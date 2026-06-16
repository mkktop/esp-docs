# ESP32-S3-Korvo-1 用户指南

> **文档版本**: v5.0 (当前版本), 兼容 v4.0
> **状态**: 量产 (Mass Production)
> **原厂文档**: [ESP-Skainet Korvo-1 User Guide](https://github.com/espressif/esp-skainet/blob/master/docs/en/hw-reference/esp32s3/user-guide-korvo-1.md)
> **中文文档**: [ESP32-S3-Korvo-1 中文用户指南](https://github.com/espressif/esp-skainet/blob/master/docs/zh_CN/hw-reference/esp32s3/user-guide-korvo-1.md)

---

## 目录

- [1. 概述](#1-概述)
- [2. 功能框图](#2-功能框图)
- [3. 主板组件](#3-主板组件)
- [4. 子板组件 (ESP32-Korvo-Mic)](#4-子板组件-esp32-korvo-mic)
- [5. 音频系统组件](#5-音频系统组件)
  - [5.1 麦克风阵列](#51-麦克风阵列)
  - [5.2 音频编解码器 ES8311](#52-音频编解码器-es8311)
  - [5.3 音频 ADC ES7210](#53-音频-adc-es7210)
  - [5.4 扬声器与功放](#54-扬声器与功放)
- [6. ESP-Skainet 语音识别 SDK](#6-esp-skainet-语音识别-sdk)
  - [6.1 概述](#61-概述)
  - [6.2 声学前端 (AFE)](#62-声学前端-afe)
  - [6.3 唤醒词检测 WakeNet](#63-唤醒词检测-wakenet)
  - [6.4 语音命令词识别 MultiNet](#64-语音命令词识别-multinet)
  - [6.5 语音合成 (TTS)](#65-语音合成-tts)
- [7. 默认固件和功能测试](#7-默认固件和功能测试)
- [8. 开始应用开发](#8-开始应用开发)
  - [8.1 必备硬件](#81-必备硬件)
  - [8.2 可选硬件](#82-可选硬件)
  - [8.3 电源选项](#83-电源选项)
  - [8.4 硬件设置](#84-硬件设置)
  - [8.5 软件设置](#85-软件设置)
- [9. 硬件参考](#9-硬件参考)
  - [9.1 GPIO 分配](#91-gpio-分配)
  - [9.2 电路分布](#92-电路分布)
  - [9.3 选择音频输出方式](#93-选择音频输出方式)
- [10. 硬件版本详情](#10-硬件版本详情)
- [11. 相关文档](#11-相关文档)

---

## 1. 概述

ESP32-S3-Korvo-1 是乐鑫 (Espressif) 推出的一款 **AI 人工智能开发板**，搭载 [ESP32-S3](https://www.espressif.com/zh-hans/products/socs/esp32-s3) SoC 芯片和乐鑫语音识别 SDK [ESP-Skainet](https://www.espressif.com/zh-hans/solutions/audio-solutions/esp-skainet/overview)。

### 核心特性

| 特性 | 说明 |
|------|------|
| **核心芯片** | ESP32-S3R8 (叠封 8 MB PSRAM) |
| **模组** | ESP32-S3-WROOM-1，16 MB Flash + 8 MB PSRAM |
| **麦克风阵列** | 三麦克风模拟阵列，间距 65 mm，适用于远场拾音 |
| **语音唤醒** | 支持中英文唤醒词检测 (WakeNet) |
| **语音命令识别** | 离线语音命令词识别 (MultiNet)，支持中英文 |
| **AI 加速** | 专门的向量指令用于加速神经网络计算和信号处理 |
| **工作温度** | -40 C ~ 85 C |
| **板尺寸** | 直径 88 mm (圆形主板设计) |
| **接口** | I/O, USB |
| **外设** | 按键, TF 卡, LED, 麦克风 |
| **参考价格** | 约 329 元人民币 |

### 应用场景

借助 ESP-Skainet，开发者可基于 ESP32-S3-Korvo-1 开发各种语音识别应用：

- 智能显示器 (Smart Displays)
- 智能插头 (Smart Plugs)
- 智能开关 (Smart Switches)
- 智能家电
- 智能音箱
- 智能照明控制

### 板卡架构

ESP32-S3-Korvo-1 开发板由 **两部分** 组成：

1. **主板 (ESP32-S3-Korvo-1)**：集成 ESP32-S3-WROOM-1 模组、功能按键、SD 卡槽、扬声器接口和 USB 接口等
2. **子板 (ESP32-Korvo-Mic)**：配置三麦克风阵列、功能按键、可寻址 LED 等

主板和子板通过 **FPC 排线** 连接。子板同时也是 [ESP32-Korvo v1.1](https://github.com/espressif/esp-skainet/blob/master/docs/en/hw-reference/esp32/user-guide-esp32-korvo-v1.1.md) 的子板。

---

## 2. 功能框图

功能框图显示了主板 ESP32-S3-Korvo-1（左侧）和子板 ESP32-Korvo-Mic（右侧）的主要组件，以及组件之间的连接方式。

```
┌─────────────────────────────────────────────────────────────┐
│                    ESP32-S3-Korvo-1 主板                      │
│                                                             │
│  ┌─────────────┐     ┌──────────────┐     ┌──────────────┐ │
│  │ ESP32-S3    │ I2S │  Audio Codec │     │   Speaker    │ │
│  │ WROOM-1     │◄───►│    ES8311    │────►│   Output     │ │
│  │ 16MB Flash  │ I2C │              │     └──────────────┘ │
│  │  8MB PSRAM  │◄───►│              │     ┌──────────────┐ │
│  │             │     └──────────────┘     │  Headphone   │ │
│  │             │                          │   Output     │ │
│  │             │     ┌──────────────┐     └──────────────┘ │
│  │             │ I2S │  Audio ADC   │                       │
│  │             │◄───►│   ES7210     │                       │
│  │             │ I2C │  (4-channel) │                       │
│  │             │◄───►│              │                       │
│  │             │     └──────┬───────┘                       │
│  │             │            │ FPC                           │
│  │             │     ┌──────┴───────┐                       │
│  │             │     │ FPC Connector│                       │
│  └─────────────┘     └──────┬───────┘                       │
│                             │                               │
│  ┌──────────┐  ┌──────────┐ │ ┌──────────┐  ┌───────────┐ │
│  │ USB Power│  │USB-UART  │ │ │MicroSD   │  │ Battery   │ │
│  │  Port    │  │  Port    │ │ │Card Slot │  │  Socket   │ │
│  └──────────┘  └──────────┘ │ └──────────┘  └───────────┘ │
│                             │                               │
└─────────────────────────────┼───────────────────────────────┘
                              │ FPC 排线
┌─────────────────────────────┼───────────────────────────────┐
│              ESP32-Korvo-Mic 子板                            │
│                             │                               │
│  ┌──────────┐  ┌──────────┐ │ ┌──────────┐                 │
│  │  Mic 1   │  │  Mic 2   │ │ │  Mic 3   │                 │
│  │(Analog)  │  │(Analog)  │ │ │(Analog)  │                 │
│  └────┬─────┘  └────┬─────┘ │ └────┬─────┘                 │
│       │              │      │      │                        │
│       └──────────────┴──────┴──────┘                        │
│                      (三麦克风阵列)                           │
│                                                             │
│  ┌──────────────────────────────────────────┐               │
│  │     12 x RGB LED (WS2812C)              │               │
│  └──────────────────────────────────────────┘               │
│                                                             │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                │
│  │ BT1│ │ BT2│ │ BT3│ │ BT4│ │ BT5│ │ BT6│ (功能按键)      │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘                │
│                                                             │
│  ┌──────────────────────────────────────────┐               │
│  │       FPC Connector (连接主板)            │               │
│  └──────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 主板组件

以下从 ESP32-S3-WROOM-1 模组开始逆时针依次介绍主板上的主要组件：

| 序号 | 主要组件 | 介绍 |
|------|----------|------|
| 1 | **ESP32-S3-WROOM-1** | ESP32-S3-WROOM-1 模组，内置 ESP32-S3R8 芯片，集成 Wi-Fi 和 Bluetooth 5 (LE) 子系统，还有专门的向量指令用于加速神经网络计算和信号处理。ESP32-S3R8 芯片叠封 8 MB PSRAM，模组还另外带有 16 MB Flash，可灵活高效读取数据。 |
| 2 | **PDM Interface (预留)** | 预留 PDM 接口，子板可选择连接使用 3 个脉冲密度调制 (PDM) 数字麦阵列。 |
| 3 | **5V to 3.3V LDO** | 模组电路的电源转换器，输入 5V，输出 3.3V。 |
| 4 | **5V Power On LED** | 开发板连接 USB 电源，并且电源开关拨至 "ON" 时，5V 电源指示灯红灯亮起。 |
| 5 | **Power Switch** | 拨动至 "ON"，开发板上电；拨动至 "OFF"，开发板断电。 |
| 6 | **Battery Socket** | 外接锂电池可作为 USB 供电的替代电源。建议电池规格：容量 >1000 mAh，输出电压 3.7V，输入电压 4.2V-5V。 |
| 7 | **Battery Charger Chip** | AP5056，1A 线性锂电池充电器。充电电源来自 USB 供电接口。 |
| 8 | **Battery Green LED** | 开发板连接 USB 电源且不接电池时绿灯亮。接电池状态下，充电完成后绿灯亮。 |
| 9 | **Battery Red LED** | 开发板连接 USB 电源且不接电池时红灯闪烁。接电池充电中红灯亮，充电完成后红灯灭。 |
| 10 | **USB Power Port** | 开发板的供电接口 (Micro-USB)。 |
| 11 | **USB-to-UART Bridge** | 单芯片 USB 至 UART 桥接器，可提供高达 3 Mbps 的传输速率。 |
| 12 | **FPC Connector** | 通过 FPC 排线连接主板和子板。 |
| 13 | **USB-to-UART Port** | 作为通信接口，通过板载 USB 至 UART 桥接器与芯片通信 (Micro-USB)。 |
| 14 | **Reset Button** | 芯片复位按键。 |
| 15 | **Boot Button** | 下载按键。按住 Boot 键的同时按一下 Reset 键进入"固件下载"模式。 |
| 16 | **MicroSD Card Slot** | 可插入 MicroSD 卡，支持 SPI 模式，用于扩充数据存储空间。 |
| 17 | **Audio_VCC33** | 音频电路的电源转换器，输入 5V，输出 3.3V。 |
| 18 | **Speaker Output** | 音频输出插槽，外接扬声器。采用 2.00 mm 排针间距，建议使用 4 欧姆 3 瓦特扬声器。 |
| 19 | **PA Chip** | 超低 EMI，无需滤波器，3 瓦特单声道 D 类音频功放。 |
| 20 | **ADC Chip** | 音频 4 通道模拟转数字芯片 (ES7210)。 |
| 21 | **Headphone Output** | 音频输出插槽，外接 3.5 mm 立体声耳机 (输出单声道信号)。 |
| 22 | **Codec_3V3 (NC)** | Codec 的额外电源选项，默认不使用、不上件。 |
| 23 | **Audio Codec Chip** | 音频编解码芯片 ES8311，低功耗单声道音频编解码器。 |

---

## 4. 子板组件 (ESP32-Korvo-Mic)

子板 ESP32-Korvo-Mic 正面和背面包含以下主要组件：

| 主要组件 | 介绍 |
|----------|------|
| **Analog Microphone (模拟麦克风)** | 与另外两个模拟麦克风组成三个模拟麦克风阵列，**间隔 65 mm**。硬件也支持两个模拟麦克风阵列，间隔 55 mm（默认不上件，软件暂不支持）。 |
| **RGB LED** | **12 个可寻址 RGB LED (WS2812C)**，可用于状态指示和视觉效果。 |
| **Function Button (功能按键)** | 板子上有 **6 个功能按键**，用户可自定义功能。 |
| **Digital Microphone (数字麦克风)** | 硬件支持三个数字麦克风阵列，间隔 65 mm（默认不上件，软件暂不支持）。 |
| **FPC Connector** | 通过 FPC 排线连接主板和子板。 |

---

## 5. 音频系统组件

### 5.1 麦克风阵列

ESP32-S3-Korvo-1 的核心音频输入组件是 **三麦克风模拟阵列**：

- **阵列配置**: 3 个模拟麦克风
- **麦克风间距**: 65 mm
- **ADC 接口**: 通过 ES7210 四通道 ADC 采集
- **通道分配**: 3 通道用于麦克风，1 通道预留 AEC 参考

**麦克风阵列优势**：

| 特性 | 说明 |
|------|------|
| **远场拾音** | 三麦克风阵列可实现远场语音采集，有效拾音距离可达数米 |
| **波束成形** | 配合 BSS 算法，实现声源定位和方向性拾音 |
| **噪声抑制** | 多麦克风协同工作，提升噪声环境下的语音质量 |
| **回声消除** | 配合 AEC 算法，实现设备自身播放音频的回声消除 |

### 5.2 音频编解码器 ES8311

[ES8311](http://www.everest-semi.com/pdf/ES8311%20PB.pdf) 是一款低功耗单声道音频编解码器，主要特性：

| 特性 | 说明 |
|------|------|
| **ADC** | 单通道，高性能多比特 Delta-Sigma ADC |
| **DAC** | 单通道，高性能多比特 Delta-Sigma DAC |
| **前置放大器** | 低噪声前置放大器 |
| **耳机驱动** | 内置耳机驱动器 |
| **数字音效** | 内置数字音效处理器 |
| **模拟混音** | 支持模拟混音功能 |
| **增益控制** | 内置增益函数 |
| **通信接口** | 通过 I2S 和 I2C 总线与 ESP32-S3 通信 |

**数据流方向**：
- **播放路径**: ESP32-S3 → I2S → ES8311 DAC → PA (功放) → 扬声器
- **录音路径**: 麦克风 (单路) → ES8311 ADC → I2S → ESP32-S3

### 5.3 音频 ADC ES7210

[ES7210](http://www.everest-semi.com/pdf/ES7210%20PB.pdf) 是一款高性能 4 通道音频 ADC，专为麦克风阵列应用设计：

| 特性 | 说明 |
|------|------|
| **通道数** | 4 通道 (3 路麦克风 + 1 路 AEC 参考) |
| **信噪比 (SNR)** | 102 dB |
| **THD+N** | -85 dB |
| **位深** | 24 位 |
| **采样频率** | 8 ~ 100 kHz |
| **数据端口** | I2S/PCM 主从串行数据端口，支持 TDM 模式 |
| **通信接口** | 通过 I2S 和 I2C 与 ESP32-S3 通信 |

### 5.4 扬声器与功放

| 组件 | 说明 |
|------|------|
| **功放芯片 (PA)** | 超低 EMI，无需滤波器，3 瓦特单声道 D 类音频功放 |
| **扬声器输出** | 2.00 mm / 0.08 英寸排针间距 |
| **推荐扬声器** | 4 欧姆，3 瓦特 |
| **耳机输出** | 3.5 mm 立体声插孔 (单声道输出) |

---

## 6. ESP-Skainet 语音识别 SDK

### 6.1 概述

[ESP-Skainet](https://github.com/espressif/esp-skainet) 是乐鑫针对语音控制设备推出的智能语音助手 SDK。它**不依赖云连接，可以完全实现离线运行**，在本地乐鑫 SoC 上即可进行唤醒词检测和语音命令词识别。

ESP-Skainet 的整体处理流水线：

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   麦克风      │───►│  声学前端     │───►│  唤醒词引擎   │───►│  命令词识别   │───►│  执行动作    │
│  音频输入     │    │   (AFE)      │    │  (WakeNet)   │    │  (MultiNet)  │    │  (用户处理)  │
└──────────────┘    │              │    │              │    │              │    └──────────────┘
                    │  - AEC       │    │ "Hi 乐鑫"    │    │ "打开空调"   │
                    │  - BSS / NS  │    │ "Hi ESP"    │    │ "Turn on..."│
                    │  - VAD       │    │              │    │              │
                    └──────────────┘    └──────────────┘    └──────────────┘
```

**核心组件**：
- **声学前端 (AFE)**: 集成 AEC、BSS、NS、VAD 等算法
- **唤醒词引擎 (WakeNet)**: 持续监听唤醒词
- **命令词识别 (MultiNet)**: 识别语音命令
- **语音合成 (TTS)**: 语音反馈 (仅中文)

### 6.2 声学前端 (AFE)

ESP-SR 的 [AFE (Audio Front-End)](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/README.html) 算法框架是构建语音用户界面 (VUI) 的关键组件。

#### AFE 算法组件

| 算法 | 全称 | 功能说明 |
|------|------|----------|
| **AEC** | Acoustic Echo Cancellation (声学回声消除) | 消除扬声器播放声音在麦克风中的回声，支持双麦处理。适用于设备播放音频时的语音识别 |
| **BSS** | Blind Source Separation (盲源分离) | 使用多麦克风检测声源方向，强化目标方向音频。在噪声环境中提高目标声源质量 |
| **NS** | Noise Suppression (噪声抑制) | 单通道处理，消除非人声噪声 (如吸尘器、空调声)，针对稳态噪声效果显著 |
| **MISO** | Multi Input Single Output (多输入单输出) | 双通道输入单通道输出，在双麦无唤醒场景下选择信噪比高的一路音频输出 |
| **VAD** | Voice Activity Detection (语音活动检测) | 实时输出当前帧的语音活动状态，过滤不包含有效语音的音频段 |
| **AGC** | Automatic Gain Control (自动增益控制) | 动态调整输出音频幅值，弱信号放大，强信号压缩 |
| **WakeNet** | 唤醒词引擎 | 基于神经网络的唤醒词模型，专为低功耗嵌入式 MCU 设计 |

#### AFE 性能优势

| 优势 | 说明 |
|------|------|
| **声学性能优越** | 已通过亚马逊 Alexa 远场测试。使用乐鑫自研唤醒词引擎，满足多语言测试要求 |
| **资源消耗低** | 基于 ESP32-S3 AI 加速器优化，仅消耗约 **22% CPU**、**48 KB SRAM** 和 **1.1 MB PSRAM** |
| **产品设计灵活** | 支持简单直观的 API，麦克风间距 **20-80 mm** 可变，硬件设计灵活 |

#### AFE 工作流程

```
输入音频 (多通道)
      │
      ▼
┌──────────────┐
│     AEC      │ ─── 声学回声消除
│ (回声消除)    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  BSS 或 NS   │ ─── 双麦用 BSS，单麦用 NS
│ (降噪/分离)   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│     VAD      │ ─── 语音活动检测
│ (语音检测)    │
└──────┬───────┘
       │
       ▼
  增强后的音频 → WakeNet 唤醒词检测
```

#### AEC 工作模式

AEC 提供多种模式以适应不同应用场景：

| 应用场景 | 模式 | 说明 |
|----------|------|------|
| 语音识别 (SR) | `AEC_MODE_SR_LOW_COST`, `AEC_MODE_SR_HIGH_PERF` | 适用于唤醒词识别场景，资源消耗低 |
| 全双工对话 (FD) | `AEC_MODE_FD_LOW_COST`, `AEC_MODE_FD_HIGH_PERF` | 线性滤波 + 非线性处理，适用于人机对话 |
| 语音通话 (VOIP) | `AEC_MODE_VOIP_LOW_COST`, `AEC_MODE_VOIP_HIGH_PERF` | 支持 8/16 kHz，建议使用 FD 模式替代 |

#### AFE 配置示例

```c
#include "esp_afe_sr_models.h"
#include "model_path.h"

// 步骤1: 初始化模型
srmodel_list_t *models = esp_srmodel_init("model");

// 步骤2: 初始化 AFE 配置
// input_format "MMNR": 麦克风1, 麦克风2, 未使用, 参考信号
afe_config_t *afe_config = afe_config_init("MMNR", models, AFE_TYPE_SR, AFE_MODE_HIGH_PERF);

// 步骤3: 创建 AFE 实例
esp_afe_sr_iface_t *afe_handle = esp_afe_handle_from_config(afe_config);
esp_afe_sr_data_t *afe_data = afe_handle->create_from_config(afe_config);

// 步骤4: 输入音频数据
int feed_chunksize = afe_handle->get_feed_chunksize(afe_data);
int feed_nch = afe_handle->get_feed_channel_num(afe_data);
int16_t *feed_buff = (int16_t *)malloc(feed_chunksize * feed_nch * sizeof(int16_t));
afe_handle->feed(afe_data, feed_buff);

// 步骤5: 获取处理结果
afe_fetch_result_t *result = afe_handle->fetch(afe_data);
int16_t *processed_audio = result->data;
vad_state_t vad_state = result->vad_state;
wakenet_state_t wakeup_state = result->wakeup_state;
```

### 6.3 唤醒词检测 WakeNet

[WakeNet](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html) 是乐鑫自研的高性能、低资源消耗的唤醒词检测算法。

#### 支持的唤醒词

| 唤醒词 | 模型名称 | 语言 |
|--------|----------|------|
| Hi 乐鑫 | wn9_hilexin | 中文 |
| Hi ESP | wn9_hiesp | 英文 |
| 你好小智 | nihaoxiaozhi | 中文 |
| 你好小鑫 | nihaoxiaoxin | 中文 |
| 小爱同学 | xiaoaitongxue | 中文 |
| Alexa | Alexa | 英文 |
| 自定义唤醒词 | - | 多语言 |

> 乐鑫免费开放 "Hi 乐鑫"、"Hi ESP" 等唤醒词，同时提供自定义唤醒词定制服务。

#### WakeNet9 技术参数

| 参数 | 数值 |
|------|------|
| **模型结构** | Dilated Convolution (空洞卷积) |
| **参数量** | < 300K |
| **每帧推理时间** | 约 3 ms (32 ms 帧) |
| **音频格式** | 16 kHz, 单声道, signed 16-bit |
| **特征提取** | MFCC (Mel 频率倒谱系数) |
| **同时支持唤醒词数** | 最多 5 个 |

#### WakeNet9 检测准确率

| 距离 | 环境 | 准确率 |
|------|------|--------|
| 1 米 | 安静环境 | **98%** |
| 1 米 | 稳态噪声 (SNR 4 dB) | 96% |
| 1 米 | 语音噪声 (SNR 4 dB) | 94% |
| 3 米 | 安静环境 | 不受影响 |

> 测试条件: 误唤醒率 1 次/12 小时

#### WakeNet 使用示例

```bash
# 配置开发板和唤醒词
idf.py set-target esp32s3
idf.py menuconfig

# 在 menuconfig 中选择:
# Audio Media HAL -> Audio hardware board -> ESP32-S3-Korvo-1
# ESP Speech Recognition -> Select wake words -> Hi,Lexin (wn9_hilexin)

# 编译并烧写
idf.py flash monitor
```

加载多个唤醒词模型 (最多 2 个同时运行)：

```bash
# menuconfig 中:
# ESP Speech Recognition -> Select wake words -> Hi,Lexin (wn9_hilexin) -> Load Multiple Wake Words
# ESP Speech Recognition -> Load Multiple Wake Words -> Hi,Lexin (wn9_hilexin)
#                                                    -> Hi,ESP (wn9_hiesp)
```

修改检测阈值：

```c
// 设置唤醒词检测阈值 (esp-sr v2.1.3+)
afe_handle->set_wakenet_threshold(afe_handle, model_index, threshold); // model_index: 1 或 2
afe_handle->reset_wakenet_threshold(afe_handle, model_index);          // 重置为默认阈值
```

### 6.4 语音命令词识别 MultiNet

[MultiNet](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_command_recognition/README.html) 命令词识别模型专为提供灵活的离线语音命令词识别而设计。

#### MultiNet 特性

| 特性 | 说明 |
|------|------|
| **最大命令数** | 支持 **200 个**中英文命令词 |
| **无需重训练** | 用户可根据需求自定义语音命令，无需重新训练模型 |
| **语言支持** | 中文和英文 (同一时间只能激活一个语言模型) |
| **工作流程** | 唤醒词触发后激活，持续监听命令 |
| **超时机制** | 无语音输入超时后自动关闭，返回唤醒监听模式 (默认 5-6 秒) |
| **多候选输出** | 识别结果最多返回 5 个候选，按概率排序 |

#### MultiNet 工作流程

```
WakeNet 检测到唤醒词
        │
        ▼
WakeNet 进入待机模式
        │
        ▼
MultiNet 激活，开始监听
        │
        ├──► "打开空调"    → Command ID: 1
        ├──► "关闭电灯"    → Command ID: 2
        ├──► "Turn on..." → Command ID: 3
        │
        ▼
   静默超时
        │
        ▼
MultiNet 关闭，WakeNet 重新激活
```

#### MultiNet 识别状态

| 状态 | 说明 |
|------|------|
| `ESP_MN_STATE_DETECTING` | 正在识别中，还未识别到目标命令词 |
| `ESP_MN_STATE_DETECTED` | 识别到目标命令词，可调用 `get_results` 获取结果 |
| `ESP_MN_STATE_TIMEOUT` | 长时间未检测到命令词，自动退出，等待下次唤醒 |

#### MultiNet 自定义命令

```c
// 添加新命令
esp_mn_commands_add(cmd_id_50, "TURN ON THE KITCHEN LIGHT");
esp_mn_commands_add(cmd_id_51, "SHUFFLE MY PLAYLIST");

// 修改现有命令
esp_mn_commands_modify(cmd_id_3, "TURN ON MY SPEAKER");

// 应用更改
esp_mn_commands_update();
```

#### 识别结果数据结构

```c
typedef struct {
    esp_mn_state_t state;
    int num;                    // 识别到的词条数目, num <= 5
    int phrase_id[ESP_MN_RESULT_MAX_NUM];  // 词条对应的 Phrase ID 列表
    float prob[ESP_MN_RESULT_MAX_NUM];     // 识别概率，从大到小排列
} esp_mn_results_t;
```

### 6.5 语音合成 (TTS)

ESP-SR 还提供语音合成 (Text-to-Speech) 功能：

- **当前支持**: 仅支持中文 TTS 模型
- **英文 TTS**: 开发中
- **用途**: 可将识别到的命令词通过语音播报反馈给用户

---

## 7. 默认固件和功能测试

每块 ESP32-S3-Korvo-1 开发板出厂时都预装了 [默认固件](https://github.com/espressif/esp-skainet/blob/master/tools/default_firmware/default_firmware_ESP32-S3-Korvo-1)，支持语音唤醒和语音命令识别功能测试。

> **注意**: 默认固件仅支持中文唤醒词和中文语音命令。如需使用英文，请参考 [ESP-Skainet 示例](https://github.com/espressif/esp-skainet/tree/master/examples) 进行配置。

### 测试所需硬件

- 1 x ESP32-S3-Korvo-1 开发板
- 1 x USB 2.0 数据线 (Standard-A to Micro-B)

### 测试步骤

1. 通过 **USB 供电接口** 连接电源。**电池指示绿灯** 应亮起。未接电池时，**电池指示红灯** 闪烁。
2. 将 **电源开关** 拨至 **ON**。红色 **5V 电源指示灯** 亮起。
3. 按下主板上的 **复位按键**。
4. 使用默认中文唤醒词 **"Hi 乐鑫"** 激活开发板。检测到唤醒词后，子板上的 **12 个 RGB LED** 逐一亮起白色，表示开发板正在等待语音命令。
5. 说出命令来控制开发板。

### 默认中文语音命令

| 语音命令 | 含义 | 响应 |
|----------|------|------|
| 关闭电灯 | Turn off the light | RGB LED 熄灭 |
| 打开白色灯 | Turn on the white light | RGB LED 亮白色 |
| 打开红色灯 | Turn on the red light | RGB LED 亮红色 |
| 打开绿色灯 | Turn on the green light | RGB LED 亮绿色 |
| 打开蓝色灯 | Turn on the blue light | RGB LED 亮蓝色 |
| 打开黄色灯 | Turn on the yellow light | RGB LED 亮黄色 |
| 打开青色灯 | Turn on the cyan light | RGB LED 亮青色 |
| 打开紫色灯 | Turn on the purple light | RGB LED 亮紫色 |

> 如果命令未被识别，RGB LED 将恢复到语音唤醒前的状态。

---

## 8. 开始应用开发

### 8.1 必备硬件

- 1 x ESP32-S3-Korvo-1 开发板
- 1 x 扬声器 (4 欧姆 3 瓦特，推荐)
- 2 x USB 2.0 数据线 (Standard-A to Micro-B)
- 1 x 电脑 (Windows、Linux 或 macOS)

### 8.2 可选硬件

- 1 x MicroSD 卡
- 1 x 锂离子电池 (容量 >1000 mAh，内置保护电路)

### 8.3 电源选项

| 供电方式 | 说明 |
|----------|------|
| **USB 供电** | 通过 USB Power Port 供电 (5V) |
| **电池供电** | 通过 Battery Socket 连接锂电池 (3.7V) |
| **充电** | AP5056 充电芯片，充电电源来自 USB Power Port |

### 8.4 硬件设置

1. 将主板和子板通过 FPC 排线牢固连接
2. 连接扬声器至 **Speaker Output** 端口
3. 插入 USB 数据线，分别连接 PC 与开发板的两个 USB 端口
4. 打开 **电源开关**，确认红色电源指示灯亮起

### 8.5 软件设置

#### 环境准备

```bash
# 1. 克隆 ESP-Skainet 仓库 (包含 ESP-SR 组件)
git clone --recursive https://github.com/espressif/esp-skainet.git

# 2. 安装 ESP-IDF (使用 ESP-Skainet 推荐的版本)
# 参考 ESP-IDF 编程指南: https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/index.html

# 3. 设置目标芯片
idf.py set-target esp32s3
```

#### 编译和烧录示例

```bash
# 配置开发板
idf.py menuconfig
# Audio Media HAL -> Audio hardware board -> ESP32-S3-Korvo-1

# 编译和烧录
idf.py flash monitor

# 退出监控: Ctrl-]
```

---

## 9. 硬件参考

### 9.1 GPIO 分配

ESP32-S3-Korvo-1 的关键 GPIO 分配：

| 功能 | 说明 |
|------|------|
| **I2S 音频接口** | 用于 ESP32-S3 与 ES8311 / ES7210 之间的音频数据传输 |
| **I2C 控制接口** | 用于 ESP32-S3 对 ES8311 / ES7210 的寄存器配置 |
| **RGB LED** | WS2812C 控制 GPIO |
| **功能按键** | 6 个功能按键 GPIO |
| **MicroSD 卡** | SPI 模式 GPIO |
| **PA 使能** | 功放芯片使能控制 |

### 9.2 电路分布

#### 独立的 USB 供电与电池供电电路

开发板支持两种独立供电方式，USB 供电和电池供电通过电源管理电路进行切换。

#### 独立的模组与音频供电电路

模组电路和音频电路使用**独立的电源转换器**：
- 模组供电: 5V → 3.3V LDO
- 音频供电: 5V → 3.3V (Audio_VCC33)

这种独立设计可以有效降低音频电路对模组电路的电源噪声干扰。

### 9.3 选择音频输出方式

ESP32-S3-Korvo-1 提供两种音频输出方式：
1. **扬声器输出** - 通过 D 类功放芯片驱动外接扬声器 (4 欧姆 3 瓦特)
2. **耳机输出** - 3.5 mm 立体声耳机插孔 (输出单声道信号)

---

## 10. 硬件版本详情

### 当前版本: v5.0

| 版本 | 说明 |
|------|------|
| **v5.0** | 当前最新版本 |
| **v4.0** | 前一版本，使用方法与 v5.0 相同 |

> 如果您使用的是 ESP32-S3-Korvo-1 v4.0，请参考 v5.0 的用户指南。v5.0 相对 v4.0 的变化请参阅原厂文档的"硬件版本"章节。

### 内含组件和包装

#### 零售订单

每块 ESP32-S3-Korvo-1 开发板的零售包装包含：
- ESP32-S3-Korvo-1 主板
- ESP32-Korvo-Mic 子板
- FPC 排线
- 螺丝和紧固件

#### 批量订单

批量订单以大纸板箱包装。

---

## 11. 相关文档

### 技术规格书

- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf) (PDF)
- [ESP32-S3-WROOM-1 & ESP32-S3-WROOM-1U 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_cn.pdf) (PDF)

### 原理图

- [ESP32-S3-Korvo-1 v5.0 主板原理图](https://dl.espressif.com/dl/schematics/SCH_ESP32-S3-Korvo-1_V6_20211201.pdf) (PDF)
- [ESP32-S3-Korvo-1 v4.0 主板原理图](https://dl.espressif.com/dl/schematics/sch_esp32-s3-korvo_v5_20211020.pdf) (PDF)
- [ESP32-Korvo-Mic 子板原理图](https://dl.espressif.com/dl/schematics/SCH_ESP32-KORVO-MIC_V1_1_20200316A.pdf) (PDF)

### PCB 布局图

- [ESP32-S3-Korvo-1 v5.0 主板 PCB 布局图](https://dl.espressif.com/dl/schematics/PCB_ESP32-S3-Korvo-1_V5_20211201.pdf) (PDF)
- [ESP32-Korvo-Mic 子板 PCB 布局图](https://dl.espressif.com/dl/schematics/PCB_ESP32-Korvo-Mic_V1_1_20200316AA.pdf) (PDF)

### 尺寸图

- [ESP32-S3-Korvo-1 v5.0 主板尺寸图](https://dl.espressif.com/dl/schematics/DXF_ESP32-S3-Korvo-1_V5_mb_20211207.pdf) (PDF)
- [ESP32-Korvo-Mic 子板正面尺寸图](https://dl.espressif.com/dl/schematics/DXF_ESP32-S3-Korvo-Mic_top_V1_1_20211111.pdf) (PDF)
- [ESP32-Korvo-Mic 子板背面尺寸图](https://dl.espressif.com/dl/schematics/DXF_ESP32-S3-Korvo-Mic_Bottom_V1_1_20211111.pdf) (PDF)

### 软件资源

| 资源 | 链接 |
|------|------|
| ESP-Skainet SDK | [https://github.com/espressif/esp-skainet](https://github.com/espressif/esp-skainet) |
| ESP-SR 语音识别框架 | [https://github.com/espressif/esp-sr](https://github.com/espressif/esp-sr) |
| ESP-IDF 编程指南 | [https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/) |
| ESP-SR 文档 | [https://docs.espressif.com/projects/esp-sr/zh_CN/latest/](https://docs.espressif.com/projects/esp-sr/zh_CN/latest/) |
| BSP: ESP32-S3-KORVO-1 | [ESP32-S3-Korvo-1 BSP](https://components.espressif.com/components/espressif/esp32_s3_korvo_1) |

### 购买渠道

| 渠道 | 链接 |
|------|------|
| 淘宝 | [ESP32-S3-Korvo-1](https://item.taobao.com/item.htm?ft=t&id=661876689259) |
| Digi-Key | [ESP32-S3-KORVO-1](https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-KORVO-1/17887573) |
| Mouser | [ESP32-S3-Korvo-1](https://www.mouser.com/ProductDetail/Espressif-Systems/ESP32-S3-Korvo-1) |
| Amazon US | [ESP32-S3-Korvo-1](https://www.amazon.com/dp/B09MQCHFCL) |

---

## 附录: ESP32-S3-Korvo-1 vs ESP32-Korvo v1.1 对比

| 特性 | ESP32-S3-Korvo-1 | ESP32-Korvo v1.1 |
|------|-------------------|-------------------|
| **芯片** | ESP32-S3R8 | ESP32 (ESP32-WROVER-E) |
| **Flash** | 16 MB | 16 MB |
| **PSRAM** | 8 MB (叠封) | 8 MB |
| **AI 加速** | 向量指令 (SIMD) | 无专用 AI 加速 |
| **麦克风阵列** | 3 麦模拟阵列 | 3 麦模拟阵列 |
| **子板** | ESP32-Korvo-Mic (共用) | ESP32-Korvo-Mic (共用) |
| **唤醒词性能** | WakeNet9 (高性能) | WakeNet5 (基础) |
| **SDK** | ESP-Skainet + ESP-SR v2.0+ | ESP-Skainet + ESP-SR v1.* |

---

> **文档来源**: 本指南基于乐鑫官方文档编写，信息截至 2026 年 6 月。如需获取最新信息，请参考[官方文档](https://github.com/espressif/esp-skainet/blob/master/docs/zh_CN/hw-reference/esp32s3/user-guide-korvo-1.md)。
