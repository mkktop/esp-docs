# ESP8685-WROOM 系列模组数据手册

> 文档编号：08
> 适用模组：ESP8685-WROOM-01 / 03 / 04 / 05 / 06 / 07（均基于 ESP8685H4）
> 来源：乐鑫官方 ESP8685-WROOM 各模组规格书、产品页

---

## 目录

1. [模组系列概述](#1-模组系列概述)
2. [核心规格（共性）](#2-核心规格共性)
3. [六款模组对比](#3-六款模组对比)
4. [选型建议](#4-选型建议)
5. [封装资源](#5-封装资源)
6. [应用场景](#6-应用场景)
7. [参考资源](#7-参考资源)

---

## 1. 模组系列概述

ESP8685-WROOM 是乐鑫基于 **ESP8685H4**（ESP32-C3 小封装衍生，QFN 4×4）芯片的模组系列，提供 **01~07 共 6 款不同形态**，覆盖贴片、竖插、超小尺寸、不同 GPIO 数与天线类型，满足各类装配与尺寸需求。

所有 WROOM 模组：
- 集成芯片：ESP8685H4
- Flash：2 或 4 MB（封装内）
- 无 PSRAM
- PCB 天线（07 除外，为单极子外接天线）

---

## 2. 核心规格（共性）

| 参数 | 规格 |
|------|------|
| 集成芯片 | ESP8685H4（QFN 4×4，15 GPIO） |
| 内核 | RISC-V 32 位单核，160 MHz |
| 无线 | 2.4 GHz Wi-Fi (802.11b/g/n) + Bluetooth 5 (LE) |
| Flash | 2 或 4 MB（封装内） |
| RAM | 400 KB SRAM |
| 工作电压 | 3.0 ~ 3.6 V（推荐 3.3 V） |
| 外设 | UART、SPI、I2C、LED PWM、I2S、RMT 等 |

---

## 3. 六款模组对比

| 模组 | 形态 | 可用 GPIO | 尺寸 (mm) | Flash | 天线 |
|------|------|-----------|-----------|-------|------|
| **WROOM-01** | 贴片 + 插针 | 15 | 16×24×3.1 | 2/4 MB | PCB |
| **WROOM-03** | 竖插 | 8 | 15×17.3×2.8 | 2/4 MB | PCB |
| **WROOM-04** | 贴片 | 13 | 16×24×3.1 | 2/4 MB | PCB |
| **WROOM-05** | 贴片 + 插针 | 5 | 15×17.3×2.8 | 2/4 MB | PCB |
| **WROOM-06** | 贴片 / 竖插两用 | 15（贴片）/ 5（竖插） | 15.8×20.3×2.7 | 2/4 MB | PCB |
| **WROOM-07** | 竖插超小 | 3 | 8.5×12.7×2.6 | 2/4 MB | 单极子外接天线焊盘 |

> GPIO 数差异源于模组形态：竖插/超小型只引出核心引脚，贴片型引出更多。

---

## 4. 选型建议

| 需求 | 推荐模组 |
|------|----------|
| 需要 GPIO 多、通用贴片 | **WROOM-01**（15 GPIO，最全） |
| 需要贴片但引脚数中等 | WROOM-04（13 GPIO） |
| 竖插、需要一定 GPIO | WROOM-03（8 GPIO） |
| 极简应用、引脚少 | WROOM-05（5 GPIO） |
| 装配方式灵活（贴片/竖插两用） | WROOM-06 |
| 超小尺寸 + 外接天线 | WROOM-07（3 GPIO，8.5×12.7 mm） |

---

## 5. 封装资源

各模组均提供：
- PCB Footprint（dxf）
- 部分提供 3D 模型（STEP / glb）

下载地址形如：`https://www.espressif.com/sites/default/files/modules-dxf/ESP8685-WROOM-XX%20PCB%20Footprint.dxf`

---

## 6. 应用场景

- 小家电（旋钮屏、体脂秤、电动牙刷）
- 智能照明（灯泡、灯带控制器）
- 可穿戴 / 便携 IoT
- 传感器节点
- 对尺寸与成本双敏感的 Wi-Fi + BLE 产品

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| WROOM-01 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-01_datasheet_cn.pdf |
| WROOM-03 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-03_datasheet_cn.pdf |
| WROOM-04 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-04_datasheet_cn.pdf |
| WROOM-05 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-05_datasheet_cn.pdf |
| WROOM-06 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-06_datasheet_cn.pdf |
| WROOM-07 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685-wroom-07_datasheet_cn.pdf |
| ESP32-C3 系列模组产品页 | https://www.espressif.com/zh-hans/products/modules#ESP32-C3 |
| 乐鑫产品选型工具 | https://products.espressif.com/#/product-selector?language=zh |

---

> **文档总结**：ESP8685-WROOM 是基于 ESP8685H4（ESP32-C3 小封装）的模组系列，共 01~07 六款形态，覆盖贴片（01/04）、竖插（03/07）、两用（06）、插针（05）等装配方式，GPIO 数 3~15 不等，Flash 2/4 MB。全系列无 PSRAM，多用 PCB 天线（07 除外）。选型核心看 GPIO 数量与装配方式：通用贴片选 WROOM-01，竖插选 03，超小+外接天线选 07。是小尺寸、低成本 Wi-Fi+BLE 产品的灵活模组方案。
