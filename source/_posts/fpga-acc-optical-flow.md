---
title: fpga-acc-optical-flow
date: 2024-12-16 19:35:04
tags:
---

# 光流法加速

**光流法（Optical Flow）**是一种计算图像序列中像素运动的技术，广泛用于计算机视觉领域，用于估计图像中物体的运动。它假定图像中的亮度在短时间内是保持恒定的，通过比较相邻帧之间的像素变化来推断运动信息。



### 1.  什么是光流法

光流法的核心假设是**亮度恒常性假设**（Brightness Constancy Assumption），即同一目标在不同帧间运动时，其亮度不会发生改变。考虑一个像素在第一帧的光强度为  $I(x,y,t)$ ，经过 $\mathrm{d}t$ 时间后，该像素在下一帧移动了 $(\mathrm{d}x, \mathrm{d}y)$，表示为：
$$
I(x,y,t) = I(x + \mathrm{d}x, y + \mathrm{d}y, t + \mathrm{d}t)
$$
假设移动距离很小，则可以应用泰勒展开：
$$
I(x + \mathrm{d}x, y + \mathrm{d}y, t + \mathrm{d}t) = I(x,y,t) + 
\frac{\partial I}{\partial x}\mathrm{d}x +
\frac{\partial I}{\partial y}\mathrm{d}y +
\frac{\partial I}{\partial t}\mathrm{d}t + 
\varepsilon
$$
其中 $\varepsilon$ 为高阶无穷小项。联立两式可以得到：
$$
\frac{\partial I}{\partial x}\frac{\mathrm{d}x}{\mathrm{d}t} +
\frac{\partial I}{\partial y}\frac{\mathrm{d}y}{\mathrm{d}t} +
\frac{\partial I}{\partial t}\frac{\mathrm{d}t}{\mathrm{d}t}
= 0
$$
令 $I_x = \frac{\partial I}{\partial x}, I_y = \frac{\partial I}{\partial y}, I_t = \frac{\partial I}{\partial t}, u = \frac{\mathrm{d}x}{\mathrm{d}t}, v = \frac{\mathrm{d}y}{\mathrm{d}t}$，则上式表示为：
$$
I_x u + I_y v + I_t = 0
$$
其中 $I_x, I_y$ 是图像空间的梯度，$I_t$ 是沿时间的梯度，$(u,v)$ 则是需要求解的光流向量。

由于光流约束方程单独一个不足以求解两个变量 $u,v$，需要引入额外约束条件。**Lucas-Kanade 方法**是一种经典的光流估计法，其基于一种新的假设，即场景中相同表面的相邻点具有相似的运动，并且其投影到图像平面上的距离也比较近。一个场景上邻近的点投影到图像上也是邻近点，且邻近点速度一致。那么，我们就可以对 n 个邻近点建立方程，求解 $u,v$ 的值，即：
$$
I_{x1} u + I_{y1} v = - I_{t1} \\
I_{x2} u + I_{y2} v = - I_{t2} \\
\cdots \\
I_{xn} u + I_{yn} v = - I_{tn}
$$
写成矩阵形式就是：
$$
\left[ \begin{array}{cc} I_{x1} & I_{y1} \\ I_{x2} & I_{y2} \\ \cdots & \cdots \\ I_{xn} & I_{yn} \end{array} \right]
\left[ \begin{array}{c} u \\ v \end{array} \right] = 
\left[ \begin{array}{c} -I_{t1} \\ -I_{t2} \\ \cdots \\ -I_{tn} \end{array} \right]
$$
令 $A = \left[ \begin{array}{cc} I_{x1} & I_{y1} \\ I_{x2} & I_{y2} \\ \cdots & \cdots \\ I_{xn} & I_{yn} \end{array} \right], \vec{V} = \left[ \begin{array}{c} u \\ v \end{array} \right], b = \left[ \begin{array}{c} I_{t1} \\ I_{t2} \\ \cdots \\ I_{tn} \end{array} \right]$，则上式记作 $A\vec{V}=-b$，采用最小二乘法得到：
$$
A^T A \vec{V} = A^T (-b)
$$
求解光流矢量为：
$$
\vec{V} = (A^T A)^{-1} A^T (-b)
$$
矩阵形式表示为：
$$
\left[ \begin{array}{c} u \\ v \end{array} \right] = 
\left[ \begin{array}{cc} \sum_{i=1}^n I_{xi}^2 & \sum_{i=1}^n I_{xi}I_{yi} \\ \sum_{i=1}^n I_{xi}I_{yi} & \sum_{i=1}^n I_{yi}^2 \end{array} \right]^{-1}
\left[ \begin{array}{c} - \sum_{i=1}^n I_{xi}I_{ti} \\ - \sum_{i=1}^n I_{yi}I_{ti} \end{array} \right]
$$
该算法的缺点是不能产生非常密集的流向量，流动信息在移动边缘和小运动方面会很快消失。它的优点是对存在噪声的图像的鲁棒性仍然是很好的。



### 2. FPGA 加速光流法

下面我们对 rosetta 基准测试 [1] 进行修改，在 FPGA 上加速光流法，计算连续 5 帧图像内像素的运动轨迹。

**2.1 计算梯度**

分别计算空间中 $x,y$ 方向上的梯度，以及时间 $z$ 上的梯度，基于多帧图像进行加权差分。

```c
void gradient_xy_calc(hls::stream<input_t> &frame_stream,
                      hls::stream<pixel_t> &gradient_x_stream,
                      hls::stream<pixel_t> &gradient_y_stream) {
    // our own line buffer
    static pixel_t buf[5][MAX_WIDTH];
#pragma HLS array_partition variable = buf complete dim = 1

    // small buffer
    pixel_t small_buf[5];
#pragma HLS array_partition variable = small_buf complete dim = 0

    // window buffer
    xf::cv::Window<5, 5, input_t> window;

    const int GRAD_WEIGHTS[] = {1, -8, 0, 8, -1};

    GRAD_XY_OUTER:for (int r = 0; r < MAX_HEIGHT + 2; r++) {
        GRAD_XY_INNER:for (int c = 0; c < MAX_WIDTH + 2; c++) {
#pragma HLS pipeline II = 1
            // read out values from current line buffer
            for (int i = 0; i < 4; i++)
                small_buf[i] = buf[i + 1][c];
            // the new value is either 0 or read from frame
            if (r < MAX_HEIGHT && c < MAX_WIDTH)
                small_buf[4] = (pixel_t)frame_stream.read();
            else if (c < MAX_WIDTH)
                small_buf[4] = 0;
            // update line buffer
            if (r < MAX_HEIGHT && c < MAX_WIDTH) {
                for (int i = 0; i < 4; i++)
                    buf[i][c] = small_buf[i];
                buf[4][c] = small_buf[4];
            } else if (c < MAX_WIDTH) {
                for (int i = 0; i < 4; i++)
                    buf[i][c] = small_buf[i];
                buf[4][c] = small_buf[4];
            }

            // manage window buffer
            if (r < MAX_HEIGHT && c < MAX_WIDTH) {
                window.shift_pixels_left();
                for (int i = 0; i < 5; i++)
                    window.insert_pixel(small_buf[i], i, 4);
            } else {
                window.shift_pixels_left();
                window.insert_pixel(0, 0, 4);
                window.insert_pixel(0, 1, 4);
                window.insert_pixel(0, 2, 4);
                window.insert_pixel(0, 3, 4);
                window.insert_pixel(0, 4, 4);
            }

            // compute gradient
            pixel_t x_grad = 0;
            pixel_t y_grad = 0;
            if (r >= 4 && r < MAX_HEIGHT && c >= 4 && c < MAX_WIDTH) {
                GRAD_XY_XYGRAD:for (int i = 0; i < 5; i++) {
                    x_grad += window.getval(2, i) * GRAD_WEIGHTS[i];
                    y_grad += window.getval(i, 2) * GRAD_WEIGHTS[i];
                }
                gradient_x_stream.write(x_grad / 12);
                gradient_y_stream.write(y_grad / 12);
            } else if (r >= 2 && c >= 2) {
                gradient_x_stream.write(0);
                gradient_y_stream.write(0);
            }
        }
    }
}

void gradient_z_calc(hls::stream<input_t> &frame1_stream,
                     hls::stream<input_t> &frame2_stream,
                     hls::stream<input_t> &frame3_stream,
                     hls::stream<input_t> &frame4_stream,
                     hls::stream<input_t> &frame5_stream,
                     hls::stream<pixel_t> &gradient_z_stream) {
    const int GRAD_WEIGHTS[] = {1, -8, 0, 8, -1};
    GRAD_Z_OUTER:for (int r = 0; r < MAX_HEIGHT; r++) {
        GRAD_Z_INNER:for (int c = 0; c < MAX_WIDTH; c++) {
#pragma HLS pipeline II = 1
            input_t f1 = frame1_stream.read();
            input_t f2 = frame2_stream.read();
            input_t f3 = frame3_stream.read();
            input_t f4 = frame4_stream.read();
            input_t f5 = frame5_stream.read();
            gradient_z_stream.write(((pixel_t)(f1 * GRAD_WEIGHTS[0] +
                                               f2 * GRAD_WEIGHTS[1] +
                                               f3 * GRAD_WEIGHTS[2] +
                                               f4 * GRAD_WEIGHTS[3] +
                                               f5 * GRAD_WEIGHTS[4])) / 12);
        }
    }
}
```

**2.2 平滑处理**

分别对 $x,y$ 方向上的梯度进行平滑处理。

```c
void gradient_weight_y(hls::stream<pixel_t> &gradient_x_stream,
                       hls::stream<pixel_t> &gradient_y_stream,
                       hls::stream<pixel_t> &gradient_z_stream,
                       hls::stream<gradient_t> &y_filt_grad_stream) {
    xf::cv::LineBuffer<7, MAX_WIDTH, gradient_t> buf;
    const pixel_t GRAD_FILTER[] = {0.0755, 0.133, 0.1869, 0.2903, 0.1869, 0.133, 0.0755};
    GRAD_WEIGHT_Y_OUTER:for (int r = 0; r < MAX_HEIGHT + 3; r++) {
        GRAD_WEIGHT_Y_INNER:for (int c = 0; c < MAX_WIDTH; c++) {
#pragma HLS pipeline II = 1
#pragma HLS dependence variable = buf inter false
            if (r < MAX_HEIGHT) {
                buf.shift_pixels_up(c);
                gradient_t tmp;
                tmp.x = gradient_x_stream.read();
                tmp.y = gradient_y_stream.read();
                tmp.z = gradient_z_stream.read();
                buf.insert_bottom_row(tmp, c);
            } else {
                buf.shift_pixels_up(c);
                gradient_t tmp;
                tmp.x = 0;
                tmp.y = 0;
                tmp.z = 0;
                buf.insert_bottom_row(tmp, c);
            }

            gradient_t acc;
            acc.x = 0;
            acc.y = 0;
            acc.z = 0;
            if (r >= 6 && r < MAX_HEIGHT) {
                GRAD_WEIGHT_Y_ACC:for (int i = 0; i < 7; i++) {
                    acc.x += buf.getval(i, c).x * GRAD_FILTER[i];
                    acc.y += buf.getval(i, c).y * GRAD_FILTER[i];
                    acc.z += buf.getval(i, c).z * GRAD_FILTER[i];
                }
                y_filt_grad_stream.write(acc);
            } else if (r >= 3) {
                y_filt_grad_stream.write(acc);
            }
        }
    }
}

void gradient_weight_x(hls::stream<gradient_t> &y_filt_grad_stream,
                       hls::stream<gradient_t> &filt_grad_stream) {
    xf::cv::Window<1, 7, gradient_t> buf;
    const pixel_t GRAD_FILTER[] = {0.0755, 0.133, 0.1869, 0.2903, 0.1869, 0.133, 0.0755};
    GRAD_WEIGHT_X_OUTER:for (int r = 0; r < MAX_HEIGHT; r++) {
        GRAD_WEIGHT_X_INNER:for (int c = 0; c < MAX_WIDTH + 3; c++) {
#pragma HLS pipeline II = 1
            buf.shift_pixels_left();
            gradient_t tmp;
            if (c < MAX_WIDTH) {
                tmp = y_filt_grad_stream.read();
            } else {
                tmp.x = 0;
                tmp.y = 0;
                tmp.z = 0;
            }
            buf.insert_pixel(tmp, 0, 6);

            gradient_t acc;
            acc.x = 0;
            acc.y = 0;
            acc.z = 0;
            if (c >= 6 && c < MAX_WIDTH) {
                GRAD_WEIGHT_X_ACC:for (int i = 0; i < 7; i++) {
                    acc.x += buf.getval(0, i).x * GRAD_FILTER[i];
                    acc.y += buf.getval(0, i).y * GRAD_FILTER[i];
                    acc.z += buf.getval(0, i).z * GRAD_FILTER[i];
                }
                filt_grad_stream.write(acc);
            } else if (c >= 3) {
                filt_grad_stream.write(acc);
            }
        }
    }
}
```

**2.3 计算外积**

通过梯度向量的外积得到结构张量矩阵的分量 $I_{xi}^2, I_{xi}I_{yi}, I_{yi}^2, I_{xi}I_{ti}, I_{yi}I_{ti}$。

```c
void outer_product(hls::stream<gradient_t> &gradient_stream,
                   hls::stream<outer_t> &outer_product_stream) {
    OUTER_OUTER:for (int r = 0; r < MAX_HEIGHT; r++) {
        OUTER_INNER:for (int c = 0; c < MAX_WIDTH; c++) {
#pragma HLS pipeline II = 1
            gradient_t grad = gradient_stream.read();
            outer_pixel_t x = (outer_pixel_t)grad.x;
            outer_pixel_t y = (outer_pixel_t)grad.y;
            outer_pixel_t z = (outer_pixel_t)grad.z;
            outer_t out;
            out.val[0] = (x * x);
            out.val[1] = (y * y);
            out.val[2] = (z * z);
            out.val[3] = (x * y);
            out.val[4] = (x * z);
            out.val[5] = (y * z);
            outer_product_stream.write(out);
        }
    }
}
```

**2.4 张量加权**

通过对结构张量进行加权以突出窗口中特定的内容，或保持局部区域的张量一致性。可以使用高斯核等进行计算。

```c
void tensor_weight_y(hls::stream<outer_t> &outer_product_stream,
                     hls::stream<tensor_t> &tensor_y_stream) {
    xf::cv::LineBuffer<3, MAX_WIDTH, outer_t> buf;
    const pixel_t TENSOR_FILTER[] = {0.3243, 0.3513, 0.3243};
    TENSOR_WEIGHT_Y_OUTER:for (int r = 0; r < MAX_HEIGHT + 1; r++) {
        TENSOR_WEIGHT_Y_INNER:for (int c = 0; c < MAX_WIDTH; c++) {
#pragma HLS pipeline II = 1
            outer_t tmp;
            buf.shift_pixels_up(c);
            if (r < MAX_HEIGHT) {
                tmp = outer_product_stream.read();
            } else {
                TENSOR_WEIGHT_Y_TMP_INIT:for (int i = 0; i < 6; i++)
                    tmp.val[i] = 0;
            }
            buf.insert_bottom_row(tmp, c);

            tensor_t acc;
            TENSOR_WEIGHT_Y_ACC_INIT:for (int k = 0; k < 6; k++)
                acc.val[k] = 0;

            if (r >= 2 && r < MAX_HEIGHT) {
                TENSOR_WEIGHT_Y_TMP_OUTER:for (int i = 0; i < 3; i++) {
                    tmp = buf.getval(i, c);
                    pixel_t k = TENSOR_FILTER[i];
                    TENSOR_WEIGHT_Y_TMP_INNER:for (int component = 0; component < 6; component++) {
                        acc.val[component] += tmp.val[component] * k;
                    }
                }
            }
            if (r >= 1) {
                tensor_y_stream.write(acc);
            }
        }
    }
}

void tensor_weight_x(hls::stream<tensor_t> &tensor_y_stream,
                     hls::stream<tensor_t> &tensor_stream) {
    xf::cv::Window<1, 3, tensor_t> buf;
    const pixel_t TENSOR_FILTER[] = {0.3243, 0.3513, 0.3243};
    TENSOR_WEIGHT_X_OUTER:for (int r = 0; r < MAX_HEIGHT; r++) {
        TENSOR_WEIGHT_X_INNER:for (int c = 0; c < MAX_WIDTH + 1; c++) {
#pragma HLS pipeline II = 1
            buf.shift_pixels_left();
            tensor_t tmp;
            if (c < MAX_WIDTH) {
                tmp = tensor_y_stream.read();
            } else {
                TENSOR_WEIGHT_X_TMP_INIT:for (int i = 0; i < 6; i++)
                    tmp.val[i] = 0;
            }
            buf.insert_pixel(tmp, 0, 2);

            tensor_t acc;
            TENSOR_WEIGHT_X_ACC_INIT:for (int k = 0; k < 6; k++)
                acc.val[k] = 0;
            if (c >= 2 && c < MAX_WIDTH) {
                TENSOR_WEIGHT_X_TMP_OUTER:for (int i = 0; i < 3; i++) {
                    tmp = buf.getval(0, i);
                    TENSOR_WEIGHT_X_TMP_INNER:for (int component = 0; component < 6; component++) {
                        acc.val[component] += tmp.val[component] * TENSOR_FILTER[i];
                    }
                }
            }
            if (c >= 1) {
                tensor_stream.write(acc);
            }
        }
    }
}
```

**2.5 求解光流**

根据结构张量的分量求解约束方程。

```c
void flow_calc(hls::stream<tensor_t> &tensor_stream,
               velocity_t outputs[MAX_HEIGHT][MAX_WIDTH]) {
    static outer_pixel_t buf[2];
    FLOW_OUTER:for (int r = 0; r < MAX_HEIGHT; r++) {
        FLOW_INNER:for (int c = 0; c < MAX_WIDTH; c++) {
#pragma HLS pipeline II = 1
            tensor_t tmp_tensor = tensor_stream.read();
            if (r >= 2 && r < MAX_HEIGHT - 2 && c >= 2 && c < MAX_WIDTH - 2) {
                calc_pixel_t t1 = (calc_pixel_t)tmp_tensor.val[0];
                calc_pixel_t t2 = (calc_pixel_t)tmp_tensor.val[1];
                calc_pixel_t t3 = (calc_pixel_t)tmp_tensor.val[2];
                calc_pixel_t t4 = (calc_pixel_t)tmp_tensor.val[3];
                calc_pixel_t t5 = (calc_pixel_t)tmp_tensor.val[4];
                calc_pixel_t t6 = (calc_pixel_t)tmp_tensor.val[5];

                calc_pixel_t denom = t1 * t2 - t4 * t4;
                calc_pixel_t numer0 = t6 * t4 - t5 * t2;
                calc_pixel_t numer1 = t5 * t4 - t6 * t1;

                if (denom != 0) {
                    buf[0] = numer0 / denom;
                    buf[1] = numer1 / denom;
                } else {
                    buf[0] = 0;
                    buf[1] = 0;
                }
            } else {
                buf[0] = buf[1] = 0;
            }

            outputs[r][c].x = (vel_pixel_t)buf[0];
            outputs[r][c].y = (vel_pixel_t)buf[1];
        }
    }
}
```

**2.6 并行计算设计**

这里沿用图像处理流水线的设计，完整代码详见 [Github](https://github.com/WenbinTeng/Terris/tree/main/app/optical-flow)。

```
       / gradient_xy_calc \
frames                      gradient_weight -> outer_product -> tensor_weight -> flow_calc
       \ gradient_z_calc  /
```



### 3. 参考文献

[1] Zhou Y, Gupta U, Dai S, et al. Rosetta: A realistic high-level synthesis benchmark suite for software programmable FPGAs[C]//Proceedings of the 2018 ACM/SIGDA International Symposium on Field-Programmable Gate Arrays. 2018: 269-278.
