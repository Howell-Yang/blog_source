---
title: Adaptive lookup tables for non-linear operators
summary: "An error-driven segmentation algorithm for fixed-point lookup tables. It refines segments only where the real fixed-point error is high, guaranteeing a 2 LSB worst-case error with 12–30 tables per operator instead of a uniform 126."
org: QCraft
period: 2026
role: Algorithm design, evaluation and integration into the conversion flow
weight: 2
metrics:
  - value: "0.217 LSB"
    label: weighted mean error over 600 LUT modules
  - value: "2 LSB"
    label: "guaranteed worst-case error (uniform 126 tables: 11,381 LSB)"
  - value: "−85%"
    label: tables vs. uniform 126-table design (avg 18.6)
topics: [Fixed-point arithmetic, Lookup tables, rsqrt, Softmax, Horizon J6M]
---

## The problem

On automotive NPUs, non-linear operators such as `rsqrt`, `reciprocal`, the `exp` inside Softmax and `sigmoid` run as fixed-point lookup tables, not floating-point math. A bad table can break a model that looks fine everywhere else. In one production model, the root cause of an accuracy gap was a single table's segment boundary in an `rsqrt` inside a normalization layer (see [Layered numerical consistency](/projects/numerical-consistency/)).

The hardware's segmented lookup table is not a global nearest-neighbor lookup:

1. **Dividing points** pick a segment for the input.
2. **Inside a segment**, a fixed-point table is read and interpolated.
3. **Outside the valid range**, linear fits on the left and right take over.

So the error comes from four things: the valid input domain, where the dividing points sit, how densely each segment is sampled, and how the left and right linear regions are handled.

## Measuring error the way the chip sees it

The reference is the IR's fake-quant output: the quantized value the model was trained and validated against. The candidate is the converter's real fixed-point lookup-table output code. For every valid integer input code *x<sub>i</sub>*:

```text
e(x_i)  = | q_LUT(x_i) − q_FQ( f(x_i) ) |          error in output LSB
E_mean  = Σ_i e(x_i) / N                            weighted by sample count
E_max   = max_i e(x_i)                              over all operators and codes
```

- **Mean LSB** is weighted over every valid input code of every table, not a simple average of per-operator averages.
- **Max LSB** exposes rare but potentially fatal local distortions, the kind that slip through aggregate metrics.
- **Valid domains** come from calibration: `rsqrt` and `reciprocal` use the IR's calibrated range; the Softmax `exp` input is never positive after the max-subtraction, so it defaults to [−12, 0] when no range is recorded.

## Three strategies compared

{{< callout >}}
More tables lower the **average** error, but they do not bound the **worst-case** error. If dividing points are not placed according to the function's shape and the true fixed-point error, high-curvature regions and linear-region boundaries can still be badly wrong.
{{< /callout >}}

| Strategy | Dividing points | Left/right linear regions | Tables per op | Guarantee |
| --- | --- | --- | --- | --- |
| A. Original 6-table | Native per op: mostly curvature-based, some evenly spaced | Original linear fits | 6 | Baseline only; no max-error target |
| B. Uniform 126-table | Valid range split into 126 equal-width segments | Not optimized | 126 | Table count only |
| C. Adaptive (this work) | Keep the 6-table boundaries, then recursively bisect | Upgraded only when the measured gain justifies it | 12–30, chosen per op | Boundaries kept, never worse than baseline, max 2 LSB |

Curvature-based placement (A) puts more boundaries where the function bends, which beats equal spacing, but six tables are too few for near-singular functions. Uniform 126 tables (B) is simple and lowers the mean, but it ignores curvature, output scale and the real fixed-point error. On `rsqrt` it wastes tables on flat regions and still leaves huge errors near extreme input codes.

## The adaptive algorithm

The original 6-table layout is the floor: the algorithm only adds boundaries, never moves or removes them. The feedback signal is the real fixed-point output error, not a floating-point approximation.

{{< pipeline "6-table baseline|keep all boundaries" "Measure real error|every input code" "Bisect worst segment|midpoint" "Check linear regions|upgrade if it pays" "Max ≤ 2 LSB?|else repeat" "Round up to ×6|hardware export" >}}

```text
for each LUT operator f with valid domain D:
    S ← segments from the original 6-table layout          # never removed
    loop:
        E ← real fixed-point error of f over every code in D, per segment
        if max(E) ≤ 2 LSB: break
        s* ← segment with the largest error
        split s* at its midpoint                            # nested refinement
        if a left/right linear region dominates the error:
            upgrade that region only if the measured gain justifies it
        if table budget exhausted and max(E) > 2 LSB:
            fail hard and report the operator               # never ship silently
    round the table count up to a multiple of 6             # export constraint
```

Because new boundaries are midpoints of existing segments, the refinement is nested. Every original boundary survives, so the result can never be worse than the baseline on any segment. Most operators settle at 12, 18, 24 or 30 tables.

## Results

Evaluated on four production graphs of a temporal tracking model, each with 150 lookup-table modules (600 in total):

| Strategy | Weighted mean error | Worst-case error | Tables per operator |
| --- | --- | --- | --- |
| Uniform 126-table | 0.270 LSB | 11,381 LSB | 126 |
| Adaptive | **0.217 LSB** | **2 LSB** | 12–30 (avg 18.6) |

{{< callout "result" >}}
The adaptive scheme beats uniform 126 tables on **both** mean and worst-case error while using about 85% fewer tables. End-to-end regression metrics moved by about 0.001 or less. The value is in removing rare local distortions, not in raising averages.
{{< /callout >}}

## Lessons

- **Measure what the hardware computes.** Floating-point approximations of a table hide exactly the errors that matter.
- **Bound the worst case, not just the mean.** One bad input code can decide a corner case on the road.
- **Refine where the error is.** A budget spent by curvature and measured error beats a bigger uniform budget.
- **Fail loudly.** An operator that cannot meet the bound should stop the export, not ship.
