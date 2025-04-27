---
title: fpga-acc-sgd
date: 2024-12-03 15:51:39
tags:
---

# 随机梯度下降算法加速

在机器学习和深度学习中，优化是模型训练的核心环节。随机梯度下降算法（Stochastic Gradient Descent, 简称 SGD）是最经典的优化算法之一，它通过迭代更新模型参数来最小化损失函数。本文将从理论与实践的角度，详细介绍 SGD 的工作原理、实现细节及其改进方法。



### 1. SGD算法的基本概念

随机梯度下降是一种通过不断迭代寻找函数极小值的优化方法。其核心思想是，在每次迭代中随机选择一个样本（或一小批样本）来估计目标函数的梯度信息，沿负梯度方向更新参数，逐步逼近最优解。这样做的优点是计算效率高，而且当数据集很大时 SGD 也能够避免陷入局部最小值。

SGD 的更新公式如下：
$$
\theta_{t+1} = \theta_t - \eta \nabla f_i(\theta_t;x_i)
$$
其中：

- $\theta$ 表示模型参数；
- $t$ 表示当前迭代次数；
- $\eta$ 表示学习率；
- $f_i$ 是对随机样本 $x_i$ 的损失函数。

随机梯度下降算法的工作流程如下：

1. 随机打乱训练数据；
2. 从训练数据中逐个取样；
3. 基于当前样本计算梯度，更新参数；
4. 重复以上过程，直到参数收敛或达到指定迭代次数。

```python
import numpy as np

# 示例目标函数：y = (x-3)^2
def loss_function(x):
    return (x - 3)**2

# 梯度计算
def gradient(x):
    return 2 * (x - 3)

# SGD 实现
def stochastic_gradient_descent(initial_x, learning_rate, num_epochs):
    x = initial_x
    history = [x]  # 保存每次迭代的参数值
    for epoch in range(num_epochs):
        grad = gradient(x)  # 计算梯度
        x = x - learning_rate * grad  # 参数更新
        history.append(x)
        print(f"Epoch {epoch+1}: x = {x:.4f}, loss = {loss_function(x):.4f}")
    return x, history

# 参数初始化
initial_x = 10  # 初始参数值
learning_rate = 0.1  # 学习率
num_epochs = 20  # 迭代次数

# 执行 SGD
final_x, history = stochastic_gradient_descent(initial_x, learning_rate, num_epochs)

print(f"最终参数值：x = {final_x:.4f}")
```



### 2. FPGA 加速 SGD 算法

下面我们对 rosetta 基准测试 [1] 进行修改，在 FPGA 上使用 SGD 算法执行垃圾邮件过滤任务。

**2.1 计算点积**

第一阶段是计算向量 $\theta_t$ 和向量 $x_i$ 的点积。$\theta_t$ 在初始化后一直被存储在加速器本地，加速器每次从内存中读取 $x_i$ 向量的一段，交给数个乘加单元并行计算，最后再通过一个加法树合并各个单元的部分和。

```c++
FeatureType compute_stage1(hls::stream<FeatureType> &theta_stream,
                           hls::stream<DataType> &data_stream) {
    FeatureType dot = 0;
    for (int i = 0; i < NUM_FEATURES / PAR_FACTOR; i++) {
#pragma HLS pipeline II = 1
        FeatureType theta = theta_stream.read();
        DataType data = data_stream.read();
        dot += theta * data;
    }
    return dot;
}
```

**2.2 计算梯度并更新参数**

第二阶段是根据点积结果进行决策，结合标签计算对应的梯度，然后更新参数 $\theta_{t+1}$。根据前一阶段计算得到的点积结果，通过 Sigmoid 函数进行决策，与训练标签进行比较，计算出梯度。最后根据梯度更新参数。

```c++
void compute_stage2(hls::stream<FeatureType> &theta_stream,
                    hls::stream<DataType> &data_stream,
                    LabelType label,
                    hls::stream<FeatureType> &result_stream,
                    FeatureType dot) {
    FeatureType prob = Sigmoid(dot);
    for (int i = 0; i < NUM_FEATURES / PAR_FACTOR; i++) {
#pragma HLS pipeline II = 1
        FeatureType theta = theta_stream.read();
        DataType data = data_stream.read();
        result_stream.write(theta + (-STEP_SIZE * (prob - label) * data));
    }
}
```

**2.3 并行计算设计**

由于每次 $i$ 的迭代都依赖于前一次参数 $\theta_t$ 的更新值，因此对 $i$ 的迭代只能构建“读-算-写”粗粒度流水线。点积的计算可以用多个乘加单元并行计算，但其并行度的瓶颈还是在于内存的读写带宽。梯度计算和参数更新也可以分块进行，最好与第一阶段的乘加单元的分块一致，令流水线充分流动。完整代码详见 [Github](https://github.com/WenbinTeng/Terris/tree/main/app/spam-filter)。

```
theta,x                                           theta_new
-------> VE #0 -- \                 / --> grad #0 --------->
theta,x                                           theta_new
-------> VE #1 -- -> dot -> Sigmoid -- -> grad #1 --------->
theta,x                                           theta_new
-------> VE #2 -- /                 \ --> grad #2 --------->
...
```



### 3. 参考文献

[1] Zhou Y, Gupta U, Dai S, et al. Rosetta: A realistic high-level synthesis benchmark suite for software programmable FPGAs[C]//Proceedings of the 2018 ACM/SIGDA International Symposium on Field-Programmable Gate Arrays. 2018: 269-278.
