---
layout:     post
title:      深度学习
subtitle:   GoogLeNet神经网络
date:       2026-04-22
author:     潇湘潞
header-img: img/stage0/li_xingyun.jpg
catalog: true
tags:
    - 深度学习

---

## 前言

在**2014**年的ImageNet图像识别挑战赛中，一个名叫*GoogLeNet* ([Szegedy *et al.*, 2015](https://zh.d2l.ai/chapter_references/zreferences.html#id162))的网络架构大放异彩。 GoogLeNet吸收了NiN中串联网络的思想，并在此基础上做了改进。 这篇论文的一个重点是**解决了什么样大小的卷积**核最合适的问题。

 卷积核的选择问题：小孩子才做选择题；

- Inception的卷积核选择和GoogLeNet的框架模型：

![Inception](https://zh.d2l.ai/_images/inception.svg) ![CooleNet](https://zh.d2l.ai/_images/inception-full.svg) 

## Inception模型代码：

```python
class Inception(nn.Module):
    def __init__(self, in_channels, c1, c2, c3, c4, **kwargs) -> None:
        super().__init__(**kwargs)

        self.p1 = nn.Sequential(
            nn.Conv2d(in_channels, c1, kernel_size=1),
            nn.ReLU()
        ) 
        self.p2 = nn.Sqeuential(
            nn.Conv2d(in_channels, c2[0], kernel_size=1),
            nn.Conv2d(c2[0], c2[1], kernel_size=3, padding=1),
            nn.ReLU()
        )
        self.p3 = nn.Sequential(
            nn.Conv2d(in_channels, c3[0], kernel_size=1),   
            nn.Conv2d(c3[0], c3[1], kernel_size=5, padding=2),
            nn.ReLU()
        )
        self.p4 = nn.Sequential(
            nn.MaxPool2d(kernel_size=3, stride=1, padding=1),
            nn.Conv2d(in_channels, c4, kernel_size=1),
            nn.ReLU()
        )
    def forward(self, X):
        p1 = self.p1(X)
        p2 = self.p2(X)
        p3 = self.p3(X)
        p4 = self.p4(X)
        return torch.cat((p1, p2, p3, p4), dim=1)
```

note：模型能被训练，钞能力很强大！

Ref：[李沐](https://zh-v2.d2l.ai/)

