# ESP32-C3-MINI-1 / MINI-1U 模组数据手册

> 文档编号：06
> 适用模组：ESP32-C3-MINI-1（PCB 天线）、ESP32-C3-MINI-1U（外接天线座子）
> 来源：乐鑫官方《ESP32-C3-MINI-1 & MINI-1U 技术规格书》、产品页

---

## 目录

1. [模组概述](#1-模组概述)
2. [核心规格](#2-核心规格)
3. [MINI-1 vs MINI-1U](#3-mini-1-vs-mini-1u)
4. [尺寸与封装](#4-尺寸与封装)
5. [引脚与 strapping](#5-引脚与-strapping)
6. [对应开发板](#6-对应开发板)
7. [应用场景](#7-应用场景)
8. [参考资源](#8-参考资源)

---

## 1. 模组概述

ESP32-C3-MINI-1 / ESP32-C3-MINI-1U 是乐鑫基于 **ESP32-C3** 芯片的**通用型 Wi-Fi + 低功耗蓝牙（BLE）双模模组**，以**体积小**著称。模组搭载 ESP32-C3FN4 / FH4 / FH4X 芯片，**4 MB flash 集成于芯片封装内**，因此模组封装尺寸更小，非常适合对空间敏感的应用。

- **ESP32-C3-MINI-1**：PCB 板载天线
- **ESP32-C3-MINI-1U**：外部天线连接器（U.FL / IPEX）

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 集成芯片 | ESP32-C3FN4 / ESP32-C3FH4 / ESP32-C3FH4X |
| 内核 | RISC-V 32 位单核，160 MHz |
| 无线 | 2.4 GHz Wi-Fi (802.11b/g/n) + Bluetooth 5 (LE) |
| Flash | **4 MB（芯片封装内）** |
| RAM | 400 KB SRAM |
| 工作电压 | 3.0 ~ 3.6 V（推荐 3.3 V） |
| 尺寸 | **13.2 × 16.6 × 2.4 mm** |

---

## 3. MINI-1 vs MINI-1U

| 项目 | MINI-1 | MINI-1U |
|------|--------|---------|
| 天线 | PCB 板载天线 | 外部天线连接器（U.FL） |
| 芯片/Flash/RAM | 相同 | 相同 |
| 尺寸 | 13.2×16.6×2.4 mm | 相近 |
| 适用 | 常规应用 | 金属外壳/远距离/外接高增益天线场景 |

---

## 4. 尺寸与封装

- **尺寸**：13.2 × 16.6 × 2.4 mm（极小）
- **贴片设计**：模组采用 SMT 贴片，适合回流焊
- 3D 模型（STEP / glb）可从官方下载

---

## 5. 引脚与 strapping

- 模组引出**大部分 GPIO**（除 flash SPI 总线外）
- **3 个 strapping 管脚**：GPIO2、GPIO8、GPIO9
  - GPIO9 默认弱上拉，控制 SPI Boot / Download Boot
  - GPIO2、GPIO8 默认浮空，建议外部上拉
- strapping 时序要求：tH ≥ 3 ms

> 完整引脚定义见模组规格书「模组原理图」与 ESP32-C3 datasheet 启动配置章节。

---

## 6. 对应开发板

ESP32-C3-MINI-1 模组被多款官方开发板采用：

| 开发板 | 说明 |
|--------|------|
| **ESP32-C3-DevKitM-1** | 入门级开发板，搭载 MINI-1，引脚全引出 |
| **ESP32-C3-DevKit-RUST-1/2** | Rust 培训板（Adafruit Feather 外形），含 IMU/温湿度/电池充电 |
| **ESP32-C3-LCDkit** | 旋钮屏评估板，1.28" SPI 屏 + EC11 编码器 |
| ESP32-C3-AWS-ExpressLink-DevKit-2 | AWS IoT ExpressLink 开发板，Arduino UNO 兼容 |

---

## 7. 应用场景

- 智能家居、工业自动化、医疗保健、消费电子
- 可穿戴 / 便携设备（体积小优势）
- 电池供电的低功耗 BLE + Wi-Fi 设备
- 需要外接天线的金属外壳设备（MINI-1U）

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3-MINI-1 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c3-mini-1_datasheet_cn.pdf |
| ESP32-C3-MINI-1 规格书（在线） | https://documentation.espressif.com/esp32-c3-mini-1_datasheet_cn.html |
| MINI-1 PCB Footprint（dxf） | https://www.espressif.com/sites/default/files/modules-dxf/ESP32-C3-MINI-1%20PCB%20Footprint.dxf |
| MINI-1 3D 模型（STEP） | https://www.espressif.com/sites/default/files/3dmodel/ESP32-C3-MINI-1%203D%20Model.STEP |
| 模组产品页 | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C3 |

---

> **文档总结**：ESP32-C3-MINI-1 / MINI-1U 是 ESP32-C3 的超小尺寸通用模组（13.2×16.6×2.4 mm），4 MB flash 封装于芯片内。MINI-1 用 PCB 天线，MINI-1U 用外接 IPEX 座子。它被 DevKitM-1、DevKit-RUST、LCDkit、AWS ExpressLink DevKit 等多款官方开发板采用，是空间受限型 Wi-Fi + BLE 产品的首选模组。
