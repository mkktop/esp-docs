# ESP32-C3 芯片封装与勘误表

> 文档编号：04
> 适用范围：ESP32-C3 / ESP8685 全系列芯片
> 来源：乐鑫官方产品页、ESP32-C3 勘误表、Datasheet 封装信息

---

## 目录

1. [封装总览](#1-封装总览)
2. [ESP32-C3 封装（QFN 5×5）](#2-esp32-c3-封装qfn-5x5)
3. [ESP8685 封装（QFN 4×4）](#3-esp8685-封装qfn-4x4)
4. [芯片变型对比](#4-芯片变型对比)
5. [勘误表（Errata）](#5-勘误表errata)
6. [PCN 与公告](#6-pcn-与公告)
7. [封装资源下载](#7-封装资源下载)
8. [参考资源](#8-参考资源)

---

## 1. 封装总览

ESP32-C3 系列芯片提供两种封装尺寸，均引脚兼容于同封装产品：

| 封装 | 尺寸 | 代表型号 | GPIO |
|------|------|----------|------|
| **QFN32** | 5×5 mm | ESP32-C3 / C3FH4 / C3FN4 / C3FH8X | 22 |
| **QFN32** | 5×5 mm | ESP32-C3FH4X / C3FH4AZ | 16 |
| **QFN24** | 4×4 mm | ESP8685（C3 的小封装衍生） | 15 |

> 封装内带 flash 的型号（FN4/FH4/FH4X/FH8X），flash 占用部分引脚，可用 GPIO 相应减少。

---

## 2. ESP32-C3 封装（QFN 5×5）

| 项目 | 规格 |
|------|------|
| 封装类型 | QFN32（Quad Flat No-leads） |
| 尺寸 | 5×5 mm |
| 引脚 | 32 |
| GPIO | 22（FH4X/FH4AZ 为 16） |
| Flash | 外接（C3）/ 封装内 4 MB（FH4/FN4/FH4X）/ 8 MB（FH8X） |
| RAM/ROM | 400 KB SRAM / 384 KB ROM / 8 KB RTC SRAM |

### 引脚分类

- **3 个 strapping 管脚**：GPIO2、GPIO8、GPIO9
- **6 个 flash 总线管脚**（封装内 flash 型号占用）
- 其余为通用 GPIO

---

## 3. ESP8685 封装（QFN 4×4）

ESP8685 是 ESP32-C3 的**更小封装**（QFN 4×4，24 引脚）衍生型号，**软件与 C3 完全一致**，仅引脚数与封装不同。

| 项目 | 规格 |
|------|------|
| 封装 | QFN24，4×4 mm |
| 核 | 单核（与 ESP32-C3 同 RISC-V 内核） |
| GPIO | 15 |
| RAM/ROM | 400 KB RAM / 384 KB ROM / 8 KB RTC SRAM |
| Flash | 2 或 4 MB（封装内） |
| 典型模组 | ESP8685-WROOM-01/03/04/05/06/07 |

> ESP8685 详细规格见 [05-ESP8685-Datasheet](05-ESP8685-Datasheet.md)。

---

## 4. 芯片变型对比

| 型号 | 封装 | Flash | GPIO | RAM | 状态 |
|------|------|-------|------|-----|------|
| ESP32-C3 | QFN 5×5 | 外接 | 22 | 400 KB | 在产 |
| ESP32-C3FH4 | QFN 5×5 | 4 MB 内置 | 22 | 400 KB | 在产 |
| **ESP32-C3FH4X** | QFN 5×5 | 4 MB 内置 | 16 | 400 KB | **推荐** |
| ESP32-C3FH8X | QFN 5×5 | 8 MB 内置 | 16 | 400 KB | 在产 |
| ESP32-C3FN4 | QFN 5×5 | 4 MB 内置 | 22 | 400 KB | EOL |
| ESP32-C3FH4AZ | QFN 5×5 | 4 MB 内置 | 16 | 400 KB | NRND |
| **ESP8685** | QFN 4×4 | 2/4 MB 内置 | 15 | 400 KB | 在产 |

---

## 5. 勘误表（Errata）

《ESP32-C3 系列芯片勘误表》描述芯片的**已知错误（bug）与规避方案**，是产品量产前必查文档。勘误按芯片版本（chip revision）列出问题及规避措施。

> ⚠️ 勘误内容会随芯片版本更新。实际开发请以官方最新勘误为准，并通过 `esp_chip_info()` 读取芯片版本，在固件中针对特定版本应用规避。

常见勘误涉及领域（具体条目见官方文档）：
- RTC/低功耗相关
- Wi-Fi/BLE 射频相关
- 外设（SPI/I2C/UART/ADC 等）边角情况
- Flash/cache 相关
- 复位与时序

> 在 ESP-IDF 中，多数已知勘误已由驱动层自动规避；启动日志会打印芯片版本（如 `rev v0.3` / `rev v0.4`）。

---

## 6. PCN 与公告

| 类型 | 说明 | 查询 |
|------|------|------|
| **PCN**（产品/工艺变更通知） | 芯片工艺、引脚、封装变更通知 | https://espressif.com/zh-hans/support/documents/pcns?keys=ESP32-C3 |
| **Advisories**（公告） | 安全、bug、兼容性、器件可靠性 | https://espressif.com/zh-hans/support/documents/advisories?keys=ESP32-C3 |
| 证书 | RF/安全/法规认证 | https://espressif.com/zh-hans/support/documents/certificates |

---

## 7. 封装资源下载

| 资源 | 链接 |
|------|------|
| ESP32-C3 封装（dxf） | https://www.espressif.com/sites/default/files/chips-dxf/ESP32-C3.dxf |
| ESP8685 Footprint | https://www.espressif.com/sites/default/files/chips-dxf/ESP8685_Footprint.asc |
| 乐鑫 KiCad 库 | https://github.com/espressif/kicad-libraries |

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3 勘误表 | https://docs.espressif.com/projects/esp-chip-errata/zh_CN/latest/esp32c3/index.html |
| ESP32-C3 芯片产品页 | https://www.espressif.com/zh-hans/products/socs?id=ESP32-C3 |
| 芯片系列对比 | https://products.espressif.com/#/product-comparison |
| ESP32-C3 Datasheet | https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_cn.pdf |
| 乐鑫产品订购信息（PDF） | https://www.espressif.com/sites/default/files/documentation/espressif_products_ordering_information_cn.pdf |

---

> **文档总结**：ESP32-C3 系列芯片有两种封装——ESP32-C3 为 QFN32 5×5 mm（GPIO 16/22），衍生型号 ESP8685 为更小的 QFN24 4×4 mm（GPIO 15），二者软件一致。封装内 flash 容量分 2/4/8 MB 多档，推荐新设计用 ESP32-C3FH4X（4 MB）或 ESP8685。量产前务必查阅《ESP32-C3 勘误表》确认芯片版本对应已知 bug 与规避方案，并关注 PCN/Advisories 获取工艺与可靠性变更。封装 dxf/KiCad 库可从官方直接下载用于 PCB 设计。
