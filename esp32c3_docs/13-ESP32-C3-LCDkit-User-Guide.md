# ESP32-C3-LCDkit 用户指南 — 旋钮屏评估开发板

> 文档编号：13
> 适用开发板：ESP32-C3-LCDkit（搭载 ESP32-C3-MINI-1）
> 来源：乐鑫官方 esp-dev-kits 用户指南、BSP 文档、产品页

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [主板组件（LCDkit_MB）](#3-主板组件lcdkit_mb)
4. [LCD 子板（LCDkit_DB）](#4-lcd-子板lcdkit_db)
5. [EC11 旋转编码器](#5-ec11-旋转编码器)
6. [红外与音频](#6-红外与音频)
7. [软件开发与 BSP](#7-软件开发与-bsp)
8. [应用示例](#8-应用示例)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. 开发板概述

ESP32-C3-LCDkit 是一款基于 **ESP32-C3 芯片**和 **SPI 接口显示屏**的评估开发板，不仅通过旋转编码器开关实现屏幕交互，还具有**音频播放**和**红外无线控制**功能。它是乐鑫**物种保护系列**开发板，由主板和子板组成。

由于 ESP32-C3 具有**成本低、功耗低、性能强**的特点，能基本满足 GUI 交互需求，在小尺寸屏幕应用场景中占据优势——是构建 SPI 接口屏方案的理想选择。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C3-MINI-1（4 MB 封装内 flash，400 KB SRAM） |
| 芯片 | ESP32-C3（RISC-V 32 位单核，160 MHz） |
| 无线 | 2.4 GHz Wi-Fi + Bluetooth 5 (LE) |
| 屏幕 | 1.28" SPI LCD，240×240，驱动 IC **GC9A01** |
| 旋转编码器 | EC11（360° 旋转 + 按键） |
| 音频 | 板载功放 + 扬声器 |
| 红外 | IR 发射 + 接收 |
| USB | USB Type-C（供电 + 下载 + JTAG） |

---

## 3. 主板组件（LCDkit_MB）

| 组件 | 说明 |
|------|------|
| **ESP32-C3-MINI-1 模组** | 通用 Wi-Fi + BLE MCU 模组，4 MB flash + 400 KB SRAM |
| UART & RGB 扩展 I/O | 2.54 mm 排针，连系统电源、UART、RGB 数据引脚 |
| RGB LED | 三色 LED，状态指示 |
| 扬声器 | 经音频功放播放 |
| LCD 屏幕连接器 | 2.54 mm 排母，连 1.28" LCD 子板 |
| USB 电源端口 | 供电，建议 ≥5V/2A；USB 通信 |
| Reset 按键 | 系统重置 |
| 5 V to 3.3 V LDO | 低压差线性稳压 |
| 3.3 V 电源指示灯 | 指示供电状态 |
| **EC11 旋转编码器** | 360° 旋转 + 按键，控制屏幕 GUI |
| 音频功放芯片 | 驱动扬声器 |
| 红外接收器 | 接收红外信号 |
| 红外功能选择端口 | 2.54 mm 排针 + 跳线帽，选红外发射或接收 |
| 红外发射器 | 发送红外信号 |

---

## 4. LCD 子板（LCDkit_DB）

- **屏幕**：1.28 英寸，SPI 接口，240×240 分辨率
- **驱动 IC**：**GC9A01**（圆形屏常用）
- 通过 2.54 mm 排母与主板连接
- 支持搭配不同 I2C/SPI 屏幕子板使用

---

## 5. EC11 旋转编码器

- **360° 旋转**：旋转产生正交脉冲，MCU 解析方向与档位
- **按键开关**：按压作为确认/选择
- 用于实现旋钮屏 GUI 的菜单导航与调节

---

## 6. 红外与音频

| 功能 | 说明 |
|------|------|
| 红外发射 | 控制红外家电（如空调、电视） |
| 红外接收 | 学习遥控器信号 |
| 红外功能选择 | 跳线帽切换发射/接收 |
| 音频播放 | 板载功放 + 扬声器，可播报语音提示 |

---

## 7. 软件开发与 BSP

ESP32-C3-LCDkit 基于 **ESP-IDF**（FreeRTOS）开发，含 LCD、ADC、RMT、SPI 等组件。乐鑫提供官方 **BSP**（板级支持包）`espressif/esp32_c3_lcdkit`，封装屏幕、IR、音频等初始化。

```bash
# 添加 BSP 组件
idf.py add-dependency "espressif/esp32_c3_lcdkit"

idf.py set-target esp32c3
idf.py menuconfig   # 配置工程选项
idf.py -p <PORT> flash monitor
```

USB Type-C 单口同时提供 5 V 供电、串口、JTAG 调试与编程。

---

## 8. 应用示例

- **Knob Panel Example**：带语音播放功能的旋钮面板 GUI 演示，为开发类似应用提供参考
- 示例仓库：`esp-dev-kits/examples/esp32-c3-lcdkit`

---

## 9. 应用场景

ESP32-C3 成本优势 + 低功耗 + SPI 屏，适合**小家电旋钮屏与小尺寸显示屏**：

- 洗衣机
- 体脂秤
- 电动牙刷
- 智能旋钮面板
- 1.28" 圆形屏交互设备

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3-LCDkit 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-lcdkit/user_guide.html |
| LCDkit 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-lcdkit/index.html |
| BSP esp32_c3_lcdkit（Component Registry） | https://components.espressif.com/components/espressif/esp32_c3_lcdkit |
| BSP README（GitHub） | https://github.com/espressif/esp-bsp/blob/master/bsp/esp32_c3_lcdkit/README.md |
| 应用示例（knob_panel） | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-c3-lcdkit |
| LCDkit 产品页 | https://www.espressif.com/zh-hans/products/devkits?id=ESP32-C3 |
| LCD 屏选型博客（SPI+LCD） | https://developer.espressif.com/blog/espressif-socs-lcd-screens/ |

---

> **文档总结**：ESP32-C3-LCDkit 是 ESP32-C3 的旋钮屏评估开发板（物种保护系列），主板+子板结构。搭载 ESP32-C3-MINI-1（4 MB flash），子板为 1.28" 240×240 SPI 圆形屏（GC9A01）。集成 EC11 旋转编码器（旋钮+按键）、扬声器+功放、红外收发，USB-C 单口供电/下载/JTAG。官方 BSP `esp32_c3_lcdkit` 一键集成，提供 Knob Panel 旋钮面板示例。凭借 C3 的成本与低功耗优势，是洗衣机、体脂秤、电动牙刷等小家电旋钮屏与小尺寸显示屏应用的理想参考平台。
