# ESP-Rust 在 ESP32-C3 上的开发详解

> 文档编号：16
> 适用芯片：ESP32-C3（RISC-V 架构，Rust 友好）
> 来源：乐鑫官方 Rust 培训项目、esp-rs GitHub、Ferrous Systems 培训手册

---

## 目录

1. [为什么在 ESP32-C3 上用 Rust](#1-为什么在-esp32-c3-上用-rust)
2. [ESP-RS 生态总览](#2-esp-rs-生态总览)
3. [std 与 no_std 两条开发路径](#3-std-与-no_std-两条开发路径)
4. [工具链安装](#4-工具链安装)
5. [第一个 Rust 工程](#5-第一个-rust-工程)
6. [常用 esp-rs 库](#6-常用-esp-rs-库)
7. [Rust 培训项目](#7-rust-培训项目)
8. [调试](#8-调试)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. 为什么在 ESP32-C3 上用 Rust

ESP32-C3 采用 **RISC-V 架构**，对 Rust 语言天然友好（Rust 一等支持 RISC-V）。在嵌入式领域用 Rust 的优势：

- **内存安全**：编译期消除空指针/缓冲区溢出/数据竞争（无 GC）
- **零成本抽象**：性能与 C 相当
- **现代工具链**：cargo 包管理、丰富的 crate 生态
- **错误前移**：把运行时错误转移到编译期

ESP32-C3 是乐鑫 Rust 支持的**首发与主力芯片**（C3 + esp-rs 工具链最成熟）。

---

## 2. ESP-RS 生态总览

[esp-rs](https://github.com/esp-rs) 是乐鑫与 Rust 社区共同维护的 ESP32 Rust 支持组织，核心项目：

| 项目 | 作用 |
|------|------|
| **espup** | ESP Rust 工具链管理器（安装 Rust + Xtensa/RISC-V 工具链） |
| **esp-idf-template** | std 项目模板（cargo generate） |
| **esp-idf-svc** | ESP-IDF 服务封装（Wi-Fi/BLE/网络/HTTP 等） |
| **esp-idf-hal** | 外设 HAL（GPIO/I2C/SPI/UART 等） |
| **esp-idf-sys** | ESP-IDF FFI 绑定 |
| **esp-riscv-rt** | RISC-V 运行时（no_std） |
| **esp-hal** | no_std 裸金属 HAL |
| **esp-backtrace** | panic 回溯 |
| **esp-println** | 串口打印 |
| **espflash / cargo-espflash** | Rust 烧录工具 |

---

## 3. std 与 no_std 两条开发路径

| 路径 | 基础 | 特点 | 适用 |
|------|------|------|------|
| **std** | Rust 标准库 + ESP-IDF | 有 std（线程/集合/IO），可用 esp-idf-svc 调 Wi-Fi/HTTP/MQTT | 入门、联网应用、快速开发 |
| **no_std** | 裸金属 | 无标准库，直接访问寄存器，体积小 | 高级、超低资源、硬实时 |

ESP32-C3（RISC-V）两条路径都支持良好。

---

## 4. 工具链安装

```bash
# 1. 安装 espup（ESP Rust 工具链管理器）
cargo install espup
espup install
# macOS/Linux: source ~/export-esp.sh

# 2. 安装 ldproxy（链接器代理）
cargo install ldproxy

# 3. 安装烧录工具
cargo install espflash cargo-espflash
```

> espup 会安装带 RISC-V target 的 Rust 工具链（ESP32-C3 是 RISC-V，不需要 Xtensa LLVM）。

---

## 5. 第一个 Rust 工程

### 5.1 std 工程（推荐入门）

```bash
cargo generate esp-rs/esp-idf-template
# 按提示选择：
#   MCU: esp32c3
#   标准库 std
```

生成后：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // 连 Wi-Fi、起 HTTP server、读 GPIO 等
    loop {
        println!("Hello from ESP32-C3 Rust!");
        thread::sleep(Duration::from_secs(1));
    }
}
```

烧录：

```bash
cargo espflash flash --monitor
# 或
cargo espflash flash --release --monitor /dev/ttyUSB0
```

### 5.2 no_std 工程

```rust
#![no_std]
#![no_main]

use esp_backtrace as _;
use esp_println::println;
use esp_riscv_rt::entry;

#[entry]
fn main() -> ! {
    let peripherals = esp32c3::Peripherals::take().unwrap();
    // 直接操作寄存器 / esp-hal 外设
    loop {
        println!("Hello no_std!");
    }
}
```

---

## 6. 常用 esp-rs 库

| crate | 用途 |
|-------|------|
| **esp-idf-svc** | Wi-Fi 连接、HTTP server/client、MQTT、BLE、NVS、定时器 |
| **esp-idf-hal** | GPIO/I2C/SPI/UART/ADC/PWM/RMT 外设 |
| **embedded-hal** | 跨平台 HAL trait（与传感器 crate 通用） |
| **esp-println** | `println!` 串口/USB 打印 |
| **log / esp-logger** | 日志 |
| **embedded-svc / wifi** | Wi-Fi 抽象 |
| **mqtt-async / rumqttc** | MQTT 客户端 |

---

## 7. Rust 培训项目

乐鑫携手 **Ferrous Systems** 推出的 ESP32-C3 Rust 培训，使用 **ESP Rust Board**（[DevKit-RUST](11-ESP32-C3-DevKit-RUST-User-Guide.md)）。

### 入门部分（std）

1. cargo generate 建立项目
2. 编写 HTTP 客户端
3. 编写 HTTP 服务器
4. 编写 MQTT 客户端（发布传感器数据、接收命令）

### 高级部分（no_std）

1. 通过 I2C 读取温湿度传感器
2. 通过同一 I2C 读 IMU
3. I2C 驱动介绍
4. 用按钮处理中断

### 参与条件

- Rust Book 前 6 章基础（第 4 章 Ownership 可不深究）
- 高级部分需嵌入式 + no_std 经验
- 硬件：ESP Rust Board 或 ESP32-C3-DevKitC-02

---

## 8. 调试

ESP32-C3 内置 **USB Serial/JTAG**，Rust 原生支持 JTAG 调试（无需外部调试器）：

```bash
# probe-rs 调试
cargo run --probe
```

配合 VSCode + probe-rs / rust-analyzer 可断点调试 Rust 固件。

---

## 9. 应用场景

- 安全性要求高的 IoT 固件（内存安全）
- Rust 学习者入门嵌入式
- 快速原型（std 路径，类似应用开发）
- 硬实时/超小固件（no_std）
- 跨平台 HAL 复用（embedded-hal 传感器 crate）

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| esp-rs 组织 | https://github.com/esp-rs |
| Rust 培训项目新闻 | https://www.espressif.com/zh-hans/news/ESP_RUST_training |
| Ferrous Systems 培训手册 | https://github.com/ferrous-systems/espressif-trainings |
| esp-idf-template | https://github.com/esp-rs/esp-idf-template |
| espup | https://github.com/esp-rs/espup |
| esp-idf-svc | https://github.com/esp-rs/esp-idf-svc |
| esp-idf-hal | https://github.com/esp-rs/esp-idf-hal |
| espflash | https://github.com/esp-rs/espflash |
| The Rust Book | https://doc.rust-lang.org/book/ |
| ESP Rust Board（DevKit-RUST） | https://github.com/esp-rs/esp-rust-board |

---

> **文档总结**：ESP32-C3 采用 RISC-V 架构，对 Rust 天然友好，是乐鑫 Rust 支持的主力芯片。ESP-RS 生态提供两条开发路径：std（基于标准库 + ESP-IDF，适合联网应用快速开发）与 no_std（裸金属，适合硬实时/超小固件）。工具链经 espup 一键安装，esp-idf-template 生成项目，cargo espflash 烧录，原生 USB/JTAG 调试。配套 Ferrous Systems 培训手册（入门 HTTP/MQTT + 高级 I2C/中断）与 ESP Rust Board 开发板，是从内存安全 IoT 固件到 Rust 嵌入式学习的完整方案。
