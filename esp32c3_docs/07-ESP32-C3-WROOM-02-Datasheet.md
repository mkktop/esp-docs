# ESP32-C3-WROOM-02 / WROOM-02U 模组数据手册

> 文档编号：07
> 适用模组：ESP32-C3-WROOM-02（PCB 天线）、ESP32-C3-WROOM-02U（外接天线座子）
> 来源：乐鑫官方《ESP32-C3-WROOM-02 & WROOM-02U 技术规格书》、产品页

---

## 目录

1. [模组概述](#1-模组概述)
2. [核心规格](#2-核心规格)
3. [WROOM-02 vs WROOM-02U](#3-wroom-02-vs-wroom-02u)
4. [尺寸与封装](#4-尺寸与封装)
5. [引脚与 strapping](#5-引脚与-strapping)
6. [与 ESP-WROOM-02 的兼容性](#6-与-esp-wroom-02-的兼容性)
7. [对应开发板](#7-对应开发板)
8. [应用场景](#8-应用场景)
9. [参考资源](#9-参考资源)

---

## 1. 模组概述

ESP32-C3-WROOM-02 / WROOM-02U 是乐鑫基于 **ESP32-C3** 芯片的通用型 Wi-Fi + 低功耗蓝牙（BLE）模组，功能强大、外设接口丰富。它与经典 **ESP-WROOM-02 / 02D** 模组**引脚兼容**，便于老产品从 ESP8266 平滑迁移到 ESP32-C3。

- **ESP32-C3-WROOM-02**：PCB 板载天线
- **ESP32-C3-WROOM-02U**：U.FL 外部天线座子

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 集成芯片 | ESP32-C3 |
| 内核 | RISC-V 32 位单核，160 MHz |
| 无线 | 2.4 GHz Wi-Fi (802.11b/g/n) + Bluetooth 5 (LE) |
| Flash | 4 MB（外部 SPI flash） |
| RAM | 400 KB SRAM |
| 工作电压 | 3.0 ~ 3.6 V（推荐 3.3 V） |
| GPIO | 19 |

---

## 3. WROOM-02 vs WROOM-02U

| 项目 | WROOM-02 | WROOM-02U |
|------|----------|-----------|
| 天线 | PCB 板载天线 | U.FL 座子接外部 IPEX 天线 |
| 尺寸 | 18×20×3.2 mm | 18×14.3×3.2 mm（无 PCB 天线，更短） |
| 芯片/Flash/RAM | 相同 | 相同 |
| 适用 | 常规应用 | 金属外壳/远距离/外接天线 |

---

## 4. 尺寸与封装

- **WROOM-02 尺寸**：18 × 20 × 3.2 mm
- **WROOM-02U 尺寸**：18 × 14.3 × 3.2 mm
- 贴片设计，提供 PCB Footprint（dxf）、3D 模型（STEP / glb）

---

## 5. 引脚与 strapping

- 模组提供 **19 个 GPIO**
- **3 个 strapping 管脚**：GPIO2、GPIO8、GPIO9
  - GPIO9 弱上拉（默认 1），控制 SPI Boot / Download Boot
  - GPIO2、GPIO8 默认浮空
- 时序：tSU ≥ 0 ms，tH ≥ 3 ms
- 启动模式：SPI Boot（GPIO9=1）/ Joint Download Boot（GPIO9=0, GPIO8=1）
- ⚠️ GPIO8 与 GPIO9 不可同时为低

---

## 6. 与 ESP-WROOM-02 的兼容性

ESP32-C3-WROOM-02 **Pin 脚兼容**经典 ESP-WROOM-02 / 02D（ESP8266 系），方便从 ESP8266 产品线迁移到 ESP32-C3（获得 RISC-V 内核 + BLE 5 + 更强安全）。

> 注意：虽然引脚兼容，但软件从 ESP8266 RTOS SDK / Arduino-ESP8266 迁移到 ESP-IDF / Arduino-ESP32 需要适配。

---

## 7. 对应开发板

ESP32-C3-WROOM-02 模组被以下开发板采用：

| 开发板 | 说明 |
|--------|------|
| **ESP32-C3-DevKitC-02** | 入门级开发板，搭载 WROOM-02，引脚全引出 |
| **ESP32-C3-Lyra** | 音频灯控开发板（ESP-ADF），麦克风+扬声器+LED 灯带+红外 |

---

## 8. 应用场景

- 智能家居、工业自动化、医疗保健、消费电子
- 从 ESP8266/ESP-WROOM-02 升级的传统产品
- 需要外接天线的设备（WROOM-02U）
- 智能照明、音频播报、灯控律动（如 C3-Lyra）

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3-WROOM-02 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c3-wroom-02_datasheet_cn.pdf |
| WROOM-02 规格书（在线） | https://documentation.espressif.com/esp32-c3-wroom-02_datasheet_cn.html |
| WROOM-02 PCB Footprint（dxf） | https://www.espressif.com/sites/default/files/modules-dxf/ESP32-C3-WROOM-02%20PCB%20Footprint.dxf |
| WROOM-02U PCB Footprint（dxf） | https://www.espressif.com/sites/default/files/modules-dxf/ESP32-C3-WROOM-02U%20PCB%20Footprint.dxf |
| WROOM-02 3D 模型（STEP） | https://www.espressif.com/sites/default/files/3dmodel/ESP32-C3-WROOM-02%203D%20Model.STEP |
| 模组产品页 | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C3 |

---

> **文档总结**：ESP32-C3-WROOM-02 / WROOM-02U 是 ESP32-C3 的通用 Wi-Fi + BLE 模组，19 GPIO、4 MB 外部 flash。WROOM-02 用 PCB 天线（18×20 mm），WROOM-02U 用 U.FL 外接天线座（18×14.3 mm）。它与经典 ESP-WROOM-02/02D 引脚兼容，便于 ESP8266 产品线升级到 ESP32-C3。被 DevKitC-02、C3-Lyra 等开发板采用。
