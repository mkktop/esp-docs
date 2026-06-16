# ESP32-C6 开发入门

> 文档编号：15
> 适用芯片：ESP32-C6
> 来源：乐鑫官方 ESP-IDF 快速入门

---

## 目录

1. [开发准备](#1-开发准备)
2. [硬件选型](#2-硬件选型)
3. [软件环境搭建](#3-软件环境搭建)
4. [第一个工程](#4-第一个工程)
5. [开发板烧录](#5-开发板烧录)
6. [下一步](#6-下一步)

---

## 1. 开发准备

| 项目 | 推荐 |
|------|------|
| 开发板 | ESP32-C6-DevKitC-1 或 DevKitM-1 |
| 数据线 | USB-C 数据传输线（非仅充电） |
| 电脑 | Windows / Linux / macOS |
| 框架 | ESP-IDF |

---

## 2. 硬件选型

| 开发板 | 模组 | 特点 |
|--------|------|------|
| **DevKitC-1** | WROOM-1（8 MB） | 通用，WROOM 系列模组 |
| **DevKitM-1** | MINI-1（4 MB） | 超小尺寸 |

---

## 3. 软件环境搭建

### 方式一：一键安装器（推荐）

- 下载 ESP-IDF 安装器（Windows/macOS/Linux）
- 安装时选择 ESP32-C6 支持

### 方式二：手动

```bash
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32c6    # Linux/macOS
# Windows: .\install.ps1 esp32c6
```

### 验证

```bash
idf.py --version
```

---

## 4. 第一个工程

```bash
cp -r $IDF_PATH/examples/get-started/hello_world .
cd hello_world
idf.py set-target esp32c6
idf.py menuconfig
idf.py -p <PORT> flash monitor
```

> `<PORT>`：Windows 为 `COMx`，Linux 为 `/dev/ttyUSB0`，macOS 为 `/dev/cu.usbserial-xxxx`。

---

## 5. 开发板烧录

### 通过 USB-C（推荐）

直接用 USB-C 数据线连接 DevKitC-1 的 **USB OTG 口**，按住 Boot + 按 Reset 进入下载模式：

```bash
idf.py -p /dev/ttyUSB0 flash monitor
```

### 通过 UART

使用板载 USB-UART Bridge 桥接：

```bash
idf.py -p /dev/ttyUSB0 flash monitor
```

---

## 6. 下一步

| 目标 | 文档 |
|------|------|
| Wi-Fi 6 / Matter / Thread 开发 | [11-WiFi6-Matter-Thread-Guide](11-ESP32-C6-WiFi6-Matter-Thread-Guide.md) |
| 低功耗开发 | [10-Low-Power-Mode-Guide](10-ESP32-C6-Low-Power-Mode-Guide.md) |
| 编程 API | [09-Programming-Guide](09-ESP32-C6-Programming-Guide.md) |
| Zigbee 开发 | [12-Zigbee-Development](12-ESP32-C6-Zigbee-Development.md) |
| 安全特性 | [14-Security-Features](14-ESP32-C6-Security-Features.md) |

---

> **文档总结**：ESP32-C6 开发基于 ESP-IDF，一键安装器或手动安装均可。开发板选 DevKitC-1（通用）或 DevKitM-1（超小）。USB-C 数据线直连 OTG 口烧录，按住 Boot + Reset 进入下载模式，`idf.py set-target esp32c6` 后 `flash monitor` 一键烧录监视。入门后按需深入 Wi-Fi 6/Matter/Thread/低功耗/Zigbee/安全各专项文档。
