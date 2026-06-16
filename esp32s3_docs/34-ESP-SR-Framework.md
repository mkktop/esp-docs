# ESP-SR 语音识别组件库详解

> 文档编号：34
> 适用芯片：ESP32 / ESP32-S3 / ESP32-P4 / ESP32-C3 / ESP32-C5 / ESP32-C6 / ESP32-S31 等
> 适用场景：离线语音唤醒、命令词识别、声学前端、声源定位、语音合成、智能音箱/中控
> 来源：乐鑫官方 ESP-SR 文档、GitHub 仓库、Component Registry、ESP-Skainet 框架
> 关联文档：本文是 [21-ESP-Skainet-Framework](21-ESP-Skainet-Framework.md) 中 ESP-SR 组件的独立深入版

---

## 目录

1. [ESP-SR 概述](#1-esp-sr-概述)
2. [组件结构与模块](#2-组件结构与模块)
3. [声学前端 AFE](#3-声学前端-afe)
4. [唤醒词引擎 WakeNet](#4-唤醒词引擎-wakenet)
5. [命令词识别 MultiNet](#5-命令词识别-multinet)
6. [VADNet 语音活动检测](#6-vadnet-语音活动检测)
7. [语音合成 Speech Synthesis](#7-语音合成-speech-synthesis)
8. [ESP32-S3 上的优化与性能](#8-esp32-s3-上的优化与性能)
9. [如何集成 ESP-SR](#9-如何集成-esp-sr)
10. [与 ESP-Skainet 的关系](#10-与-esp-skainet-的关系)
11. [参考资源](#11-参考资源)

---

## 1. ESP-SR 概述

### 1.1 什么是 ESP-SR

ESP-SR（Espressif Speech Recognition）是乐鑫自研的**语音识别核心算法组件库**，帮助开发者基于 ESP32 / ESP32-S3 等芯片构建 AI 语音解决方案。它提供唤醒词检测、命令词识别、声学前端增强、语音合成等完整能力。

ESP-SR 既可作为独立组件被任意 ESP-IDF 工程集成（通过 Component Registry），也已被 ESP-Skainet 框架内置集成。

### 1.2 核心能力一览

| 模块 | 能力 |
|------|------|
| **AFE（声学前端）** | AEC 回声消除、NS 降噪、BSS 盲源分离、VAD 语音检测、DOA 声源定位 |
| **WakeNet（唤醒词）** | 低功耗持续监听唤醒词，最多同时 5 个 |
| **MultiNet（命令词）** | 离线命令词识别，最多 200 个中英文命令，无需重训练即可自定义 |
| **VADNet** | 基于神经网络的语音活动检测，可替代传统 WebRTC VAD |
| **Speech Synthesis（TTS）** | 中文语音合成 |

---

## 2. 组件结构与模块

```
┌─────────────────────────────────────────────────┐
│                ESP-SR 组件库                     │
├─────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   AFE    │  │ WakeNet  │  │ MultiNet │     │
│  │ 声学前端 │  │ 唤醒词   │  │ 命令词   │     │
│  └──────────┘  └──────────┘  └──────────┘     │
│  ┌──────────┐  ┌──────────┐                    │
│  │ VADNet   │  │ Speech   │                    │
│  │ 语音检测 │  │ Synthesis│                    │
│  └──────────┘  └──────────┘                    │
├─────────────────────────────────────────────────┤
│          Model Storage（模型存储）               │
│      Flash Partition / PSRAM                    │
├─────────────────────────────────────────────────┤
│          ESP-IDF / FreeRTOS                     │
├─────────────────────────────────────────────────┤
│    ESP32 / ESP32-S3 / ESP32-P4 / ESP32-C3 ...  │
└─────────────────────────────────────────────────┘
```

### 输入音频要求

- 采样率：**16 kHz**
- 声道：**单声道**
- 编码：**signed 16-bit**
- 来源：I2S/ADC 数字麦克风，或 flash/SD 卡中的 wav/pcm 文件

---

## 3. 声学前端 AFE

### 3.1 集成的算法

| 算法 | 说明 |
|------|------|
| **AEC**（Acoustic Echo Cancellation） | 回声消除，消除扬声器播放声音的回声干扰（支持全双工） |
| **VAD**（Voice Activity Detection） | 语音活动检测 |
| **BSS**（Blind Source Separation） | 盲源分离，双麦克风分离目标声源与噪声 |
| **NS**（Noise Suppression） | 单麦克风场景降噪 |
| **DOA**（Direction of Arrival） | 声源定位，判断语音来源方向 |

### 3.2 认证

乐鑫双麦克风 AFE 已通过 **Amazon Alexa Built-in 设备的软件音频前端解决方案**认证，达到商用级语音增强质量。

> AFE 详细工作流程、Amazon Alexa 认证、DOA 等已在 [21-ESP-Skainet-Framework](21-ESP-Skainet-Framework.md) 第 3 节展开，此处不再重复。

---

## 4. 唤醒词引擎 WakeNet

### 4.1 技术特点

- 基于 Dilated Convolution（空洞卷积）神经网络
- WakeNet9 模型不到 300K 参数
- 32ms 帧平均运行时间 3ms
- 最多同时支持 **5 个**唤醒词

### 4.2 支持的唤醒词

| 唤醒词 | 类型 |
|--------|------|
| Hi, Lexin（Hi 乐鑫） | 免费官方 |
| Hi, ESP | 免费官方 |
| 你好小智 / 你好小鑫 | 免费官方 |
| Alexa / 天猫精灵 / 小爱同学 | 支持 |
| 自定义唤醒词 | 乐鑫定制服务 / TTS 样本训练 |

### 4.3 版本与适用芯片

| 版本 | 适用 |
|------|------|
| WakeNet 9 / 9l | ESP32、ESP32-S3、ESP32-P4（9l 为高速响应优化） |
| WakeNet 9s | ESP32-C3、ESP32-C5、ESP32-C6（无 PSRAM/SIMD 芯片） |

---

## 5. 命令词识别 MultiNet

### 5.1 核心优势

| 特性 | 说明 |
|------|------|
| 完全离线 | 本地运行，无需网络，保护隐私 |
| 无需重训练自定义 | 用户可随时添加命令词，**无需重新训练模型** |
| 大容量 | 最多 **200 个**中英文命令 |
| 多语言 | 支持中文 + 英文 |
| 同义映射 | 多种说法可映射到同一命令 ID |

### 5.2 版本

| 版本 | 特点 |
|------|------|
| MultiNet 6 | 基于 Conformer 结构 |
| MultiNet 7 | 基于 Zipformer 结构，参数更少、准确率更高（语音噪声场景 +2.6%） |

---

## 6. VADNet 语音活动检测

VADNet 是乐鑫 2025 年 2 月发布的基于神经网络的语音活动检测模型，可替代传统 WebRTC VAD，提供更高检测准确率。可作为 ESP-SR 框架的一部分独立使用或集成进 AFE 流程。

---

## 7. 语音合成 Speech Synthesis

ESP-SR 提供中文 TTS（Text-To-Speech）功能，将识别到的命令词语音播报出来。英文 TTS 开发中。

**TTS Pipeline V3 更新**：支持中文、英文、日语、法语唤醒词训练，计划支持韩语、西班牙语、葡萄牙语、德语、俄语、阿拉伯语。

---

## 8. ESP32-S3 上的优化与性能

ESP32-S3 是 ESP-SR 的**最佳目标平台**之一：

| 特性 | 对语音识别的意义 |
|------|------------------|
| 向量指令（SIMD） | 加速神经网络矩阵乘法、卷积、FFT、量化运算 |
| 双核 Xtensa LX7 240MHz | 强大并行处理 |
| 大容量 Octal PSRAM | 容纳神经网络模型 |
| 高带宽 Octal Flash | 快速加载模型 |

### 模型存储方案

| 方案 | 说明 |
|------|------|
| Flash Partition | 模型存 flash 分区，启动加载 |
| PSRAM | 模型加载到 PSRAM 运行，更快 |

---

## 9. 如何集成 ESP-SR

### 9.1 作为独立组件（推荐）

ESP-SR 已发布到 ESP Component Registry，可在任意 ESP-IDF 工程中直接添加：

```bash
idf.py add-dependency "espressif/esp-sr"
```

### 9.2 在 ESP-Skainet 中使用

ESP-Skainet 已将 ESP-SR 作为内置组件集成，克隆 ESP-Skainet 即可：

```bash
git clone --recursive https://github.com/espressif/esp-skainet.git
```

### 9.3 menuconfig 配置要点

```
Audio Media HAL → Audio hardware board → 选择开发板（如 ESP32-S3-Korvo-1）
ESP Speech Recognition → Select wake words → 选择唤醒词（如 wn9_hilexin）
ESP Speech Recognition → Select speech commands model → 选择 MultiNet 语言
ESP Speech Recognition → Load Multiple Wake Words → 多唤醒词（最多 5 个）
```

### 9.4 核心代码骨架

```c
#include "esp_afe_sr_models.h"
#include "esp_wn_iface.h"
#include "esp_mn_iface.h"

// 1. AFE 配置 + WakeNet
afe_config_t afe_config = AFE_CONFIG_DEFAULT();
afe_config.wakenet_init = true;
afe_config.wakenet_model_name = "wn9_hilexin";
esp_afe_sr_iface_t *afe_handle = esp_afe_sr_iface_init();

// 2. MultiNet 命令词
esp_mn_commands_add(1, "turn on the light");
esp_mn_commands_add(2, "turn off the light");
esp_mn_commands_update();

// 3. 处理流
afe_fetch_result_t *res = afe_handle->fetch(afe_data);
if (res->wakeup_state == WAKEUP_DETECTED)   { /* 唤醒 */ }
if (res->wakeup_state == WAKEUP_CMD) {
    int cmd = multinet->get_results(res->data);  /* 命令 ID */
}
```

> 完整代码示例与 MQTT 结合的语音控制物联网设备实现，详见 [21-ESP-Skainet-Framework](21-ESP-Skainet-Framework.md) 第 10–12 节。

---

## 10. 与 ESP-Skainet 的关系

| 项目 | 定位 |
|------|------|
| **ESP-SR** | 语音识别**核心算法组件库**（AFE / WakeNet / MultiNet / VADNet / TTS），可被任意工程集成 |
| **ESP-Skainet** | 语音助手**应用框架**，内置 ESP-SR 作为算法核心，提供开箱即用的唤醒/命令识别应用与示例 |

简言之：**ESP-Skainet 用 ESP-SR**。如果只是要算法能力并自己搭框架，用 ESP-SR；如果想要现成的语音助手方案，用 ESP-Skainet。

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-SR GitHub | https://github.com/espressif/esp-sr |
| ESP-SR 组件注册表 | https://components.espressif.com/components/espressif/esp-sr |
| ESP-SR 文档（英文，ESP32-S3） | https://docs.espressif.com/projects/esp-sr/en/latest/esp32s3/getting_started/readme.html |
| ESP-SR 文档（中文，ESP32-S3） | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/getting_started/readme.html |
| AFE 声学前端文档 | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/README.html |
| WakeNet 唤醒词文档 | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html |
| MultiNet 命令词文档 | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_command_recognition/README.html |
| 语音合成文档 | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_synthesis/readme.html |
| 唤醒词定制流程 | https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32/wake_word_engine/ESP_Wake_Words_Customization.html |
| ESP-Skainet 框架 | https://github.com/espressif/esp-skainet |

---

> **文档总结**：ESP-SR 是乐鑫自研的语音识别核心算法组件库，包含 AFE（声学前端：AEC/NS/BSS/VAD/DOA）、WakeNet（唤醒词，≤5 个同检，9 系列针对不同芯片）、MultiNet（命令词，≤200 个中英文，无需重训练自定义）、VADNet（神经网络 VAD）、Speech Synthesis（中文 TTS）五大模块。它既可经 Component Registry 独立集成到任意 ESP-IDF 工程，也作为 ESP-Skainet 框架的算法内核内置。ESP32-S3 凭借向量指令加速、大容量 PSRAM 与双核算力，是 ESP-SR 的最佳目标平台，配合 Korvo-1/2、BOX-3、VoCat 等开发板可快速落地离线语音产品。
