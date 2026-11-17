---
title: AR navigation perception SDK
summary: "A perception SDK for AR navigation on phones and car head units: a multi-stream parallel framework with automatic profiling. 12 features delivered in four months with CPU usage under 70%, shipped on the JMC CX743 production vehicle."
org: Tencent Maps
period: 2021 – 2022
role: SDK design and on-device model deployment
weight: 8
video: /public/img/AR_Navi_Map.mp4
metrics:
  - value: "12"
    label: perception features in four months
  - value: "< 70%"
    label: CPU usage on the head unit
  - value: "1"
    label: production vehicle launch (JMC CX743)
topics: [Mobile / in-car SDK, Parallel pipelines, Profiling, On-device deployment]
---

## The problem

AR navigation overlays guidance on the live camera view, so perception has to run in real time on hardware that is also running navigation, maps and the car's own software. A phone or head-unit CPU has little headroom. A feature that is accurate but late is useless, and an SDK that starves the rest of the system will not ship.

## Design

{{< pipeline "Camera frames" "Parallel perception streams|road, parking slots, …" "Results to navigation" "AR guidance" >}}

- **Multi-stream parallel framework.** Perception tasks run as parallel streams, so a slow task does not hold up the others and new tasks can be added without rewriting the pipeline.
- **Automatic profiling built in.** Every stream reports its timing and CPU cost continuously, so performance regressions show up during development, not in the car.
- **On-device models** deployed and tuned for the target hardware.

## Results

<div class="media-grid">
<figure><video src="/public/img/nano_det.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>Road-scene perception</figcaption></figure>
<figure><video src="/public/img/parkingSlotDetection.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>Parking-slot detection</figcaption></figure>
<figure><video src="/public/img/AR_Navi_Map.mp4" muted autoplay loop playsinline preload="metadata"></video><figcaption>AR navigation</figcaption></figure>
</div>

{{< callout "result" >}}
Twelve perception features delivered in four months, with CPU usage held under 70%. The system launched on the JMC CX743 production vehicle. Its returned data now helps keep Tencent Maps up to date.
{{< /callout >}}

## Visual positioning SDK

My work also included the SDK for the visual positioning system used alongside AR navigation.

<figure><img src="/public/img/vps_sdk_design.png" alt="Visual positioning SDK architecture" loading="lazy"><figcaption>Visual positioning SDK design</figcaption></figure>

## Lessons

- **Budget the CPU first, then the features.** Real-time on shared hardware is a scheduling problem as much as a model problem.
- **Make performance visible all the time.** Built-in profiling caught regressions long before integration.
