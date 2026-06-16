# ESP32-C3-Lyra 用户指南 — 音频灯控开发板

> 文档编号：12
> 适用开发板：ESP32-C3-Lyra V2.0（搭载 ESP32-C3-WROOM-02）
> 来源：乐鑫官方 ESP-ADF 用户指南、产品页

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [板载组件详解](#3-板载组件详解)
4. [电源系统](#4-电源系统)
5. [音频系统](#5-音频系统)
6. [LED 灯带控制](#6-led-灯带控制)
7. [红外控制](#7-红外控制)
8. [功能按键](#8-功能按键)
9. [软件开发（ESP-ADF）](#9-软件开发esp-adf)
10. [应用场景](#10-应用场景)
11. [参考资源](#11-参考资源)

---

## 1. 开发板概述

ESP32-C3-Lyra 是乐鑫基于 **ESP32-C3** 推出的**音频灯控开发板**，对麦克风、扬声器以及 LED 灯带的控制，可满足客户对**超高性价比的音频播报机**以及**律动灯带**的产品开发需求。它属于 ESP-ADF（音频开发框架）生态开发板。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C3-WROOM-02（4 MB 外部 SPI flash，PCB 天线） |
| 芯片 | ESP32-C3（RISC-V 32 位单核，160 MHz） |
| 无线 | 2.4 GHz Wi-Fi + Bluetooth 5 (LE) |
| 音频 | ECM 麦克风 + NS4150 功放 + 扬声器接口 |
| LED | 可寻址灯带接口 + RGB 灯带接口 |
| 红外 | IR 发射 + 接收 |
| 按键 | 6 功能键 + Boot + Reset |
| USB | 2 个（电源 + USB 转 UART） |
| 电源 | 5 V USB 或 12 V DC |

---

## 3. 板载组件详解

| 组件 | 说明 |
|------|------|
| **ESP32-C3-WROOM-02 模组** | 通用 Wi-Fi + BLE 模组，4 MB 外部 SPI flash，PCB 天线（02U 兼容，需外接天线） |
| **扬声器功放 NS4150** | EMI、3 W 单声道 D 类功放，放大 ESP32-C3 PDM_TX 音频驱动扬声器 |
| **扬声器输出端口** | 推荐 4 Ω、3 W 扬声器，2.00 mm 间距 |
| **6 个功能按键** | MODE、COLOR、PLAY/PAUSE、SET、VOL+/LM+、VOL-/LM- |
| **Boot/Reset 按键** | Boot + Reset 进入固件上传；Reset 单按复位 |
| **USB-to-UART 端口** | PC 与模组通信 |
| **USB-to-UART 桥接 CP2102N** | 软件下载与调试，3 Mbps |
| **USB 电源端口** | 5 V 系统供电，建议 ≥5 V/2 A |
| **系统电源开关** | ON/OFF 控制 5 V 系统电源 |
| **LED 灯带电源选择开关** | 选 USB 5 V 或 12 V 直流供电 |
| **12 V DC 端口** | 最大 2 A，5.5/2.5 mm 插孔 |
| **12V→5V 降压 MP2313** | 1 A、2 MHz 高效同步降压 |
| **可寻址 LED 灯带端口** | 4×1P 2.54 mm，支持 WS2811/WS2812，5 V/12 V，RMT 或 SPI 控制 |
| **RGB LED 灯带端口** | 4×1P 2.54 mm，常规 RGB 灯带（不可寻址），PWM 控制 |
| **系统电源 LED** | 电源开关 ON 时红色 |
| **红外接收 IRM-H638T/TR2** | 微型贴片红外接收器，解调输出由 C3 解码 |
| **红外发射 IR67-21C/TR8** | 红外发光二极管 |
| **麦克风** | 板载 ECM 麦克风，经晶体管放大送 ADC |
| **系统 LED（WS2812C）** | RGB 灯，由 GPIO 控制，指示音频应用状态 |

---

## 4. 电源系统

| 供电方式 | 说明 |
|----------|------|
| **5 V USB** | USB 电源端口，建议 ≥5 V/2 A |
| **12 V DC** | 直流适配器，最大 2 A，经 MP2313 降到 5 V |

- LED 灯带电源选择开关：根据灯带电压（5V/12V）选择 USB 5V 或 12V 直流
- 系统电源开关：ON/OFF 5 V 系统电源

---

## 5. 音频系统

- **输入**：ECM 麦克风 → 晶体管放大 → ESP32-C3 ADC
- **输出**：ESP32-C3 PDM_TX → NS4150 D 类功放 → 扬声器（4 Ω 3 W）
- 用途：音频播报机、语音提示、律动检测

---

## 6. LED 灯带控制

| 灯带类型 | 接口 | 控制方式 |
|----------|------|----------|
| **可寻址 LED 灯带**（WS2811/WS2812） | 4×1P 2.54 mm | 单线控制，RMT 或 SPI 发命令 |
| **RGB LED 灯带**（常规，不可寻址） | 4×1P 2.54 mm | 各颜色独立线路，PWM 控制 |

支持 5 V 与 12 V 两种灯带。

---

## 7. 红外控制

- **红外发射器**（IR67-21C/TR8）：向外界发送红外信号（控制家电）
- **红外接收器**（IRM-H638T/TR2）：接收并解调红外信号，由 C3 解码（学习遥控器）

---

## 8. 功能按键

| 按键 | 功能 |
|------|------|
| MODE | 模式切换 |
| COLOR | 颜色切换 |
| PLAY/PAUSE | 播放/暂停 |
| SET | 设置 |
| VOL+/LM+ | 音量+/律动模式+ |
| VOL-/LM- | 音量-/律动模式- |
| Boot | 下载模式 |
| Reset | 系统复位 |

---

## 9. 软件开发（ESP-ADF）

ESP32-C3-Lyra 基于 **ESP-ADF**（乐鑫音频开发框架）开发，提供音频播放、录音、处理、蓝牙音箱等能力。

```bash
idf.py set-target esp32c3
idf.py -p <PORT> flash monitor
```

示例涵盖：音频播报、麦克风律动检测、LED 灯带节拍同步、红外学习等。

---

## 10. 应用场景

- 超高性价比**音频播报机**（提示音、语音播报）
- **律动灯带**（音乐节奏同步灯效）
- 智能照明控制（可寻址/RGB 灯带）
- 红外家电遥控（学习型遥控器）
- 小家电音频 + 灯效一体化方案

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3-Lyra V2.0 用户指南（中文） | https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-c3-lyra-v2.0.html |
| ESP32-C3-Lyra 用户指南 | https://docs.espressif.com/projects/esp-adf/zh_CN/latest/design-guide/dev-boards/user-guide-esp32-c3-lyra.html |
| Lyra 产品页 | https://www.espressif.com/zh-hans/products/devkits?id=ESP32-C3 |
| ESP-ADF GitHub | https://github.com/espressif/esp-adf |
| C3-Lyra V2.0 用户指南 PDF | https://www.espressif.com/zh-hans/support/download/documents/development-board |

---

> **文档总结**：ESP32-C3-Lyra 是 ESP32-C3 的音频灯控开发板（ESP-ADF 生态），搭载 WROOM-02 模组。集成 ECM 麦克风、NS4150 3W 功放、扬声器接口、可寻址（WS2812）与 RGB 双灯带接口、红外收发、6 个功能按键，支持 5V USB 或 12V DC 供电。是超高性价比音频播报机、律动灯带、智能照明与红外遥控一体化方案的开发与量产参考，适合小家电音频+灯效场景。
