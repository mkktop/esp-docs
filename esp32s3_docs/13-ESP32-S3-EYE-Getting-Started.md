# ESP32-S3-EYE 入门指南 — 图像识别 AI 开发板

> **文档来源**: 基于乐鑫官方 Espressif ESP-WHO 文档、ESP32-S3-EYE v2.2 用户指南及 BSP 组件文档整理。

---

## 目录

- [1. 产品概述](#1-产品概述)
- [2. 硬件架构](#2-硬件架构)
- [3. 摄像头](#3-摄像头)
- [4. 麦克风](#4-麦克风)
- [5. LCD 显示屏](#5-lcd-显示屏)
- [6. 电池与供电](#6-电池与供电)
- [7. 主板组件详解](#7-主板组件详解)
- [8. 子板组件详解](#8-子板组件详解)
- [9. 与 ESP-EYE 的对比](#9-与-esp-eye-的对比)
- [10. 快速入门](#10-快速入门)
- [11. 应用程序开发](#11-应用程序开发)
- [12. ESP-WHO 开发框架](#12-esp-who-开发框架)
- [13. 应用场景](#13-应用场景)
- [14. 相关资源与链接](#14-相关资源与链接)

---

## 1. 产品概述

ESP32-S3-EYE 是乐鑫（Espressif）推出的一款小型 AI（人工智能）开发板，搭载 ESP32-S3 芯片和乐鑫 AI 开发框架 ESP-WHO。开发板配置一个 200 万像素的摄像头、一个 LCD 显示屏和一个数字麦克风，适用于图像识别和音频处理等应用。

板上还配有 8 MB 八线 PSRAM（Octal PSRAM）和 8 MB Flash，具有充足的存储空间。此外，ESP32-S3 芯片还为开发板提供了 Wi-Fi 图传和 USB 端口调试等功能。您可以使用 ESP-WHO 开发各种 AIoT（人工智能物联网）应用，例如智能门铃、监控系统、人脸识别打卡机等。

### 核心规格一览

| 参数 | 规格 |
|------|------|
| SoC | ESP32-S3 |
| 模组 | ESP32-S3-WROOM-1 |
| 处理器 | 240 MHz Xtensa 32-bit LX7 双核 |
| 内置 SRAM | 512 KB |
| PSRAM | 8 MB（Octal SPI，芯片叠封） |
| Flash | 8 MB |
| Wi-Fi | 802.11 b/g/n |
| 蓝牙 | Bluetooth 5 (LE) |
| AI 加速 | 专用向量指令，加速神经网络计算和信号处理 |
| 工作温度 | -40 ~ 85 °C |
| 尺寸 | 主板 60 x 30 mm，子板 31.9 x 30 mm |
| 参考价格 | 约 45 USD / 299 CNY |

---

## 2. 硬件架构

ESP32-S3-EYE 开发板包含两部分，主板和子板通过排针连接：

- **主板（ESP32-S3-EYE-MB）**：配置 ESP32-S3-WROOM-1 模组、摄像头、SD 卡槽、数字麦克风、USB 接口和功能按键等。
- **子板（ESP32-S3-EYE-SUB）**：配置 LCD 显示屏等。

### 功能框图

```
  +-------------------+          +-------------------+
  |   ESP32-S3-EYE-MB |          |  ESP32-S3-EYE-SUB |
  |                   |          |                   |
  |  +-------------+ |          |  +-------------+  |
  |  | ESP32-S3    | |          |  |   1.3" LCD   |  |
  |  | WROOM-1     |<--+--排针--+-->|  240x240    |  |
  |  | 8MB Flash   | |          |  |  (SPI)      |  |
  |  | 8MB PSRAM   | |          |  +-------------+  |
  |  +------+------+ |          |                   |
  |         |        |          |  +-------------+  |
  |  +------+-+ +---+---+ +---+|  |  Strapping   |  |
  |  |Camera  | | Mic   | |USB||  |  Pins        |  |
  |  |OV2640  | |(I2S)  | |   ||  +-------------+  |
  |  +--------+ +-------+ +---+|                   |
  |                        |   +-------------------+
  |  +------+ +--------+  |
  |  |SD Card| |Accel.  |  |
  |  | Slot  | |QMA7981 |  |
  |  +------+ +--------+  |
  |                        |
  |  +------+ +--------+  |
  |  |Battery| |Buttons |  |
  |  |Charger| |(x6)    |  |
  |  +------+ +--------+  |
  +------------------------+
```

---

## 3. 摄像头

ESP32-S3-EYE 板载一个 OV2640 摄像头模块，是人脸检测与识别应用的核心传感器。

| 参数 | 规格 |
|------|------|
| 型号 | OV2640 |
| 像素 | 200 万像素 |
| 视角 | 66.5° |
| 最高分辨率 | 1600 x 1200 |
| 接口 | DVP（Digital Video Port） |
| 驱动库 | esp32-camera（https://github.com/espressif/esp32-camera） |
| 独立供电 | 3.3V 转 1.5V LDO + 3.3V 转 2.8V LDO |

摄像头通过 DVP 接口与 ESP32-S3 连接，开发程序时可以根据需要配置不同的分辨率（如 240x240 用于 LCD 显示，更高分辨率用于图像处理）。摄像头模组具有独立的供电电路（1.5V 和 2.8V LDO），确保图像采集的稳定性。

---

## 4. 麦克风

ESP32-S3-EYE 配备一个数字 MEMS 麦克风，用于语音活动检测（VAD）和自动语音识别（ASR）。

| 参数 | 规格 |
|------|------|
| 类型 | MEMS 数字麦克风 |
| 通信接口 | I2S |
| 灵敏度 | -26 dBFS |
| 信噪比（SNR） | 61 dB |
| 供电 | 3.3V |

麦克风配合 ESP-SR 语音识别框架，可实现语音唤醒和命令词识别功能。出厂固件默认支持中文唤醒词"Hi 乐鑫"和英文唤醒词"Hi ESP"。

---

## 5. LCD 显示屏

子板（ESP32-S3-EYE-SUB）上搭载一块 1.3 英寸彩色 LCD 显示屏，用于实时显示摄像头画面和检测结果。

| 参数 | 规格 |
|------|------|
| 尺寸 | 1.3 英寸 |
| 分辨率 | 240 x 240 |
| 接口 | SPI 总线 |
| LCD 驱动 IC | ST7789 |
| 背光 | 可软件控制 |

LCD 显示屏通过 SPI 总线与 ESP32-S3 连接。在实际应用中，摄像头采集的画面可以实时显示在 LCD 上，同时叠加人脸检测框、特征点标记、识别结果等信息。

---

## 6. 电池与供电

### 6.1 供电方式

ESP32-S3-EYE 支持两种供电方式（二选一）：

| 供电方式 | 说明 |
|----------|------|
| USB 供电 | 通过 Micro-USB 接口提供 5V 电源 |
| 外接锂电池 | 通过电池焊点连接外部锂电池 |

### 6.2 电池规格

| 参数 | 推荐规格 |
|------|----------|
| 类型 | 锂电池（需带保护电路板和电流保险器） |
| 容量 | > 1000 mAh |
| 输出电压 | 3.7 V |
| 输入电压（充电） | 4.2 V - 5 V |

### 6.3 电池充电芯片

开发板集成了 ME4054BM5G-N 1A 线性锂电池充电器（ThinSOT 封装），充电电源来自 USB 接口。

### 6.4 电池指示灯

| 状态 | 指示灯表现 |
|------|------------|
| USB 连接、未接电池 | 红灯闪烁 |
| USB 连接、电池充电中 | 红灯常亮 |
| USB 连接、电池充电完成 | 红灯熄灭 |

---

## 7. 主板组件详解

以下按逆时针顺序，从摄像头开始依次介绍主板（ESP32-S3-EYE-MB）正反面的主要组件：

| 序号 | 组件 | 说明 |
|------|------|------|
| 1 | Camera（摄像头） | OV2640，200 万像素，66.5° 视角，最高 1600x1200 |
| 2 | Module Power LED（模组电源指示灯） | 绿色 LED，USB 供电时亮起，可通过 GPIO3 配置不同状态 |
| 3 | Pin Headers（排针） | 连接子板的排母 |
| 4 | 5V to 3.3V LDO | 电源转换器，输入 5V，输出 3.3V |
| 5 | Digital Microphone（数字麦克风） | MEMS 数字麦，I2S 通信，灵敏度 -26 dBFS，SNR 61 dB |
| 6 | FPC Connector（FPC 接口） | 通过 FPC 排线连接主板和子板 |
| 7 | Function Button（功能按键） | 6 个功能按键，除 RST 外均可配置 |
| 8 | ESP32-S3-WROOM-1 | 模组，内置 ESP32-S3R8 芯片，8MB PSRAM + 8MB Flash |
| 9 | MicroSD Card Slot（MicroSD 卡槽） | 用于扩充数据存储空间 |
| 10 | 3.3V to 1.5V LDO | 摄像头电源转换器 |
| 11 | 3.3V to 2.8V LDO | 摄像头电源转换器 |
| 12 | USB Port（USB 接口） | Micro-USB，用于供电和通信（GPIO19/GPIO20） |
| 13 | Battery Soldering Points（电池焊点） | 焊接电池母座插头，外接锂电池 |
| 14 | Battery Charger Chip（电池充电芯片） | ME4054BM5G-N，1A 线性锂电池充电器 |
| 15 | Battery Red LED（电池指示红灯） | 电池充电状态指示 |
| 16 | Accelerometer（加速度传感器） | QMA7981 三轴加速度传感器，用于屏幕旋转等 |

---

## 8. 子板组件详解

子板（ESP32-S3-EYE-SUB）从 LCD 显示屏开始逆时针方向的主要组件：

| 组件 | 说明 |
|------|------|
| LCD Display（LCD 显示屏） | 1.3 英寸 LCD，SPI 总线连接 |
| Strapping Pins（Strapping 管脚） | 从主板引出的四个 strapping 管脚，空闲测试点 |
| Female Headers（排母） | 连接主板上的排针 |
| LCD FPC Connector（LCD FPC 接口） | 通过 FPC 排线连接子板和 LCD 显示屏 |
| LCD_RST | LCD_RST 测试点，可用于重启 LCD 显示屏 |

---

## 9. 与 ESP-EYE 的对比

ESP32-S3-EYE 相比前代 ESP-EYE 支持更多功能，主要差异如下：

| 特性 | ESP32-S3-EYE | ESP-EYE |
|------|--------------|---------|
| 内置芯片 | ESP32-S3 | ESP32 |
| PSRAM | 8 MB 八线 PSRAM | 8 MB 四线 PSRAM |
| Flash | 8 MB | 4 MB |
| LCD 显示屏 | 有（1.3" 240x240） | 无 |
| 加速度传感器 | 有（QMA7981） | 无 |
| 外接电池 | 支持（可选） | 不支持 |
| USB 至 UART 桥接器 | 不需要（ESP32-S3 USB Serial/JTAG 提供） | 需要 |
| 天线连接器 | 不需要（模组内置天线） | 需要 |
| AI 加速 | 专用向量指令 | 无专用 AI 指令 |

---

## 10. 快速入门

### 10.1 开箱即用 — 默认固件功能测试

ESP32-S3-EYE 出厂即烧录默认固件，可立即体验语音唤醒、语音命令识别、人脸检测和识别功能。

**所需硬件**：
- 1 套 ESP32-S3-EYE 开发板
- 1 根 USB 2.0 数据线（标准 A 型转 Micro-B 型）

### 10.2 上电步骤

1. 确保开发板完好无损，主板与子板已组装好。
2. 使用 USB 数据线连接开发板的 USB 接口与电源。
3. 上电反应：
   - **模组电源指示灯**亮起几秒钟（固件加载中）。
   - **模组电源指示灯**熄灭（固件加载完成），默认进入人脸识别模式。
   - LCD 显示屏播放实时画面。

### 10.3 按键操作

| 操作 | 按键 | 功能 |
|------|------|------|
| 检测人脸 | — | 面对摄像头，检测到人脸后显示屏出现蓝色方框 |
| 录入人脸 ID | MENU | 为检测到的人脸录入 ID（从 1 开始） |
| 人脸识别 | UP+ | 识别已知人脸，显示 ID；未知人脸显示 "WHO?" |
| 删除人脸 ID | PLAY | 删除最新录入的人脸 ID，显示剩余 ID 数 |

### 10.4 语音操作

1. 说出唤醒词"Hi 乐鑫"（中文）或"Hi ESP"（英文）唤醒开发板。唤醒后模组电源指示灯亮起。
2. 说出命令词操作开发板。

**人脸识别模式命令词**：

| 中文命令词 | 设备反馈 |
|-----------|----------|
| 添加人脸 | 录入人脸 ID |
| 识别一下 | 显示识别到的人脸 ID，无法识别则显示 "WHO?" |
| 删除人脸 | 删除最新人脸 ID，显示剩余数量 |

**模式切换命令词**：

| 中文命令词 | 设备反馈 |
|-----------|----------|
| 人脸识别 | 检测到人脸时显示蓝色方框 |
| 移动侦测 | 检测到物体移动时左上角显示蓝色实心方框 |
| 仅显示 | 仅显示摄像头实时画面 |
| 停止工作 | 停止工作，仅显示乐鑫标志 |

---

## 11. 应用程序开发

### 11.1 必备硬件

- 1 x ESP32-S3-EYE 开发板
- 1 x USB 2.0 数据线（标准 A 型转 Micro-B 型）
- 1 x 电脑（Windows、Linux 或 macOS）

### 11.2 可选硬件

- 外接锂电池（容量 > 1000 mAh，输出 3.7V）
- MicroSD 卡（用于扩展存储）

### 11.3 软件环境搭建

ESP32-S3-EYE 的开发基于 ESP-WHO 框架，该框架构建于 ESP-IDF 之上。

**当前版本要求**：ESP-WHO 需要 ESP-IDF v5.5.x。

#### 步骤 1：安装 ESP-IDF

```bash
# 使用 EIM（ESP-IDF Installation Manager）安装 ESP-IDF v5.5.x
# 选择 Custom installation，选择 ESP-IDF v5.5.x
```

#### 步骤 2：克隆 ESP-WHO 仓库

```bash
git clone https://github.com/espressif/esp-who.git
```

#### 步骤 3：编译人脸识别示例

```bash
# 进入示例目录
cd esp-who/examples/human-face-recognition

# 激活 ESP-IDF 环境
source ~/.espressif/tools/activate_idf_v5.5.4.sh

# 设置板级配置
export IDF_EXTRA_ACTIONS_PATH=<path-to-esp-who>/esp-who/tools/
idf.py -DSDKCONFIG_DEFAULTS=sdkconfig.bsp.esp32_s3_eye set-target esp32s3

# 编译
idf.py build

# 烧录
idf.py -p <PORT> flash

# 监视
idf.py -p <PORT> monitor
```

#### 步骤 4：验证运行

启动后，串口日志将显示摄像头、按键和 LVGL 图形栈的初始化信息：

```
I (1240) cam_hal: cam init ok
I (1254) camera: Detected OV2640 camera
I (1732) LVGL: Starting LVGL task
I (1853) S3-EYE: Setting LCD backlight: 100%
I (2053) button: IoT Button Version: 4.1.5
```

LCD 屏幕上将显示摄像头实时画面，将摄像头对准人脸即可看到面部特征点标记。

---

## 12. ESP-WHO 开发框架

### 12.1 框架概述

ESP-WHO 是乐鑫构建在 ESP 芯片上的图像处理开发平台，为开发计算机视觉应用提供实用的框架，包括：

- **人脸检测（Face Detection）**：检测图像中的人脸位置
- **人脸识别（Face Recognition）**：识别已知人脸的身份
- **行人检测（Pedestrian Detection）**：检测图像中的行人
- **二维码识别（QR Code Recognition）**：识别二维码内容

ESP-WHO 基于 ESP-DL 库进行深度学习推理，并集成了摄像头、显示屏、存储等常用外设。

### 12.2 组件层级架构

ESP-WHO 采用基于 FreeRTOS 的任务驱动架构：

```
[Camera Input] --> [Frame Capture] --> [Detection] --> [Recognition (可选)] --> [Output (LCD/Terminal)]
```

| 组件 | 说明 |
|------|------|
| who_frame_cap | 帧捕获管道，WhoFrameCapNode 为基础处理节点 |
| who_detect | 检测模块，运行 ESP-DL 检测模型（人脸、行人等） |
| who_recognition_core | 识别模块，执行特征提取和匹配，内部状态机管理 ENROLL/RECOGNIZE/DELETE |

### 12.3 BSP 支持

通过板级支持包（BSP）抽象硬件差异，同一应用代码可在不同目标板（如 ESP32-S3-EYE、ESP32-P4-Function-EV-Board）上运行。

ESP32-S3-EYE 的 BSP 能力：

| 能力 | 控制器/编解码器 | 状态 |
|------|-----------------|------|
| 显示屏 | ST7789 | 支持 |
| LVGL 端口 | - | 支持 |
| 触摸 | - | 不支持 |
| 按键 | - | 支持 |
| 音频（麦克风） | - | 支持 |
| SD 卡 | - | 支持 |
| LED | - | 支持 |
| 摄像头 | OV2640 | 支持 |

---

## 13. 应用场景

ESP32-S3-EYE 适用于多种 AIoT 视觉应用：

| 应用场景 | 说明 |
|----------|------|
| 智能门铃 / 猫眼门铃 | 通过人脸识别自动识别访客身份 |
| 安防监控 | 人脸检测、移动侦测、异常报警 |
| 人脸识别考勤终端 | 企业考勤打卡，支持多人脸注册和识别 |
| 二维码识别 | 读取并解析二维码内容 |
| 语音交互设备 | 语音唤醒和命令词控制 |
| Wi-Fi 图传 | 通过 Wi-Fi 实时传输图像数据 |
| 边缘 AI 终端 | 在设备端完成 AI 推理，无需云端 |
| 教育与原型开发 | AI 视觉应用的入门和学习 |

---

## 14. 相关资源与链接

### 官方资源

| 资源 | 链接 |
|------|------|
| ESP-WHO GitHub 仓库 | https://github.com/espressif/esp-who |
| ESP32-S3-EYE 入门指南（中文） | https://github.com/espressif/esp-who/blob/master/docs/zh_CN/get-started/ESP32-S3-EYE_Getting_Started_Guide.md |
| ESP32-S3-EYE 入门指南（英文） | https://github.com/espressif/esp-who/blob/master/docs/en/get-started/ESP32-S3-EYE_Getting_Started_Guide.md |
| ESP-WHO 中文说明 | https://github.com/espressif/esp-who/blob/master/README_CN.md |
| ESP-DL 深度学习库 | https://github.com/espressif/esp-dl |
| ESP32-S3-EYE BSP | https://components.espressif.com/components/espressif/esp32_s3_eye |
| ESP32-S3-EYE 默认固件 | https://github.com/espressif/esp-who/tree/master/default_bin/esp32-s3-eye |
| ESP32-S3-EYE 示例代码 | https://github.com/espressif/esp-who/tree/master/examples/esp32-s3-eye |
| ESP32-S3-WROOM-1 数据手册 | https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_cn.pdf |
| ESP-WHO 入门教程（博客） | https://developer.espressif.com/blog/2026/05/esp-who-get-started/ |

### 购买渠道

- AliExpress：ESP32-S3-EYE
- Digi-Key：ESP32-S3-EYE
- Mouser：ESP32-S3-EYE
- Amazon US：ESP32-S3-EYE
- Robu India：ESP32-S3-EYE

### 技术规格

| 参数 | 值 |
|------|-----|
| 产品型号（MPN） | ESP32-S3-EYE |
| 量产状态 | 量产中（Mass Production） |
| Flash | 8 MB |
| PSRAM | 8 MB |
| 接口 | Camera, LCD screen, Mic, Buttons, TF Card |
| 温度范围 | -40 ~ 85 °C |

---

> **文档版本**: 基于截至 2026 年 06 月的官方文档整理。如需最新信息，请参考乐鑫官方文档。
