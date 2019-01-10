---
layout:     post                    # 使用的布局（不需要改）
title:      细化算法集合               # 标题 
subtitle:   各类细化算法
date:       2019-01-10              # 时间
author:     Kelly Yang                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - c++
    - opencv
---

> 引言：本文主要是对当前细化代码以及原理做一个分析总结，持续更新ing

## 前言
细化是将图像的线条从多像素宽度减少到单位像素宽度过程的简称，一些文章经常将细化结果描述为"骨架化"。

一般要求：
* 1、保证细化后骨架的连通性；
* 2、对原图像的细节特征要保持的较好；
* 3、细化的结果是原线条的中心线；
* 4、线条的端点保留完好；
* 5、线条交叉点不能发生畸变；
* 6、细化所用时间不能太长。

### 算法
#### 非迭代算法
非迭代细化算法是不以像素为基础的，它们通过一次遍历的方式产生线条的某一中值或中心线，而不检查所有单个像素，即这种算法是一次即产生骨架；如`基于距离变换`的方法，`游程长度编码`细化等。这类算法中最简单的方法是通过扫描确定每段线的中心点，然后把它们连接成一副骨架，这种算法拥有计算机处理速度的优势，但是会有容易产生噪声点的缺陷。
#### 迭代算法
即重复删除图像边缘满足一定条件的像素，最终得到单像素宽带骨架。
迭代方法依据其检查像素的方法又可以再分成：

串行算法：
在串行算法中，通过在每次迭代中用固定的次序检查像素来判断是否删除像素，在第n次迭代中像素p的删除取决于到目前为止执行过的所有操作，也就是必须在第(n-1)次迭代结果和第n次检测像素的基础之上进行像素删除操作；即是否删除像素在每次迭代的执行中是固定顺序的，它不仅取决于前次迭代的结果，也取决于本次迭代中已处理过像素点分布情况。

并行算法：
在并行算法中，第n次迭代中像素的删除只取决于(n-1)次迭代后留下的结果，因此所有像素能在每次迭代中以并行的方式独立的被检测；即像素点删除与否与像素值图像中的顺序无关，仅取决于前次迭代效果

## 1、基于数学形态学的细化算法
数学形态学细化算法的主要思想是源于击中击不中变换的形态学算法。在给定一系列具有一定形状的结构元素后，顺序循环地删除满足击中变换的像素。
细化公式：$$ A\otimes B=A-\left ( A\otimes B \right ) $$

## 2、Zhang快速并行算法
Zhang快速并行算法是Zhang在1984年提出来的。以二值图像为例，判断一个点该不该删除时要根据它的八个相邻点的情况来判断。总结一下，有如下的判据：
* 1、内部点不能删除
* 2、孤立点不能删除
* 3、直线端点不能删除
* 4、如果P是边界点，去掉P后，如果连通分量不增加，则P可以删除。

该算法步骤如下：假设一幅图像中的3*3区域，对各点标记名称P1、P2、...、P9，其中P1位于中心，如下图

![Image text](https://raw.githubusercontent.com/KellyYang233/KellyYang233.github.io/master/img/balingyu.png)

### 算法步骤
1、其中P1=1为黑色点，如果同时满足以下四个条件，则删除P1，即令P1=0

* 1、P1周围的8个点中，白点的个数介于2至6，则 2≤N(P1)≤6；其中，N(P1)=P2+P3+...+P9;这一步是为了排除孤立点和内部点。
* 2、设Z(P1)是以P2、...、P9为序时这些点的值从0到1变化的次数,也就是表示从P2按顺序到P9的过程中，相应的点的数值从0->1的变化次数。当对所有边界点都检查完毕后，将所有标记了的点除去。
* 3、P2×P4×P6=0;
* 4、P4×P6×P8=0;

2、同1，仅将上面3条件改为P2×P4×P8=0;4条件改为P2×P6×P8=0。同样当对所有边界点都检验完毕后，将所有标记了的点除去。

3、以上两步操作构成一次迭代，直至没有点再满足标记条件，这时剩下的点组成区域即为细化后的骨架。

**代码**
```
void thinimage(Mat &srcimage)//单通道、二值化后的图像
{
    vector<Point> deletelist1;
    int Zhangmude[9];
    int nl = srcimage.rows;
    int nc = srcimage.cols;
    while (true)
    {
        for (int j = 1; j<(nl - 1); j++)
        {
            uchar* data_last = srcimage.ptr<uchar>(j - 1);
            uchar* data = srcimage.ptr<uchar>(j);
            uchar* data_next = srcimage.ptr<uchar>(j + 1);
            for (int i = 1; i<(nc - 1); i++)
            {
                if (data[i] == 255)
                {
                    Zhangmude[0] = 1;
                    if (data_last[i] == 255) Zhangmude[1] = 1;
                    else  Zhangmude[1] = 0;
                    if (data_last[i + 1] == 255) Zhangmude[2] = 1;
                    else  Zhangmude[2] = 0;
                    if (data[i + 1] == 255) Zhangmude[3] = 1;
                    else  Zhangmude[3] = 0;
                    if (data_next[i + 1] == 255) Zhangmude[4] = 1;
                    else  Zhangmude[4] = 0;
                    if (data_next[i] == 255) Zhangmude[5] = 1;
                    else  Zhangmude[5] = 0;
                    if (data_next[i - 1] == 255) Zhangmude[6] = 1;
                    else  Zhangmude[6] = 0;
                    if (data[i - 1] == 255) Zhangmude[7] = 1;
                    else  Zhangmude[7] = 0;
                    if (data_last[i - 1] == 255) Zhangmude[8] = 1;
                    else  Zhangmude[8] = 0;
                    int whitepointtotal = 0;
                    for (int k = 1; k < 9; k++)
                    {
                        whitepointtotal = whitepointtotal + Zhangmude[k];
                    }
                    if ((whitepointtotal >= 2) && (whitepointtotal <= 6))
                    {
                        int ap = 0;
                        if ((Zhangmude[1] == 0) && (Zhangmude[2] == 1)) ap++;
                        if ((Zhangmude[2] == 0) && (Zhangmude[3] == 1)) ap++;
                        if ((Zhangmude[3] == 0) && (Zhangmude[4] == 1)) ap++;
                        if ((Zhangmude[4] == 0) && (Zhangmude[5] == 1)) ap++;
                        if ((Zhangmude[5] == 0) && (Zhangmude[6] == 1)) ap++;
                        if ((Zhangmude[6] == 0) && (Zhangmude[7] == 1)) ap++;
                        if ((Zhangmude[7] == 0) && (Zhangmude[8] == 1)) ap++;
                        if ((Zhangmude[8] == 0) && (Zhangmude[1] == 1)) ap++;
                        if (ap == 1)
                        {
                            if ((Zhangmude[1] * Zhangmude[7] * Zhangmude[5] == 0) && (Zhangmude[3] * Zhangmude[5] * Zhangmude[7] == 0))
                            {
                                deletelist1.push_back(Point(i, j));
                            }
                        }
                    }
                }
            }
        }
        if (deletelist1.size() == 0) break;
        for (size_t i = 0; i < deletelist1.size(); i++)
        {
            Point tem;
            tem = deletelist1[i];
            uchar* data = srcimage.ptr<uchar>(tem.y);
            data[tem.x] = 0;
        }
        deletelist1.clear();

        for (int j = 1; j<(nl - 1); j++)
        {
            uchar* data_last = srcimage.ptr<uchar>(j - 1);
            uchar* data = srcimage.ptr<uchar>(j);
            uchar* data_next = srcimage.ptr<uchar>(j + 1);
            for (int i = 1; i<(nc - 1); i++)
            {
                if (data[i] == 255)
                {
                    Zhangmude[0] = 1;
                    if (data_last[i] == 255) Zhangmude[1] = 1;
                    else  Zhangmude[1] = 0;
                    if (data_last[i + 1] == 255) Zhangmude[2] = 1;
                    else  Zhangmude[2] = 0;
                    if (data[i + 1] == 255) Zhangmude[3] = 1;
                    else  Zhangmude[3] = 0;
                    if (data_next[i + 1] == 255) Zhangmude[4] = 1;
                    else  Zhangmude[4] = 0;
                    if (data_next[i] == 255) Zhangmude[5] = 1;
                    else  Zhangmude[5] = 0;
                    if (data_next[i - 1] == 255) Zhangmude[6] = 1;
                    else  Zhangmude[6] = 0;
                    if (data[i - 1] == 255) Zhangmude[7] = 1;
                    else  Zhangmude[7] = 0;
                    if (data_last[i - 1] == 255) Zhangmude[8] = 1;
                    else  Zhangmude[8] = 0;
                    int whitepointtotal = 0;
                    for (int k = 1; k < 9; k++)
                    {
                        whitepointtotal = whitepointtotal + Zhangmude[k];
                    }
                    if ((whitepointtotal >= 2) && (whitepointtotal <= 6))
                    {
                        int ap = 0;
                        if ((Zhangmude[1] == 0) && (Zhangmude[2] == 1)) ap++;
                        if ((Zhangmude[2] == 0) && (Zhangmude[3] == 1)) ap++;
                        if ((Zhangmude[3] == 0) && (Zhangmude[4] == 1)) ap++;
                        if ((Zhangmude[4] == 0) && (Zhangmude[5] == 1)) ap++;
                        if ((Zhangmude[5] == 0) && (Zhangmude[6] == 1)) ap++;
                        if ((Zhangmude[6] == 0) && (Zhangmude[7] == 1)) ap++;
                        if ((Zhangmude[7] == 0) && (Zhangmude[8] == 1)) ap++;
                        if ((Zhangmude[8] == 0) && (Zhangmude[1] == 1)) ap++;
                        if (ap == 1)
                        {
                            if ((Zhangmude[1] * Zhangmude[3] * Zhangmude[5] == 0) && (Zhangmude[3] * Zhangmude[1] * Zhangmude[7] == 0))
                            {
                                deletelist1.push_back(Point(i, j));
                            }
                        }
                    }
                }
            }
        }
        if (deletelist1.size() == 0) break;
        for (size_t i = 0; i < deletelist1.size(); i++)
        {
            Point tem;
            tem = deletelist1[i];
            uchar* data = srcimage.ptr<uchar>(tem.y);
            data[tem.x] = 0;
        }
        deletelist1.clear();
    }
}
```