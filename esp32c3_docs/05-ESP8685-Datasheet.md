# ESP8685 数据手册详解（ESP32-C3 小封装衍生）

> 文档编号：05
> 适用范围：ESP8685 / ESP8685H4 芯片
> 来源：乐鑫官方 ESP8685 Datasheet、产品选型页

---

## 目录

1. [产品概述](#1-产品概述)
2. [核心规格](#2-核心规格)
3. [与 ESP32-C3 的关系](#3-与-esp32-c3-的关系)
4. [对应模组](#4-对应模组)
5. [外设与安全](#5-外设与安全)
6. [应用场景](#6-应用场景)
7. [参考资源](#7-参考资源)

---

## 1. 产品概述

ESP8685 是 ESP32-C3 的**超小封装衍生芯片**，采用 **QFN 4×4 mm（24 引脚）**封装。它继承 ESP32-C3 的全部特性——RISC-V 32 位单核、2.4 GHz Wi-Fi、Bluetooth 5 (LE)、完善的安全机制——但体积更小、引脚更少，非常适合对尺寸敏感的 IoT 产品。

ESP8685 与 ESP32-C3 **软件完全兼容**，同一份 ESP-IDF 固件可直接烧录，仅引脚分配因封装不同而需调整。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 内核 | RISC-V 32 位单核，160 MHz（RV32IMC） |
| 无线 | 2.4 GHz Wi-Fi (802.11b/g/n) + Bluetooth 5 (LE) |
| 封装 | QFN24，4×4 mm |
| GPIO | 15 |
| SRAM | 400 KB |
| ROM | 384 KB |
| RTC SRAM | 8 KB |
| Flash（封装内） | 2 或 4 MB |
| 安全 | RSA-3072 安全启动、AES-128-XTS flash 加密、HMAC、数字签名、硬件加密加速 |

---

## 3. 与 ESP32-C3 的关系

| 维度 | ESP8685 | ESP32-C3 |
|------|---------|----------|
| 内核 | RISC-V RV32IMC 单核 160 MHz | 同 |
| 软件 | ESP-IDF 完全兼容 | 同 |
| 封装 | QFN 4×4 mm | QFN 5×5 mm |
| 引脚 | 24 | 32 |
| GPIO | 15 | 16/22 |
| Flash | 2/4 MB 封装内 | 外接/4/8 MB 封装内 |
| 定位 | 超小尺寸 IoT | 通用低成本 IoT |

> ESP8684 是另一款 C 系列小封装芯片（非 C3 衍生），不要混淆。

---

## 4. 对应模组

ESP8685 芯片对应丰富的 WROOM 模组系列，覆盖不同形态（贴片/竖插）与 GPIO 数：

| 模组 | 形态 | GPIO | Flash | 天线 |
|------|------|------|-------|------|
| ESP8685-WROOM-01 | 贴片+插针 | 15 | 2/4 MB | PCB |
| ESP8685-WROOM-03 | 竖插 | 8 | 2/4 MB | PCB |
| ESP8685-WROOM-04 | 贴片 | 13 | 2/4 MB | PCB |
| ESP8685-WROOM-05 | 贴片+插针 | 5 | 2/4 MB | PCB |
| ESP8685-WROOM-06 | 贴片/竖插两用 | 15 / 5 | 2/4 MB | PCB |
| ESP8685-WROOM-07 | 竖插超小 | 3 | 2/4 MB | 单极子外接天线 |

> 各 WROOM 模组尺寸约 8.5×12.7 ~ 16×24 mm，均无 PSRAM。详见 [08-ESP8685-WROOM-Datasheet](08-ESP8685-WROOM-Datasheet.md)。

---

## 5. 外设与安全

ESP8685 外设与 ESP32-C3 一致（受引脚数限制，可用通道数减少）：

- 通信：UART、SPI、I2C、I2S、LED PWM、RMT
- 模拟：12 位 SAR ADC、温度传感器
- 定时器：通用定时器、系统定时器、看门狗
- 安全：RSA-3072 安全启动、AES-128-XTS flash 加密、HMAC、数字签名、SHA/AES/RSA 加速器、RNG、时钟毛刺检测

---

## 6. 应用场景

- 体积受限的智能家电（小尺寸屏、旋钮屏）
- 可穿戴 / 便携 IoT 设备
- 智能照明（灯泡、灯带）
- 传感器节点
- 低功耗 BLE 信标 / 数据记录器
- 需要成本与体积双优化的 Wi-Fi + BLE 产品

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| ESP8685 数据手册（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685_datasheet_cn.pdf |
| ESP8685 数据手册（英文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp8685_datasheet_en.pdf |
| ESP32-C3 系列产品页（含 ESP8685） | https://www.espressif.com/zh-hans/products/socs#ESP32-C3 |
| ESP8685 模组产品页 | https://www.espressif.com/zh-hans/products/modules#ESP32-C3 |
| ESP8685 Footprint（asc） | https://www.espressif.com/sites/default/files/chips-dxf/ESP8685_Footprint.asc |

---

> **文档总结**：ESP8685 是 ESP32-C3 的超小封装（QFN 4×4 mm，24 引脚）衍生芯片，软件与 C3 完全兼容，仅封装更小、GPIO 减至 15、Flash 封装内 2/4 MB。它继承了 RISC-V 单核 160 MHz、Wi-Fi + BLE 5、全套安全机制，是对尺寸与成本敏感的 IoT 产品（可穿戴、小家电、智能照明、传感器节点）的理想选择。对应 ESP8685-WROOM-01~07 多种形态模组，满足贴片/竖插/超小等不同装配需求。
