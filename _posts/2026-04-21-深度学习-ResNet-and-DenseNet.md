---
layout:     post
title:      深度学习
subtitle:   ResNet和DenseNet
date:       2026-04-22
author:     潇湘潞
header-img: img/stage0/li_xingyun.jpg
catalog: true
tags:
    - 深度学习

---

## 前言

**何恺明**等人提出了*残差网络*（ResNet） ([He *et al.*, 2016](https://zh.d2l.ai/chapter_references/zreferences.html#id60))。 残差网络的核心思想是：每个附加层都应该更容易地包含原始函数作为其元素之一。 凭借它，ResNet赢得了2015年ImageNet大规模视觉识别挑战赛。

ResNet沿用了VGG完整的3×3卷积层设计，*残差块*（residual block）如下图：

![residual-block](https://zh.d2l.ai/_images/resnet-block.svg)

其中，1×1是调正通道数和分辨率；

ResNet对应的数学表达：
$$
f(x) = x + g(x)
$$
其中，$f(x)$ 分解为两部分相加：一个简单的线性项和一个复杂的非线性项。

进一步扩展：
$$
f(x) = [x, f(x)]
$$
其中，$f(x)$ 分解为两部分在通道上拼接：一个简单的线性项和一个复杂的非线性项。

![ResNet and DenseNet](https://zh.d2l.ai/_images/densenet-block.svg)

对应，ResNet和DenseNet均可加入1×1 Conv，均为调正通道数和分辨率；

Ref：[李沐](https://zh-v2.d2l.ai/)

