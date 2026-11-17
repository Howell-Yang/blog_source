---
title: QNN Web, one-click model conversion for every chip
summary: "An internal service that turns one uploaded model into engines for every target chip. Built in 2022–2023 to remove manual, error-prone conversions; rebuilt in 2026 for reliability and given an MCP interface so AI agents can drive it."
org: QCraft
period: 2022 – 2026
role: Designer and main developer (v1); architecture lead (v2 and MCP layer)
weight: 5
metrics:
  - value: "4,273"
    label: production model conversions
  - value: "30–60 → 5 min"
    label: build time with timing cache
  - value: "≈ −80%"
    label: manual effort per conversion
topics: [TensorRT, Horizon J5 / J6, NVIDIA Orin, Qualcomm, gRPC, MCP, Internal tooling]
---

## The problem

Driving programs ship to chips at several price points, and every model must be compiled separately for each one. Before QNN Web, an engineer would:

1. log in to each hardware box;
2. hand-write a JSON config for the converter (one typo meant re-uploading and resubmitting);
3. wait 30–60 minutes for each TensorRT engine build.

That was during production releases roughly every two weeks, across many models.

## Version 1: one upload, every target

{{< pipeline "Web front end|upload ONNX / IR" "Config generator|from the graph" "gRPC dispatch" "Conversion servers|Orin · J5 · J6" "Engines + logs|download" >}}

- **Front and back end separated, talking over gRPC**, including file transfer. A user uploads once and gets engines for every target.
- **Configs generated from the ONNX graph**: input names, shapes and types are read from the model, so nothing is typed by hand.
- **TensorRT timing cache**: kernel-selection results are reused across builds of similar models, cutting a typical build from 30–60 minutes to about 5.
- **New hardware at almost no cost**: adding Orin and later J6 targets meant adding a conversion server, not changing every user's workflow.

{{< callout "result" >}}
4,273 production conversions across several automaker programs, with about 80% less manual effort per conversion. Model owners could convert their own models during fast production iteration.
{{< /callout >}}

## Version 2: rebuilt for reliability (2026)

As production work grew, three problems appeared: conversions failed intermittently, users could not tell what state a job was in, and submitting several jobs at once blocked the service.

| Problem | Change |
| --- | --- |
| Unknown job state | Every job's state is stored in a database and updated as it moves, including a new "transferring" state. The page shows a status column and a restart button. |
| Chatty front end | The page queries status only when it is refreshed, instead of polling constantly. |
| Blocking on concurrent submissions | A task queue; a job already queued or already converted is not submitted again. |
| Wasted conversions | LibTorch conversions are split by type, so only the requested variants are built. |
| Disk filling up | Artifacts stay on the conversion machines, not the front end. Periodic cleanup backs up IR files first and marks cleaned jobs as such. The database stores paths, not model files. |
| Fragile front/back-end communication | The front end hands each request to a conversion server and returns that server's address. Each server keeps its own record of finished work. Heartbeat, status and file-transfer services were added. |

Qualcomm and Orin conversion paths were added in the same rebuild.

## An MCP layer for AI agents

In 2026 I wrapped the service in a Model Context Protocol (MCP) server, so coding agents such as Cursor can drive conversions as tools instead of through a web page.

{{< pipeline "Agent|Cursor or script" "MCP server|protocol only" "Task scheduler" "Conversion server" "Result|direct link or stream" >}}

The MCP layer does **no conversion work itself**. It exposes four tools:

1. **Upload IR**: validate and upload a local model file.
2. **Submit task**: choose the target type and precision.
3. **Query status**: poll a job's state.
4. **Get result**: return a download link, or stream the file.

The real work stays in the existing scheduler and conversion servers, so the agent path and the web path share one implementation. An engineer can now ask an agent to convert a file to a given target and return the local path of the result, or call the same tools from a Python client inside a pipeline.

## Lessons

- **Remove the step, not just the typo.** Generating configs from the graph beat validating hand-written configs.
- **State belongs in a database.** Most reliability problems came from job state that lived only in memory or in a browser tab.
- **Keep protocol layers thin.** The MCP server only translates; one conversion path means one place to fix bugs.
