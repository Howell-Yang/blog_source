---
title: Layered numerical consistency and release gates
summary: "A method that compares every stage of the deployment chain, from vehicle data to on-board post-processing, with the stage before it on identical inputs, so a mismatch has exactly one possible source. Plus evidence-graded release gates that keep unverified results off the car."
org: QCraft
period: 2024 – 2026
role: Method design, tooling direction and adoption across model teams
weight: 3
metrics:
  - value: "12"
    label: stage-to-stage checkpoints, camera data to on-board output
  - value: "5"
    label: evidence grades, source → vehicle
topics: [Numerical alignment, LibTorch, Horizon HBIR / HBM, HIL / SIL, Release engineering]
---

## The problem

"The on-board result differs from the float model" is the most common, and least useful, bug report in model deployment. Between a model in PyTorch and the same model on the vehicle there are a dozen transformations: data capture, preprocessing, export, fake quantization, an intermediate representation, two runtimes, a compiler, integer kernels and post-processing. Each can add error, and the errors compound. Without structure, debugging means guessing.

## The method: compare neighbors, not endpoints

Compare each stage only with the stage before it, on **identical inputs** and **identical post-processing**. Each comparison then has exactly one stage that can be responsible.

The data side:

{{< pipeline "Vehicle data" "HIL|hardware-in-the-loop" "SIL|software-in-the-loop" "Offline format" "Offline preprocessing" "On-board preprocessing" >}}

The model side:

{{< pipeline "Float" "Chip-oriented export" "QAT, fake-quant off" "QAT" "IR → rebuilt QAT" "LibTorch" "QAT HBIR" "Quantized HBIR" "HBM|on chip" "On-board post-proc." >}}

## Error sources at each checkpoint

| # | Comparison | Typical error source | How it is checked |
| --- | --- | --- | --- |
| 1 | Vehicle data vs HIL vs SIL | NV12 frames used directly vs converted through RGB | Dump SIL inputs into the offline format and diff |
| 2 | Offline preprocessing vs data conversion | RGB vs NV12 paths; upsample–downsample round trips | Tensor diff on the same frames |
| 3 | Offline vs on-board preprocessing | Different implementations of the same transform | Tensor diff on the same frames |
| 4 | Float vs chip-oriented export | YUV444 input instead of RGB; replaced upsampling | Intermediate-feature and output diff |
| 5 | Export vs QAT with fake-quant disabled | Conv–BN fusion; plugin ops whose fake-quant cannot be switched off | Output diff, per-layer if needed |
| 6 | QAT vs float | The quantization error itself | Task metrics plus per-layer error |
| 7 | QAT → IR → rebuilt QAT | Serialization and rebuild differences | Bit-level output comparison |
| 8 | QAT model vs LibTorch (Python vs C++) | GPU type, CUDA version, occasional run-specific outliers | Off-board vs on-board consistency check |
| 9 | LibTorch vs QAT HBIR | Lookup-table conversion happens here | Per-operator comparison, LUT error check |
| 10 | QAT HBIR vs quantized HBIR | Fake-quant replaced by true integer arithmetic | Quantization-scale check |
| 11 | Quantized HBIR vs HBM | Compiler | Vendor verification tool |
| 12 | Off-board vs on-board post-processing, and vehicle vs offline end to end | Implementation drift; frame sync | On-board sync test |

## Case studies

**A lookup-table boundary in `rsqrt`.** One production model showed a gap between LibTorch and the chip. We reproduced it off-board by comparing plain LibTorch with LibTorch using the chip's lookup tables, then bisected by layer. The error came from one table's split point in an `rsqrt` inside a normalization layer. The fix was local. The lasting change was to run lookup-table error checks early in conversion, which later became the [adaptive lookup-table algorithm](/projects/adaptive-lut/).

**Map model coordinate errors.** For a vector-map model, stage isolation found three separate problems: an `align_corners` mismatch in upsampling between back ends, insufficient precision in a pose-delta tensor, and an inverse operation executed at the wrong point in the graph.

**Temporal models.** Tracking models carry state between frames, so a tiny numeric difference in one frame propagates into many later ones. Checks for these models must cover whole sequences, not single frames.

## Release gates: how far has a result been proven?

Consistency tooling tells you where an error is. Release gates decide what may ship. Every claim about a quantized model is tagged with the highest level it has passed:

{{< pipeline "Source|code reviewed" "Build|artifacts compile" "Simulation|offline metrics" "Board|runs on chip" "Vehicle|road tests" >}}

- An IR upload is not a working on-chip model, and a good offline metric is not a board result. Status reports say which level a result has reached.
- **New model features merge behind a default-off flag.** They are switched on only after offline, regression-test and release validation.
- **A regression set of past quantization failures** runs before every release, so a fixed problem cannot quietly return.
- **Quantization changes are written into the release notes**, so downstream teams can link a behavior change to its cause.

{{< callout "result" >}}
The method turned "results don't match" into a reproducible bug with a known location. The regression set caught quantization issues before release on three production programs.
{{< /callout >}}

## Lessons

- **Isolate before you fix.** Compare neighbors on identical inputs; never diff the two ends of a 12-stage chain.
- **Data and post-processing are part of the model.** Several "quantization bugs" were preprocessing differences.
- **Name the evidence level.** Most release arguments disappear once everyone agrees how far a result has actually been proven.
