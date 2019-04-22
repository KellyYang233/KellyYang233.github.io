---
layout:     post                    # 使用的布局（不需要改）
title:      双边滤波               # 标题 
subtitle:   
date:       2019-04-22              # 时间
author:     Kelly Yang                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - opencv
---

# 双边滤波Bilateral Filter

双边滤波（Bilateral Filter）是非线性滤波中的一种。这是一种结合图像的空间邻近度与像素值相似度的处理办法。在滤波时，该滤波方法同时考虑空间临近信息与颜色相似信息，在滤除噪声、平滑图像的同时，又做到边缘保存。
双边滤波采用了两个高斯滤波的结合，一个负责计算空间邻近度的权值，也就是常用的高斯滤波器原理。而另一个负责计算像素值相似度的权值。在两个高斯滤波的同时作用下，就是双边滤波。
ref：https://blog.csdn.net/qq_36359022/article/details/80198890

## 公式
![](https://i.imgur.com/egoqWyj.png)
g(i, j)代表输出点；
S(i, j)的是指以(i,j)为中心的(2N+1)(2N+1)的大小的范围；
f(k, l)代表(多个)输入点；
w(i, j, k, l)代表经过两个高斯函数计算出的值(这里还不是权值)