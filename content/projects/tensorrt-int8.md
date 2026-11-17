---
title: TensorRT INT8 for cloud GPU models, and onnx2trt
summary: "An INT8 quantization and deployment pipeline for Tencent Maps' cloud GPU models: cosine similarity above 0.999 to float, latency −20%, GPU utilization and memory −10%, and roughly ¥3M saved in compute. The calibration tooling became the open-source onnx2trt."
org: Tencent Maps
period: 2021 – 2022
role: Owner of quantization, deployment and testing for cloud models
weight: 7
links:
  - label: onnx2trt on GitHub
    url: https://github.com/Howell-Yang/onnx2trt
metrics:
  - value: "> 0.999"
    label: output cosine similarity, INT8 vs float
  - value: "−20%"
    label: inference latency
  - value: "≈ ¥3M"
    label: compute cost saved
topics: [TensorRT, ONNX, INT8 calibration, Cloud GPU, Open source]
---

## The problem

Tencent Maps' cloud models used more GPU capacity than the budget allowed. INT8 inference was the obvious lever. The obstacle was accuracy: quantized models lost precision, and converting and testing each model by hand did not scale.

## The pipeline

{{< pipeline "PyTorch model" "ONNX export" "Calibration|several algorithms" "Accuracy check|cosine vs float" "TensorRT INT8 engine" "Serving + tests" >}}

1. **Export to ONNX** as the common input format.
2. **Calibrate** with more than one algorithm: ONNX Runtime's quantization calibrators and TensorRT's own calibrators, behind one interface.
3. **Evaluate by output similarity.** Compare the quantized model's outputs with the float model's using cosine similarity. A calibration method that optimizes this similarity directly was added alongside the standard ones.
4. **Build and test the engine automatically**: latency, GPU utilization, memory and accuracy, so every model went through the same gate.

## Results

{{< callout "result" >}}
Quantized outputs kept cosine similarity above 0.999 with the float models. Inference latency fell by about 20%, and GPU utilization and memory each by about 10%, saving roughly ¥3M in compute cost.
{{< /callout >}}

## onnx2trt

I released the calibration tooling as [onnx2trt](https://github.com/Howell-Yang/onnx2trt), an open-source tool for ONNX → TensorRT INT8 quantization that:

- combines ONNX quantization with TensorRT's built-in calibration algorithms;
- exposes an interface for custom calibration algorithms;
- includes the cosine-similarity-driven calibration method.

## Lessons

- **Judge quantization by output similarity to the float model**, not by the calibrator's own statistic.
- **Automate the test, not just the conversion.** A conversion that is not measured the same way every time cannot be compared.
