# ESP32-C6 芯片封装与勘误表

> 文档编号：04
> 适用范围：ESP32-C6 / ESP32-C6FH4 / ESP32-C6FH8 全系列
> 来源：乐鑫官方产品页、ESP32-C6 勘误表

---

## 目录

1. [封装总览](#1-封装总览)
2. [QFN40 封装](#2-qfn40-封装)
3. [QFN32 封装](#3-qfn32-封装)
4. [芯片变型对比](#4-芯片变型对比)
5. [勘误表（Errata）](#5-勘误表errata)
6. [封装资源下载](#6-封装资源下载)
7. [参考资源](#7-参考资源)

---

## 1. 封装总览

ESP32-C6 系列提供两种 QFN 封装：

| 封装 | 尺寸 | GPIO | 代表型号 |
|------|------|------|----------|
| **QFN40** | 5×5 mm | 30 | ESP32-C6（外接 flash） |
| **QFN32** | 5×5 mm | 22 | ESP32-C6FN4 |

> QFN40 和 QFN32 封装尺寸相同（5×5 mm），但引脚数不同。

---

## 2. QFN40 封装

| 项目 | 规格 |
|------|------|
| 封装类型 | QFN40（Quad Flat No-leads） |
| 尺寸 | 5×5 mm |
| 引脚 | 40 |
| GPIO | **30 个**可编程 GPIO |
| 适用 | ESP32-C6、C6FH4、C6FH8 |

---

## 3. QFN32 封装

| 项目 | 规格 |
|------|------|
| 封装类型 | QFN32 |
| 尺寸 | 5×5 mm |
| 引脚 | 32 |
| GPIO | **22 个**可编程 GPIO |

---

## 4. 芯片变型对比

| 型号 | 封装 | GPIO | Flash | 状态 |
|------|------|------|-------|--------|
| ESP32-C6 | QFN40 | 30 | 外接（最大 16 MB） | 在产 |
| ESP32-C6FH4 | QFN40 | 30 | 4 MB 内置 | **EOL**（停产） |
| ESP32-C6FH8 | QFN40 | 30 | 8 MB 内置 | **EOL**（停产） |
| ESP32-C6FN4 | QFN32 | 22 | 4 MB 内置 | 在产 |

> 新设计推荐使用裸芯片 ESP32-C6（外接 flash）或 QFN32 的 ESP32-C6FN4。

---

## 5. 勘误表（Errata）

《ESP32-C6 勘误表》列出芯片已知问题及规避方案，按 chip revision 分述。量产前必查。

> ⚠️ 勘误内容随芯片版本更新，请以官方最新文档为准。启动日志会打印芯片版本（如 `rev v0.1`），固件可据此条件编译规避。

常见涉及领域（具体条目见官方文档）：
- HP/LP 双核唤醒时序
- Wi-Fi 6 / 802.15.4 共存
- USB OTG 边角情况
- Deep-sleep LP 核独立运行
- Flash 接口

---

## 6. 封装资源下载

| 资源 | 链接 |
|------|------|
| 乐鑫 KiCad 库 | https://github.com/espressif/kicad-libraries |
| 封装 Footprint | 参考官方 esp-hardware-design-guidelines |

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6 勘误表 | https://docs.espressif.com/projects/esp-chip-errata/zh_CN/latest/esp32c6/index.html |
| ESP32-C6 产品页 | https://www.espressif.com/zh-hans/products/socs/esp32-c6 |
| ESP32-C6 封装 | https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32c6/index.html |
| 乐鑫产品选型工具 | https://products.espressif.com/#/product-selector?language=zh |
| 芯片系列对比 | https://products.espressif.com/#/product-comparison |

---

> **文档总结**：ESP32-C6 有 QFN40（30 GPIO）和 QFN32（22 GPIO）两种 5×5 mm 封装，均为 QFN 引脚阵列。主力在产型号为裸芯片 ESP32-C6（外接 flash，最大 16 MB）和 QFN32 的 ESP32-C6FN4（4 MB 内置）。C6FH4/FH8（内置 flash 型号）已停产。新设计推荐 ESP32-C6 外接 flash 或 ESP32-C6FN4。量产前务必查阅最新勘误表确认芯片版本对应问题与规避。
