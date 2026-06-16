# ESP32-S3-LCD-EV-Board 用户指南 — LCD 评估板

> **文档来源**: 基于乐鑫官方 Espressif ESP-Dev-Kits 文档、BSP 组件文档及 ESP32-S3-LCD-EV-Board 用户指南（v1.5）整理。

---

## 目录

- [1. 产品概述](#1-产品概述)
- [2. 硬件架构与核心规格](#2-硬件架构与核心规格)
- [3. LCD 子板与屏幕面板](#3-lcd-子板与屏幕面板)
- [4. 触摸屏](#4-触摸屏)
- [5. 主板组件详解](#5-主板组件详解)
- [6. 音频系统](#6-音频系统)
- [7. 快速入门](#7-快速入门)
- [8. 应用程序开发](#8-应用程序开发)
- [9. 示例项目](#9-示例项目)
- [10. 应用场景](#10-应用场景)
- [11. 相关资源与链接](#11-相关资源与链接)

---

## 1. 产品概述

ESP32-S3-LCD-EV-Board 是乐鑫（Espressif）推出的一款基于 ESP32-S3 芯片的屏幕交互开发板，专门用于评估和验证 ESP32-S3 屏幕应用。通过搭配不同类型的 LCD 子板，可以驱动 IIC、SPI、8080 以及 RGB 接口的 LCD 显示屏，满足用户对多种不同分辨率以及接口的触摸屏应用产品的开发需求。

同时，开发板还搭载双麦克风阵列，支持语音识别和近/远场语音唤醒，具有触摸屏交互和语音交互功能，是一款集视觉与语音于一体的 HMI（人机交互）评估平台。

### 产品型号

除非另有说明，本文中的 ESP32-S3-LCD-EV-Board 同时指以下两款开发板：

| 型号 | 配置 | 在售子板 |
|------|------|----------|
| **ESP32-S3-LCD-EV-Board** | 搭配 3.95" 480 x 480 RGB LCD | SUB2 |
| **ESP32-S3-LCD-EV-Board-2** | 搭配 4.3" 800 x 480 RGB LCD | SUB3 |

### 核心特性

- **嵌入式模组**：板载 ESP32-S3-WROOM-1 模组，内置 16 MB Flash + 16 MB Octal PSRAM（v1.5 版本）
- **多接口屏幕支持**：支持 RGB、8080 并口、SPI、I2C 接口屏幕
- **音频系统**：板载音频编解码器（Codec）+ 音频功放，支持双麦克风拾音
- **USB 下载调试**：板载 USB 转串口芯片，支持 USB Type-C 接口下载和调试
- **触摸交互**：配备电容式触摸屏
- **语音交互**：支持离线语音唤醒和识别

---

## 2. 硬件架构与核心规格

### 2.1 核心芯片

| 参数 | 规格 |
|------|------|
| SoC | ESP32-S3 |
| 模组 | ESP32-S3-WROOM-1-N16R16V（v1.5）/ N16R8（v1.4） |
| 处理器 | 240 MHz Xtensa 32-bit LX7 双核 |
| 内置 SRAM | 512 KB |
| Wi-Fi | 802.11 b/g/n |
| 蓝牙 | Bluetooth 5 (LE) |
| AI 加速 | 专用向量指令，支持神经网络加速和信号处理 |

### 2.2 存储配置

| 版本 | Flash | PSRAM |
|------|-------|-------|
| v1.5 | 16 MB Quad Flash（1.8V） | 16 MB Octal PSRAM（1.8V，8 线） |
| v1.4 | 16 MB Quad Flash（1.8V） | 8 MB Octal PSRAM（1.8V） |

### 2.3 整体规格

| 参数 | 规格 |
|------|------|
| 尺寸 | 123 x 100 x 20 mm |
| 工作温度 | -40 ~ 85 °C |
| 参考价格 | 约 59 USD / 369 CNY |
| 量产状态 | 量产中（Mass Production） |

### 2.4 功能框图

ESP32-S3-LCD-EV-Board 的主要组件和连接关系：

```
              +-----------------------+
              |    ESP32-S3-WROOM-1   |
              |   16MB Flash          |
              |   16MB Octal PSRAM    |
              +----------+------------+
                         |
         +---------------+---------------+
         |               |               |
   +-----+-----+   +-----+-----+   +----+------+
   |  LCD 子板  |   |  音频系统  |   |  USB      |
   |  连接器    |   |           |   |  接口     |
   | (2.54mm)   |   | ES8311   |   | Type-C    |
   |            |   | ES7210   |   | CP2102N   |
   | RGB/8080/  |   | NS4150   |   |           |
   | SPI/I2C    |   |           |   +-----------+
   +-----+------+   +-----+-----+
         |                |
   +-----+------+   +----+------+
   | 触摸屏     |   | 双麦克风  |
   | FT5x06/    |   | 阵列     |
   | GT911/     |   | (Left+   |
   | GT1151     |   |  Right)  |
   +------------+   +-----------+
```

---

## 3. LCD 子板与屏幕面板

主板可搭配以下三种不同类型的 LCD 子板使用。通过更换子板，可方便地接入不同接口和分辨率的屏幕。

### 3.1 LCD 子板总览

| 子板名称 | 屏幕尺寸（英寸） | 分辨率 | LCD 驱动芯片（接口） | 触摸驱动芯片 | BSP 支持 |
|----------|-----------------|--------|----------------------|-------------|----------|
| ESP32-S3-LCD-EV-Board-SUB1 | 0.96 | 128 x 64 | SSD1315 (I2C) | 无 | 暂不支持 |
| | 2.40 | 320 x 240 | ST7789V (SPI) | XPT2046 | 暂不支持 |
| ESP32-S3-LCD-EV-Board-SUB2 | 3.50 | 480 x 320 | ST7796S (8080) | GT911 | 暂不支持 |
| | **3.95** | **480 x 480** | **GC9503CV (RGB)** | **FT5x06** | **支持** |
| ESP32-S3-LCD-EV-Board-SUB3 | **4.30** | **800 x 480** | **ST7262E43 (RGB)** | **GT1151** | **支持** |

### 3.2 SUB1 子板

ESP32-S3-LCD-EV-Board-SUB1 提供了两种屏幕接口：

- **0.96 英寸 I2C 接口屏**：SSD1315 驱动芯片，128 x 64 分辨率，OLED 单色屏
- **2.4 英寸 SPI 接口屏**：ST7789V 驱动芯片，320 x 240 分辨率，XPT2046 电阻触摸

> 注意：该子板暂未做 BSP 适配。

### 3.3 SUB2 子板（出厂搭载于 ESP32-S3-LCD-EV-Board）

ESP32-S3-LCD-EV-Board-SUB2 提供了两种屏幕接口：

- **3.95 英寸 RGB 接口触摸屏**（出厂贴装）：GC9503CV 驱动芯片，480 x 480 分辨率，RGB565 接口，FT5x06 电容触摸
- **3.5 英寸 8080 并口屏**（可选）：ST7796S 驱动芯片，480 x 320 分辨率，GT911 电容触摸

#### SUB2 关键参数（3.95 英寸 RGB 屏）

| 参数 | 规格 |
|------|------|
| 屏幕尺寸 | 3.95 英寸 |
| 分辨率 | 480 x 480 |
| 接口 | RGB565 |
| LCD 驱动 IC | GC9503CV |
| 刷新率 | 60 Hz |
| 触摸驱动 IC | FT5x06 |
| 触摸类型 | 电容式多点触摸 |

### 3.4 SUB3 子板（出厂搭载于 ESP32-S3-LCD-EV-Board-2）

ESP32-S3-LCD-EV-Board-SUB3 仅支持 4.3 英寸 RGB 接口触摸屏。

#### SUB3 关键参数

| 参数 | 规格 |
|------|------|
| 屏幕尺寸 | 4.3 英寸 |
| 分辨率 | 800 x 480 |
| 接口 | RGB565 |
| LCD 驱动 IC | ST7262E43 |
| 像素时钟 | 18 MHz |
| 刷新率 | 35 Hz |
| 触摸驱动 IC | GT1151 |
| 触摸类型 | 电容式多点触摸 |

#### SUB3 RGB 时序参数

```c
{
    .pclk_hz = 18 * 1000 * 1000,
    .h_res = 800,
    .v_res = 480,
    .hsync_pulse_width = 40,
    .hsync_back_porch = 40,
    .hsync_front_porch = 48,
    .vsync_pulse_width = 23,
    .vsync_back_porch = 32,
    .vsync_front_porch = 13,
    .flags.pclk_active_neg = true,
}
```

---

## 4. 触摸屏

ESP32-S3-LCD-EV-Board 的 LCD 子板均配备电容式触摸面板，实现流畅的人机交互。

| 子板 | 触摸驱动 IC | 接口 | 触摸类型 |
|------|------------|------|----------|
| SUB2 (3.95") | FT5x06 | I2C | 电容式多点触摸 |
| SUB3 (4.3") | GT1151 | I2C | 电容式多点触摸 |

触摸屏配合 LVGL 图形库使用，可以构建丰富的交互式图形用户界面（GUI），支持点击、滑动、长按、缩放等手势操作。

---

## 5. 主板组件详解

**ESP32-S3-LCD-EV-Board-MB** 主板是整个套件的核心，集成了 ESP32-S3-WROOM-1 模组，并提供与 LCD 子板连接的端口。以下按逆时针顺序介绍主要组件：

| 组件 | 说明 |
|------|------|
| **ESP32-S3-WROOM-1-N16R16V 模组** | 通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，搭载 ESP32-S3 芯片，内置 16 MB Flash 和 16 MB PSRAM。具有强大的神经网络运算能力和信号处理能力 |
| **Reset 按键** | 单独按下重置系统 |
| **Boot 按键** | 长按 Boot 键再按 Reset 键可进入固件上传模式，通过串口或 USB 上传固件 |
| **扩展连接器** | 连接所有 IO 扩展芯片管脚、系统电源管脚及部分模组管脚 |
| **I/O 扩展芯片** | TCA9554，8 位通用并行 I/O 扩展芯片，通过 I2C 通信控制 IO 口模式和输出电平 |
| **LCD 子板连接器** | 2.54 mm 间距连接器，可连接三种不同类型的 LCD 子板 |
| **LED** | RGB 三色 LED，可配置状态行为指示 |
| **USB-to-USB 端口** | 为系统供电（与 USB-to-UART 端口二选一），建议使用至少 5V/2A 电源适配器。用于 PC 与 ESP32-S3-WROOM-1 的 USB 通信 |
| **USB-to-UART 端口** | 为系统供电（与 USB-to-USB 端口二选一），用于 PC 与模组的串口通信 |
| **左侧麦克风** | 板载麦克风，连接至音频模数转换器 |
| **右侧麦克风** | 板载麦克风，连接至音频模数转换器 |
| **音频模数转换器** | ES7210，高性能低功耗 4 通道音频 ADC，支持声学回声消除（AEC） |
| **USB-to-UART 桥接器** | CP2102N，提供高达 3 Mbps 的传输速率 |
| **音频编解码芯片** | ES8311，低功耗单声道音频编解码器（ADC + DAC + 前置放大器 + 耳机驱动器） |
| **音频功率放大器** | NS4150，低 EMI、3W 单声道 D 类音频功放 |
| **扬声器连接器** | 通过音频功放驱动外部扬声器 |

---

## 6. 音频系统

ESP32-S3-LCD-EV-Board 配备完整的音频输入输出系统，支持语音交互。

### 6.1 音频输入（录音）

| 组件 | 型号 | 说明 |
|------|------|------|
| 麦克风阵列 | 双板载麦克风 | 左右两个麦克风，用于远场语音拾音 |
| 音频 ADC | ES7210 | 4 通道高性能音频模数转换器，支持声学回声消除（AEC），非常适合音乐和语音应用 |

### 6.2 音频输出（播放）

| 组件 | 型号 | 说明 |
|------|------|------|
| 音频编解码器 | ES8311 | 低功耗单声道音频编解码器，包含 ADC、DAC、低噪声前置放大器、耳机驱动器、数字音效等 |
| 音频功率放大器 | NS4150 | 低 EMI、3W 单声道 D 类音频功放 |
| 扬声器连接器 | - | 连接外部扬声器（可选配件） |

### 6.3 音频连接总线

音频芯片通过以下总线与 ESP32-S3-WROOM-1 模组连接：

- **I2S 总线**：用于音频数据传输
- **I2C 总线**：用于音频芯片的配置和控制

---

## 7. 快速入门

### 7.1 版本确认

请查看主板 ESP32-S3-LCD-EV-Board-MB 背面的丝印版本号，以确认开发板版本：

- **v1.5 版本**：配备 ESP32-S3-WROOM-1-N16R16V 模组（16 MB Flash + 16 MB PSRAM）
- **v1.4 及以下版本**：配备 ESP32-S3-WROOM-1-N16R8 模组（16 MB Flash + 8 MB PSRAM）

### 7.2 必备硬件

- 1 x ESP32-S3-LCD-EV-Board-MB（主板）
- 1 x LCD 子板（SUB2 或 SUB3）
- 1 x USB 2.0 数据线（标准 A 型转 Type-C 型）
- 1 x 电脑（Windows、Linux 或 macOS）

> 注意：请确保使用适当的 USB 数据线。部分数据线仅可用于充电，无法用于数据传输和程序烧录。

### 7.3 可选硬件

- 1 x 扬声器（连接扬声器连接器）

### 7.4 硬件设置

1. **连接 LCD 子板**：将 LCD 子板对准主板上的 LCD 子板连接器端口，轻轻按压使其牢固连接。
2. **连接 USB 数据线**：将 USB 数据线分别连接 PC 与开发板的 USB 端口之一（USB-to-USB 或 USB-to-UART）。
3. **上电**：LCD 屏幕亮起后，可以用手指与触摸屏进行交互。

> 供电建议：使用至少 5V/2A 电源适配器供电，以保证供电稳定。USB-to-USB 端口和 USB-to-UART 端口两者选一即可。

---

## 8. 应用程序开发

### 8.1 开发框架

ESP32-S3-LCD-EV-Board 的开发框架为 **ESP-IDF**，基于 FreeRTOS 的乐鑫 SoC 开发框架，具有众多组件（LCD、ADC、RMT、SPI 等）。

- **ESP-IDF 版本要求**：v5.0.1 及以上，推荐使用最新的 release/v5.1 分支
- **LCD 应用开发指南**：参考 ESP-IoT-Solution 编程指南中的 LCD 部分

### 8.2 环境搭建

了解如何快速设置开发环境，请前往 ESP-IDF 快速入门指南 > 安装章节。

### 8.3 编译与烧录

```bash
# 设置目标芯片
idf.py set-target esp32s3

# 配置 BSP（在 menuconfig 中选择对应的 LCD 子板）
idf.py menuconfig
# 导航到 Application Configuration -> Select BSP
# 选择 ESP32-S3-LCD-EV-Board 及对应子板

# 编译项目
idf.py build

# 烧录固件
idf.py -p <PORT> flash

# 监视串口输出
idf.py -p <PORT> monitor
```

### 8.4 PSRAM 120M DDR 配置

为了获得 RGB LCD 的最佳性能，可启用 PSRAM 120M DDR 功能。详细配置方法请参考 ESP-FAQ LCD 章节。

> **重要提示**：v1.5 版本的开发板配备 ESP32-S3-WROOM-1-N16R16V 模组，目前不支持 PSRAM 120M DDR 功能。如果启用，可能会出现芯片上电后冻结然后复位的问题。

### 8.5 BSP 组件

乐鑫为 ESP32-S3-LCD-EV-Board 提供了官方 BSP，可通过 ESP Component Registry 获取：

| 组件 | 名称 | 说明 |
|------|------|------|
| 完整 BSP | `espressif/esp32_s3_lcd_ev_board` | 包含 LVGL 集成 |
| 精简 BSP | `espressif/esp32_s3_lcd_ev_board_noglib` | 不包含 LVGL |

BSP 支持的能力：

| 能力 | 控制器/编解码器 | 状态 |
|------|-----------------|------|
| 显示屏 | - | 支持 |
| 触摸 | - | 支持 |
| 按键 | - | 支持 |
| 音频（扬声器） | ES8311 | 支持 |
| 音频（麦克风） | ES7210 | 支持 |
| LVGL 端口 | - | 支持 |

---

## 9. 示例项目

ESP-Dev-Kits 仓库为 ESP32-S3-LCD-EV-Board 提供了丰富的示例项目，可在 ESP-IDF release/v5.1 分支下直接编译烧录：

| 示例名称 | 说明 |
|----------|------|
| **86-Box Demo** | 86 型智能面板演示 |
| **86-Box Smart Panel** | 86 型智能面板完整应用 |
| **Smart Panel** | 智能家居中控面板 |
| **LVGL Demos** | LVGL 图形库示例集合 |
| **USB Keyboard** | USB 键盘功能演示 |
| **USB Camera** | USB 摄像头画面显示 |
| **USB File System** | USB 文件系统演示 |

### 出厂固件

| 开发板 | 子板 | 固件 | 说明 |
|--------|------|------|------|
| ESP32-S3-LCD-EV-Board | SUB2 (480x480) | 86-Box Smart Panel | 86 型智能面板示例 |
| ESP32-S3-LCD-EV-Board-2 | SUB3 (800x480) | Smart Panel | 智能家居面板示例 |

可通过 ESP Launchpad 在线一键烧录示例固件。

---

## 10. 应用场景

ESP32-S3-LCD-EV-Board 适用于多种屏幕交互产品的原型构建：

| 应用场景 | 说明 |
|----------|------|
| **智能中控屏** | 智能家居中央控制屏幕，集成设备控制和状态显示 |
| **86 型开关面板** | 替代传统墙面开关的智能触摸面板 |
| **家电控制面板** | 洗衣机、烤箱、空调等家电的智能控制面板 |
| **电动车仪表盘** | 电动自行车或电动滑板车的数据显示仪表盘 |
| **可视门铃** | 带触摸屏的可视对讲门铃系统 |
| **工业 HMI** | 工业设备的触摸屏人机交互界面 |
| **物联网网关** | 带 GUI 显示的物联网网关设备 |
| **智能音箱** | 带屏幕的智能语音助手设备 |

---

## 11. 相关资源与链接

### 官方文档

| 资源 | 链接 |
|------|------|
| ESP32-S3-LCD-EV-Board 用户指南 v1.5（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-lcd-ev-board/user_guide.html |
| ESP32-S3-LCD-EV-Board 用户指南 v1.4（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-lcd-ev-board/user_guide_v1.4.html |
| ESP32-S3-LCD-EV-Board 总览页面 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-lcd-ev-board/index.html |
| 用户指南（英文） | https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32s3/esp32-s3-lcd-ev-board/user_guide.html |
| BSP 组件 | https://components.espressif.com/components/espressif/esp32_s3_lcd_ev_board |
| BSP API 文档 | https://github.com/espressif/esp-bsp/blob/master/bsp/esp32_s3_lcd_ev_board/API.md |

### 原理图与硬件资源

| 子板 | 原理图链接 |
|------|-----------|
| SUB1 | SCH_ESP32-S3-LCD-Ev-Board-SUB1_V1.0 |
| SUB2 | SCH_ESP32-S3-LCD-EV-Board-SUB2_V1.2/V1.5 |
| SUB3 | SCH_ESP32-S3-LCD-EV-Board-SUB3_V1.1/V1.3 |

### 示例代码与开发资源

| 资源 | 链接 |
|------|------|
| 示例代码仓库 | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-s3-lcd-ev-board |
| ESP-IDF 快速入门 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html |
| ESP-IoT-Solution LCD 编程指南 | https://docs.espressif.com/projects/esp-iot-solution/zh_CN/latest/display/lcd/index.html |
| ESP-FAQ LCD 常见问题 | https://docs.espressif.com/projects/esp-faq/en/latest/software-framework/peripherals/lcd.html |
| ESP Launchpad（在线烧录） | https://espressif.github.io/esp-launchpad/ |

### 购买渠道

- 淘宝：ESP32-S3-LCD-EV-Board / ESP32-S3-LCD-EV-Board-2
- AliExpress：ESP32-S3-LCD-EV-Board
- Digi-Key：ESP32-S3-LCD-EV-BOARD
- Mouser：ESP32-S3-LCD-EV-Board
- Amazon US：ESP32-S3-LCD-EV-Board / ESP32-S3-LCD-EV-Board-2
- Robu India：ESP32-S3-LCD-EV-Board

### 技术规格

| 参数 | 值 |
|------|-----|
| 产品型号（MPN） | ESP32-S3-LCD-EV-Board |
| 量产状态 | 量产中（Mass Production） |
| Flash | 16 MB |
| PSRAM | 8 MB（v1.4）/ 16 MB（v1.5） |
| 接口 | LCD Screen, Microphones, Speaker, LED, Buttons |
| 通信接口 | I/O, USB-C, UART |
| 模组 | ESP32-S3-WROOM-1 |

---

> **文档版本**: 基于截至 2026 年 06 月的官方文档整理。如需最新信息，请参考乐鑫官方文档。
