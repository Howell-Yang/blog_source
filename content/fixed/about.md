---
author: howellyang
title: 关于我
url: /about/
sidebar: true
weight: "1"
type: fixed
---

<br>
欢迎来到我的博客! 我叫杨豪, 我将在这里发布关于技术的一些思考，一些好用的小工具，以及一些教程。我目前就职于腾讯CSIG-地图平台部，技术领域是利用计算机视觉算法，进行自动驾驶、三维见图、视觉动作捕捉等研究。<br>
<br>

# 个人信息

- **姓名** 杨豪
- **出生年月** 1992-12
- **工作年限** 3 年
- **博客** http://www.howellyang.com
- **Github** https://github.com/Howell-Yang
- **手机** 13260129163
- **Email** howellyang@buaa.edu.cn

<br>

# 教育经历

- 2016 年 9 月 ~ 2019 年 1 月 北京航空航天大学 模式识别 硕士
- 2012 年 9 月 ~ 2016 年 7 月 北京航空航天大学 自动化 本科

<br>

# 工作经历

## 腾讯(2021 年 7 月 ~ 至今）

### GPU 算力优化

我负责优化云端模型的量化、部署和测试。针对 GPU 资源消耗过多的问题，搭建了基于 TensorRT 的模型量化和部署的流程。遇到的主要问题是量化后的模型存在精度损失，最后调研了一系列的模型量化算法，并开发了模型量化和转换工具，来实现自动化的模型转化和测试。最终量化模型的输出的 cosine 相似度高于 0.999, 同时模型 infer 的耗时降低 20%，GPU 的占用率降低 10%，GPU 显存降低 10%。实现了降本增效，节省三百万左右的算力成本。

### NBT 评测网站项目

负责开发一个内部使用的评测网站，方便所有人进行模型&数据管理、算法评测、结果管理、数据共享。并且预留接口，以实现 badcase 分析、回流数据管理、建图结果展示等功能。

<div style="column-count:3;text-align: center; justify-content: center;">
<p style="text-align: center; justify-content: center;">
<img src="/public/img/nbt_eval_design.png"  alt="" height="180" width="360" class="pic"/>
<p style="text-align:center; justify-content: center;">评测平台架构</p>
</p>

<p style="text-align: center; justify-content: center;">
<img src="/public/img/nbt_eval_design0.png"  alt="" height="180" width="360" class="pic"/>
<p   style="text-align:center; justify-content: center;">评测平台接口设计</p>
</p>

<p style="text-align: center; justify-content: center;">
<img src="/public/img/nbt_eval_v1.png"  alt="" height="180" width="360" class="pic"/>
<p   style="text-align:center; justify-content: center;">评测平台页面效果图</p>
</p>
</div>

<div id="outerdiv" style="position:fixed;top:0;left:0;background:rgba(0,0,0,0.7);z-index:2;width:100%;height:100%;display:none;">
    <div id="innerdiv" style="position:absolute;">
        <img id="bigimg" style="border:5px solid #fff;" src="" />
  </div>
</div>

### AR 导航项目

将模型部署到手机、车机端对于 SDK 的性能要求较高，需要在完成复杂功能的基础上，尽可能的降低耗时和 CPU 占用率。针对这个需求，设计了多流程并行的框架，并加入了自动 profile 功能。共完成功能 12 项，实现实时的 AR 导航功能，并将 CPU 占用率控制在 70%以下。在四个月的时间里完成了个 AR 导航中的感知功能，并正式在江铃 CX743 车辆上线。后续该车正式量产，并实现了数据的回传，能有效辅助腾讯地图的数据更新。不同模块的效果如下:

<div style="column-count:3;text-align: center; justify-content: center;">

<p style="text-align: center; justify-content: center;">
<video src="/public/img/nano_det.mp4" muted  alt=""  type="video/mp4" height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
<p  style="text-align:center; justify-content: center;">道路感知结果</p>
</p>

<p style="text-align: center; justify-content: center;">
<video src="/public/img/parkingSlotDetection.mp4"  muted alt=""  height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
<p  style="text-align:center; justify-content: center;">停车位检测结果</p>
</p>

<p style="text-align: center; justify-content: center;">
<video src="/public/img/AR_Navi_Map.mp4"  muted alt=""  height="180" width="360" controls="controls" autoplay="autoplay"  loop="loop">
<p  style="text-align:center; justify-content: center;">AR导航效果</p>
</p>

</div>

## 依图科技 （ 2019 年 4 月 ~ 2021 年 7 月 ）

### 脸行贵阳-地铁刷脸项目

我在此项目负责了哪些工作，分别在哪些地方做得出色/和别人不一样/成长快，这个项目中，我最困难的问题是什么，我采取了什么措施，最后结果如何。这个项目中，我最自豪的技术细节是什么，为什么，实施前和实施后的数据对比如何，同事和领导对此的反应如何。

### 机器翻译项目

我在此项目负责了哪些工作，分别在哪些地方做得出色/和别人不一样/成长快，这个项目中，我最困难的问题是什么，我采取了什么措施，最后结果如何。这个项目中，我最自豪的技术细节是什么，为什么，实施前和实施后的数据对比如何，同事和领导对此的反应如何。

# 开源项目

（对于程序员来讲，没有什么比 Show me the code 能有说服力了）

- [onnx2trt](https://github.com/Howell-Yang/onnx2trt)：
  - 用于进行 onnx 模型到 tensorRT 模型的 int8 量化
  - 集成了 onnx_quantization 和 tensorRT 自带的模型量化算法
  - 开放模型量化接口，用于实现自定义的模型量化算法
  - 实现了基于 cosine 优化的模型量化算法
- [blog_source](https://github.com/Howell-Yang/blog_source)
  - 基于 hugo 开发的个人博客模板网站
  - 加入了一系列的格式优化和说明文档
  - 用于进行个人博客网站的开发

# 技术文章

- [tensorRT 模型量化教程] TODO
- [个人博客网站教程] TODO
- [paper reading] TODO

# 演讲和讲义

- 2014 架构师大会演讲 [如何通过 Docker 优化内部开发](http://ftqq.com)
- 7 月 T9 晋升 PPT [vps_sdk 的开发](http://ftqq.com)

# 技能清单

- 编程语言 C++/Python
- 计算机视觉 检测、分割、分类
- 自然语言处理 机器翻译
- 魔性训练框架 tensorflow/pytorch/caffe
- 模型部署框架 onnx/TNN/MNN/PaddleLite/TensorRT
- 前端框架 HTML/CSS/JavaScript
- 后端框架 Flask/MongoDB
- 基础开发 Git/Bash/Gcc/Cmake

<script>
    $(function() {
        $('img').click(function() {
            var _this = $(this);// 将当前的pimg元素作为_this传入函数
            imgShow("#outerdiv", "#innerdiv", "#bigimg", _this);
        });
    });
 
    function imgShow(outerdiv, innerdiv, bigimg, _this) {
        var src = _this.attr("src");// 获取当前点击的pimg元素中的src属性
        $(bigimg).attr("src", src);// 设置#bigimg元素的src属性
 
        /* 获取当前点击图片的真实大小，并显示弹出层及大图 */
        $("<img/>").attr("src", src).load(function() {
            var windowW = $(window).width();// 获取当前窗口宽度
            var windowH = $(window).height();// 获取当前窗口高度
            var realWidth = 1000;// 获取图片真实宽度
            var realHeight = 1000;// 获取图片真实高度
            var imgWidth, imgHeight;
            var scale = 1;// 缩放尺寸，当图片真实宽度和高度大于窗口宽度和高度时进行缩放
 
            if (realHeight > windowH * scale) {// 判断图片高度
                imgHeight = windowH * scale;// 如大于窗口高度，图片高度进行缩放
                imgWidth = imgHeight / realHeight * realWidth;// 等比例缩放宽度
                if (imgWidth > windowW * scale) {// 如宽度扔大于窗口宽度
                    imgWidth = windowW * scale;// 再对宽度进行缩放
                }
            } else if (realWidth > windowW * scale) {// 如图片高度合适，判断图片宽度
                imgWidth = windowW * scale;// 如大于窗口宽度，图片宽度进行缩放
                imgHeight = imgWidth / realWidth * realHeight;// 等比例缩放高度
            } else {// 如果图片真实高度和宽度都符合要求，高宽不变
                imgWidth = realWidth;
                imgHeight = realHeight;
            }
            $(bigimg).css("width", imgWidth);// 以最终的宽度对图片缩放
 
            var w = (windowW - imgWidth) / 2;// 计算图片与窗口左边距
            var h = (windowH - imgHeight) / 2;// 计算图片与窗口上边距
            $(innerdiv).css({
                "top" : h,
                "left" : w
            });// 设置#innerdiv的top和left属性
            $(outerdiv).fadeIn("fast");// 淡入显示#outerdiv及.pimg
        });
 
        $(outerdiv).click(function() {// 再次点击淡出消失弹出层
            $(this).fadeOut("fast");
        });
    }
</script>
