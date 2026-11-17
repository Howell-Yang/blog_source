---
title: 云端 GPU 模型的 TensorRT INT8 与 onnx2trt
type: projects
ui_lang: zh
summary: "为腾讯地图云端 GPU 模型搭建的 INT8 量化部署流程：与浮点 cosine 相似度高于 0.999，耗时降低 20%，GPU 占用和显存各降低 10%，节省约 300 万元算力成本。其中的校准工具开源为 onnx2trt。"
org: 腾讯地图
period: 2021 – 2022
role: 负责云端模型的量化、部署与测试
weight: 7
links:
  - label: GitHub 上的 onnx2trt
    url: https://github.com/Howell-Yang/onnx2trt
metrics:
  - value: "> 0.999"
    label: INT8 与浮点输出的 cosine 相似度
  - value: "−20%"
    label: 推理耗时
  - value: "≈ 300 万元"
    label: 节省的算力成本
topics: [TensorRT, ONNX, INT8 校准, 云端 GPU, 开源]
---

## 问题

腾讯地图的云端模型消耗的 GPU 资源超出了预算。INT8 推理是显而易见的手段，难点在精度：量化模型会掉精度，而每个模型都手工转换、手工测试，没法规模化。

## 流程

{{< pipeline "PyTorch 模型" "导出 ONNX" "校准|多种算法" "精度检查|与浮点比 cosine" "TensorRT INT8 engine" "部署 + 测试" >}}

1. **导出 ONNX**，作为统一的输入格式。
2. **多种算法校准**：在同一接口下集成 ONNX Runtime 的量化校准和 TensorRT 自带的校准算法。
3. **用输出相似度评估。** 用 cosine 相似度比较量化模型和浮点模型的输出。另外增加了一种直接优化这一相似度的校准方法，与标准方法并列使用。
4. **自动编译并测试 engine**：耗时、GPU 占用、显存和精度，每个模型都经过同一道关卡。

## 结果

{{< callout "result" >}}
量化后的输出与浮点模型的 cosine 相似度保持在 0.999 以上。推理耗时降低约 20%，GPU 占用和显存各降低约 10%，节省约 300 万元算力成本。
{{< /callout >}}

## onnx2trt

我把校准工具开源为 [onnx2trt](https://github.com/Howell-Yang/onnx2trt)，一个用于 ONNX → TensorRT INT8 量化的开源工具：

- 集成 ONNX 量化与 TensorRT 自带的校准算法；
- 开放自定义校准算法的接口；
- 包含基于 cosine 相似度的校准方法。

## 经验

- **用与浮点模型的输出相似度来评判量化**，而不是校准器自己的统计量。
- **自动化测试，而不只是自动化转换。** 测量方式不统一的转换结果没法相互比较。
