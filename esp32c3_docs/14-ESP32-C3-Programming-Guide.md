# ESP32-C3 编程指南（ESP-IDF）

> 文档编号：14
> 适用框架：ESP-IDF（乐鑫官方 IoT 开发框架，基于 FreeRTOS）
> 来源：乐鑫官方 ESP-IDF 编程指南（esp32c3）、Get Started、API Reference

---

## 目录

1. [ESP-IDF 概述](#1-esp-idf-概述)
2. [环境搭建](#2-环境搭建)
3. [第一个工程](#3-第一个工程)
4. [构建/烧录/监视](#4-构建烧录监视)
5. [C3 特有配置](#5-c3-特有配置)
6. [常用 API 模块](#6-常用-api-模块)
7. [Wi-Fi 与蓝牙开发](#7-wi-fi-与蓝牙开发)
8. [外设开发要点](#8-外设开发要点)
9. [OTA 与分区](#9-ota-与分区)
10. [低功耗相关 API](#10-低功耗相关-api)
11. [参考资源](#11-参考资源)

---

## 1. ESP-IDF 概述

ESP-IDF（Espressif IoT Development Framework）是乐鑫官方为 ESP32 系列芯片提供的开发框架，基于 **FreeRTOS**，提供 Wi-Fi/BLE 协议栈、丰富的外设驱动、文件系统、OTA、安全等组件。ESP32-C3 从 ESP-IDF v4.2+ 开始官方支持。

ESP-IDF 编程指南（esp32c3 目标）是 ESP32-C3 软件开发的文档中心，覆盖 Get Started、API Reference、API Guides 等。

---

## 2. 环境搭建

### 2.1 安装方式

- **官方一键安装器**（Windows/macOS/Linux）：自动安装工具链、ESP-IDF、Python 依赖
- **手动**：`git clone --recursive https://github.com/espressif/esp-idf.git` + `install.sh esp32c3`

### 2.2 验证

```bash
idf.py --version
```

---

## 3. 第一个工程

```bash
# 复制 hello_world 示例
cp -r $IDF_PATH/examples/get-started/hello_world .
cd hello_world

# 设置目标芯片
idf.py set-target esp32c3

# 配置（可选）
idf.py menuconfig

# 编译烧录监视
idf.py -p <PORT> flash monitor
```

启动后串口应打印 `Hello world!`。

---

## 4. 构建/烧录/监视

| 命令 | 作用 |
|------|------|
| `idf.py set-target esp32c3` | 设置目标芯片 |
| `idf.py build` | 编译 |
| `idf.py -p <PORT> flash` | 烧录 |
| `idf.py -p <PORT> monitor` | 串口监视（Ctrl+] 退出） |
| `idf.py -p <PORT> flash monitor` | 一键编译+烧录+监视 |
| `idf.py menuconfig` | 配置工程（sdkconfig） |
| `idf.py fullclean` | 完全清理 |
| `idf.py add-dependency "espressif/xxx"` | 添加组件 |

### 端口

- Windows：`COMx`
- Linux：`/dev/ttyUSBx`
- macOS：`/dev/cu.usbserial-xxxx`

> ESP32-C3 支持原生 USB（GPIO18/19）下载，无需外部 USB 转串口。

---

## 5. C3 特有配置

ESP32-C3 单核 RISC-V，配置要点：

| 配置 | 说明 |
|------|------|
| `CONFIG_IDF_TARGET="esp32c3"` | 目标（set-target 自动设置） |
| **USB CDC on Boot** | menuconfig → 启用后日志走 USB CDC |
| Flash 大小 | menuconfig → Serial flasher config → Flash size（4 MB 等） |
| Strapping | GPIO2/8/9，不可同时低 |
| BLE 协议栈 | NimBLE（推荐，低功耗）或 Bluedroid |

---

## 6. 常用 API 模块

| 模块 | 说明 |
|------|------|
| **GPIO** | `gpio_config()`、`gpio_set_level()`、中断 |
| **UART** | `uart_driver_install()`、读写 |
| **I2C** | `i2c_param_config()`、`i2c_master_write_to_device()` |
| **SPI** | `spi_bus_initialize()`、设备添加 |
| **ADC** | `adc_oneshot`（推荐新 API） |
| **LED PWM（LEDC）** | `ledc_timer_config()`、`ledc_set_duty()` |
| **RMT** | 红外、WS2812 等时序协议 |
| **TWAI（CAN）** | `twai_driver_install()` |
| **定时器** | `esp_timer`、通用定时器 |
| **FreeRTOS** | 任务、队列、信号量、事件组 |
| **NVS** | 非易失存储 `nvs_flash_init()` |
| **Wi-Fi / BLE** | 见第 7 节 |
| **lwIP / mDNS** | 网络 |

---

## 7. Wi-Fi 与蓝牙开发

### 7.1 Wi-Fi

```c
esp_netif_init();
esp_event_loop_create_default();
esp_netif_create_default_wifi_sta();

wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
esp_wifi_init(&cfg);
esp_wifi_set_mode(WIFI_MODE_STA);
esp_wifi_set_config(...);
esp_wifi_start();
esp_wifi_connect();
```

支持 Station、SoftAP、Station+SoftAP、混杂模式（4 虚拟接口）。

### 7.2 Bluetooth LE

ESP32-C3 推荐 **NimBLE** 协议栈（比 Bluedroid 更省内存）：

- menuconfig → Bluetooth → Host → NimBLE
- 配合 `esp_idf_components/bluetooth` 示例开发 BLE 外设/中心

### 7.3 Wi-Fi + BLE 共存

ESP32-C3 Wi-Fi 与 BLE 共用同一射频，IDF 自动处理时分共存。

---

## 8. 外设开发要点

- **GPIO 矩阵**：几乎所有外设可映射到任意 GPIO（少数固定管脚除外）
- **封装内 flash 型号**：GPIO 数减少，flash 总线管脚不可外用
- **USB Serial/JTAG**：免调试器，原生 USB 下载 + JTAG 断点
- **TWAI**：兼容 CAN 2.0，注意终端电阻

---

## 9. OTA 与分区

| 类型 | 说明 |
|------|------|
| **App OTA** | `esp_ota_*` API，A/B 双分区切换，安全升级 |
| **分区表** | `partitions.csv` 自定义 nvs/otadata/app/fatfs/phy 等 |
| **安全 OTA** | 结合 Secure Boot v2 + 签名验证 |

```c
esp_ota_handle_t handle;
esp_ota_begin(update_partition, OTA_WITH_SEQUENTIAL_WRITES, &handle);
esp_ota_write(handle, data, size);
esp_ota_end(handle);
esp_ota_set_boot_partition(update_partition);
esp_restart();
```

---

## 10. 低功耗相关 API

```c
// Deep-sleep（唤醒源 + 进入）
esp_sleep_enable_timer_wakeup(us);
esp_deep_sleep_start();          // 唤醒后相当于重启

// Light-sleep
esp_sleep_enable_gpio_wakeup(...);
esp_light_sleep_start();         // 唤醒后状态保留

// 断电域配置
esp_sleep_pd_config(domain, option);

// Wi-Fi/BLE 自动省电
// Modem-sleep + Auto Light-sleep（保持连接）
```

> 低功耗完整指南见 [15-ESP32-C3-Low-Power-Mode-Guide](15-ESP32-C3-Low-Power-Mode-Guide.md)。

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF 编程指南（ESP32-C3 中文） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/index.html |
| Get Started（C3） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/get-started/index.html |
| API Reference | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/index.html |
| ESP-IDF GitHub | https://github.com/espressif/esp-idf |
| ESP-IDF 示例 | https://github.com/espressif/esp-idf/tree/master/examples |
| esp-bsp（板级支持包） | https://github.com/espressif/esp-bsp |
| ESP Component Registry | https://components.espressif.com/ |
| 如何建立串口连接 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/get-started/establish-serial-connection.html |
| ESP-IDF 版本兼容性 | https://github.com/espressif/esp-idf/blob/master/COMPATIBILITY_CN.md |

---

> **文档总结**：ESP32-C3 编程基于 ESP-IDF（FreeRTOS）。开发流程：一键安装器搭环境 → `idf.py set-target esp32c3` → menuconfig 配置 → `idf.py flash monitor` 烧录监视。C3 特性：单核 RISC-V、原生 USB 下载/JTAG、NimBLE 低功耗蓝牙、4 虚拟 Wi-Fi 接口、Wi-Fi/BLE 自动共存。API 覆盖 GPIO/UART/I2C/SPI/ADC/LEDC/RMT/TWAI/NVS/OTA 等全外设。配合 esp-bsp 板级支持包与 Component Registry 组件可快速开发。低功耗经 Deep-sleep（5 µA）/Light-sleep（130 µA）+ Modem-sleep 实现。
