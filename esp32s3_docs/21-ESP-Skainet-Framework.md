# ESP-Skainet 语音识别框架详解

> 文档编号：21  
> 适用芯片：ESP32 / ESP32-S2 / ESP32-S3 / ESP32-S31 / ESP32-P4 / ESP32-C3 / ESP32-C5 / ESP32-C6  
> 适用项目：离线语音唤醒、语音命令词识别、智能音箱、智能家居控制  
> 来源：乐鑫官方 ESP-Skainet / ESP-SR 文档、GitHub 仓库、DevCon23 演讲

---

## 目录

1. [ESP-Skainet 概述](#1-esp-skainet-概述)
2. [框架架构](#2-框架架构)
3. [声学前端（AFE）](#3-声学前端afe)
4. [唤醒词引擎（WakeNet）](#4-唤醒词引擎wakenet)
5. [语音命令词识别（MultiNet）](#5-语音命令词识别multinet)
6. [VAD 语音活动检测（VADNet）](#6-vad-语音活动检测vadnet)
7. [语音合成（Speech Synthesis）](#7-语音合成speech-synthesis)
8. [ESP32-S3 AI 加速能力](#8-esp32-s3-ai-加速能力)
9. [支持的开发板](#9-支持的开发板)
10. [如何使用 ESP-Skainet](#10-如何使用-esp-skainet)
11. [自定义命令词](#11-自定义命令词)
12. [与 MQTT 项目结合](#12-与-mqtt-项目结合)
13. [参考资源](#13-参考资源)

---

## 1. ESP-Skainet 概述

### 1.1 什么是 ESP-Skainet

ESP-Skainet 是乐鑫推出的智能语音助手开发框架，以最便捷的方式支持基于乐鑫 ESP32 系列芯片的唤醒词识别和语音命令词识别应用程序开发。使用 ESP-Skainet，开发者可以轻松构建唤醒词检测和离线语音命令识别应用。

ESP-Skainet 的核心语音算法由 **ESP-SR**（Espressif Speech Recognition）组件提供，ESP-Skainet 已将 ESP-SR 作为内置组件集成，无需单独下载。

### 1.2 核心功能

ESP-Skainet 提供以下核心功能：

| 功能 | 说明 |
|------|------|
| **唤醒词检测（WakeNet）** | 始终监听，检测预设的唤醒词（如"Hi 乐鑫"、"Hi ESP"、"Alexa"） |
| **语音命令词识别（MultiNet）** | 唤醒后识别离线语音命令，支持自定义命令 |
| **声学前端（AFE）** | 回声消除（AEC）、噪声抑制（NS）、盲源分离（BSS）、语音活动检测（VAD） |
| **VAD 语音活动检测（VADNet）** | 基于神经网络的语音活动检测模型 |
| **语音合成（TTS）** | 中文语音合成功能 |

### 1.3 完整的语音交互流程

```
┌──────────┐    ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  麦克风   │───>│  AFE    │───>│ WakeNet  │───>│ MultiNet │───>│ 执行命令  │
│  音频输入 │    │ 声学增强 │    │ 唤醒词检测│    │ 命令词识别│    │ 控制设备  │
└──────────┘    └─────────┘    └──────────┘    └──────────┘    └──────────┘
                                    │                │
                                    │   唤醒前待机    │   唤醒后激活
                                    v                v
                              持续监听          超时后回到待机
```

---

## 2. 框架架构

### 2.1 ESP-SR 组件结构

ESP-SR 是 ESP-Skainet 的核心算法组件，包含以下模块：

```
┌─────────────────────────────────────────────────┐
│                ESP-SR 框架                       │
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

### 2.2 输入音频

输入音频流可以来自：
- **麦克风**：通过 I2S/ADC 接口的数字麦克风
- **音频文件**：存储在 flash 或 SD 卡中的 wav/pcm 格式音频文件

音频要求：
- 采样率：16 kHz
- 声道：单声道
- 编码：signed 16-bit

---

## 3. 声学前端（AFE）

### 3.1 AFE 概述

声学前端（Audio Front-End, AFE）集成了多种音频增强算法，确保在各种环境下都能获得高质量的语音识别效果。

### 3.2 AFE 集成的算法

| 算法 | 说明 |
|------|------|
| **AEC（Acoustic Echo Cancellation）** | 回声消除，消除扬声器播放声音的回声干扰 |
| **VAD（Voice Activity Detection）** | 语音活动检测，判断音频中是否包含人声 |
| **BSS（Blind Source Separation）** | 盲源分离，使用双麦克风分离目标声源和噪声 |
| **NS（Noise Suppression）** | 噪声抑制，单麦克风场景下的降噪处理 |

### 3.3 AFE 工作流程

```
音频输入 ──> AEC（回声消除）──> 降噪 ──> VAD（语音检测）──> 输出增强音频
                               │
                    ┌──────────┴──────────┐
                    │                      │
              双麦克风：BSS          单麦克风：NS
              盲源分离               噪声抑制
```

### 3.4 Amazon Alexa 认证

乐鑫的双麦克风声学前端（AFE）已通过 **Amazon Alexa Built-in 设备的软件音频前端解决方案**认证，达到了商用级别的语音增强质量。

### 3.5 新增功能（2026 年更新）

- **新 AEC 算法**：用于全双工场景的改进回声消除算法
- **DOA（Direction of Arrival）**：声源定位算法，判断语音来源方向
- **VADNet**：基于神经网络的语音活动检测模型，可替代传统 WebRTC VAD

---

## 4. 唤醒词引擎（WakeNet）

### 4.1 WakeNet 概述

WakeNet 是乐鑫的语音唤醒引擎，专为低功耗嵌入式 MCU 设计，提供高性能、低内存消耗的唤醒词检测算法。WakeNet 使设备能够持续待命，等待唤醒词触发。

### 4.2 技术特点

- **基于神经网络**：使用 Dilated Convolution（空洞卷积）模型结构
- **参数量少**：WakeNet9 模型不到 300K 参数
- **运行速度快**：32 毫秒帧的平均运行时间为 3 毫秒
- **多唤醒词支持**：最多同时支持 5 个唤醒词
- **MFCC 特征提取**：使用 Mel-Frequency Cepstral Coefficients 提取语音特征

### 4.3 版本演进

| 版本 | 状态 | 适用芯片 |
|------|------|---------|
| WakeNet 1-8 | 已停止使用 | - |
| WakeNet 9 | 推荐 | ESP32、ESP32-S3、ESP32-P4 |
| WakeNet 9l | 新增 | ESP32、ESP32-S3、ESP32-P4（高速响应优化） |
| WakeNet 9s | 新增 | ESP32-C3、ESP32-C5、ESP32-C6（无 PSRAM/SIMD 芯片） |

### 4.4 WakeNet9 在 ESP32-S3 上的性能

| 测试场景 | 距离 | 识别准确率 |
|---------|------|-----------|
| 安静环境 | 1 米 | 98% |
| 平稳噪声（SNR 4dB） | 1 米 | 96% |
| 语音噪声（SNR 4dB） | 1 米 | 94% |
| 安静环境 | 3 米 | 不受影响 |

> 误报率：每 12 小时误触发 1 次

### 4.5 支持的唤醒词

| 唤醒词 | 说明 |
|--------|------|
| **Hi, Lexin**（Hi 乐鑫） | 免费官方唤醒词 |
| **Hi, ESP** | 免费官方唤醒词 |
| **你好小智**（nihaoxiaozhi） | 免费官方唤醒词 |
| **你好小鑫**（nihaoxiaoxin） | 免费官方唤醒词 |
| **小爱同学**（xiaoaitongxue） | 支持 |
| **Alexa** | 支持 |
| **天猫精灵** | 支持 |
| **自定义唤醒词** | 乐鑫提供定制服务 |

### 4.6 自定义唤醒词

乐鑫提供两种自定义唤醒词的方式：

1. **乐鑫定制服务**：参考"乐鑫语音唤醒词定制流程"文档，由乐鑫团队训练定制
2. **TTS 样本训练**：使用 TTS Pipeline 生成的样本训练唤醒词
   - TTS Pipeline V3 支持中文、英文、日语、法语训练
   - 计划支持韩语、西班牙语、葡萄牙语、德语、俄语、阿拉伯语

### 4.7 WakeNet 使用方法

WakeNet 目前包含在声学前端 AFE 中，默认为运行状态：

```c
// 如需禁用 WakeNet
afe_config.wakenet_init = false;

// 运行时临时关闭/打开 WakeNet
afe_handle->disable_wakenet(afe_data);
afe_handle->enable_wakenet(afe_data);

// 修改检测阈值
afe_handle->set_wakenet_threshold(afe_handle, model_index, threshold);
afe_handle->reset_wakenet_threshold(afe_handle, model_index);  // 恢复默认
```

---

## 5. 语音命令词识别（MultiNet）

### 5.1 MultiNet 概述

MultiNet 是乐鑫的离线语音命令词识别模型，专为提供灵活的离线语音命令词识别而设计。唤醒词被检测到后，MultiNet 激活并监听语音命令。

### 5.2 核心优势

- **离线识别**：完全离线运行，无需网络连接，保护用户隐私
- **灵活自定义**：用户可轻松添加自己的语音命令词，**无需重新训练模型**
- **大容量支持**：支持最多 **200 个**中英文命令词
- **多语言**：支持中文和英文命令词识别
- **同义命令**：多个不同的说法可以映射到同一个命令 ID

### 5.3 支持的命令词示例

| 中文命令 | 英文命令 |
|---------|---------|
| 打开空调 | Turn on the light |
| 打开卧室灯 | Increase the temperature |
| 关闭电视 | Turn off my soundbox |
| 调高音量 | Turn on the kitchen light |
| 切换模式 | Shuffle my playlist |

### 5.4 MultiNet 工作原理

MultiNet 的工作流程包含两个主要分支：

```
┌──────────────────────────────────────────────────────┐
│                  MultiNet 工作流程                    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  分支 1（上）：神经网络识别                            │
│  音频输入 ──> MFCC 特征提取 ──> 神经网络分类           │
│                                                      │
│  分支 2（下）：命令匹配                                │
│  预定义命令列表 ──> 命令 FST（有限状态转换器）          │
│                                                      │
│  神经网络输出 + 命令 FST ──> 最终命令 ID              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

- **神经网络**：基于 Conformer / Zipformer 结构
- **命令 FST**：有限状态转换器，将语音输出与预定义命令匹配
- **灵活映射**：多个说法可以映射到同一命令 ID（如"太热了"、"调低温度"、"让它凉快点"都映射到命令 ID 2）

### 5.5 MultiNet 版本

| 版本 | 特点 |
|------|------|
| MultiNet 6 | 基于 Conformer 结构，支持中英文 |
| MultiNet 7 | 基于 Zipformer 结构，参数量更少，准确率更高 |

**MultiNet7 相比 MultiNet6 的提升：**

| 环境 | 提升幅度 |
|------|---------|
| 安静环境 | +1.0% |
| 平稳噪声 | +1.5% |
| 语音噪声 | +2.6% |
| 参数量 | 减少 100 万参数 |

### 5.6 命令词超时机制

唤醒词被检测到后，MultiNet 激活并持续监听命令。当检测到足够长的静音时（通常 5-6 秒），MultiNet 被禁用，WakeNet 重新激活，进入待机状态。

---

## 6. VAD 语音活动检测（VADNet）

VADNet 是乐鑫 2025 年 2 月发布的基于神经网络的语音活动检测模型。它可以替代传统的 WebRTC VAD，提供更高的检测准确率。

VADNet 作为 ESP-SR 框架的一部分，可以独立使用或集成到 AFE 流程中。

---

## 7. 语音合成（Speech Synthesis）

ESP-SR 提供语音合成（Text-To-Speech, TTS）功能：

- **当前支持**：中文 TTS
- **英文 TTS**：开发中
- **用途**：将识别到的命令词通过语音播报出来，提升用户体验

**TTS Pipeline V3 更新（2026 年 4 月）：**
- 支持唤醒词训练语言：中文、英文、日语、法语
- 计划支持：韩语、西班牙语、葡萄牙语、德语、俄语、阿拉伯语

---

## 8. ESP32-S3 AI 加速能力

ESP32-S3 在 AI 语音处理方面具有显著优势：

### 8.1 硬件加速

| 特性 | 说明 |
|------|------|
| **向量指令** | 专为加速神经网络计算和信号处理设计的 SIMD 指令 |
| **双核 Xtensa LX7** | 240 MHz 主频，强大的并行处理能力 |
| **大容量 PSRAM** | 支持 8 MB 八线 PSRAM，满足神经网络模型存储需求 |
| **高带宽 Flash** | 支持八线 SPI Flash，快速加载模型 |

### 8.2 ESP32-S3 专属优化

ESP32-S3 的向量指令可以显著加速以下运算：
- 矩阵乘法（神经网络核心运算）
- FFT（快速傅里叶变换，音频特征提取）
- 卷积运算
- 量化/反量化

### 8.3 模型存储方案

| 方案 | 说明 |
|------|------|
| **Flash Partition** | 模型存储在 flash 分区中，启动时加载 |
| **PSRAM** | 模型加载到 PSRAM 中运行，速度更快 |

---

## 9. 支持的开发板

### 9.1 ESP32-S3-Korvo-1

推荐用于语音唤醒和命令词识别的最佳体验开发板：

- 三麦克风阵列，适合远场语音拾取
- ESP32-S3-WROOM-1 模组（ESP32-S3R8，8 MB PSRAM，16 MB flash）
- ES8311 编解码芯片 + ES7210 四通道 ADC
- 扬声器输出、耳机输出
- SD 卡槽、USB 接口
- 电池供电支持

### 9.2 ESP32-S3-Korvo-2

- 双麦克风阵列
- 集成 LCD、Camera、TF 卡
- 支持 JPEG 视频流处理
- ES8311 + ES7210 音频芯片组合

### 9.3 ESP32-S3-EYE

- 200 万像素摄像头 + LCD 显示屏 + 麦克风
- 8 MB 八线 PSRAM + 8 MB flash
- 集成语音唤醒、语音命令识别和人脸检测功能

### 9.4 ESP-BOX 系列

- ESP32-S3-BOX：集成触摸屏、麦克风阵列、扬声器
- ESP32-S3-BOX-3：最新版本
- ESP32-S3-BOX-Lite：精简版

---

## 10. 如何使用 ESP-Skainet

### 10.1 环境准备

```bash
# 克隆 ESP-Skainet 仓库（已包含 ESP-SR 组件）
git clone --recursive https://github.com/espressif/esp-skainet.git

# 确保 ESP-IDF 已安装（版本请参考 ESP-Skainet README 中的推荐版本）
```

### 10.2 唤醒词检测示例

```bash
cd esp-skainet/examples/wake_word_detection/wakenet

# 设置目标芯片
idf.py set-target esp32s3

# 配置开发板和唤醒词
idf.py menuconfig
# Audio Media HAL -> Audio hardware board -> ESP32-S3-Korvo-1
# ESP Speech Recognition -> Select wake words -> Hi,Lexin (wn9_hilexin)

# 编译烧录
idf.py flash monitor
```

### 10.3 使用 AFE 接口的唤醒词检测

如果使用双麦克风或需要更多语音增强算法：

```bash
cd esp-skainet/examples/wake_word_detection/afe

idf.py set-target esp32s3
idf.py menuconfig
# Audio Media HAL -> Audio hardware board -> ESP32-S3-Korvo-1
# ESP Speech Recognition -> Select wake words -> Hi,Lexin (wn9_hilexin)

idf.py flash monitor
```

### 10.4 加载多个唤醒词

ESP-Skainet 支持同时加载多个唤醒词模型（最多两个同时运行）：

```bash
idf.py menuconfig
# ESP Speech Recognition -> Select wake words -> Hi,Lexin (wn9_hilexin) -> Load Multiple Wake Words
# ESP Speech Recognition -> Load Multiple Wake Words ->
#     [x] Hi,Lexin (wn9_hilexin)
#     [x] Hi,ESP (wn9_hiesp)
```

### 10.5 完整语音交互示例

```c
#include "esp_afe_sr_models.h"
#include "esp_wn_iface.h"
#include "esp_mn_iface.h"

void app_main(void)
{
    // 1. 配置 AFE
    afe_config_t afe_config = AFE_CONFIG_DEFAULT();
    afe_config.wakenet_init = true;  // 启用 WakeNet
    afe_config.wakenet_model_name = "wn9_hilexin";

    // 2. 创建 AFE
    esp_afe_sr_iface_t *afe_handle = esp_afe_sr_iface_init();

    // 3. 初始化 MultiNet
    esp_mn_iface_t *multinet = esp_mn_iface_init();
    esp_mn_commands_add(1, "turn on the light");
    esp_mn_commands_add(2, "turn off the light");
    esp_mn_commands_add(3, "increase temperature");
    esp_mn_commands_update();

    // 4. 处理音频流
    while (true) {
        // 获取 AFE 处理结果
        afe_fetch_result_t *res = afe_handle->fetch(afe_data);

        if (res->wakeup_state == WAKEUP_DETECTED) {
            // 唤醒词检测到，激活 MultiNet
            printf("Wake word detected!\n");
        }

        if (res->wakeup_state == WAKEUP_CMD) {
            // 命令词识别结果
            int command_id = multinet->get_results(res->data);
            printf("Command ID: %d\n", command_id);
            // 根据 command_id 执行相应操作
            switch (command_id) {
                case 1: /* 打开灯 */ break;
                case 2: /* 关闭灯 */ break;
                case 3: /* 调高温度 */ break;
            }
        }
    }
}
```

---

## 11. 自定义命令词

### 11.1 添加命令词

```c
// 添加新命令
esp_mn_commands_add(50, "turn on the kitchen light");
esp_mn_commands_add(51, "shuffle my playlist");

// 修改已有命令
esp_mn_commands_modify(1, "turn on my speaker");  // 将 "turn on my soundbox" 改为 "turn on my speaker"

// 应用更改
esp_mn_commands_update();
```

### 11.2 多语言命令

MultiNet 支持中文和英文命令词。在 menuconfig 中选择语言模型：

```
ESP Speech Recognition -> Select speech commands model -> MultiNet (Chinese/English)
```

一次只能激活一个语言模型，但可以在烧录前切换。

---

## 12. 与 MQTT 项目结合

ESP-Skainet 可以与 MQTT 项目深度结合，实现**语音控制的物联网设备**：

### 12.1 语音控制智能家居

```
用户说话 "Hi 乐鑫，打开客厅灯"
  └─> WakeNet 检测到 "Hi 乐鑫"
      └─> MultiNet 识别命令 "打开客厅灯" -> 命令 ID 1
          └─> 通过 MQTT 发布消息到 topic "home/light/living_room"
              └─> 智能灯泡订阅该 topic 并打开
```

### 12.2 MQTT 消息发布示例

```c
void on_command_detected(int command_id) {
    switch (command_id) {
        case 1:  // "打开客厅灯"
            esp_mqtt_client_publish(mqtt_client, "home/light/living_room", "ON", 0, 1, 0);
            break;
        case 2:  // "关闭客厅灯"
            esp_mqtt_client_publish(mqtt_client, "home/light/living_room", "OFF", 0, 1, 0);
            break;
        case 3:  // "调高温度"
            esp_mqtt_client_publish(mqtt_client, "home/thermostat/setpoint", "26", 0, 1, 0);
            break;
    }
}
```

### 12.3 优势

- **完全离线**：语音识别在本地完成，无需云端，保护隐私
- **低延迟**：从说话到 MQTT 消息发送仅需毫秒级延迟
- **低成本**：单个 ESP32-S3 芯片即可完成全部功能

---

## 13. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-Skainet GitHub | <https://github.com/espressif/esp-skainet> |
| ESP-Skainet 中文 README | <https://github.com/espressif/esp-skainet/blob/master/README_cn.md> |
| ESP-SR GitHub | <https://github.com/espressif/esp-sr> |
| ESP-SR 组件注册表 | <https://components.espressif.com/components/espressif/esp-sr> |
| WakeNet 唤醒词文档 | <https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/wake_word_engine/README.html> |
| MultiNet 命令词文档 | <https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/speech_command_recognition/README.html> |
| AFE 声学前端文档 | <https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32s3/audio_front_end/index.html> |
| 唤醒词定制流程 | <https://docs.espressif.com/projects/esp-sr/zh_CN/latest/esp32/wake_word_engine/ESP_Wake_Words_Customization.html> |
| ESP-Skainet 发布新闻 | <https://www.espressif.com/zh-hans/news/ESP-Skainet_Released> |
| DevCon23 演讲视频 | <https://www.youtube.com/watch?v=txXVJlKr-RY> |
| ESP32-S3-Korvo-1 用户指南 | <https://github.com/espressif/esp-skainet/blob/master/docs/zh_CN/hw-reference/esp32s3/user-guide-korvo-1.md> |
| 唤醒词检测示例 | <https://github.com/espressif/esp-skainet/tree/master/examples/wake_word_detection> |

---

> **文档总结**：ESP-Skainet 是乐鑫为 ESP32 系列芯片打造的离线语音识别框架，核心包含三个组件：声学前端 AFE（回声消除、降噪、声源分离）、唤醒词引擎 WakeNet（高性能低功耗，支持 5 个唤醒词同时检测）、语音命令词识别 MultiNet（支持 200 个中英文命令，无需重新训练即可自定义）。ESP32-S3 凭借向量指令加速、大容量 PSRAM 和强大的双核计算能力，成为 ESP-Skainet 的最佳目标平台。与 MQTT 项目结合，可实现完全离线的语音控制物联网设备，从说话到 MQTT 消息发布仅需毫秒级延迟。
