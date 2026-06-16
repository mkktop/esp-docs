# ESP32-C3-DevKit-RUST（ESP Rust Board）用户指南

> 文档编号：11
> 适用开发板：ESP32-C3-DevKit-RUST-1 / RUST-2（ESP Rust Board）
> 来源：乐鑫官方 esp-dev-kits 用户指南、ESP Rust 培训项目新闻、esp-rs GitHub

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格与板载外设](#2-核心规格与板载外设)
3. [RUST-1 vs RUST-2](#3-rust-1-vs-rust-2)
4. [ESP-RS Rust 生态](#4-esp-rs-rust-生态)
5. [Adafruit Feather 外形](#5-adafruit-feather-外形)
6. [快速入门](#6-快速入门)
7. [Rust 培训项目](#7-rust-培训项目)
8. [应用场景](#8-应用场景)
9. [参考资源](#9-参考资源)

---

## 1. 开发板概述

ESP32-C3-DevKit-RUST（ESP Rust Board）是乐鑫与 Rust 社区共同开发的开发板，专为 **Rust 语言**物联网开发与培训设计。它基于 ESP32-C3 单核 RISC-V MCU（2.4 GHz Wi-Fi + Bluetooth 5 LE），板载 ESP32-C3-MINI-1 模组、6DoF IMU、温湿度传感器、锂离子电池充电器和 USB（Type-C）接口，采用 **Adafruit Feather 外形尺寸**，对面包板十分友好。

RUST-2 是 RUST-1 的升级版，集成温湿度传感器与锂电池充电电路，更适合便携式与传感器驱动应用。

---

## 2. 核心规格与板载外设

| 参数/组件 | 规格 |
|-----------|------|
| 模组 | ESP32-C3-MINI-1（4 MB 封装内 flash） |
| 芯片 | ESP32-C3（RISC-V 32 位单核，160 MHz） |
| 无线 | 2.4 GHz Wi-Fi + Bluetooth 5 (LE) |
| **IMU** | 6DoF（6 轴）惯性测量单元 |
| **温湿度传感器** | 板载（RUST-2 标配） |
| **锂电池充电电路** | 板载，支持便携供电（RUST-2 标配） |
| USB | USB Type-C（I/O，供电+下载+调试） |
| 外形 | Adafruit Feather 兼容 |
| 面包板 | 友好 |

---

## 3. RUST-1 vs RUST-2

| 维度 | RUST-1 | RUST-2 |
|------|--------|--------|
| 模组 | ESP32-C3-MINI-1 | ESP32-C3-MINI-1 |
| 传感器 | IMU | IMU + **温湿度传感器** |
| 供电 | USB | USB + **锂电池充电电路** |
| 用途 | Rust 培训基础 | 便携式 + 传感器驱动应用 |
| 仓库 | esp-rs/esp-rust-board（v1.2） | esp-dev-kits/esp32-c3-devkit-rust-2 |

---

## 4. ESP-RS Rust 生态

ESP Rust Board 配套 ESP-RS Rust 生态：

- **esp-rs**（GitHub）：乐鑫 Rust 支持，包含 std 与 no_std 两条路径
- **_std 路径**：基于 Rust 标准库，类似 ESP-IDF 的开发体验（HTTP/MQTT 等）
- **no_std 路径**：嵌入式裸金属，直接访问寄存器与中断
- **培训手册**：[Ferrous Systems 在线培训](https://github.com/ferrous-systems/espressif-trainings)，分入门与高级两部分

---

## 5. Adafruit Feather 外形

- 兼容 Adafruit Feather 系列尺寸与排针布局
- 可直接插入面包板
- 与 Feather 扩展板（FeatherWings）物理兼容
- 板载外设齐全，培训时**无需处理电线**即可演示 Rust 性能

> 开发板获 CERN 开放式硬件许可证，完全开源。

---

## 6. 快速入门

### 硬件准备

- ESP32-C3-DevKit-RUST-1 或 RUST-2
- USB Type-C 数据线
- 电脑

### Rust 工具链

```bash
# 安装 espup（ESP Rust 工具链管理器）
cargo install espup
espup install
# 安装 ldproxy（链接器代理）
cargo install ldproxy

# 使用 cargo generate 建立项目
cargo generate esp-rs/esp-idf-template
```

### 烧录

```bash
# std 项目
cargo espflash flash --monitor

# 或用 ESP-IDF（与传统 C 开发一致）
idf.py set-target esp32c3
idf.py -p <PORT> flash monitor
```

> Rust 开发详见 [16-ESP-Rust-on-ESP32-C3](16-ESP-Rust-on-ESP32-C3.md)。

---

## 7. Rust 培训项目

乐鑫携手 **Ferrous Systems** 推出的 ESP32-C3 Rust 培训项目，以一本在线培训手册为材料，可自学或小组培训，含编程练习与 Troubleshooting。

### 入门部分（基于 Rust 标准库 std）

1. 用 cargo generate 建立项目
2. 编写 HTTP 客户端
3. 编写 HTTP 服务器
4. 编写 MQTT 客户端：发布传感器数据、接收订阅命令

### 高级部分（基于 no_std，嵌入式 Rust）

1. 通过 I2C 读取温湿度传感器
2. 通过同一 I2C 总线读取 IMU
3. I2C 驱动介绍
4. 用按钮处理中断

### 参与条件

- 掌握基础 Rust（Rust Book 前 6 章，第 4 章 Ownership 可不深究）
- 高级部分需嵌入式基础 + no_std Rust 经验
- 硬件：ESP Rust Board 或 ESP32-C3-DevKitC-02

---

## 8. 应用场景

- Rust 语言 IoT 开发学习与培训
- 便携式传感器应用（RUST-2，电池供电）
- IMU / 温湿度数据采集
- 低功耗 Wi-Fi + BLE Rust 项目原型
- Feather 生态扩展（FeatherWing）

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| DevKit-RUST-2 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-devkit-rust-2/user_guide.html |
| ESP Rust Board GitHub（esp-rs） | https://github.com/esp-rs/esp-rust-board |
| Rust 培训项目新闻 | https://www.espressif.com/zh-hans/news/ESP_RUST_training |
| Ferrous Systems 培训手册 | https://github.com/ferrous-systems/espressif-trainings |
| ESP-RS 组织（Rust 生态） | https://github.com/esp-rs |
| esp-idf-template（项目模板） | https://github.com/esp-rs/esp-idf-template |
| espup 工具链管理器 | https://github.com/esp-rs/espup |

---

> **文档总结**：ESP32-C3-DevKit-RUST（ESP Rust Board）是乐鑫与 Rust 社区联合打造、专为 Rust 语言 IoT 开发与培训的开发板，基于 ESP32-C3-MINI-1，Adafruit Feather 外形、面包板友好。RUST-2 升级版集成 IMU、温湿度传感器与锂电池充电电路，支持便携传感器应用。配套 ESP-RS Rust 生态（std/no_std 双路径）与 Ferrous Systems 培训手册（入门 HTTP/MQTT + 高级 I2C/中断），是 ESP32-C3 上用 Rust 进行物联网开发的首选硬件。开源（CERN OHL）。
