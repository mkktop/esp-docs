# ESP-AT 在 ESP32-C3 上的使用指南

> 文档编号：17
> 适用芯片：ESP32-C3（作为 AT 固件运行设备，被外部主机通过 AT 指令控制）
> 来源：乐鑫官方 ESP-AT 文档（esp32c3）、AT 命令集、Sleep AT 示例

---

## 目录

1. [ESP-AT 概述](#1-esp-at-概述)
2. [ESP32-C3 上的 AT 固件](#2-esp32-c3-上的-at-固件)
3. [AT 命令集](#3-at-命令集)
4. [Wi-Fi 相关 AT 命令](#4-wi-fi-相关-at-命令)
5. [BLE 相关 AT 命令](#5-ble-相关-at-命令)
6. [低功耗 AT 命令（Sleep）](#6-低功耗-at-命令sleep)
7. [烧录 AT 固件](#7-烧录-at-固件)
8. [典型应用](#8-典型应用)
9. [参考资源](#9-参考资源)

---

## 1. ESP-AT 概述

ESP-AT 是乐鑫提供的**AT 指令固件**，烧录到 ESP32-C3 后，设备可作为**外部主机 MCU 的 Wi-Fi/BLE 通信协处理器**。主机 MCU（如 STM32、PLC）通过 UART 串口发送文本 AT 指令，即可让 ESP32-C3 完成 Wi-Fi 连网、TCP/UDP/MQTT/HTTP 通信、BLE 收发等，无需自己实现复杂网络协议栈。

ESP-AT 让 ESP32-C3 化身为一个**低成本的 Wi-Fi/BLE 模块**（类似传统 WiFi 模块用法）。

---

## 2. ESP32-C3 上的 AT 固件

ESP-AT 为 ESP32-C3 提供专门的固件与文档（`esp-at/esp32c3` 路径）。特性：

- 支持 Wi-Fi STA/SoftAP
- 支持 BLE 5（外设/中心角色）
- 支持 TCP/UDP/SSL/MQTT/HTTP/MQTT
- 支持低功耗模式（Modem-sleep / Light-sleep / Deep-sleep）
- 默认通过 UART0 收发 AT 命令

---

## 3. AT 命令集

AT 命令格式：

| 类型 | 格式 | 示例 |
|------|------|------|
| 测试 | `AT+<cmd>=?` | `AT+CWMODE=?` |
| 查询 | `AT+<cmd>?` | `AT+CWMODE?` |
| 设置 | `AT+<cmd>=<params>` | `AT+CWMODE=1` |
| 执行 | `AT+<cmd>` | `AT+RST` |

常用基础命令：

| 命令 | 作用 |
|------|------|
| `AT` | 测试 |
| `AT+RST` | 重启 |
| `AT+GMR` | 查版本 |
| `AT+RESTORE` | 恢复出厂 |
| `ATE0` / `ATE1` | 关/开回显 |

---

## 4. Wi-Fi 相关 AT 命令

| 命令 | 作用 |
|------|------|
| `AT+CWMODE=<mode>` | 设置 Wi-Fi 模式（1=STA, 2=AP, 3=STA+AP） |
| `AT+CWJAP="<ssid>","<pwd>"` | 连接 AP |
| `AT+CWQAP` | 断开 AP |
| `AT+CIPSTA?` | 查询 STA IP |
| `AT+CWSAP` | 设置 SoftAP |
| `AT+CWLAP` | 扫描 AP |
| `AT+CIPSTART` | 建立 TCP/UDP/SSL 连接 |
| `AT+CIPSEND` | 发送数据 |
| `AT+CIPSERVER` | 建立 TCP server |
| `AT+CIPMUX` | 多连接模式 |
| `AT+MQTTUSERCFG` | MQTT 配置 |
| `AT+MQTTCONN` | MQTT 连接 |
| `AT+MQTTPUB` | MQTT 发布 |

---

## 5. BLE 相关 AT 命令

| 命令 | 作用 |
|------|------|
| `AT+BLEINIT=<role>` | 初始化 BLE（2=服务端/peripheral） |
| `AT+BLEADDR?` | 查询 BLE 地址 |
| `AT+BLEADVPARAM` | 设置广播参数 |
| `AT+BLEADVSTART` / `AT+BLEADVSTOP` | 开始/停止广播 |
| `AT+BLEGATTS...` | GATT 服务端操作 |
| `AT+BLEGATTC...` | GATT 客户端操作 |

---

## 6. 低功耗 AT 命令（Sleep）

ESP32-C3 采用先进电源管理，ESP-AT 支持四种功耗模式：

| 模式 | AT 设置值 | 说明 |
|------|-----------|------|
| Active | 默认 | 射频工作 |
| **Modem-sleep** | `AT+SLEEP=1` | CPU 运行，Wi-Fi/BT 基带与射频关闭（RF 按 AP DTIM 定期开） |
| **Light-sleep** | `AT+SLEEP=2` | CPU 暂停，RTC 工作，按监听间隔定期唤醒 RF |
| **Deep-sleep** | `AT+SLEEP=3` | CPU 与大部分外设掉电，仅 RTC 工作 |

### 示例：Wi-Fi Modem-sleep

```
AT+CWMODE=1
AT+CWJAP="espressif","1234567890"
AT+SLEEP=1
```
> RF 将根据 AP 的 DTIM 定期关闭。

### 示例：Wi-Fi Light-sleep（监听间隔 3）

```
AT+CWMODE=1
AT+CWJAP="espressif","1234567890",,,,3
AT+SLEEP=2
```
> CPU 自动休眠，RF 按 CWJAP 设置的监听间隔定期关闭。

### 示例：BLE 广播态 Light-sleep

```
AT+BLEINIT=2
AT+BLEADVPARAM=1600,1600,0,0,7,0,0,"00:00:00:00:00:00"   // 1 s 间隔
AT+BLEADVSTART
AT+CWINIT=0          // 禁用 Wi-Fi
AT+SLEEP=2
```

> 单 BLE Light-sleep 配置外部 32 kHz 晶振可降至 µA 级；无外部 32 kHz 则约 mA 级。

---

## 7. 烧录 AT 固件

```bash
# 下载官方 ESP32-C3 AT 固件 bin
# 或自行编译：
git clone --recursive https://github.com/espressif/esp-at.git
cd esp-at
# build.py 配置芯片为 esp32c3
./build.py menuconfig
./build.py build
./build.py -p <PORT> flash
```

烧录后，用串口工具（115200 8N1）发送 `AT` 应返回 `OK`。

---

## 8. 典型应用

| 应用 | 说明 |
|------|------|
| MCU + C3 联网模块 | 主机 MCU（STM32/PLC）通过 AT 让 C3 联网 |
| 低功耗数据采集 | C3 周期醒来连 Wi-Fi 上传，其余 Deep-sleep |
| BLE 网关 | C3 作 BLE 外设/中心，经 AT 与主机交互 |
| 替代传统 WiFi 模块 | ESP8266 AT 升级到 C3 AT（获 BLE + 更强安全） |
| MQTT/HTTP IoT 节点 | AT+MQTT/AT+CIPSTART 实现 IoT 上云 |

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-AT 文档（ESP32-C3 中文） | https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32c3/index.html |
| AT 命令集 | https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32c3/AT_Command_Set/index.html |
| Sleep AT 示例 | https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32c3/AT_Command_Examples/sleep_at_examples.html |
| Wi-Fi AT 命令 | https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32c3/AT_Command_Set/Wi-Fi_AT_Commands.html |
| BLE AT 命令 | https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32c3/AT_Command_Set/BLE_AT_Commands.html |
| ESP-AT GitHub | https://github.com/espressif/esp-at |
| AT 固件下载 | https://www.espressif.com/zh-hans/support/download/at |

---

> **文档总结**：ESP-AT 是乐鑫的 AT 指令固件，把 ESP32-C3 变成外部主机 MCU 的 Wi-Fi/BLE 通信协处理器，主机经 UART 发 AT 文本命令即可完成联网、TCP/UDP/MQTT/HTTP、BLE 收发，无需自实现协议栈。支持 Wi-Fi STA/SoftAP、BLE 5、SSL 与四种低功耗模式（Modem-sleep/Light-sleep/Deep-sleep，经 `AT+SLEEP=1/2/3` 设置）。ESP32-C3 是替代传统 ESP8266 AT 模块、为 MCU 增加低成本 Wi-Fi+BLE 能力的理想方案。
