---
title: AR 导航感知 SDK
type: projects
ui_lang: zh
summary: "面向手机和车机的 AR 导航感知 SDK：多流程并行框架加自动 profile。四个月完成 12 项功能，CPU 占用低于 70%，在江铃 CX743 量产车型上线。"
org: 腾讯地图
period: 2021 – 2022
role: SDK 设计与端侧模型部署
weight: 8
video: /public/img/AR_Navi_Map.mp4
metrics:
  - value: "12"
    label: 项感知功能，四个月完成
  - value: "< 70%"
    label: 车机 CPU 占用
  - value: "1"
    label: 款量产车型上线（江铃 CX743）
topics: [手机 / 车机 SDK, 并行流水线, Profiling, 端侧部署]
---

## 问题

AR 导航把引导信息叠加在实时相机画面上，所以感知必须在硬件上实时运行，而同一硬件还在运行导航、地图和车辆自身的软件。手机或车机的 CPU 余量很小。准确但延迟高的功能没有用，挤占系统其他部分资源的 SDK 也不可能上线。

## 设计

{{< pipeline "相机帧" "并行感知流程|道路、车位……" "结果送导航" "AR 引导" >}}

- **多流程并行框架。** 各感知任务作为并行的流程运行，慢任务不会拖住其他任务，新增任务也不需要重写流水线。
- **内置自动 profile。** 每个流程持续上报耗时和 CPU 开销，性能回退在开发阶段就能发现，而不是等到上车。
- **端侧模型**针对目标硬件部署和调优。

## 结果

<div class="media-grid">
<figure><video src="/public/img/nano_det.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>道路场景感知</figcaption></figure>
<figure><video src="/public/img/parkingSlotDetection.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>车位检测</figcaption></figure>
<figure><video src="/public/img/AR_Navi_Map.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>AR 导航</figcaption></figure>
</div>

{{< callout "result" >}}
四个月完成 12 项感知功能，CPU 占用控制在 70% 以下。系统在江铃 CX743 量产车型上线，回传的数据帮助腾讯地图保持更新。
{{< /callout >}}

## 视觉定位 SDK

我的工作还包括与 AR 导航配套使用的视觉定位系统 SDK。

<figure><img src="/public/img/vps_sdk_design.png" alt="视觉定位 SDK 架构" loading="lazy"><figcaption>视觉定位 SDK 设计</figcaption></figure>

## 经验

- **先定 CPU 预算，再定功能。** 在共享硬件上做实时，既是模型问题，也是调度问题。
- **让性能始终可见。** 内置 profile 让性能回退在集成之前很久就被发现。
