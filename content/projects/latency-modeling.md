---
title: Predicting on-board latency before writing code
summary: "A block-level latency model, calibrated on the board, that predicted an enlarged occupancy model within 0.7%. It ranked six deployment options before any code was written and led to a backbone shared by five perception models."
org: QCraft
period: 2025 – 2026
role: Latency modeling, deployment planning and the OneModel phase-2 design
weight: 4
metrics:
  - value: "0.7%"
    label: error, 19.38 ms predicted vs 19.25 ms measured
  - value: "6"
    label: deployment options ranked by frame-budget share
  - value: "5–9 ms"
    label: estimated saving from a shared backbone
topics: [Latency modeling, Profiling, BEV occupancy, Multi-task backbone, Deployment planning]
---

## The question

Product asked for two upgrades to the occupancy network at once: a much larger output range (from 28.8 m × 86.4 m to about 48 m × 136 m) and a frame rate doubled from 5 Hz to 10 Hz. Either one alone roughly doubles the occupancy model's cost. The chip also runs detection/tracking and map models, so the real question was:

> Can both upgrades ship together? If not, what has to change elsewhere to make room?

Answering by implementing and measuring each variant would take weeks. I answered it with a model instead.

## Step 1: profile by block, on the board

The network was split into blocks, and each block's latency was measured on the board, not only read from the compiler's estimate. Board timings differed from the compiler profile (about 8.8 ms profiled vs 10.0 ms measured for the whole model), and the gap was not uniform across blocks.

## Step 2: decide how each block scales

Keeping the 0.3 m output resolution, the output grid grows from 96 × 288 to 160 × 448 cells, about **2.6×**. Each block was then classified by how its cost depends on that grid:

| Block | Depends on output grid? | Board measurement | Estimate for larger range |
| --- | --- | --- | --- |
| Image backbone + neck (5 cameras) | No: works in image space | 3.92 ms | 3.92 ms |
| View transform (image → BEV) | Yes, proportional | 0.92 ms | 2.39 ms |
| Point-cloud branch | Partly (neck scales) | 0.48 ms | 1.33 ms |
| Temporal fusion | Yes | 0.69 ms | 1.79 ms |
| BEV encoder (backbone + neck) | Yes | 1.88 ms | 4.90 ms |
| Near-field high-resolution head | No: fixed region | 0.45 ms | 0.45 ms |
| Main occupancy head | Yes | 1.77 ms | 4.61 ms |
| **Whole model** | | **≈ 10.0 ms** | **19.38 ms** |

{{< callout "result" >}}
**Measured after implementation: 19.25 ms**, against 19.38 ms predicted, a 0.7% error. The image encoder is the only large block unaffected by the range change, which made it the obvious thing to share.
{{< /callout >}}

{{< callout "caveat" >}}
The 0.7% holds for this model. A similar estimate for a different view-transform design missed by more. The value is the loop of predict, measure and correct, not one number.
{{< /callout >}}

## Step 3: rank the deployment options

With per-block costs known, combinations of models, frame rates and sharing could be priced as frame-budget share (at 10 Hz) without building any of them:

| Option | Change | Share of 10 Hz budget |
| --- | --- | --- |
| 1 | Today: three models deployed separately | 40.1% |
| 2 | + larger occupancy range | 44.8% |
| 3 | + occupancy at 10 Hz | 54.5% |
| 4 | + occupancy reuses the image backbone and neck | 50.6% |
| 5 | + map model reuses them too | 46.2% |
| 6 | + map model at 3.3 Hz | 42.3% |

The conclusion was clear before any code: **a larger range plus 10 Hz only fits if the image backbone and neck are shared**. Deployment becomes two-stage. One backbone model produces feature maps, and the detection, map and occupancy heads consume them. Lowering the map model's frame rate gives further headroom if needed.

## Step 4: OneModel, one backbone for five perception models

This led to the OneModel phase-2 plan I wrote: a single image backbone and neck shared by five perception models across 9 cameras at 512 × 1024. The goal was to buy latency now and capacity later.

{{< pipeline "9 cameras" "Shared backbone + neck" "Detection & tracking" "Map" "Occupancy" "Front-view perception" "Traffic lights" >}}

- **Latency budget.** Sharing saves an estimated 5–9 ms per frame. That saving is the budget for a larger backbone, so the plan first evaluated a backbone of about twice the capacity.
- **Sizing the backbone from existing models.** Features from each task's released float checkpoint were extracted, and PCA measured how redundant each stage's channels were. Redundant dimensions could shrink; information-dense ones had to stay.
- **Initial weights without retraining from scratch.** Channels were aligned across the existing checkpoints, PCA selected the dimensions to keep, and closed-form linear solves produced the neck and per-task projections. Reconstruction error was checked on held-out clips.
- **Training.** Each step trains on one task's data across all GPUs, rotating through the tasks by planned exposure. This reuses each task's data loaders and keeps distributed training consistent.
- **Rollout.** First adapt each model to the shared architecture and release it on its own. Then merge pairs, starting with tracking + occupancy. Then merge all five. A merge ships only if no task regresses.

## Lessons

- **Model before you build.** A block-level model answered a question that would otherwise have taken weeks of implementation.
- **Measure on the device.** Compiler profiles and board timings disagree in exactly the places that matter.
- **Find the invariant.** The part whose cost does not change with the requirement (the image encoder) was the part worth sharing.
