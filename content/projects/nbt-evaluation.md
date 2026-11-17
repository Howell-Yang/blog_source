---
title: NBT evaluation platform
summary: "An internal web platform for model and data management, evaluation runs, result tracking and data sharing, designed with interfaces for bad-case analysis, returned-data management and map-building visualization."
org: Tencent Maps
period: 2021 – 2022
role: Designer and developer
weight: 9
image: /public/img/nbt_eval_v1.png
metrics:
  - value: "1"
    label: place for models, data, evaluations and results
topics: [Evaluation, Web development, MLOps]
---

## The problem

Teams needed one shared place to manage models and data, run algorithm evaluations, keep results and share data, instead of each person tracking these separately.

## Design

The platform puts models, data, evaluation jobs and results behind one web interface, with an API so other tools can plug in.

<figure><img src="/public/img/nbt_eval_design.png" alt="Evaluation platform architecture" loading="lazy"><figcaption>Platform architecture</figcaption></figure>

- **Model and data management** in one place.
- **Algorithm evaluation** run through the platform, so results are produced the same way.
- **Result management and data sharing** across the team.
- **Interfaces kept open** for bad-case analysis, returned-data management and map-building result display.

<figure><img src="/public/img/nbt_eval_design0.png" alt="Evaluation platform interface design" loading="lazy"><figcaption>Interface design</figcaption></figure>

## The product

<div class="media-grid">
<figure><img src="/public/img/nbt_eval_v1.png" alt="Evaluation platform, version 1" loading="lazy"><figcaption>Version 1</figcaption></figure>
<figure><img src="/public/img/nbt_eval_v2.png" alt="Evaluation platform, version 2" loading="lazy"><figcaption>Version 2</figcaption></figure>
</div>

## Lessons

- **Comparability is a product feature.** A standard evaluation path matters more than any single metric.
- **Design the interfaces before the features.** Leaving room for bad-case analysis and data return keeps the platform extensible.
