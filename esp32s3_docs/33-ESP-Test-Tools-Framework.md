# ESP-Test-Tools 框架详解 — 射频测试与产线测试工具集

> 文档编号：33
> 适用芯片：ESP32-S3（及 ESP 全系列）
> 适用场景：射频（RF）性能测试、Wi-Fi/BLE 非信令与信令测试、产线（生产）测试、认证测试
> 来源：乐鑫官方 esp-test-tools 文档、ESP-IDF RF Calibration 指南、EspRFTestTool

---

## 目录

1. [ESP-Test-Tools 概述](#1-esp-test-tools-概述)
2. [测试阶段划分](#2-测试阶段划分)
3. [Wi-Fi 测试项](#3-wi-fi-测试项)
4. [Bluetooth LE 测试项](#4-bluetooth-le-测试项)
5. [RF 测试环境搭建](#5-rf-测试环境搭建)
6. [EspRFTestTool 工具](#6-esprftesttool-工具)
7. [RF 校准（PHY Calibration）](#7-rf-校准phy-calibration)
8. [典型测试流程](#8-典型测试流程)
9. [参考资源](#9-参考资源)

---

## 1. ESP-Test-Tools 概述

### 1.1 什么是 ESP-Test-Tools

ESP-Test-Tools 是乐鑫提供的**射频（RF）测试与产线测试工具集**，用于评估和验证基于 ESP 芯片/模组/设备的无线通信性能，确保产品在各种场景下的通信质量。它面向**研发阶段验证**与**量产阶段产线测试**两个场景。

### 1.2 核心组成

| 组成 | 说明 |
|------|------|
| **RF Test Firmware** | 烧录到被测设备（DUT）的射频测试固件，支持各种 TX/RX 测试模式 |
| **EspRFTestTool** | 运行在 PC 端的射频测试工具套件，串口下发命令控制 DUT |
| **测试仪器控制软件** | 配合 CMW500 / CMW270 / CBT 等综合测试仪使用 |

---

## 2. 测试阶段划分

ESP-Test-Tools 文档按研发流程划分为两个阶段：

| 阶段 | 说明 |
|------|------|
| **Development Stage（研发阶段）** | 评估 Wi-Fi/BLE 射频性能，验证发射功率、频谱质量、误码率等关键指标 |
| **Production Stage（产线/生产阶段）** | 产线量产测试，快速判定每台出厂设备的射频是否合格 |

> 本文档主要聚焦 ESP32-S3 在 Development Stage 的 RF 测试项。

---

## 3. Wi-Fi 测试项

### 3.1 Wi-Fi 非信令测试（Non-Signaling Test）

又称**定频测试**，直接控制设备发射特定信号而**不建立数据连接**，用于评估关键 RF 性能指标：

- 发射功率（TX Power）
- 频谱质量（频谱模板、EVM）
- 误码率 / 误差率

> 这是最常用的产线与研发射频验证手段，速度快、可控性强。

### 3.2 Wi-Fi 信令测试（Signaling Test）

评估和验证无线网络设备的 Wi-Fi 信令功能，关注不同场景下稳定可靠的通信。重点评估**空口（OTA）性能**：

- **TRP**（Total Radiated Power，总辐射功率）
- **TIS**（Total Isotropic Sensitivity，总全向灵敏度）

### 3.3 Wi-Fi 自适应性测试（Adaptivity Test）

模拟各种网络条件和负载，考察设备在发射速率、信道选择、功率水平的实时调整能力，优化整体网络性能与稳定性。**主要针对欧洲/日本等地区的法规符合性**（Listen-Before-Talk 等）。

### 3.4 Wi-Fi 阻塞测试（Blocking Test）

评估设备在**强干扰环境**下的接收性能。通过引入高强度干扰信号，测量接收灵敏度与抗干扰能力，确保复杂无线环境下的可靠运行。

---

## 4. Bluetooth LE 测试项

| 测试项 | 说明 |
|--------|------|
| **BLE 非信令测试** | 控制设备发射特定信号而不建立连接，评估发射功率、频谱特性、误码率 |
| **BLE DTM 测试**（Direct Test Mode） | 直接控制设备进入特定发射/接收模式，评估发射功率、接收灵敏度、频谱特性 |
| **BLE 阻塞测试** | 评估设备在其他无线信号干扰环境下的稳定性与性能，确保符合相关标准 |
| **BLE 自适应性测试** | 确保跳频期间满足性能标准，尤其当 BLE 信号功率谱密度（PSD）超过 10 dBm/MHz 时，避免干扰其他无线设备 |

---

## 5. RF 测试环境搭建

以 BLE DTM 测试为例，典型环境如下：

```
   ┌──────┐  USB   ┌──────────────┐  UART0 ┌──────┐
   │  PC  │◄──────►│ USB-to-UART  │◄──────►│      │
   │      │        │   板 ×2      │        │ DUT  │
   │ EspRF│        └──────┬───────┘ UART1 └──┬───┘
   │ Test │               │                  │
   │ Tool │          ┌────▼────┐   RF 线缆   │
   └──┬───┘          │ Tester  │◄───────────►│
      │              │ CMW500/ │   (被测设备)  │
      └─────────────►│ CMW270/ │              │
        仪器控制      │ CBT     │              │
                     └─────────┘              │
```

| 角色 | 说明 |
|------|------|
| **PC** | 经 USB 连接 USB-to-UART 板，需安装 EspRFTestTool、仪器控制软件、UART 板驱动 |
| **Tester（综合测试仪）** | 测试 DUT 在不同模式下的 RF 性能，常用 CMW500、CMW270 或蓝牙测试仪 CBT |
| **USB-to-UART 板 ×2** | 分别连接 UART0（串口命令输入/日志）和 UART1（连接测试仪） |
| **DUT（被测设备）** | 基于 ESP32-S3 芯片/模组设计的产品 |

### DTM 默认引脚与注意事项

- ESP 提供的 DTM 固件默认使用 **GPIO4、GPIO5** 作为 UART1 测试引脚，可通过 UART0 串口命令修改
- DUT 的 **CHIP_EN 引脚默认上拉**；若产品设计未上拉，需手动将 CHIP_EN 连到 3V3
- 部分串口通信板内部已交换 RXD/TXD，无需再交叉，按实际情况接线
- ESP32-S3 具备**上电自校准**功能，**RF 线缆必须在 DUT 上电前连接到测试仪**

---

## 6. EspRFTestTool 工具

EspRFTestTool 是乐鑫 PC 端射频测试工具套件，提供：

| 功能 | 说明 |
|------|------|
| 串口命令输入 | 通过 UART 向 DUT 下发 TX/RX/CW 等测试命令 |
| 仪器协同 | 配合 CMW500 等综合测试仪完成自动化测试 |
| 日志查看 | 实时查看 DUT 测试日志 |

EspRFTestTool 配套的 RF 测试固件基于 ESP-IDF PHY 层 API（如 `esp_phy_wifi_tx()`、`esp_phy_ble_tx()`、`esp_phy_test_start_stop()` 等）实现。

---

## 7. RF 校准（PHY Calibration）

RF 测试离不开 PHY 校准机制。ESP32-S3 在 RF 初始化时支持三种校准方式：

| 校准方式 | 触发条件 | 说明 |
|----------|----------|------|
| **Partial Calibration（部分校准）** | 默认方式，基于 NVS 中存储的全校准数据 | 启动最快，需 `CONFIG_ESP_PHY_CALIBRATION_AND_DATA_STORAGE=y` |
| **Full Calibration（全校准）** | NVS 不存在/被擦除/MAC 变更/PHY 版本变更/校准数据损坏 | 比部分校准多耗时约 100ms，效果最佳 |
| **No Calibration（不校准）** | 仅用于 Deep-sleep 唤醒 | 最省时 |

相关 API：

```c
// 触发全校准的兜底手段：擦除 NVS 中的 PHY 校准数据
esp_err_t esp_phy_erase_cal_data_in_nvs(void);

// PHY modem 控制
void esp_phy_enable(esp_phy_modem_t modem);   // 启用 PHY（WiFi/BT）
void esp_phy_disable(esp_phy_modem_t modem);

// RF 测试模式
void esp_phy_rftest_init(void);
void esp_phy_rftest_config(uint8_t conf);     // conf=1 进入 RF 测试模式
void esp_phy_test_start_stop(uint8_t value);  // value=3 开始 TX/RX，0 结束

// Wi-Fi 发射
void esp_phy_wifi_tx(uint32_t chan, esp_phy_wifi_rate_t rate,
                     int8_t backoff, uint32_t length_byte,
                     uint32_t packet_delay, uint32_t packet_num);
void esp_phy_wifi_tx_tone(uint32_t start, uint32_t chan, uint32_t backoff);  // CW 载波
```

> 详见 [02-ESP32-S3-Technical-Reference-Manual](02-ESP32-S3-Technical-Reference-Manual.md) 与 ESP-IDF `api-guides/RF_calibration` 文档。

---

## 8. 典型测试流程

以产线 Wi-Fi 非信令测试为例：

```
1. DUT 烧录 RF Test Firmware（或 DTM 固件）
2. RF 线缆连接 DUT 天线馈点 ↔ 综合测试仪（上电前连接）
3. DUT 上电自校准
4. PC 端 EspRFTestTool 经 UART 下发命令：
     esp_phy_test_start_stop(3)              // 进入测试
     esp_phy_wifi_tx(chan, rate, backoff, …) // 定频发射
5. 综合测试仪测量：发射功率 / 频谱模板 / EVM / 误码率
6. 判定合格 → 进入下一工位；不合格 → 标记返修
```

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-Test-Tools 文档（中文） | https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32s3/development_stage/index.html |
| RF 测试项总览 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/index.html |
| Wi-Fi 非信令测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/wifi_non_signaling_test.html |
| Wi-Fi 信令测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/wifi_signaling_test.html |
| Wi-Fi 自适应性测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/wifi_adaptivity_test.html |
| Wi-Fi 阻塞测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/wifi_blocking_test.html |
| BLE 非信令测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/bt_ble_non_signaling_test.html |
| BLE DTM 测试 | https://docs.espressif.com/projects/esp-test-tools/en/latest/esp32s3/development_stage/rf_test_items/ble_dtm_test.html |
| RF Calibration 指南 | https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/RF_calibration.html |
| EspRFTestTool 下载 | https://www.espressif.com/en/support/download/other-tools |

---

> **文档总结**：ESP-Test-Tools 是乐鑫面向 ESP 芯片/模组/设备的射频测试与产线测试工具集，包含 RF Test Firmware（烧录到 DUT）、EspRFTestTool（PC 端控制）和综合测试仪协同三部分。覆盖 Wi-Fi 非信令/信令/自适应性/阻塞测试、BLE 非信令/DTM/阻塞/自适应性测试，分研发阶段（Development Stage）和产线阶段（Production Stage）。其底层依赖 ESP32-S3 的 PHY 校准机制（Partial/Full/No 三种校准方式）和 PHY 测试 API（`esp_phy_wifi_tx`、`esp_phy_rftest_config` 等）。是 ESP32-S3 产品量产射频合格判定与法规认证的关键工具。
