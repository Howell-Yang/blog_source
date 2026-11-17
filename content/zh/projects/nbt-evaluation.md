---
title: NBT 评测平台
type: projects
ui_lang: zh
summary: "内部评测网站：模型与数据管理、算法评测、结果管理和数据共享，并预留 badcase 分析、回流数据管理和建图结果展示接口。"
org: 腾讯地图
period: 2021 – 2022
role: 设计与开发
weight: 9
image: /public/img/nbt_eval_v1.png
metrics:
  - value: "1"
    label: 个平台统一管理模型、数据、评测和结果
topics: [评测, Web 开发, MLOps]
---

## 问题

团队需要一个共享的地方来管理模型和数据、运行算法评测、保存结果和共享数据，而不是每个人各自记录。

## 设计

平台把模型、数据、评测任务和结果统一放在一个 Web 界面之后，并提供 API 供其他工具接入。

<figure><img src="/public/img/nbt_eval_design.png" alt="评测平台架构" loading="lazy"><figcaption>平台架构</figcaption></figure>

- **模型与数据管理**集中在一处。
- **算法评测**通过平台运行，结果以同样的方式产生。
- **结果管理与数据共享**面向整个团队。
- **预留接口**用于 badcase 分析、回流数据管理和建图结果展示。

<figure><img src="/public/img/nbt_eval_design0.png" alt="评测平台界面设计" loading="lazy"><figcaption>界面设计</figcaption></figure>

## 产品

<div class="media-grid">
<figure><img src="/public/img/nbt_eval_v1.png" alt="评测平台第一版" loading="lazy"><figcaption>第一版</figcaption></figure>
<figure><img src="/public/img/nbt_eval_v2.png" alt="评测平台第二版" loading="lazy"><figcaption>第二版</figcaption></figure>
</div>

## 经验

- **可比性本身就是产品功能。** 统一的评测路径比任何单个指标都重要。
- **先设计接口，再设计功能。** 为 badcase 分析和数据回流预留位置，平台才能持续扩展。
