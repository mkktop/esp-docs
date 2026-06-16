# ESP32-C6 编程指南（ESP-IDF）

> 文档编号：09
> 适用框架：ESP-IDF（乐鑫官方 IoT 开发框架，基于 FreeRTOS）
> 来源：乐鑫官方 ESP-IDF 编程指南（esp32c6）

---

## 目录

1. [编程概述](#1-编程概述)
2. [环境搭建](#2-环境搭建)
3. [第一个工程](#3-第一个工程)
4. [HP + LP 双核开发](#4-hp--lp-双核开发)
5. [Wi-Fi 6 编程](#5-wi-fi-6-编程)
6. [蓝牙 BLE 开发](#6-蓝牙-ble-开发)
7. [Thread / Zigbee 开发](#7-thread--zigbee-开发)
8. [USB OTG 编程](#8-usb-otg-编程)
9. [常用 API 模块](#9-常用-api-模块)
10. [参考资源](#10-参考资源)

---

## 1. 编程概述

ESP32-C6 编程基于 **ESP-IDF**（FreeRTOS），与 C3 开发流程高度一致，主要区别：

| 维度 | ESP32-C6 | ESP32-C3 |
|------|-----------|---------|
| 目标芯片 | `esp32c6` | `esp32c3` |
| 双核 | HP + LP 双核 | 单核 |
| Wi-Fi | Wi-Fi 6 (802.11ax) | Wi-Fi 4 |
| 802.15.4 | Thread + Zigbee | 无 |
| USB | USB OTG + USB Serial/JTAG | USB Serial/JTAG |
| BLE | BT 5.3 | BT 5 |

---

## 2. 环境搭建

```bash
# 安装方式与 C3 完全一致
idf.py install-esp32c6
# 或
espup install
idf.py set-target esp32c6
```

---

## 3. 第一个工程

```bash
cp -r $IDF_PATH/examples/get-started/hello_world .
cd hello_world
idf.py set-target esp32c6
idf.py menuconfig
idf.py -p <PORT> flash monitor
```

---

## 4. HP + LP 双核开发

### 4.1 双核基本概念

| 核 | 频率 | 主要任务 | 典型使用 |
|-----|------|----------|----------|
| HP | 160 MHz | Wi-Fi/BT/802.15.4 协议栈 + 用户应用 | 主程序 |
| LP | 20 MHz | BLE 监听、低功耗外设轮询 | Deep-sleep 期间独立任务 |

### 4.2 LP 核代码（ESP-IDF LP Core 例程）

LP 核可独立运行低功耗任务，由 HP 核启动：

```c
// HP 核启动 LP 核
esp_lp_core_boot_config_t cfg = {
    .lp_core_boot_mode = LP_CORE_BOOT_BOOT,
};
esp_err_t err = esp_lp_core_launcher_start(&cfg);
```

LP 核代码位于 `lp_core` 例程，编译为 `lp_core` 分区。

---

## 5. Wi-Fi 6 编程

### 5.1 Wi-Fi 6 新特性

| 特性 | API/配置 | 说明 |
|------|----------|------|
| OFDMA | menuconfig 开启 | 多用户上行/下行 |
| MU-MIMO | menuconfig 开启 | 下行多用户 |
| TWT | `esp_wifi_ftm_init()` | 目标唤醒时间省电 |
| BSS Color | menuconfig | 空间复用 |

### 5.2 Wi-Fi STA 连接

```c
esp_netif_init();
esp_event_loop_create_default();
esp_netif_create_default_wifi_sta();
wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
esp_wifi_init(&cfg);
esp_wifi_set_mode(WIFI_MODE_STA);
esp_wifi_start();
esp_wifi_connect();
```

> Wi-Fi 6 配置与 C3 基本一致，menuconfig 中可开启 802.11ax 特性。

---

## 6. 蓝牙 BLE 开发

ESP32-C6 使用 **NimBLE**（推荐）或 Bluedroid：

```bash
# menuconfig → Bluetooth → Host → NimBLE
```

NimBLE 比 Bluedroid 更省内存（适合 C6 双核协作）。

---

## 7. Thread / Zigbee 开发

### 7.1 Thread（Matter-over-Thread）

```bash
# menuconfig → OpenThread → Enable Thread
# 或使用 esp-idf 的 openthread 例程
```

### 7.2 Zigbee

```bash
# menuconfig → Zigbee → Enable Zigbee
```

ESP32-C6 支持 **Zigbee 3.0** 认证协议栈，可与 HomeKit/Zigbee2MQTT 集成。

---

## 8. USB OTG 编程

ESP32-C6 支持 **USB 2.0 OTG**（全速 12 Mbps）：

```c
// USB OTG 设备模式示例
usb_device_config_t config = {
    .speed = USB_SPEED_FULL,
};
esp_usb_otg_init(&config);
```

应用场景：USB HID 设备、USB 串口（CDC）、USB 存储等。

---

## 9. 常用 API 模块

| 模块 | 说明 |
|------|------|
| Wi-Fi | `esp_wifi_*`（Wi-Fi 6 特性） |
| BLE | NimBLE `esp_nimble_hci` |
| 802.15.4 | OpenThread / Zigbee SDK |
| USB OTG | `esp_usb_otg_*` |
| LP 核 | `esp_lp_core_*` |
| 低功耗 | `esp_sleep_*` |
| TWAI | `twai_*`（CAN 2.0） |
| MCPWM | `mcpwm_*` |
| PARLIO | `parlio_*` |

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF 编程指南（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/index.html |
| Get Started（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/get-started/index.html |
| Wi-Fi 指南（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/wifi.html |
| LP Core 例程 | https://github.com/espressif/esp-idf/tree/master/examples/system/lp_core |

---

> **文档总结**：ESP32-C6 编程基于 ESP-IDF，流程与 C3 一致。核心区别：目标为 `esp32c6`；HP+LP 双核可协作（LP 核独立处理 BLE 监听/TWT 任务）；Wi-Fi 6（OFDMA/MU-MIMO/TWT）；802.15.4（Thread + Zigbee）；USB OTG（全速 12 Mbps）；BLE 用 NimBLE 更省内存。
