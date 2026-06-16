# ESP-DSP 数字信号处理库详解

> 文档编号：35
> 适用芯片：ESP32 / ESP32-S3 / ESP32-P4 / ESP32-S31（官方优化）/ 全系列（ANSI C 参考实现）
> 适用场景：音频处理、FFT 频谱分析、滤波、神经网络/DSP 向量加速、信号处理应用
> 来源：乐鑫官方 ESP-DSP 文档、GitHub 仓库、Component Registry、API Reference、Benchmarks、Changelog

---

## 目录

1. [ESP-DSP 概述](#1-esp-dsp-概述)
2. [库结构与优化策略](#2-库结构与优化策略)
3. [信号处理（1D）API：dsps 前缀](#3-信号处理1dapidps-前缀)
4. [矩阵运算 API：dspm 前缀](#4-矩阵运算apidspm-前缀)
5. [ESP32-S3 性能基准](#5-esp32-s3-性能基准)
6. [示例工程](#6-示例工程)
7. [如何集成 ESP-DSP](#7-如何集成-esp-dsp)
8. [版本演进要点](#8-版本演进要点)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. ESP-DSP 概述

### 1.1 什么是 ESP-DSP

ESP-DSP 是乐鑫为所有 Espressif 芯片提供的**官方数字信号处理（DSP）库**，作为 ESP-IDF 组件使用。它包含针对 **ESP32、ESP32-S3、ESP32-P4**（及 ESP32-S31）CPU 架构优化的高性能计算函数与类，用于音频处理、FFT、滤波、矩阵运算、向量数学等场景。

### 1.2 核心能力

| 功能 | 说明 |
|------|------|
| **矩阵乘法** | 整型/浮点矩阵运算（C++ Mat 类） |
| **点积（Dot Product）** | 浮点/整型向量点积 |
| **FFT** | 快速傅里叶变换（radix-2 / radix-4，浮点/16 位定点） |
| **IIR** | 无限脉冲响应滤波器（含双二阶 biquad、立体声） |
| **FIR** | 有限脉冲响应滤波器（含抽取、多速率 Multirate） |
| **向量数学** | 加/减/乘、标量运算等基本向量操作 |
| **卡尔曼滤波** | 扩展卡尔曼滤波（EKF） |
| **DCT/DST** | 离散余弦/正弦变换（含 DCT-IV、DST-IV） |
| **窗函数 / 生成器 / 卷积** | FFT 窗、复音生成、2D 卷积、相关运算 |

---

## 2. 库结构与优化策略

### 2.1 双实现策略

ESP-DSP 对每个函数提供两种实现：

| 实现 | 后缀 | 说明 |
|------|------|------|
| **优化实现** | `_ae32`（ESP32）/ `_aes3`（ESP32-S3）/ `_arp4`（RISC-V/P4） | 汇编编写，针对特定 CPU 架构深度优化 |
| **参考实现** | `_ansi` | ANSI C 编写，可在任意平台编译运行 |

编译时通过宏自动选择当前芯片对应的优化版本，对开发者透明。

### 2.2 数据类型支持

- **单精度浮点**（32-bit float）—— 主力数据类型
- **16 位有符号整型**（int16）—— 定点优化
- **int8**（1.8.x 新增，行点积、矩阵×向量）

### 2.3 头文件

只需包含一个头文件即可使用全部 API：

```c
#include "esp_dsp.h"
```

---

## 3. 信号处理（1D）API：dsps 前缀

一维信号处理函数统一使用 `dsps` 前缀：

| 模块 | 代表函数 |
|------|----------|
| 点积 | `dsps_dotprod_f32`、`dsps_dotprod_s16` |
| FFT | `dsps_fft2r_fc32`（radix-2）、`dsps_fft4r_fc32`（radix-4）、`dsps_fft2r_sc16`（定点） |
| 位反转 | `dsps_bit_rev_fc32` |
| 复数转实数 | `dsps_cplx2reC_fc32` |
| IIR | `dsps_biquad_f32`（biquad 滤波器） |
| FIR | `dsps_fir_f32`、`dsps_fird_f32`（抽取 FIR）、`dsps_fird_s16` |
| 基本数学 | `dsps_mul_f32`、`dsps_mulc_f32`、`dsps_add_f32`、`dsps_sub_f32` |
| 窗函数 | Hanning / Hamming / Blackman 等窗生成 |
| 生成器 | `dsps_cplx_gen()`（复音生成） |
| DCT/DST | DCT、DST（含 IV 型） |
| 卷积/相关 | 1D/2D 卷积、相关 |
| 卡尔曼 | 扩展卡尔曼滤波（EKF） |

### FFT 初始化示例

```c
// 初始化 FFT 系数表（radix-2，浮点）
esp_err_t dsps_fft2r_init_fc32(float *fft_table_buff, int table_size);
void     dsps_fft2r_deinit_fc32(void);

// 执行 FFT
dsps_fft2r_fc32(data, N, w);      // N 点复数 FFT
dsps_bit_rev_fc32(data, N);       // 位反转
dsps_cplx2reC_fc32(data, N);      // 拆分为两个实信号谱
```

---

## 4. 矩阵运算 API：dspm 前缀

矩阵运算使用 `dspm` 前缀，并提供 C++ `Mat` 类（支持子矩阵操作）：

| 函数 | 说明 |
|------|------|
| `dspm_mult_f32` | 浮点矩阵乘法 C=A×B（如 16×16） |
| `dspm_mult_s16` | 16 位定点矩阵乘法 |
| `dspm_mult_3x3x1_f32` / `dspm_mult_3x3x3_f32` | 3×3 小矩阵优化 |
| `dspm_mult_4x4x1_f32` / `dspm_mult_4x4x4_f32` | 4×4 小矩阵优化 |
| `Mat` 类（C++） | 面向对象矩阵，支持子矩阵、行列点积 |

---

## 5. ESP32-S3 性能基准

ESP32-S3 凭借 **ESP-NN 优化的 aes3 指令集**，多项指标显著优于 ESP32（单位：CPU 周期，越小越快）：

### 5.1 FFT（radix-2，浮点）

| 点数（复数） | ESP32 O2 | **ESP32-S3 O2** |
|:---:|:---:|:---:|
| 64 | 4544 | **3970** |
| 256 | 23210 | **20139** |
| 1024 | 113205 | **97847** |

### 5.2 定点 FFT（16-bit，ESP32-S3 优势极大）

| 点数 | ESP32 | **ESP32-S3** |
|:---:|:---:|:---:|
| 64 | 8775 | **774** |
| 256 | 45746 | **3412** |
| 1024 | 226154 | **15623** |

### 5.3 矩阵乘法

| 运算 | ESP32 | **ESP32-S3** |
|------|:---:|:---:|
| `dspm_mult_f32` 16×16 | 24659 | **6280** |
| `dspm_mult_s16` 16×16 | 24697 | **2004** |

### 5.4 点积 / FIR / IIR

| 运算 | ESP32 | **ESP32-S3** |
|------|:---:|:---:|
| `dsps_dotprod_f32` N=256 | 1047 | **432** |
| `dsps_dotprod_s16` N=256 | 437 | **307** |
| `dsps_fir_f32` (1024样本,256系数) | 1078691 | **443671** |
| `dsps_biquad_f32` (1024样本) | 17442 | 17552 |

> 结论：定点 FFT、矩阵乘法、点积、FIR 在 ESP32-S3 上提升尤为显著，得益于 SIMD 向量指令优化。

---

## 6. 示例工程

ESP-DSP 提供完整示例，覆盖各 API 用法：

| 示例 | 说明 |
|------|------|
| `examples/dotprod` | 点积计算与性能对比 |
| `examples/basic_math` | 向量基本数学运算 |
| `examples/fft` | FFT 频谱分析（双信号、加窗、位反转） |
| `examples/fft_window` | 窗函数 + FFT |
| `examples/iir` | IIR 滤波器 |
| `examples/fir` | FIR 滤波器 |
| `examples/matrix` | Mat 类矩阵运算 |
| `examples/kalman` | 扩展卡尔曼滤波 |
| `examples/conv2d` | 2D 卷积 |

---

## 7. 如何集成 ESP-DSP

### 7.1 从 Component Registry 获取（推荐）

```bash
idf.py add-dependency "espressif/esp-dsp"
```

### 7.2 基本使用流程

```c
#include "esp_dsp.h"

// 1. 初始化库
esp_err_t ret = dsps_fft2r_init_fc32(NULL, CONFIG_DSP_MAX_FFT_SIZE);
assert(ret == ESP_OK);

// 2. 生成输入信号、加窗
// 3. 执行 FFT + 位反转
dsps_fft2r_fc32(fft_data, N, dsps_fft_w_table_fc32);
dsps_bit_rev_fc32(fft_data, N);

// 4. 用完释放
dsps_fft2r_deinit_fc32();
```

### 7.3 menuconfig 关键项

```
CONFIG_DSP_MAX_FFT_SIZE   // 最大 FFT 点数（决定系数表大小）
```

---

## 8. 版本演进要点

| 版本 | 时间 | 要点 |
|------|------|------|
| 1.8.2 | 2026-05 | 修复 int8 行点积 |
| 1.8.1 | 2026-04 | 新增 int8 行点积、int8 矩阵×向量 |
| 1.8.0 | 2026-04 | 支持 ESP32-S31 |
| 1.7.0 | 2025-06 | 多速率 FIR、基于 Multirate FIR 的重采样器 |
| 1.6.0 | 2025-04 | 立体声 IIR、DCT-IV/DST-IV、FFT2R/4R 针对 esp32 & esp32s3 优化 |
| 1.4.x | 2023–2024 | 大量 esp32s3 专项优化（FIR/biquad/memcpy/s8/s16） |

---

## 9. 应用场景

| 场景 | 涉及 API |
|------|----------|
| 音频频谱分析 | FFT + 窗函数 |
| 音频均衡/滤波 | IIR biquad、FIR |
| 语音前处理（配合 ESP-SR） | FFT、向量运算 |
| 神经网络推理加速 | 矩阵乘法、点积（与 ESP-NN 协同） |
| 传感器信号滤波 | FIR、IIR、卡尔曼 |
| 图像处理 | 2D 卷积 |
| 重采样 | Multirate FIR Resampler |

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-DSP GitHub | https://github.com/espressif/esp-dsp |
| ESP-DSP Component Registry | https://components.espressif.com/components/espressif/esp-dsp |
| ESP-DSP 概述 | https://docs.espressif.com/projects/esp-dsp/en/latest/esp-dsp-library.html |
| ESP-DSP API Reference | https://docs.espressif.com/projects/esp-dsp/en/latest/esp-dsp-apis.html |
| ESP-DSP Benchmarks | https://docs.espressif.com/projects/esp-dsp/en/latest/esp-dsp-benchmarks.html |
| ESP-DSP Examples | https://docs.espressif.com/projects/esp-dsp/en/latest/esp-dsp-examples.html |
| ESP-DSP Applications | https://docs.espressif.com/projects/esp-dsp/en/latest/esp-dsp-applications.html |
| Changelog | https://github.com/espressif/esp-dsp/blob/master/CHANGELOG.md |

---

> **文档总结**：ESP-DSP 是乐鑫官方 DSP 库，作为 ESP-IDF 组件提供 FFT、IIR/FIR 滤波、矩阵乘法、点积、卡尔曼滤波、DCT/DST、窗函数等完整算法。采用"汇编优化实现（_ae32/_aes3/_arp4）+ ANSI C 参考实现（_ansi）"双策略，支持浮点与 16 位定点。ESP32-S3 凭借 SIMD 向量指令在定点 FFT（64 点仅 774 周期 vs ESP32 的 8775）、矩阵乘法（s16 快 12 倍）、点积、FIR 等指标上提升巨大，是音频处理、信号滤波、神经网络加速的理想平台。可通过 `idf.py add-dependency "espressif/esp-dsp"` 一键集成。
