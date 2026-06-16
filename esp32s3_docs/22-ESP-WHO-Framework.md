# ESP-WHO 图像识别框架详解

> 文档编号：22  
> 适用芯片：ESP32 / ESP32-S3 / ESP32-P4（ESP32 和 ESP32-S2 暂未适配新版本）  
> 适用项目：人脸检测与识别、行人检测、二维码识别、AIoT 视觉应用  
> 来源：乐鑫官方 ESP-WHO / ESP-DL 文档、GitHub 仓库、ESP32-S3-EYE 用户指南

---

## 目录

1. [ESP-WHO 概述](#1-esp-who-概述)
2. [框架架构](#2-框架架构)
3. [ESP-DL 深度学习库](#3-esp-dl-深度学习库)
4. [帧捕获管线](#4-帧捕获管线frame-cap-pipeline)
5. [人脸检测](#5-人脸检测)
6. [人脸识别](#6-人脸识别)
7. [行人检测](#7-行人检测)
8. [二维码识别](#8-二维码识别)
9. [摄像头支持](#9-摄像头支持)
10. [ESP32-S3 视觉能力](#10-esp32-s3-视觉能力)
11. [支持的开发板](#11-支持的开发板)
12. [如何使用 ESP-WHO](#12-如何使用-esp-who)
13. [与 MQTT 项目结合](#13-与-mqtt-项目结合)
14. [参考资源](#14-参考资源)

---

## 1. ESP-WHO 概述

### 1.1 什么是 ESP-WHO

ESP-WHO（Espressif WHO，Where is Human Object）是乐鑫基于其芯片打造的图像处理开发平台。它提供了人脸检测、人脸识别、行人检测、二维码识别等示例，开发者可以基于这些示例衍生出丰富的实际应用。

ESP-WHO 基于 **ESP-DL**（Espressif Deep Learning Library）开发，是乐鑫为 AIoT 领域推出的核心视觉识别开发框架。

### 1.2 新版本特性（重大重构）

ESP-WHO 经历了全面重构，新版本带来以下改进：

| 特性 | 说明 |
|------|------|
| **适配新版 ESP-DL** | 使用重构后的 ESP-DL 深度学习推理库 |
| **支持 ESP32-P4** | 新增对 ESP32-P4 芯片的支持 |
| **异步执行** | 摄像头和深度学习模型异步执行，实现更高帧率 |
| **LVGL 图形支持** | 集成 LVGL 轻量级多功能图形库，可开发图形应用 |
| **新增行人检测模型** | 新增行人检测深度学习模型 |
| **任务化架构** | 基于 FreeRTOS 的任务架构，提供一致的生命周期管理 |

### 1.3 支持的芯片

| 芯片 | ESP-IDF v5.3 | ESP-IDF v5.4 | ESP-IDF v5.5 |
|------|-------------|-------------|-------------|
| ESP32-S3 | 支持 | 支持 | 支持 |
| ESP32-P4 | 支持 | 支持 | 支持 |
| ESP32 | 旧分支支持 | - | - |
| ESP32-S2 | 旧分支支持 | - | - |

> 注意：ESP32 和 ESP32-S2 及部分示例（猫脸检测、颜色检测）暂未适配新分支，可在旧分支 [release/v1.1.0](https://github.com/espressif/esp-who/tree/release/v1.1.0) 中找到。

---

## 2. 框架架构

### 2.1 分层架构

```
┌─────────────────────────────────────────────────┐
│              Application（应用层）                │
│   (human_face_recognition, object_detect...)    │
├─────────────────────────────────────────────────┤
│          ESP-WHO 组件                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │WhoFrameCap│  │WhoDetect │  │WhoRecogni│     │
│  │ 帧捕获    │  │ 目标检测  │  │ tion     │     │
│  └──────────┘  └──────────┘  └──────────┘     │
├─────────────────────────────────────────────────┤
│          ESP-DL（深度学习库）                     │
│  Models: Face Detect, Face Recognition,         │
│         Pedestrian Detect, MNIST, ...           │
├─────────────────────────────────────────────────┤
│          BSP / 外设驱动                           │
│  Camera, LCD, Button, SD Card, LED, Touch       │
├─────────────────────────────────────────────────┤
│          ESP-IDF（物联网开发框架）                 │
│          FreeRTOS + LWIP + ...                  │
├─────────────────────────────────────────────────┤
│    ESP32-S3 / ESP32-P4 SoC                     │
└─────────────────────────────────────────────────┘
```

### 2.2 任务化架构

ESP-WHO 使用基于 FreeRTOS 的任务化架构：

- **WhoTaskBase**：基础任务类，定义一致的生命周期
- **WhoTask**：标准任务类，扩展异步操作支持
- **WhoFrameCapNode**：帧捕获节点基类，作为处理阶段的基础
- **WhoDetect**：检测类，消费帧并运行 ESP-DL 检测模型
- **WhoRecognitionCore**：识别核心类，扩展检测功能，包含特征提取和匹配

### 2.3 典型工作流

```
[Camera Input] --> [Frame Capture] --> [Detection] --> [Recognition (Optional)] --> [Output (LCD/Terminal)]
                                          │
                                          └──────> [Skip] ──────> [Output]
```

节点之间通过 **Queue（队列）** 和 **Event Bit（事件位）** 连接，形成异步管线。

---

## 3. ESP-DL 深度学习库

### 3.1 ESP-DL 概述

ESP-DL（Espressif Deep Learning Library）是乐鑫为 ESP 系列芯片开发的优化深度学习推理库。ESP-WHO 的所有 AI 功能都建立在 ESP-DL 之上。

### 3.2 ESP-DL 提供的模型

| 模型 | 功能 | 适用芯片 |
|------|------|---------|
| **Human Face Detect** | 人脸检测 | ESP32-S3, ESP32-P4 |
| **Human Face Recognition** | 人脸识别（特征提取+匹配） | ESP32-S3, ESP32-P4 |
| **Pedestrian Detect** | 行人检测 | ESP32-S3, ESP32-P4 |
| **MNIST** | 手写数字识别 | ESP32-S3, ESP32-P4 |
| **Cat Face Detect** | 猫脸检测 | 旧版本 |
| **Color Detect** | 颜色检测 | 旧版本 |

### 3.3 人脸检测模型详情

ESP-DL 提供多种人脸检测模型：

| 模型名称 | 类型 | 输入尺寸 | 特点 |
|---------|------|---------|------|
| MSR_S8_V1 + MNP_S8_V1 | 两阶段模型 | 120x160x3 -> 48x48x3 | 第一阶段预测候选，第二阶段精细化 |
| ESPDET_PICO_224_224_FACE | 单阶段模型 | 224x224x3 | 平衡速度和精度 |
| ESPDET_PICO_416_416_FACE | 单阶段模型 | 416x416x3 | 高精度检测 |

**模型延迟（ESP32-S3）：**

| 模型 | 预处理(ms) | 模型推理(ms) | 后处理(ms) |
|------|-----------|-------------|-----------|
| msr_s8_v1_s3 | 3.8 | 33.1 | 0.3 |
| mnp_s8_v1_s3 | 0.6 | 5.8 | 0.1 |
| espdet_pico_224_224_face_s8_s3 | 7.2 | 131.6 | 0.8 |
| espdet_pico_416_416_face_s8_s3 | 21.8 | 437.0 | 1.3 |

---

## 4. 帧捕获管线（Frame Cap Pipeline）

### 4.1 帧捕获节点

`who_frame_cap` 组件负责帧捕获管线。`WhoFrameCapNode` 是处理阶段的基类，节点之间通过队列和事件位连接，形成异步管线。

### 4.2 异步管线优势

新版本的 ESP-WHO 将摄像头采集和深度学习模型推理改为异步执行：
- 摄像头采集帧后立即放入队列
- 检测模型从队列中取帧进行处理
- 摄像头可以继续采集下一帧
- 实现更高的帧率（FPS）

### 4.3 帧处理序列图

```
Camera        WhoFrameCap     WhoDetectBase     WhoRecognition     Display
  │                │                │                 │                │
  │── Raw Frame ──>│                │                 │                │
  │                │── Processed ──>│                 │                │
  │                │     Frame      │                 │                │
  │                │           [Run Detection Model]  │                │
  │                │                │── Detection ───>│                │
  │                │                │     Results     │                │
  │                │                │── Detection ──────────────────>│
  │                │                │     Results     │                │
  │                │                │           [Run Recognition]     │
  │                │                │                 │── Results ───>│
  │                │                │                 │                │
```

---

## 5. 人脸检测

### 5.1 人脸检测功能

人脸检测是 ESP-WHO 的核心功能之一，能够在图像中定位人脸位置，并标注关键面部特征点。

### 5.2 检测输出

人脸检测输出包括：
- **人脸边界框**：`[score, x1, y1, x2, y2]`
- **关键面部特征点**：
  - 左眼位置 (left_eye)
  - 右眼位置 (right_eye)
  - 鼻子位置 (nose)
  - 左嘴角 (left_mouth)
  - 右嘴角 (right_mouth)

**输出示例：**
```
I (955) human_face_detect: [score: 0.936285, x1: 100, y1: 64, x2: 193, y2: 191]
I (955) human_face_detect: left_eye: [117, 114], left_mouth: [120, 160], nose: [132, 143], right_eye: [157, 112], right_mouth: [151, 160]
```

### 5.3 检测 API

```cpp
// 创建人脸检测器
HumanFaceDetect *detect = new HumanFaceDetect();

// 运行检测
dl::image::img_t img = {.data=DATA, .width=WIDTH, .height=HEIGHT, .pix_type=PIX_TYPE};
detect->run(img);
```

---

## 6. 人脸识别

### 6.1 人脸识别功能

人脸识别在人脸检测的基础上进一步提取人脸特征，并与已注册的人脸特征进行比对，实现身份识别。

### 6.2 识别流程

```
人脸检测 ──> 人脸对齐 ──> 特征提取 ──> 特征匹配 ──> 身份 ID
                                              │
                                     ┌────────┴────────┐
                                     │                  │
                               匹配成功            匹配失败
                               返回 ID            返回"WHO?"
```

### 6.3 特征存储

每个人脸特征占用 **2050 字节**：
- 2 字节：ID
- 2048 字节：特征数据

支持的存储方式：
| 存储方式 | 配置选项 | 说明 |
|---------|---------|------|
| Flash 分区 (FATFS) | `CONFIG_DB_FATFS_FLASH` | 存储在 1MB flash 分区 |
| SD 卡 (FATFS) | `CONFIG_DB_FATFS_SDCARD` | 存储在 SD 卡 |
| SPIFFS | `CONFIG_DB_SPIFFS` | 存储在 SPIFFS 文件系统 |

### 6.4 识别状态机

WhoRecognitionCore 管理内部状态机，支持三种操作模式：

| 模式 | 说明 |
|------|------|
| **ENROLL（注册）** | 录入新人脸特征到数据库 |
| **RECOGNIZE（识别）** | 将当前人脸与数据库中的特征进行匹配 |
| **DELETE（删除）** | 从数据库中删除人脸特征 |

### 6.5 识别输出

```
id: 3, sim: 0.750027
```

- `id`：匹配到的人脸 ID
- `sim`：相似度（越接近 1 越相似）

---

## 7. 行人检测

行人检测是 ESP-WHO 新版本中新增的检测模型，能够在图像中检测和定位行人。

使用与 `WhoDetect` 相同的框架，只需更换检测模型即可。

---

## 8. 二维码识别

ESP-WHO 提供二维码识别功能，可以识别图像中的 QR Code 并解码。

---

## 9. 摄像头支持

### 9.1 支持的摄像头型号

ESP-WHO 通过 `esp32-camera` 驱动支持多种摄像头传感器：

| 摄像头 | 分辨率 | 特点 |
|--------|--------|------|
| OV2640 | 200 万像素 | 最常用，性价比高 |
| OV3660 | 300 万像素 | 更高分辨率 |
| OV5640 | 500 万像素 | 高分辨率 |
| GC0308 | 30 万像素 | 低成本 |
| GC032A | 30 万像素 | 低成本 |

### 9.2 摄像头接口

ESP32-S3 通过 **DVP（Digital Video Port）** 或 **SPI** 接口连接摄像头：

- **DVP 接口**：并行数据传输，高帧率
- **SPI 接口**：串行传输，占用引脚少

---

## 10. ESP32-S3 视觉能力

### 10.1 硬件优势

ESP32-S3 在图像处理方面的优势：

| 特性 | 说明 |
|------|------|
| **向量指令** | SIMD 指令加速神经网络推理和图像处理 |
| **大容量 PSRAM** | 支持 8 MB 八线 PSRAM，存储高分辨率图像帧 |
| **高带宽 Flash** | 八线 SPI Flash，快速加载模型 |
| **双核 240MHz** | 并行处理：一核采集图像，一核运行 AI 推理 |
| **LCD 接口** | 直接驱动 LCD 显示屏 |
| **DVP 接口** | 高速摄像头数据采集 |
| **Wi-Fi 图传** | 支持通过 Wi-Fi 传输图像数据 |

### 10.2 性能数据

ESP32-S3 上的典型推理性能（以人脸检测为例）：

| 操作 | 耗时（ESP32-S3） |
|------|-----------------|
| 图像预处理 | 3.8 ms |
| 模型推理（MSR_S8_V1） | 33.1 ms |
| 后处理 | 0.3 ms |
| 总计 | ~37 ms（约 27 FPS） |

### 10.3 异步管线优化

新版本 ESP-WHO 的异步管线进一步提升了帧率：
- 摄像头采集和模型推理并行执行
- 采集不阻塞推理，推理不阻塞采集
- 整体帧率接近采集帧率

---

## 11. 支持的开发板

### 11.1 ESP32-S3-EYE

ESP32-S3-EYE 是 ESP-WHO 的主要目标开发板：

| 特性 | 规格 |
|------|------|
| SoC | ESP32-S3 |
| 摄像头 | 200 万像素 OV2640 |
| 显示屏 | LCD（ST7789） |
| 麦克风 | 数字麦克风 |
| 按钮 | 多个功能按钮 |
| 存储 | 8 MB 八线 PSRAM + 8 MB flash |
| SD 卡 | 支持 MicroSD 卡 |
| IMU | 支持加速度传感器 |
| Wi-Fi | 支持图传 |

**出厂默认固件功能：**
- 语音唤醒（"Hi ESP" / "Hi 乐鑫"）
- 语音命令识别
- 人脸检测和识别
- 移动侦测
- 实时画面显示

### 11.2 ESP32-S3-Korvo-2

| 特性 | 规格 |
|------|------|
| SoC | ESP32-S3 |
| 摄像头 | 支持（通过连接器） |
| 显示屏 | LCD（ILI9341） |
| 音频 | ES8311 + ES7210 |
| 存储 | 16 MB Flash + 8 MB PSRAM |
| 触摸 | 支持（TT21100） |
| SD 卡 | 支持 |
| LED | 支持 |

### 11.3 ESP32-P4-Function-EV-Board

新一代 ESP32-P4 开发板，提供更强的 AI 计算能力：

| 特性 | 规格 |
|------|------|
| SoC | ESP32-P4 |
| 音频 | ES8311（编解码+麦克风+扬声器） |
| 显示屏 | 多种 LCD 支持（EK79007, ILI9881C, LT8912B） |
| 触摸 | 支持（GT911） |
| SD 卡 | 支持 |

### 11.4 Maple Eye ESP32-S3

第三方开发板 AnalogLamb Maple Eye ESP32-S3：
- 与 ESP32-S3-EYE 兼容
- 200 万像素摄像头
- 双 LCD 显示屏
- 数字麦克风
- 8 MB 八线 PSRAM + 8 MB flash
- MicroSD 卡接口

---

## 12. 如何使用 ESP-WHO

### 12.1 环境要求

- **ESP-IDF 版本**：v5.5.x（当前最新要求）
- 安装方法：使用 EIM（ESP-IDF Installation Manager）

### 12.2 快速开始

```bash
# 1. 克隆 ESP-WHO 仓库
git clone https://github.com/espressif/esp-who.git

# 2. 设置环境变量
# Linux:
export IDF_EXTRA_ACTIONS_PATH=/path_to_esp-who/tools/

# Windows (PowerShell):
$Env:IDF_EXTRA_ACTIONS_PATH="/path_to_esp-who/tools/"

# Windows (CMD):
set IDF_EXTRA_ACTIONS_PATH=/path_to_esp-who/tools/

# 3. 进入人脸识别示例
cd esp-who/examples/human_face_recognition

# 4. 设置目标和开发板
idf.py -DSDKCONFIG_DEFAULTS=sdkconfig.bsp.esp32_s3_eye set-target esp32s3

# 5. （可选）配置选项
idf.py menuconfig

# 6. 编译烧录
idf.py -p COM3 flash monitor
```

### 12.3 使用 VSCode

在 `.vscode/settings.json` 中配置：

```json
{
  "idf.currentSetup": "<path-to-esp-idf>/v5.5.4/esp-idf",
  "idf.openOcdConfigs": ["board/esp32s3-builtin.cfg"],
  "idf.customExtraVars": {
    "IDF_EXTRA_ACTIONS_PATH": "<path-to-esp-who>/esp-who/tools",
    "SDKCONFIG_DEFAULTS": "sdkconfig.bsp.esp32_s3_eye",
    "IDF_TARGET": "esp32s3"
  }
}
```

然后在 ESP-IDF 终端中运行 `idf.py reconfigure`，最后 Build、Flash、Monitor。

### 12.4 启动日志

正常启动时可以看到以下日志：

```
I (1240) cam_hal: cam init ok
I (1254) camera: Camera PID=0x26 VER=0x42 MIDL=0x7f MIDH=0xa2
I (1254) camera: Detected OV2640 camera
I (1254) camera: Detected camera at address=0x30
[...]
I (1732) LVGL: Starting LVGL task
I (1853) S3-EYE: Setting LCD backlight: 100%
[...]
W (1966) dl::Model: Minimize() will delete variables not used in model inference
[...]
I (2053) button: IoT Button Version: 4.1.5
I (2057) gpio: GPIO[0]| InputEn: 1| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0
I (2065) main_task: Returned from app_main()
```

### 12.5 人脸识别操作

**使用按钮操作：**
1. 面对摄像头，检测到人脸后显示蓝色方框
2. 按 **MENU** 键：录入人脸 ID
3. 按 **UP+** 键：进行人脸识别，显示匹配的 ID 或 "WHO?"
4. 按 **PLAY** 键：删除最新录入的人脸 ID

**使用语音命令操作：**

先说唤醒词 "Hi 乐鑫" 唤醒设备，然后使用以下命令：

| 语音命令 | 设备响应 |
|---------|---------|
| 添加人脸 | 录入人脸 ID |
| 识别一下 | 显示识别到的人脸 ID 或 "WHO?" |
| 删除人脸 | 删除最新人脸 ID |
| 人脸识别 | 进入人脸识别模式 |
| 移动侦测 | 进入移动侦测模式 |
| 仅显示 | 仅显示实时画面 |
| 停止工作 | 停止工作 |

### 12.6 人脸检测 C++ API

```cpp
#include "human_face_detect.hpp"

// 创建检测器
HumanFaceDetect *detect = new HumanFaceDetect();

// 准备图像
dl::image::img_t img = {
    .data = image_data,
    .width = WIDTH,
    .height = HEIGHT,
    .pix_type = dl::image::PIX_TYPE_RGB888
};

// 运行检测
std::vector<dl::detect::result_t> results = detect->run(img);

// 遍历检测结果
for (const auto& res : results) {
    printf("score: %f, box: [%d, %d, %d, %d]\n",
           res.score, res.box[0], res.box[1], res.box[2], res.box[3]);
}
```

### 12.7 扩展检测回调

ESP-WHO 支持自定义检测回调，例如检测到人脸时点亮 LED：

```cpp
void on_face_detected(const std::vector<dl::detect::result_t>& results) {
    if (!results.empty()) {
        // 有人脸被检测到
        gpio_set_level(LED_GPIO, 1);  // 点亮 LED
    } else {
        gpio_set_level(LED_GPIO, 0);  // 熄灭 LED
    }
}
```

---

## 13. 与 MQTT 项目结合

ESP-WHO 可以与 MQTT 项目结合，实现 **AI 视觉物联网应用**：

### 13.1 门禁系统

```
摄像头检测人脸 ──> 人脸识别 ──> 匹配成功？
                                │
                    ┌───────────┴───────────┐
                    │                       │
                   是                      否
                    │                       │
            MQTT 发布             MQTT 发布
            "door/access"        "door/alert"
            "USER_ID:3"          "Unknown face detected"
                    │                       │
            门禁系统开门           发送告警通知
```

### 13.2 MQTT 消息发布示例

```c
void on_face_recognized(int face_id, float similarity) {
    char payload[64];
    snprintf(payload, sizeof(payload),
             "{\"id\":%d,\"sim\":%.2f,\"action\":\"access\"}",
             face_id, similarity);
    esp_mqtt_client_publish(mqtt_client, "vision/face_recognition",
                            payload, 0, 1, 0);
}

void on_unknown_face_detected() {
    esp_mqtt_client_publish(mqtt_client, "vision/alert",
                            "{\"type\":\"unknown_face\",\"timestamp\":...}",
                            0, 1, 0);
}
```

### 13.3 智能监控

- **人脸计数**：通过 MQTT 上报当前画面中的人脸数量
- **陌生人告警**：检测到未注册人脸时通过 MQTT 发送告警
- **行人检测**：检测到行人时上报事件
- **图像传输**：通过 Wi-Fi 将检测帧传输到服务器

### 13.4 语音+视觉融合

ESP32-S3-EYE 同时支持 ESP-WHO（视觉）和 ESP-Skainet（语音），可实现：
- 语音命令 "添加人脸" -> 录入新人脸
- 语音命令 "识别一下" -> 人脸识别并 MQTT 上报
- 人脸识别 + 语音播报识别结果

---

## 14. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-WHO GitHub | <https://github.com/espressif/esp-who> |
| ESP-WHO 中文 README | <https://github.com/espressif/esp-who/blob/master/README_CN.md> |
| ESP-DL GitHub | <https://github.com/espressif/esp-dl> |
| ESP-WHO 入门教程 | <https://developer.espressif.com/blog/2026/05/esp-who-get-started/> |
| ESP32-S3-EYE 用户指南（中文） | <https://github.com/espressif/esp-who/blob/master/docs/zh_CN/get-started/ESP32-S3-EYE_Getting_Started_Guide.md> |
| 人脸检测模型文档 | <https://github.com/espressif/esp-dl/blob/master/models/human_face_detect/README.md> |
| 人脸识别模型文档 | <https://github.com/espressif/esp-dl/blob/master/models/human_face_recognition/README.md> |
| 人脸识别示例 | <https://github.com/espressif/esp-who/tree/master/examples/human_face_recognition> |
| 人脸检测示例 | <https://github.com/espressif/esp-dl/blob/master/examples/human_face_detect/README.md> |
| ESP-EYE 产品页面 | <https://www.espressif.com/zh-hans/products/devkits/esp-eye/overview> |
| Maple Eye ESP32-S3 新闻 | <https://www.espressif.com/zh-hans/news/Maple_Eye_ESP32-S3> |
| ESP32-S3-Korvo-2 用户指南 | <https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2.html> |
| ESP32-P4 Function EV Board | <https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32p4/esp32-p4-function-ev-board/user_guide.html> |
| esp32-camera 组件 | <https://components.espressif.com/components/espressif/esp32-camera> |
| ESP-TFLite-Micro（TensorFlow Lite） | <https://components.espressif.com/components/espressif/esp-tflite-micro> |
| LVGL 图形库 | <https://lvgl.io/> |

---

> **文档总结**：ESP-WHO 是乐鑫为 ESP32-S3 和 ESP32-P4 打造的图像识别开发框架，基于 ESP-DL 深度学习推理库。新版本经过全面重构，采用异步管线架构（摄像头采集与 AI 推理异步执行），实现了更高的帧率。核心功能包括人脸检测（支持两阶段和单阶段模型）、人脸识别（特征提取+匹配，支持 Flash/SD 卡/SPIFFS 存储）、行人检测和二维码识别。ESP32-S3 凭借向量指令加速、8 MB PSRAM 和 DVP 摄像头接口，能够实现约 27 FPS 的人脸检测。与 MQTT 项目结合，可构建门禁系统、智能监控、陌生人告警等 AIoT 视觉物联网应用。
