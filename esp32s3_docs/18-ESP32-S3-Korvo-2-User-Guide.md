# ESP32-S3-Korvo-2 用户指南

> **文档版本**: V3.1 (当前版本), 兼容 V3.0
> **状态**: 量产 (Mass Production)
> **原厂文档**: [ESP32-S3-Korvo-2 User Guide (ESP-ADF)](https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2.html)
> **BSP 组件**: [esp32_s3_korvo_2](https://components.espressif.com/components/espressif/esp32_s3_korvo_2)

---

## 目录

- [1. 概述](#1-概述)
- [2. 功能框图与架构](#2-功能框图与架构)
- [3. 硬件组件介绍](#3-硬件组件介绍)
- [4. 音频系统](#4-音频系统)
  - [4.1 双麦克风阵列](#41-双麦克风阵列)
  - [4.2 音频 ADC ES7210](#42-音频-adc-es7210)
  - [4.3 音频编解码器 ES8311](#43-音频编解码器-es8311)
  - [4.4 功率放大器 NS4150](#44-功率放大器-ns4150)
  - [4.5 AEC 回声消除电路](#45-aec-回声消除电路)
- [5. DSP 与音频处理](#5-dsp-与音频处理)
  - [5.1 ESP32-S3 AI/DSP 加速](#51-esp32-s3-aidsp-加速)
  - [5.2 声学前端 AFE](#52-声学前端-afe)
  - [5.3 语音唤醒 WakeNet](#53-语音唤醒-wakenet)
  - [5.4 语音命令词识别 MultiNet](#54-语音命令词识别-multinet)
- [6. 多媒体外设](#6-多媒体外设)
  - [6.1 LCD 显示屏](#61-lcd-显示屏)
  - [6.2 摄像头](#62-摄像头)
  - [6.3 microSD 卡](#63-microsd-卡)
- [7. BSP 板级支持包](#7-bsp-板级支持包)
- [8. 开始开发](#8-开始开发)
  - [8.1 必备硬件](#81-必备硬件)
  - [8.2 可选硬件](#82-可选硬件)
  - [8.3 硬件设置](#83-硬件设置)
  - [8.4 软件设置](#84-软件设置)
- [9. 内含组件和包装](#9-内含组件和包装)
- [10. AEC 电路设计详解](#10-aec-电路设计详解)
- [11. 应用场景](#11-应用场景)
- [12. 相关文档与资源](#12-相关文档与资源)

---

## 1. 概述

ESP32-S3-Korvo-2 是乐鑫 (Espressif) 推出的一款基于 ESP32-S3 的**多媒体解决方案开发板**。与 Korvo-1 专注于语音识别不同，Korvo-2 在语音功能基础上增加了 LCD 显示屏、摄像头等多媒体外设，是一款综合性的音视频开发平台。

### 核心特性

| 特性 | 说明 |
|------|------|
| **核心芯片** | ESP32-S3 (ESP32-S3-WROOM-1 模组) |
| **存储** | 16 MB Flash + 8 MB PSRAM |
| **麦克风阵列** | **双麦克风 (2 Mic) 阵列**，支持语音识别和近/远场语音唤醒 |
| **音频编解码** | ES8311 (编解码器) + ES7210 (ADC) |
| **功放** | NS4150，3W D 类功放 |
| **LCD 显示** | 支持 LCD 扩展板 (ILI9341, 触摸屏) |
| **摄像头** | 支持 DVP 摄像头 (OV3660) |
| **JPEG 视频流** | 支持 JPEG 视频流处理 |
| **SD 卡** | 支持 microSD 卡 |
| **工作温度** | -40 C ~ 85 C |
| **板尺寸** | 115 x 80 mm |
| **接口** | I/O, USB |
| **外设** | 按键, TF 卡, Camera, LCD |
| **参考价格** | 约 329 元人民币 |

### 版本信息

| 版本 | 状态 |
|------|------|
| **V3.0** | 已发布 |
| **V3.1** | 当前版本 (与 V3.0 使用方法相同) |

> BSP 组件兼容 V3.0 和 V3.1 版本。

### 与 Korvo-1 的定位对比

| 特性 | ESP32-S3-Korvo-1 | ESP32-S3-Korvo-2 |
|------|-------------------|-------------------|
| **定位** | AI 语音识别开发板 | 多媒体开发板 |
| **麦克风** | 3 麦克风阵列 | 2 麦克风阵列 |
| **LCD 显示** | 无 | 支持 (ILI9341) |
| **触摸屏** | 无 | 支持 (GT911 / TT21100) |
| **摄像头** | 无 | 支持 (OV3660) |
| **主要 SDK** | ESP-Skainet | ESP-ADF (Audio Dev Framework) |
| **板形态** | 圆形 (88 mm 直径) | 矩形 (115 x 80 mm) |
| **板结构** | 主板 + 子板 (FPC 连接) | 单板设计 |

---

## 2. 功能框图与架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    ESP32-S3-Korvo-2 V3.1                         │
│                                                                  │
│  ┌────────────────┐                                              │
│  │ ESP32-S3       │     ┌────────────────┐    ┌───────────────┐ │
│  │ WROOM-1        │ I2S │  Audio Codec   │    │   扬声器       │ │
│  │ 16MB Flash     │◄───►│    ES8311      │───►│  (Speaker)    │ │
│  │  8MB PSRAM     │ I2C │   (ADC+DAC)    │    └───────────────┘ │
│  │                │◄───►│                │                      │
│  │  AI/DSP 加速   │     └───────┬────────┘    ┌───────────────┐ │
│  │  (SIMD)        │             │ DAC 输出     │   耳机/音频    │ │
│  │                │     ┌───────┴────────┐    │   输出         │ │
│  │                │     │  Audio PA      │    └───────────────┘ │
│  │                │     │   NS4150       │                      │
│  │                │     │  (3W D-Class)  │                      │
│  │                │     └────────────────┘                      │
│  │                │                                             │
│  │                │     ┌────────────────┐                      │
│  │                │ I2S │  Audio ADC     │  ◄── 左侧麦克风      │
│  │                │◄───►│   ES7210       │  ◄── 右侧麦克风      │
│  │                │ I2C │  (4-Channel)   │  ◄── AEC 参考信号    │
│  │                │◄───►│                │                      │
│  └──────┬─────────┘     └────────────────┘                      │
│         │                                                         │
│    ┌────┴────────────────────────────────────┐                   │
│    │            外设接口                      │                   │
│    │  ┌──────┐ ┌──────┐ ┌──────┐ ┌────────┐ │                   │
│    │  │ LCD  │ │Camera│ │microSD│ │功能按键 │ │                   │
│    │  │Connector│ │Conn │ │ Slot │ │ x6    │ │                   │
│    │  └──────┘ └──────┘ └──────┘ └────────┘ │                   │
│    │  ┌──────┐ ┌──────┐ ┌──────────────────┐│                   │
│    │  │ USB  │ │ USB  │ │ Battery Socket   ││                   │
│    │  │ Power│ │ UART │ │ + Charger (AP5056)││                   │
│    │  └──────┘ └──────┘ └──────────────────┘│                   │
│    └────────────────────────────────────────┘                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. 硬件组件介绍

以下按照顺时针的顺序依次介绍开发板上的主要组件：

| 主要组件 | 介绍 |
|----------|------|
| **ESP32-S3-WROOM-1 模组** | ESP32-S3-WROOM-1 模组是通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，搭载 ESP32-S3 系列芯片。除具有丰富的外设接口外，模组还拥有强大的神经网络运算能力和信号处理能力，适用于 AIoT 领域的多种应用场景，例如唤醒词检测和语音命令识别、人脸检测和识别、智能家居、智能家电、智能控制面板、智能扬声器等。 |
| **左侧麦克风 (Left Microphone)** | 板载麦克风，连接至 ADC (ES7210)。 |
| **音频模数转换器 (Audio ADC Chip)** | [ES7210](http://www.everest-semi.com/pdf/ES8311%20PB.pdf) 是一款用于麦克风阵列应用的高性能、低功耗 4 通道音频模数转换器，非常适合音乐和语音应用。此外，ES7210 也可以用作声学回声消除 (AEC) 的参考信号采集。 |
| **音频编解码芯片 (Audio Codec Chip)** | [ES8311](http://www.everest-semi.com/pdf/ES8311%20PB.pdf) 是一种低功耗单声道音频编解码器，包含单通道 ADC、单通道 DAC、低噪声前置放大器、耳机驱动器、数字音效、模拟混音和增益功能。通过 I2S 和 I2C 总线与 ESP32-S3-WROOM-1 模组连接。 |
| **音频功率放大器 (Audio PA Chip)** | NS4150 是一款低 EMI、3W 单声道 D 类音频功率放大器，用于放大来自音频编解码芯片的音频信号，以驱动扬声器。 |
| **右侧麦克风 (Right Microphone)** | 板载麦克风，连接至 ADC (ES7210)。 |
| **扬声器输出端口 (Speaker Output Port)** | 可通过音频功率放大器的支持，实现外部扬声器播放功能。 |
| **USB-to-UART 桥接器** | 单芯片 USB-UART 桥接器 CP2102N，为软件下载和调试提供高达 3 Mbps 的传输速率。 |
| **USB-to-UART 端口** | 用于 PC 端与 ESP32-S3-WROOM-1 模组的通信。 |
| **USB 电源端口** | 为整个系统提供电源。建议使用至少 5V/2A 电源适配器供电，保证供电稳定。 |
| **电池接口** | 用于连接单节锂离子电池的两针接口。 |
| **电源开关** | 向下拨动开启开发板电源，向上拨动关闭开发板电源。 |
| **电池充电管理芯片** | AP5056 恒流/恒压线性电源管理芯片，用于单节锂离子电池充电管理。 |
| **功能按键** | 六个按键：REC、MUTE、PLAY、SET、VOL- 和 VOL+，与 ESP32-S3-WROOM-1 模组连接。 |
| **Boot/Reset 按键** | Boot：长按 Boot 键再按 Reset 键可启动固件上传模式。Reset：单独按下重置系统。 |
| **microSD 插槽** | 支持一线模式的 microSD 卡，可存储或播放音频文件。 |
| **LCD 连接器** | 0.5 mm 间距的 FPC 连接器，用以连接 LCD 扩展板。 |
| **系统 LED** | 两个通用 LED（绿色和红色），由模组控制，可作状态指示。 |
| **摄像头连接器** | 通过连接器外接摄像头模组至开发板，实现图像传输。 |

---

## 4. 音频系统

### 4.1 双麦克风阵列

ESP32-S3-Korvo-2 搭载 **双麦克风 (2 Mic) 阵列**，适用于语音识别和近/远场语音唤醒产品开发。

#### 麦克风阵列配置

| 参数 | 说明 |
|------|------|
| **麦克风数量** | 2 个 |
| **类型** | 模拟麦克风 |
| **连接方式** | 通过 ES7210 ADC 采集 |
| **AEC 参考通道** | ES7210 第 3 通道用于 AEC 参考信号 |
| **通道布局** | 输入格式通常为 `MMNR` (麦克风1, 麦克风2, 未使用, 参考信号) |

#### 双麦阵列优势

| 特性 | 说明 |
|------|------|
| **360 度拾音** | 配合 ESP32-S3 AFE 算法，仅使用两个麦克风即可实现 360 度拾音 |
| **盲源分离 (BSS)** | 双麦 BSS 算法实现目标声源和干扰音分离 |
| **波束成形** | 检测声源方向，强化目标方向音频 |
| **远场唤醒** | 支持远场语音唤醒和识别 |
| **Amazon Alexa 认证** | 双麦 AFE 方案已通过 Amazon Alexa Software AFE 认证 |

### 4.2 音频 ADC ES7210

[ES7210](http://www.everest-semi.com/pdf/ES7210%20PB.pdf) 是高性能、低功耗 4 通道音频 ADC：

| 特性 | 说明 |
|------|------|
| **通道数** | 4 通道 |
| **通道分配** | 通道 1-2: 麦克风; 通道 3: AEC 参考信号; 通道 4: 预留 |
| **信噪比 (SNR)** | 102 dB |
| **THD+N** | -85 dB |
| **位深** | 24 位 |
| **采样频率** | 8 ~ 100 kHz |
| **数据端口** | I2S/PCM 主从串行数据端口，支持 TDM 模式 |
| **通信接口** | I2S + I2C |
| **工作模式** | 支持从模式 (由 ESP32-S3 控制) |

### 4.3 音频编解码器 ES8311

[ES8311](http://www.everest-semi.com/pdf/ES8311%20PB.pdf) 是低功耗单声道音频编解码器：

| 特性 | 说明 |
|------|------|
| **ADC** | 单通道多比特 Delta-Sigma ADC |
| **DAC** | 单通道多比特 Delta-Sigma DAC |
| **前置放大器** | 低噪声前置放大器 |
| **耳机驱动** | 内置耳机驱动器 |
| **数字音效** | 内置数字音效处理器 |
| **模拟混音** | 支持模拟混音 |
| **增益控制** | 内置增益函数 |
| **通信接口** | I2S (数据) + I2C (控制) |

**音频数据流**：

```
播放路径:
ESP32-S3 → I2S → ES8311 DAC → NS4150 PA → 扬声器

录音路径:
麦克风 → ES7210 ADC → I2S → ESP32-S3

AEC 参考:
ES8311 DAC 输出 → ES7210 通道3 → ESP32-S3 (AEC 算法)
```

### 4.4 功率放大器 NS4150

NS4150 是板载的 D 类音频功放：

| 参数 | 数值 |
|------|------|
| **类型** | D 类功放 |
| **输出功率** | 3W |
| **EMI 特性** | 低 EMI，无需滤波器 |
| **通道** | 单声道 |
| **输入** | 来自 ES8311 DAC 的模拟音频信号 |
| **输出** | 驱动外接扬声器 |

### 4.5 AEC 回声消除电路

ESP32-S3-Korvo-2 设计了专用的 AEC (声学回声消除) 参考信号采集电路，这是其音频系统的核心设计之一。

#### AEC 参考信号源

开发板有**两路兼容的回声参考信号设计**：

| 信号源 | 说明 | 默认状态 |
|--------|------|----------|
| **Codec (ES8311) DAC 输出** | 从 ES8311 的 DAC_AOUTLN/DAC_AOUTLP 采集 | **默认推荐** |
| **PA (NS4150) 输出** | 从 PA 的 PA_OUTL+/PA_OUTL- 采集 | 备选 (需焊接 R132, R140) |

> 默认推荐使用 ES8311 DAC 输出作为回声参考信号，此时电阻 R132、R140 不上件 (NC)。

#### AEC 信号路径

```
ES8311 DAC 输出 (DAC_AOUTLN/DAC_AOUTLP)
         │
         ▼
ES7210 ADC 通道 3 (ADC_MIC3P/ADC_MIC3N)
         │
         ▼
    ESP32-S3 (用于 AEC 算法)
```

#### AEC 的作用

AEC 电路使得设备在**播放音频的同时**仍能准确识别语音命令：

1. 设备通过扬声器播放音乐或语音提示
2. 扬声器声音被麦克风拾取形成回声
3. AEC 算法使用参考信号消除回声
4. 清理后的音频送入 WakeNet / MultiNet 进行识别

---

## 5. DSP 与音频处理

### 5.1 ESP32-S3 AI/DSP 加速

ESP32-S3-Korvo-2 的音频处理能力来源于 ESP32-S3 芯片的 AI/DSP 加速特性：

| 特性 | 说明 |
|------|------|
| **向量指令 (SIMD)** | 专门用于加速神经网络计算和信号处理 |
| **双核处理器** | Xtensa 32 位 LX7 双核，最高 240 MHz |
| **PSRAM** | 8 MB Octal PSRAM，为 AI 模型提供充足内存 |
| **Flash** | 16 MB Quad Flash，存储固件和模型数据 |

#### AFE 算法资源消耗

基于 ESP32-S3 AI 加速器优化后的 AFE 算法资源消耗：

| 资源 | 消耗量 |
|------|--------|
| **CPU 占用** | 约 12-22% |
| **内部 SRAM** | 约 48 KB |
| **PSRAM** | 约 1.1 MB |
| **总存储空间** | 约 460 KB (220 KB 内存 + 240 KB 外部存储) |

### 5.2 声学前端 AFE

ESP32-S3-Korvo-2 使用 ESP-SR 的 [AFE (Audio Front-End)](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/README.html) 声学前端算法框架。

#### AFE 算法组件

| 算法 | 功能 | Korvo-2 适用场景 |
|------|------|------------------|
| **AEC** | 声学回声消除，消除扬声器回声 | 设备播放音频时仍能识别语音 |
| **BSS** | 盲源分离，双麦检测声源方向 | 双麦阵列实现方向性拾音 |
| **NS** | 噪声抑制，消除非人声噪声 | 家电、空调等稳态噪声环境 |
| **MISO** | 多输入单输出 | 双麦场景信噪比选择 |
| **VAD** | 语音活动检测 | 过滤无效音频段，节省资源 |
| **AGC** | 自动增益控制 | 动态调整音频幅值 |
| **WakeNet** | 唤醒词引擎 | 持续监听唤醒词 |

#### AFE 典型配置 (ESP32-S3-Korvo-2)

基于 BSP 日志中的实际配置参数：

```
/********** General AFE Parameter **********/
pcm_config.total_ch_num: 4
pcm_config.mic_num: 2: [ ch0, ch2 ]
pcm_config.ref_num: 1: [ ch1 ]
pcm_config.sample_rate: 16000
afe_type: SR
afe_mode: HIGH PERF
afe_perferred_core: 0
afe_perferred_priority: 5
afe_ringbuf_size: 50
memory_alloc_mode: 3
afe_linear_gain: 1.0
fixed_first_channel: true

/********** AEC (Acoustic Echo Cancellation) **********/
aec_init: true
aec mode: SR_HIGH_PERF
aec_filter_length: 4

/********** SE (Speech Enhancement, Microphone Array Processing) **********/
se_init: true, model: BSS

/********** NS (Noise Suppression) **********/
ns_init: false
ns model name: WEBRTC

/********** VAD (Voice Activity Detection) **********/
vad_init: false
vad_mode: 0

/********** WakeNet (Wake Word Engine) **********/
wakenet_init: false
wakenet_model_name: NULL

/********** AGC (Automatic Gain Control) **********/
agc_init: false
agc_mode: WAKENET
agc_compression_gain_db: 9
agc_target_level_dbfs: 9
```

#### 输入数据格式

Korvo-2 的音频输入格式通常为 `MMNR`：

| 通道 | 数据内容 |
|------|----------|
| ch0 (M) | 左侧麦克风 |
| ch1 (M) | 右侧麦克风 |
| ch2 (N) | 未使用 |
| ch3 (R) | AEC 播放参考信号 |

#### AFE 工作流程

```
四通道音频输入 (MMNR)
    │
    ▼
┌────────────────────┐
│       AEC          │ ─── 消除回声 (使用通道3参考信号)
│  (回声消除)         │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│    BSS / NS        │ ─── 双麦用 BSS 盲源分离
│  (降噪/声源分离)    │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│       VAD          │ ─── 语音活动检测
│  (语音检测)         │
└─────────┬──────────┘
          │
          ▼
   单通道增强音频 → WakeNet → MultiNet
```

#### AFE API 使用示例

```c
#include "esp_afe_sr_models.h"
#include "model_path.h"

// 步骤1: 初始化模型
srmodel_list_t *models = esp_srmodel_init("model");

// 步骤2: 初始化 AFE 配置 (Korvo-2 双麦配置)
afe_config_t *afe_config = afe_config_init("MMNR", models, AFE_TYPE_SR, AFE_MODE_HIGH_PERF);

// 步骤3: 创建 AFE 实例
esp_afe_sr_iface_t *afe_handle = esp_afe_handle_from_config(afe_config);
esp_afe_sr_data_t *afe_data = afe_handle->create_from_config(afe_config);

// 步骤4: 输入音频 (4通道交错数据)
int feed_chunksize = afe_handle->get_feed_chunksize(afe_data);
int feed_nch = afe_handle->get_feed_channel_num(afe_data); // 4 channels
int16_t *feed_buff = (int16_t *)malloc(feed_chunksize * feed_nch * sizeof(int16_t));
afe_handle->feed(afe_data, feed_buff);

// 步骤5: 获取处理后的单通道音频和状态
afe_fetch_result_t *result = afe_handle->fetch(afe_data);
int16_t *processed_audio = result->data;
wakenet_state_t wakeup_state = result->wakeup_state;
```

### 5.3 语音唤醒 WakeNet

ESP32-S3-Korvo-2 与 Korvo-1 使用相同的 [WakeNet](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html) 唤醒词引擎。

#### 支持的唤醒词

| 唤醒词 | 说明 |
|--------|------|
| **Hi 乐鑫** (wn9_hilexin) | 中文唤醒词 |
| **Hi ESP** (wn9_hiesp) | 英文唤醒词 |
| **你好小智** | 中文唤醒词 |
| **你好小鑫** | 中文唤醒词 |
| **小爱同学** | 中文唤醒词 |
| **Alexa** | 英文唤醒词 |
| **自定义唤醒词** | 乐鑫提供定制服务 |

#### WakeNet9 性能

| 参数 | 数值 |
|------|------|
| **模型参数量** | < 300K |
| **每帧推理时间** | 约 3 ms (32 ms 帧) |
| **音频格式** | 16 kHz, 单声道, 16-bit |
| **特征提取** | MFCC |
| **同时唤醒词** | 最多 5 个 |
| **1米安静环境准确率** | 98% |
| **1米噪声环境 (SNR 4dB)** | 94-96% |

### 5.4 语音命令词识别 MultiNet

[MultiNet](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_command_recognition/README.html) 提供离线语音命令词识别：

| 特性 | 说明 |
|------|------|
| **最大命令数** | 200 个中英文命令 |
| **无需重训练** | 自定义命令无需重新训练模型 |
| **工作流程** | WakeNet 唤醒后激活 |
| **超时退出** | 静默超时后返回唤醒模式 |

#### 命令词自定义

```c
// 添加命令词
esp_mn_commands_add(cmd_id_1, "打开空调");      // 中文
esp_mn_commands_add(cmd_id_2, "TURN ON THE LIGHT"); // 英文
esp_mn_commands_update(); // 应用更改
```

---

## 6. 多媒体外设

### 6.1 LCD 显示屏

ESP32-S3-Korvo-2 支持 LCD 扩展板，通过 0.5 mm FPC 连接器连接。

#### LCD 规格 (ESP32-S3-Korvo-2-LCD 扩展板)

| 参数 | 说明 |
|------|------|
| **LCD 控制器** | ILI9341 |
| **触摸控制器** | GT911 或 TT21100 (自动检测) |
| **接口** | SPI (I80 接口) |
| **图形库** | LVGL (通过 esp_lvgl_port) |
| **兼容性** | 需 ESP-IDF >= 5.4 |

#### BSP LCD 能力

| 能力 | 控制器 | 组件 |
|------|--------|------|
| DISPLAY | ILI9341 | esp_lcd_ili9341 |
| TOUCH | GT911, TT21100 | esp_lcd_touch_gt911 / esp_lcd_touch_tt21100 |
| LVGL_PORT | - | esp_lvgl_port ^2 |

> LCD 显示屏为可选配件，需单独购买 ESP32-S3-Korvo-2-LCD 扩展板。

### 6.2 摄像头

ESP32-S3-Korvo-2 通过摄像头连接器支持外接 DVP 摄像头模组。

| 参数 | 说明 |
|------|------|
| **推荐摄像头** | OV3660 |
| **接口** | DVP (Digital Video Port) |
| **视频处理** | 支持 JPEG 视频流处理 |
| **BSP 组件** | esp_video ~2.0 |

### 6.3 microSD 卡

| 参数 | 说明 |
|------|------|
| **接口模式** | 一线模式 (1-bit mode) / SDMMC |
| **用途** | 存储音频文件、图片、视频等数据 |
| **BSP 依赖** | ESP-IDF >= 5.4 |

---

## 7. BSP 板级支持包

ESP32-S3-Korvo-2 提供完整的 [BSP (Board Support Package)](https://components.espressif.com/components/espressif/esp32_s3_korvo_2)，简化开发流程。

### BSP 能力总览

| 能力 | 支持 | 控制器/编解码器 | 组件 |
|------|------|-----------------|------|
| DISPLAY | ✅ | ILI9341 | esp_lcd_ili9341 |
| LVGL_PORT | ✅ | - | esp_lvgl_port ^2 |
| TOUCH | ✅ | GT911, TT21100 | esp_lcd_touch_gt911 / tt21100 |
| BUTTONS | ✅ | - | espressif/button ^4 |
| AUDIO | ✅ | - | esp_codec_dev ~1.5 |
| AUDIO_SPEAKER | ✅ | ES8311 | - |
| AUDIO_MIC | ✅ | ES7210 | - |
| SDCARD | ✅ | - | idf >= 5.4 |
| LED | ✅ | - | idf + led_indicator ^2 |
| CAMERA | ✅ | OV3660 | esp_video ~2.0 |

### 音频编程接口

BSP 提供标准化的音频编程接口：

```c
// 初始化 I2S 音频
esp_err_t bsp_audio_init(const i2s_std_config_t *i2s_config);

// 初始化扬声器 (ES8311 DAC)
esp_codec_dev_handle_t spk_codec_dev = bsp_audio_codec_speaker_init();

// 初始化麦克风 (ES7210 ADC)
esp_codec_dev_handle_t mic_codec_dev = bsp_audio_codec_microphone_init();

// 播放音频
esp_codec_dev_set_out_vol(spk_codec_dev, DEFAULT_VOLUME);
esp_codec_dev_open(spk_codec_dev, &fs);
esp_codec_dev_write(spk_codec_dev, wav_bytes, bytes_read);
esp_codec_dev_close(spk_codec_dev);
```

### 通过 IDF Component Manager 使用 BSP

在项目的 `idf_component.yml` 中添加依赖：

```yaml
dependencies:
  espressif/esp32_s3_korvo_2: "^1.0.0"
```

---

## 8. 开始开发

### 8.1 必备硬件

- 1 x ESP32-S3-Korvo-2 V3.1 开发板
- 1 x 扬声器
- 2 x USB 2.0 数据线 (Standard-A to Micro-B)
- 1 x 电脑 (Windows、Linux 或 macOS)

> 请确保使用适当的 USB 数据线。部分数据线仅可用于充电，无法用于数据传输和编程。

### 8.2 可选硬件

- 1 x LCD 扩展板 (ESP32-S3-Korvo-2-LCD)
- 1 x 摄像头模组 (OV3660)
- 1 x microSD 卡
- 1 x 锂离子电池 (务必使用内置保护电路的锂电池)

### 8.3 硬件设置

1. 连接扬声器至 **扬声器输出** 端口
2. (可选) 连接 LCD 扩展板至 LCD 连接器
3. (可选) 连接摄像头模组至摄像头连接器
4. 插入 USB 数据线，分别连接 PC 与开发板的两个 USB 端口
5. 此时绿色待机指示灯应亮起。若电池未连接，红色充电指示灯每隔几秒闪烁一次
6. 打开 **电源开关**
7. 红色电源指示灯应亮起

### 8.4 软件设置

#### ESP-ADF 开发环境

ESP32-S3-Korvo-2 主要使用 [ESP-ADF (Audio Development Framework)](https://github.com/espressif/esp-adf) 进行开发：

```bash
# 1. 克隆 ESP-ADF
git clone --recursive https://github.com/espressif/esp-adf.git

# 2. 设置 ESP-IDF (使用 ESP-ADF 推荐版本)
# 参考: https://docs.espressif.com/projects/esp-adf/zh_CN/latest/get-started/index.html

# 3. 设置目标芯片
idf.py set-target esp32s3

# 4. 编译和烧录
idf.py flash monitor
```

#### ESP-Skainet 语音开发

ESP32-S3-Korvo-2 同样可以用于 ESP-Skainet 语音识别开发：

```bash
# 克隆 ESP-Skainet
git clone --recursive https://github.com/espressif/esp-skainet.git

# 配置开发板
idf.py set-target esp32s3
idf.py menuconfig
# Audio Media HAL -> Audio hardware board -> ESP32-S3-Korvo-2

# 编译和烧录
idf.py flash monitor
```

#### BSP 方式开发

使用 BSP 组件简化硬件初始化：

```cmake
# 在项目中添加 BSP 依赖
# idf_component.yml:
dependencies:
  espressif/esp32_s3_korvo_2: "^1.0.0"
```

```c
#include "bsp/esp-bsp.h"

// BSP 自动初始化所有硬件外设
bsp_display_start();      // 初始化 LCD
bsp_audio_init(NULL);     // 初始化音频 (使用默认配置)
bsp_display_brightness_set(100); // 设置亮度
```

---

## 9. 内含组件和包装

### 可购买的产品

| 产品型号 | 说明 |
|----------|------|
| **ESP32-S3-Korvo-2** | 主板 (含双麦克风阵列) |
| **ESP32-S3-Korvo-2-LCD** | LCD 扩展板配件 |

### 配件包含

主板可分开购买，配件包含：
- LCD 扩展板：ESP32-S3-Korvo-2-LCD
- 摄像头
- 连接器：20 针 FPC 线
- 紧固件：安装螺栓 (x8) + 螺丝 (x4)

### 购买渠道

| 渠道 | 链接 |
|------|------|
| 淘宝 | [ESP32-S3-Korvo-2](https://item.taobao.com/item.htm?id=660827533046) |
| 京东 | [ESP32-S3-Korvo-2](https://item.jd.com/10206944840952.html) |
| Digi-Key | [ESP32-S3-KORVO-2](https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-KORVO-2/15822448) |
| Mouser | [ESP32-S3-KORVO-2](https://www.mouser.com/ProductDetail/Espressif-Systems/ESP32-S3-KORVO-2) |
| Amazon US | [ESP32-S3-Korvo-2](https://www.amazon.com/dp/B09JZ8J7FY) |

---

## 10. AEC 电路设计详解

ESP32-S3-Korvo-2 的 AEC 电路设计是其音频系统的核心亮点，确保设备在播放音频时仍能准确识别语音。

### AEC 设计原理

```
                    ┌─────────────────────────────────────────┐
                    │              音频播放链路                 │
                    │                                         │
  数字音频 ──► ES8311 DAC ──► NS4150 PA ──► 扬声器 ──► 空气传播
                    │                       │            │
                    │    ┌──────────────┐   │            │
                    │    │ AEC 参考      │   │            │
                    │    │ 信号采集      │   │            ▼
                    │    │ (ADC ch3)    │   │      ┌──────────┐
                    │    └──────┬───────┘   │      │  麦克风   │
                    │           │           │      │ (回声)   │
                    │           ▼           │      └────┬─────┘
                    │     ES7210 ADC        │           │
                    │     通道3             │           │
                    │           │           │           │
                    └───────────┼───────────┼───────────┘
                                │           │
                                ▼           ▼
                         ┌─────────────────────┐
                         │    ESP32-S3 AEC     │
                         │    回声消除算法      │
                         │                     │
                         │  参考信号 - 回声    │
                         │     = 干净语音      │
                         └─────────────────────┘
```

### 信号源选择

| 信号源 | 引脚 | 默认 | 说明 |
|--------|------|------|------|
| ES8311 DAC 输出 | DAC_AOUTLN / DAC_AOUTLP | **推荐** | 更早的参考点，延迟更小 |
| NS4150 PA 输出 | PA_OUTL+ / PA_OUTL- | 备选 | 包含 PA 失真，R132/R140 需上件 |

### AEC 算法性能

基于 ESP32-S3 的 AEC 算法资源消耗 (16 kHz 采样率)：

| 模式 | 内部 RAM (KB) | PSRAM (KB) | 每帧耗时 (ms) | CPU (%) |
|------|---------------|------------|---------------|---------|
| SR_LOW_COST | 18.8 | 64.0 | 2.29/32 | 7.2 |
| SR_HIGH_PERF | 8.2 | 100.1 | 4.51/32 | 14.1 |
| FD_LOW_COST | 30.9 | 90.0 | 6.28/32 | 19.6 |
| FD_HIGH_PERF | 20.3 | 126.2 | 8.08/32 | 25.3 |

> 测试条件: ESP32-S3 @ 240 MHz, 32 ms 帧长

---

## 11. 应用场景

ESP32-S3-Korvo-2 凭借其丰富的多媒体和音频功能，适用于多种应用场景：

### 语音交互产品

| 应用 | 说明 |
|------|------|
| 智能音箱 | 语音唤醒 + 语音命令 + 音频播放 |
| 智能家电控制面板 | LCD 显示 + 语音控制 + 触摸交互 |
| 智能门铃 | 摄像头 + 语音对讲 + LCD 显示 |
| 语音助手 | 离线语音唤醒和命令识别 |

### 多媒体产品

| 应用 | 说明 |
|------|------|
| 网络摄像头 | JPEG 视频流处理 + 音频采集 |
| 智能相框 | LCD 图片显示 + TF 卡存储 |
| 视频对讲 | 摄像头 + LCD + 双向音频 |
| 安防监控 | 摄像头 + 语音告警 |

### IoT 网关

| 应用 | 说明 |
|------|------|
| 智能家居中控 | Wi-Fi + 蓝牙 + 语音控制 + LCD |
| 工业监控终端 | 传感器数据 + 语音告警 + 显示 |
| 教育设备 | 语音交互 + 多媒体教学 |

---

## 12. 相关文档与资源

### 官方文档

| 文档 | 链接 |
|------|------|
| ESP32-S3-Korvo-2 用户指南 (中文) | [ESP-ADF 文档](https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2.html) |
| ESP32-S3-Korvo-2 V3.0 用户指南 | [V3.0 文档](https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2-v3.0.html) |
| BSP API 文档 | [API.md](https://github.com/espressif/esp-bsp/blob/master/bsp/esp32_s3_korvo_2/API.md) |

### 技术规格书

- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf) (PDF)
- [ESP32-S3-WROOM-1 & ESP32-S3-WROOM-1U 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_cn.pdf) (PDF)

### 芯片规格书

| 芯片 | 文档 |
|------|------|
| ES8311 (音频编解码器) | [ES8311 PB](http://www.everest-semi.com/pdf/ES8311%20PB.pdf) |
| ES7210 (4通道ADC) | [ES7210 PB](http://www.everest-semi.com/pdf/ES7210%20PB.pdf) |

### 软件资源

| 资源 | 链接 |
|------|------|
| ESP-ADF (音频开发框架) | [https://github.com/espressif/esp-adf](https://github.com/espressif/esp-adf) |
| ESP-Skainet (语音识别 SDK) | [https://github.com/espressif/esp-skainet](https://github.com/espressif/esp-skainet) |
| ESP-SR (语音识别框架) | [https://github.com/espressif/esp-sr](https://github.com/espressif/esp-sr) |
| ESP-BSP (板级支持包) | [BSP: ESP32-S3-Korvo-2](https://github.com/espressif/esp-bsp/blob/master/bsp/esp32_s3_korvo_2/README.md) |
| ESP-IDF | [编程指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/) |
| ESP-SR 文档 | [AFE 文档](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/README.html) |

### Component Registry

| 组件 | 链接 |
|------|------|
| BSP 完整版 | [esp32_s3_korvo_2](https://components.espressif.com/components/espressif/esp32_s3_korvo_2) |
| BSP 无 GLib 版 | [esp32_s3_korvo_2_noglib](https://components.espressif.com/components/espressif/esp32_s3_korvo_2_noglib) |
| Board Manager | [esp_board_manager](https://components.espressif.com/components/espressif/esp_board_manager) |

### 兼容示例

| 示例 | 说明 |
|------|------|
| Audio Example | 播放和录制 WAV 文件 |
| ESP Launchpad | 在线烧录示例 |

### AFE 算法文档

| 算法 | 文档链接 |
|------|----------|
| AFE 声学前端 | [AFE README](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/README.html) |
| AEC 声学回声消除 | [AEC README](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/acoustic_echo_cancellation/README.html) |
| WakeNet 唤醒词 | [WakeNet README](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html) |
| MultiNet 命令词 | [MultiNet README](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_command_recognition/README.html) |

---

## 附录 A: ESP32-S3-Korvo-2 在乐鑫 Board Manager 中的注册信息

| 字段 | 值 |
|------|-----|
| Board Name | ESP32-S3 Korvo2 V3 |
| Chip | ESP32-S3 |
| Audio | ES8311 + ES7210 |
| SD Card | SDMMC |
| LCD | ILI9341 |
| LCD Touch | TT21100 / GT911 自动检测 |
| Camera | DVP Camera (OV3660) |
| Button | ADC Button |

---

## 附录 B: 系统日志示例 (音频初始化)

以下是 ESP32-S3-Korvo-2 V3 启动时的典型音频初始化日志：

```
I (1062) MAIN: [1/4] Initializing board...
I (1066) PERIPH_I2C: I2C master bus initialized successfully
I (1072) PERIPH_I2S: I2S[0] TDM,  TX, ws: 45, bclk: 9, dout: 8, din: 10
I (1078) PERIPH_I2S: I2S[0] initialize success
I (1083) PERIPH_I2S: I2S[0] TDM, RX, ws: 45, bclk: 9, dout: 8, din: 10
I (1089) PERIPH_I2S: I2S[0] initialize success

I (1126) ES8311: Work in Slave mode
I (1129) DEV_AUDIO_CODEC: Successfully initialized codec: audio_dac
I (1130) DEV_AUDIO_CODEC: Create esp_codec_dev success, chip:es8311

I (1143) DEV_AUDIO_CODEC: ADC over I2S is enabled
I (1155) ES7210: Work in Slave mode
I (1162) ES7210: Enable ES7210_INPUT_MIC1
I (1165) ES7210: Enable ES7210_INPUT_MIC2
I (1167) ES7210: Enable ES7210_INPUT_MIC3
I (1170) ES7210: Enable TDM mode
I (1173) DEV_AUDIO_CODEC: Successfully initialized codec: audio_adc
I (1176) DEV_AUDIO_CODEC: Create esp_codec_dev success, chip:es7210

I (1191) BOARD_MANAGER: Name: esp32_s3_korvo2_v3
I (1195) BOARD_MANAGER: Chip: esp32s3
I (1199) BOARD_MANAGER: Version: 1.3.0
I (1203) BOARD_MANAGER: Description: ESP32-S3 Korvo2 V3 Development Board
I (1213) BOARD_MANAGER: Manufacturer: ESPRESSIF
```

---

> **文档来源**: 本指南基于乐鑫官方文档和 ESP-BSP 文档编写，信息截至 2026 年 6 月。如需获取最新信息，请参考[官方文档](https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2.html)。
