---
title: "From PTQ to QAD: quantization that matches the float model"
summary: "How the quantization method for production driving models evolved over four generations, and why mass production needs case-level consistency with the float model rather than a good average metric."
org: QCraft
period: 2023 – 2026
role: Quantization lead; algorithm design and rollout across the team
weight: 1
metrics:
  - value: "2 → 0.5"
    label: points of quantization loss, PTQ → QAT, on five perception models
  - value: "4"
    label: method generations, each driven by a new model family
  - value: "25 / 388"
    label: key metrics / shared scalars tracked per end-to-end candidate
topics: [PTQ, QAT, QAD, LSQ, Distillation, Sensitivity analysis, Horizon J6M]
---

## The problem

Driving models at QCraft are released roughly every two weeks, and quantization is the last step before the chip. Early in a program, quantization was judged by one number: the drop in offline metrics had to stay within a threshold. Late in mass production that is no longer enough. Downstream teams tolerate almost no regression, so quantization also answers for scenario regression tests and for issues found on the vehicle.

Two facts made this hard:

- **Stability is not free.** Model owners expect quantization to be stable when the architecture does not change. In practice, any shift in the distribution of weights or activations (new data, a longer training run) changes the quantized result.
- **Averages hide cases.** An automatic emergency braking (AEB) target detected even one frame later than in the float model is a failure. A bug fixed in the float model must also be fixed in the INT8 model. A good average metric guarantees neither.

So the requirement became: **a quantized model that behaves like the float model, case by case, and is stable enough to run with one command and no tuning.**

## The workflow

Every model goes through the same staged procedure. Each stage has an exit test, and a model only moves to the more expensive stage when the cheaper one fails.

{{< pipeline "Float model" "Adapt to framework|verify zero loss" "PTQ|calibrate" "Sensitivity analysis|per layer" "QAT / QAD|task + distillation" "Export IR|compile for chip" >}}

1. **Adapt the float model to the quantization framework**, then confirm the adaptation itself costs nothing. Operator substitutions, conv–BN fusion and the chip's input path (YUV instead of RGB) all change numerics before any quantization happens.
2. **Calibrate with selected data.** The calibration set must cover the widest practical range of scenarios, and it includes cases that caused quantization issues in earlier releases.
3. **Evaluate PTQ.** If it meets the target, export. PTQ is cheap and, because it does not change the weights, stays closest to the float model.
4. **Find the sensitive layers.** Two tools: cumulative error (how far each layer's output has drifted from float, end to end) and single-layer error (quantize one layer at a time). For a sharper ranking, start from an all-INT16 (W16A16) model and lower one layer at a time to INT8. Sensitive layers stay at INT16.
5. **Train if needed: QAT or QAD**, with a task loss and a distillation loss against the float model, iterating until the target is met. Then export to the intermediate representation (IR) and compile for the chip.

## Generation 1: scale-search PTQ (2023)

The first production models (BEV detection, mono 3D, multi-task perception) were quantized with PTQ. Hand-tuning PTQ was slow and fragile: every model update meant re-tuning clipping ranges, and min/max or percentile statistics often picked poor scales for long-tailed activations.

I built an **automatic quantization-parameter search**. It searches for a good set of quantization parameters on calibration data, instead of deriving them from activation statistics and hand-tuning. It removed most of the manual work from routine releases, and a patent application was filed for it.

The limit was accuracy: PTQ on complex multi-task models still lost around 2 points.

## Generation 2: LSQ-based AutoQuant (2024)

Map and temporal tracking models combine convolutions with transformer blocks. Transformer activations have strong outliers, which uniform statistics-based PTQ handles badly. Full QAT recovers accuracy but rewrites the weights, so the model drifts from float on rare inputs.

AutoQuant sits between the two. Its design goals were:

- improve task accuracy over PTQ;
- keep intermediate features close to the float model, not just the outputs;
- change the weights as little as possible;
- learn the quantization parameters (scales) directly;
- work on hybrid CNN + transformer graphs.

The method learns step sizes with LSQ (learned step-size quantization) on a small unlabeled set. Weight changes are bounded, for example AdaRound-style rounding choices instead of free updates, and intermediate features are supervised against the float model. An alternative I evaluated was QAT restricted to the most sensitive layers. It is cheaper, but only as good as the sensitivity ranking.

Graph-level rewrites help before any training. **Cross-layer equalization** balances weight ranges between adjacent linear layers. **LayerNorm–Linear fusion** removes one quantized tensor: on the chip, LayerNorm is decomposed, and its last step (a per-channel scale and offset) folds into the next linear layer.

```text
LayerNorm:   y1 = (x − mean(x)) / std(x)
             y2 = γ ⊙ y1 + β
Linear:      y3 = W·y2 + b
Fused:       y3 = (W·diag(γ))·y1 + (W·β + b)
```

## Generation 3: distillation-based QAT (2025)

On five production perception models, QAT brought the quantization loss from about 2 points (PTQ) to about 0.5. But QAT with only a task loss has a structural problem:

{{< callout "caveat" >}}
**QAT changes the operating point.** Training on the task loss lets the quantized model pick a new precision/recall trade-off. It may remove some false positives and add some false negatives, fix a float bug or introduce a new one. The quantization team then delivers a retrained model, not a low-precision version of the float model, and nobody can tell whether a vehicle issue belongs to the model team or the quantization team.
{{< /callout >}}

Distillation-based QAT keeps the task loss but adds the float model as a teacher. Training starts from the PTQ model, and intermediate features are pulled toward the float model's features on the same input. Improvements made to the float model then carry over to the vehicle.

The engineering around it mattered as much as the loss:

- **Reproducibility:** remove avoidable randomness, so two QAT runs are comparable.
- **Data management:** dedicated calibration and QAT datasets. When float training data is added or removed, the QAT set changes in the same proportions.
- **Code promotion path:** float main branch → quantization main branch → isolated experiment branch → quantization main branch → master.
- **Cost awareness:** a QAT run takes tens of GPUs for 12–24 hours per model, so the cheaper stages come first.

## Generation 4: quantization-aware distillation (QAD) for end-to-end models (2026)

Planning and end-to-end driving models output trajectories, where small numerical differences compound over time. We ran QAT and AutoQuant/distillation side by side on a planning model and an end-to-end model, with the same data. Both reached high aggregate metrics, but the distillation-based method was clearly better on consistency and on long-tail cases.

QAD makes distillation the primary objective and scales the data. Two changes go with it:

- **Data:** training clips are selected to be diverse rather than numerous, and clips for specific known issues are collected and added.
- **Controlled experiments:** for the World2Action end-to-end model, each candidate changes one factor at a time, grouped into families. Every candidate reports the same 25 key metrics and 388 shared scalars, and failed candidates are kept as evidence.

What those experiments established:

| Factor varied | Finding |
| --- | --- |
| Distillation type (none, feature, full, QAD only) | Task loss alone or feature distillation alone is not enough; task supervision cannot be removed. |
| Temporal state (scene reset, independent teacher history) | Resetting state at scene boundaries and giving the teacher its own history are prerequisites. |
| Late-training weight drift | Anchoring weights late in training stops a numerical divergence seen in one candidate. |
| Horizon balancing, route gating, curricula | Some gave local gains offline but not consistent vehicle improvements, so they were not promoted. |

Promoted candidates went to on-vehicle testing. Anything not yet tested on the vehicle stays labeled an offline candidate.

## Lessons

- **PTQ is closer to float; QAT scores higher.** Production needs both properties, which is what QAD is for.
- **The float model is the specification.** Measure quantization by how closely it reproduces float behavior, case by case.
- **Escalate cost only when needed.** Adapt, calibrate and analyze sensitivity before spending GPU-days on training.
- **Keep the negative results.** They define where each method works and stop the same experiment from being run twice.
