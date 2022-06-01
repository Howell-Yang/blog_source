---
author: howellyang
title: 关于我
url: /about/
sidebar: true
weight: "1"
type: fixed
---

<div>
<img src="/public/img/personal_log.png" alt="" title="Howell Yang" width="256" height="128" />
</div>
<br>

欢迎来到我的博客! 我叫杨豪, 我将在这里发布关于技术的一些思考，一些好用的小工具，以及一些教程。<br>
<br>

# 个人信息
 - **姓名**   杨豪
 - **出生年月** 1992-12 
 - **工作年限** ：3年
 - **博客**  http://www.howellyang.com
 - **Github** https://github.com/Howell-Yang
 - **手机** 13260129163
 - **Email** howellyang@buaa.edu.cn

<br>

# 教育经历
- 2016年9月 ~ 2019年1月    北京航空航天大学 模式识别 **硕士**
- 2012年9月 ~ 2016年7月    北京航空航天大学 自动化  **本科**

<br>

# 工作经历
## 腾讯(2021年7月 ~ 至今）
### GPU算力优化 
我负责优化云端模型的量化、部署和测试。针对GPU资源消耗过多的问题，搭建了基于TensorRT的模型量化和部署的流程。遇到的主要问题是量化后的模型存在精度损失，最后调研了一系列的模型量化算法，并开发了模型量化和转换工具，来实现自动化的模型转化和测试。最终量化模型的输出的cosine相似度高于0.999, 同时模型infer的耗时降低20%，GPU的占用率降低10%，GPU显存降低10%。实现了降本增效，节省三百万左右的算力成本。

### NBT评测网站项目
负责开发一个内部使用的评测网站，方便所有人进行模型&数据管理、算法评测、结果管理、数据共享。并且预留接口，以实现badcase分析、回流数据管理、建图结果展示等功能。 

<multi-column-container>
<p>
<img src="/public/img/nbt_eval_design.png" alt="" height="180" width="360">
</p>

<p>
<img src="/public/img/nbt_eval_design0.png" alt="" height="180" width="360">
</p>

<p>
<img src="/public/img/nbt_eval_v1.png" alt="" height="180" width="360">
</p>

</multi-column-container>

### AR导航项目 
将模型部署到手机、车机端对于SDK的性能要求较高，需要在完成复杂功能的基础上，尽可能的降低耗时和CPU占用率。针对这个需求，设计了多流程并行的框架，并加入了自动profile功能。共完成功能12项，实现实时的AR导航功能，并将CPU占用率控制在70%以下。在四个月的时间里完成了个AR导航中的感知功能，并正式在江铃CX743车辆上线。后续该车正式量产，并实现了数据的回传，能有效辅助腾讯地图的数据更新。不同模块的效果如下:
<p>
<img src="/public/img/vps_sdk_design.png" alt="" title="Howell Yang" width="1080" height="720" />
</p>

<multi-column-container>
<p>
<video src="/public/img/nano_det.mp4" height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
</p>

<p>
<video src="/public/img/parkingSlotDetection.mp4" height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
</p>

<p>
<video src="/public/img/AR_Navi_Map.mp4" height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
</p>

</multi-column-container>



## 依图科技 （ 2019年4月 ~ 2021年7月 ）

### 脸行贵阳-地铁刷脸项目
我在此项目负责了哪些工作，分别在哪些地方做得出色/和别人不一样/成长快，这个项目中，我最困难的问题是什么，我采取了什么措施，最后结果如何。这个项目中，我最自豪的技术细节是什么，为什么，实施前和实施后的数据对比如何，同事和领导对此的反应如何。


### 机器翻译项目 
我在此项目负责了哪些工作，分别在哪些地方做得出色/和别人不一样/成长快，这个项目中，我最困难的问题是什么，我采取了什么措施，最后结果如何。这个项目中，我最自豪的技术细节是什么，为什么，实施前和实施后的数据对比如何，同事和领导对此的反应如何。

# 开源项目
（对于程序员来讲，没有什么比Show me the code能有说服力了）
  - [onnx2trt](https://github.com/Howell-Yang/onnx2trt)：
    - 用于进行onnx模型到tensorRT模型的int8量化
    - 集成了onnx_quantization和tensorRT自带的模型量化算法
    - 开放模型量化接口，用于实现自定义的模型量化算法
    - 实现了基于cosine优化的模型量化算法
  - [blog_source](https://github.com/Howell-Yang/blog_source)
    - 基于hugo开发的个人博客模板网站
    - 加入了一系列的格式优化和说明文档
    - 用于进行个人博客网站的开发

# 技术文章

- [tensorRT模型量化教程] TODO
- [个人博客网站教程] TODO
- [paper reading] TODO

# 演讲和讲义
- 2014架构师大会演讲：[如何通过Docker优化内部开发](http://ftqq.com)
- 7月T9晋升PPT：[云计算的前生今世](http://ftqq.com)


# 技能清单
- 编程语言：C++/Python
- 计算机视觉：检测、分割、分类
- 自然语言处理：机器翻译
- 魔性训练框架：tensorflow/pytorch/caffe
- 模型部署框架：onnx/TNN/MNN/PaddleLite/TensorRT
- 前端框架：HTML/CSS/JavaScript
- 后端框架: Flask/MongoDB
- 基础开发：Git/Bash/Gcc/Cmake
