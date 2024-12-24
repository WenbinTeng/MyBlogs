---
title: fpga-acc-vj
date: 2024-12-24 10:38:15
tags:
---

# Viola-Jones 算法加速

Viola-Jones 算法是一个经典的用于对象检测的算法，最著名的是用于人脸检测。由 Paul Viola 和 Michael Jones 于 2001 年提出，该算法在实时人脸检测任务中表现出色，因其速度和准确性成为计算机视觉领域的基石。本文将带您深入了解 Viola-Jones 算法的核心原理及其应用。



### 1. Viola-Jones 算法概念与流程

Viola-Jones 算法在 AdaBoost 算法的基础上，使用 Haar 特征和积分图技术来进行人脸检测。

**1.1 Haar 特征**

Viola-Jones 算法采用 Haar 特征来表示图像中的局部信息。这些特征是黑白矩形区域的简单组合，通过计算像素和的差异来捕捉图像的边缘、线条和特定模式。

Haar 特征由一个或多个矩形区域组成，通常分为黑色和白色两种颜色。通过计算黑色区域和白色区域的像素和的差值，可以提取局部特征信息，例如：

- 边缘特征：由两个矩形组成，用于检测亮到暗的过渡区域（如人脸边缘）。
- 线条特征：由三个矩形组成，用于检测两侧亮、中间暗的区域（如嘴唇）。
- 中心-周围特征：由四个矩形组成，用于对于中心区域和周围区域，可以检测特定的对称结构。

**1.2 积分图**

Haar 特征的一个难点是大量特征需要快速计算，尤其是在高分辨率图像中。为了解决这个问题，Viola-Jones 算法引入了 积分图（Integral Image）。

积分图实际上就是一个二维的前缀和数组，积分图中的一个元素就代表在原图中该元素与原图的左上角元素构成的矩形的面积。积分图的定义公式为：
$$
ii(x,y) = \sum_{x' \leq x, y' \leq y} i(x',y')
$$
扫描一次图像即可构建积分图，其步骤为：

1. 用 $s(i,j)$ 表示第 $i$ 行方向的累加和，初始化 $s(i,-1)=0$；

2. 用 $ii(i,j)$ 表示积分图，初始化 $ii(-1,i)=0$；

3. 逐行扫描图像，递归计算每个像素 $f(i,j)$ 行方向的累加和 $s(i,j)$ 与积分图 $ii(i,j)$ 的值：
   $$
   \begin{align*}
   s(i,j) &= s(i,j-1)+f(i,j) \\
   ii(i,j) &= ii(i-1)+s(i,j)
   \end{align*}
   $$

4. 扫描到达图像右下角的像素时，积分图就构造完成。

**1.3 AdaBoost**

AdaBoost（Adaptive Boosting）是 Freund 和 Schapire 于 1995 年提出的一种 Boosting 算法。它通过将多个弱分类器（性能略优于随机猜测的分类器）组合成一个强分类器，从而显著提高整体的分类准确性。Viola-Jones  算法使用 AdaBoost 算法从一组巨大的随机特征池中选取对人脸检测具有帮助的特征。

**1.4 级联分类器**

级联分类器可以通过增加分类器的个数以提高检测率的同时降低误识率。多个分类器串联形成一个决策链，当前分类器对图像作出评价，如果产生 True 结果则送入下一个分类器进行后续处理，如果产生 False 结果则判断当前窗口图像不包含目标。



### 2. FPGA 加速 Viola-Jones 算法

下面我们对 rosetta 基准测试 [1] 进行修改，在 FPGA 上使用 Viola-Jones 算法执行人脸检测任务。

**2.1 弱分类器**

```c
int classifier0(int_II II[WINDOW_SIZE][WINDOW_SIZE], int stddev) {
    static int coord[12];
#pragma HLS array_partition variable = coord complete dim = 0

    int sum0, sum1, sum2, final_sum;
    int return_value;
    coord[0] = II[4][6];
    coord[1] = II[4][18];
    coord[2] = II[13][6];
    coord[3] = II[13][18];

    coord[4] = II[7][6];
    coord[5] = II[7][18];
    coord[6] = II[10][6];
    coord[7] = II[10][18];

    coord[8] = 0;
    coord[9] = 0;
    coord[10] = 0;
    coord[11] = 0;

    sum0 = (coord[0] - coord[1] - coord[2] + coord[3]) * -4096;
    sum1 = (coord[4] - coord[5] - coord[6] + coord[7]) * 12288;
    sum2 = (coord[8] - coord[9] - coord[10] + coord[11]) * 0;
    final_sum = sum0 + sum1 + sum2;

    if (final_sum >= (-129 * stddev))
        return_value = -567;
    else
        return_value = 534;
    return return_value;
}
```

**2.2 强分类器与级联分类器**

```c
int cascade_classifier(point_t pt, int_II II[WINDOW_SIZE][WINDOW_SIZE], int_SII SII[SQ_SIZE][SQ_SIZE]) {
    /* Hard-Coding Classifier 0 */
    static int s0[9];
    int stage_sum = 0;
    s0[0] = classifier0(II, stddev);
    s0[1] = classifier1(II, stddev);
    s0[2] = classifier2(II, stddev);
    s0[3] = classifier3(II, stddev);
    s0[4] = classifier4(II, stddev);
    s0[5] = classifier5(II, stddev);
    s0[6] = classifier6(II, stddev);
    s0[7] = classifier7(II, stddev);
    s0[8] = classifier8(II, stddev);
    stage_sum = s0[0] + s0[1] + s0[2] + s0[3] + s0[4] + s0[5] + s0[6] + s0[7] + s0[8];
    
    /* Hard-Coding Classifier 1 */
    //...
}
```

**2.3 积分图**

```c
/** Integral Image Window buffer ( 625 registers )*/
static int_II II[WINDOW_SIZE][WINDOW_SIZE];
#pragma HLS array_partition variable = II complete dim = 0

SET_IIi:for (int i = 0; i < WINDOW_SIZE; i++) {
#pragma HLS unroll
    SET_IIj:for (int j = 0; j < WINDOW_SIZE; j++) {
#pragma HLS unroll
        II[i][j] = II[i][j] + (I[i][j + 1] - I[i][0]);
    }
}
```

**2.4 并行计算设计**

这里的级联分类器并没有采用链式决策结构，因为提前退出对于固定的硬件架构来说没有收益。这里的级联分类器采用强分类器之间的流水线架构，强分类器内部的多个弱分类器并行计算。完整的代码详见 [Github](https://github.com/WenbinTeng/Terris/tree/main/app/face-detection)。

```
                 ----------------      ----------------
                | weak_filter #0 |    | weak_filter #0 |
image_scalar -> | weak_filter #1 | -> | weak_filter #1 | -> ...
                | weak_filter #2 |    | weak_filter #2 |
                 ----------------      ----------------
                 strong_filter #0      strong_filter #1
```



### 3. 参考文献

[1] Zhou Y, Gupta U, Dai S, et al. Rosetta: A realistic high-level synthesis benchmark suite for software programmable FPGAs[C]//Proceedings of the 2018 ACM/SIGDA International Symposium on Field-Programmable Gate Arrays. 2018: 269-278.
