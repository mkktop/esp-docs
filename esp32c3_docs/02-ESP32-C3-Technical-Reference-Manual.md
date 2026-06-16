# ESP32-C3 技术参考手册详解

> 文档编号：02
> 适用范围：ESP32-C3 全系列芯片
> 来源：乐鑫官方《ESP32-C3 技术参考手册》、ESP-IDF 编程指南（esp32c3）、esp-risc-v CPU 文档

---

## 目录

1. [技术参考手册概述](#1-技术参考手册概述)
2. [ESP-RISC-V CPU](#2-esp-risc-v-cpu)
3. [地址分布与 CSR](#3-地址分布与-csr)
4. [中断与异常](#4-中断与异常)
5. [芯片 Boot 控制](#5-芯片-boot-控制)
6. [低功耗管理](#6-低功耗管理)
7. [外设与通信接口](#7-外设与通信接口)
8. [存储与 Flash](#8-存储与-flash)
9. [eFuse 控制器](#9-efuse-控制器)
10. [USB 串口/JTAG 控制器](#10-usb-串口jtag-控制器)
11. [参考资源](#11-参考资源)

---

## 1. 技术参考手册概述

《ESP32-C3 技术参考手册》（TRM）是 ESP32-C3 芯片的**最详尽使用说明**，提供存储器和外设的详细使用方法。如果说 datasheet 是"芯片是什么"，TRM 就是"芯片怎么用"。

TRM 按章节组织，核心章节包括：ESP-RISC-V CPU、复位和时钟、芯片 Boot 控制、eFuse、IO MUX 与 GPIO、各种外设控制器（SPI、I2C、UART、I2S、TWAI、LED PWM、RMT、ADC、GDMA 等）、RTC 与低功耗管理、安全（AES/SHA/RSA/HMAC/DS）、USB Serial/JTAG 等。

> 本章重点抽取开发者最常查询的几块内容。完整细节请直接查阅官方 PDF。

---

## 2. ESP-RISC-V CPU

### 2.1 架构

ESP-RISC-V CPU 是基于 RISC-V ISA 的 **32 位内核**，包括：
- 基本整数（I）
- 乘法/除法（M）
- 压缩（C）

标准扩展。内核为 **4 级有序标量流水线**，针对面积、功耗、性能优化。CPU 包含中断控制器（INTC）、调试模块（DM）和系统总线（SYS BUS）接口。

### 2.2 特性

- 时钟高达 160 MHz
- 通过 IRAM/DRAM 接口**零等待**访问片上 SRAM 与 cache
- INTC：多达 **31 个向量中断**，可配置优先级与阈值
- 调试模块符合 **RISC-V Debug v0.13**，支持 JTAG/USB 端口
- 调试器经系统总线（SBA）直接访问存储器与外设
- **8 个硬件断点/观察点**触发器
- 物理内存保护（PMP），最多 **16 个区域**
- 32 位 AHB 系统总线
- 可配置核心性能指标事件

---

## 3. 地址分布与 CSR

### 3.1 CPU 地址分布

| 名称 | 描述 | 起始地址 | 结束地址 | 访问 |
|------|------|----------|----------|------|
| IRAM | 指令地址空间 | 0x4000_0000 | 0x47FF_FFFF | 读/写 |
| DRAM | 数据地址空间 | 0x3800_0000 | 0x3FFF_FFFF | 读/写 |
| DM | 调试地址空间 | 0x2000_0000 | 0x27FF_FFFF | 读/写 |
| AHB | AHB 地址空间 | IRAM/DRAM/DM 之外 | — | 读/写 |

### 3.2 CSR（配置与状态寄存器）

ESP-RISC-V 实现了标准 RISC-V 特权架构 CSR + 自定义 CSR：

| 类别 | 代表 CSR | 说明 |
|------|----------|------|
| 机器模式信息 | mvendorid / marchid / mimpid / mhartid | 厂商/架构/实现/线程号 |
| 异常设置 | mstatus / misa / mtvec | 状态/ISA/异常向量 |
| 异常处理 | mscratch / mepc / mcause / mtval | 暂存/返回PC/原因/值 |
| PMP | pmpcfg0~3 / pmpaddr0~15 | 物理内存保护（16 区域） |
| 触发器 | tselect / tdata1 / tdata2 / tcontrol | 断点/观察点 |
| 调试模式 | dcsr / dpc / dscratch0/1 | 调试控制 |
| 性能计数器（自定义） | mpcer / mpcmr / mpccr | 程序计数器事件 |
| **GPIO 访问（自定义）** | cpu_gpio_oen / cpu_gpio_in / cpu_gpio_out | 通过 CSR 直接读写 GPIO |

> ⚠️ 对只读 CSR 执行写入/置位/清除操作会触发**非法指令异常**。

---

## 4. 中断与异常

- INTC 支持 **31 个向量中断**，每个可配置优先级与阈值级别
- 异常处理寄存器：mepc（异常返回 PC）、mcause（异常原因）、mtval（异常值）、mtvec（异常向量基址）
- 调试模块提供 8 个硬件触发器，可用于断点/观察点

---

## 5. 芯片 Boot 控制

### 5.1 Strapping 管脚

ESP32-C3 共有 **GPIO2、GPIO8、GPIO9** 三个 strapping 管脚（GPIO9 默认内部上拉，其余浮空），复位时被采样锁存，复位后作为普通 GPIO。

### 5.2 系统启动模式

| 启动模式 | GPIO9 | GPIO8 | GPIO3 | GPIO2 |
|----------|:-----:|:-----:|:-----:|:-----:|
| **SPI Boot（默认）** | 1 | x | x | x |
| Joint Download Boot | 0 | 1 | x | x |
| SPI Download Boot | 0 | 0 | 0 | 1 |
| 无效组合（应避免） | 0 | 0 | x | 0 |

#### SPI Boot 细分

- **常规 flash 启动**：支持安全启动，ROM bootloader 从 flash 加载二级引导程序，再启动应用程序
- **直接启动（Direct Boot）**：不支持安全启动，程序直接从 flash 运行（需 bin 前 2 字为 `0xaedb041d`）

#### Joint Download Boot

支持两种下载方式：
- USB-Serial-JTAG Download Boot
- UART Download Boot

### 5.3 控制 Boot 行为的 eFuse

| eFuse | 作用 |
|-------|------|
| EFUSE_DIS_FORCE_DOWNLOAD | =1 禁用软件强制 Download Boot |
| EFUSE_DIS_DOWNLOAD_MODE | =1 禁用 Joint Download Boot |
| EFUSE_ENABLE_SECURITY_DOWNLOAD | =1 仅允许明文 flash 读写擦除 |
| EFUSE_DIS_DIRECT_BOOT | =1 禁用 Direct Boot |

### 5.4 ROM 日志打印控制

ROM 启动日志可输出到 UART0 / USB Serial/JTAG，由 `EFUSE_UART_PRINT_CONTROL` + GPIO8、`EFUSE_USB_PRINT_CHANNEL` 控制（默认同时打印 UART0 和 USB）。

> ⚠️ **GPIO8 与 GPIO9 不可同时为低电平**。

---

## 6. 低功耗管理

ESP32-C3 的低功耗管理由以下模块实现：

| 模块 | 作用 |
|------|------|
| **PMU（功耗管理单元）** | 控制向模拟/RTC/数字电源域供电 |
| 电源隔离单元 | 保证各电源域独立工作 |
| 低功耗时钟 | 为低功耗电源域提供时钟 |
| RTC 定时器 | 48 位 always-on 计数器，RTC 时钟下记录事件 |
| 8 个 32 位 always-on 保留寄存器 | 不受 deep-sleep 影响，存不可丢失数据 |
| **6 个 always-on 管脚** | 不受 deep-sleep 影响，可作唤醒源或普通 GPIO |
| RTC 快速内存 | 8 KB SRAM，与 CPU_CLK 同频 |
| 调压器 | 数字系统调压器（3.3V→1.1V）+ 低功耗调压器（3.3V→1.1V） |

### 6.1 时钟源

| 时钟类型 | 可选时钟源 | 作用域 |
|----------|-----------|--------|
| RTC 快速时钟 | RC_FAST_CLK 的 n 分频（默认）、XTAL_DIV_CLK | RTC 寄存器 |
| RTC 慢速时钟 | XTAL32K_CLK、RC_FAST_DIV_CLK、RC_SLOW_CLK（默认） | 功耗管理系统 |
| Wireless 时钟 | XTAL32K / RC_FAST 分频 / RTC_SLOW / XTAL | 低功耗下 Wi-Fi/BT |

### 6.2 欠压检测器（Brownout）

检查 VDD3P3_RTC / VDD3P3_CPU / VDDA1 / VDDA2 电压，低于阈值（默认 2.7 V）时触发，可选芯片复位或系统复位。

> 低功耗详细模式与唤醒源见 [15-ESP32-C3-Low-Power-Mode-Guide](15-ESP32-C3-Low-Power-Mode-Guide.md)。

---

## 7. 外设与通信接口

ESP32-C3 集成的外设控制器（TRM 各章节）：

| 外设 | 说明 |
|------|------|
| **GDMA** | 通用 DMA，3 RX + 3 TX 通道，支持 SPI/I2S/UART/AES/SHA 等 |
| **SPI** | SPI0/1（flash）、SPI2（通用 FSPI） |
| **I2C** | 1 个，支持主/从 |
| **UART** | 2 个（UART0/UART1） |
| **I2S** | 1 个，含 DMA 支持，PDM 兼容 |
| **TWAI®** | 兼容 ISO 11898-1（CAN 2.0） |
| **LED PWM** | 6 通道，可调频率与占空比 |
| **RMT** | 红外收发，2 TX + 2 RX |
| **通用定时器** | 2 个 54 位 |
| **系统定时器** | 52 位 |
| **ADC** | 2 个 12 位 SAR ADC，6 通道 |
| **温度传感器** | 内置 |
| **GPIO / IO MUX** | GPIO 交换矩阵 + IO MUX |
| **eFuse 控制器** | 见第 9 节 |
| **USB Serial/JTAG** | 见第 10 节 |
| **World Controller** | 双隔离执行环境支持 |

---

## 8. 存储与 Flash

- **ROM** 384 KB：存 bootloader 与内核功能
- **SRAM** 400 KB：含 16 KB cache 专用
- **RTC SRAM** 8 KB：低功耗模式数据保持
- **Flash 访问**：通过 SPI0/1 + cache 加速，支持 SPI/Dual SPI/Quad SPI/QPI；支持 ICP（在线编程）

---

## 9. eFuse 控制器

eFuse 是 **4096 位**一次性可编程存储器，用户可用高达 **1792 位**。eFuse 只能烧写一次（0→1 不可逆），用于：

- 启动模式控制（见第 5 节）
- ROM 日志打印控制
- 安全启动密钥、flash 加密密钥
- 芯片版本、禁用功能开关
- 自定义系统参数

烧写参考 TRM「eFuse 控制器」章节，或用 `espefuse.py` 工具。

---

## 10. USB 串口/JTAG 控制器

ESP32-C3 内置全速 **USB Serial/JTAG 控制器**（GPIO18=D-、GPIO19=D+）：

| 功能 | 说明 |
|------|------|
| 固件下载 | 免外部 USB 转串口芯片，直接 USB 烧录 |
| 串口日志 | USB CDC 串口输出（需 menuconfig 开启 USB CDC on Boot） |
| JTAG 调试 | 原生 JTAG，无需外部调试器，支持 RISC-V 硬件断点 |
| 强制 Boot 模式切换 | 可在 SPI Boot 与 Joint Download Boot 间强制切换 |

> 对应 strapping 默认从 USB 下载：GPIO2/8 拉高、GPIO9 拉低。

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3 技术参考手册（PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c3_technical_reference_manual_cn.pdf |
| ESP32-C3 技术参考手册（在线） | https://documentation.espressif.com/esp32-c3_technical_reference_manual_cn.html |
| ESP-IDF 编程指南（ESP32-C3） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/index.html |
| ESP32-C3 H/W 硬件参考 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/hw-reference/index.html |
| esp-risc-v CPU 仓库 | https://github.com/espressif/esp-riscv-elf |
| espefuse 工具 | https://docs.espressif.com/projects/esptool/en/latest/esp32c3/espefuse/index.html |

---

> **文档总结**：ESP32-C3 技术参考手册（TRM）是芯片存储器与外设的权威使用指南。核心内容包括：ESP-RISC-V 32 位 RV32IMC 单核 CPU（160 MHz，PMP 16 区域，8 断点）；Boot 控制（GPIO2/8/9 strapping，SPI/Joint Download/SPI Download 三种启动模式，eFuse 控制 ROM 日志与下载行为）；低功耗管理（PMU + RTC 定时器 + 6 个 always-on 管脚 + 8 个 always-on 寄存器 + 双调压器 + Brownout）；丰富外设控制器（GDMA/SPI/I2C/UART/I2S/TWAI/LED PWM/RMT/ADC 等）；内置 USB Serial/JTAG（免调试器下载与 JTAG）。配合 datasheet 构成 C3 开发的完整硬件依据。
