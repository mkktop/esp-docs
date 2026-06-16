# ESP32-C6 安全特性详解

> 文档编号：14
> 适用芯片：ESP32-C6
> 来源：乐鑫官方 Datasheet、TRM

---

## 目录

1. [安全特性概述](#1-安全特性概述)
2. [安全启动（Secure Boot）](#2-安全启动secure-boot)
3. [Flash 加密](#3-flash-加密)
4. [TEE 可信执行环境](#4-tee-可信执行环境)
5. [加密加速器](#5-加密加速器)
6. [与 C3 安全对比](#6-与-c3-安全对比)
7. [参考资源](#7-参考资源)

---

## 1. 安全特性概述

ESP32-C6 安全机制比 C3 更强，新增 TEE 和 AES-256：

| 特性 | ESP32-C6 | ESP32-C3 |
|------|:---------:|:--------:|
| Secure Boot | RSA-3072 | RSA-3072 |
| Flash 加密 | **AES-128/256-XTS** | AES-128-XTS |
| **TEE** | **✓** | ✗ |
| ECC | ✓ | ✗ |
| SHA-256 | ✓ | ✓ |
| RSA | ✓ | ✓ |
| HMAC | ✓ | ✓ |
| RNG | ✓ | ✓ |
| 时钟毛刺检测 | ✓ | ✓ |

---

## 2. 安全启动（Secure Boot）

基于 **RSA-3072** 验签，确保只有签名固件可启动：

- ROM bootloader 验签二级 bootloader
- 二级 bootloader 验签应用程序
- eFuse 中烧写公钥哈希

---

## 3. Flash 加密

基于 **AES-128/256-XTS**（C3 仅 AES-128-XTS）：

- 支持 AES-256-XTS（更强）
- 加密范围可配置
- OTP 中的密钥安全存储

---

## 4. TEE 可信执行环境

**ESP32-C6 新增特性**，硬件隔离关键软件区域：

- 将 SoC 资源（内存/外设）分配给 Trusted OS
- 非 Trusted 代码无法访问 TEE 资源
- 适用于：安全存储、支付、数字版权

---

## 5. 加密加速器

| 加速器 | 算法 |
|--------|------|
| AES | AES-128/256 |
| SHA | SHA-256 / SHA-384 / SHA-512 |
| ECC | 椭圆曲线加密（比 RSA 更高效） |
| RSA | RSA-3072 |
| HMAC | — |
| RNG | 真随机数 |

---

## 6. 与 C3 安全对比

| 特性 | C6 | C3 |
|------|:--:|:--:|
| TEE | ✓（新增） | ✗ |
| Flash 加密 | AES-**256**-XTS | AES-128-XTS |
| ECC | ✓（新增） | ✗ |
| 适用场景 | 高安全 IoT / 支付 / DRM | 标准 IoT 安全 |

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6 Datasheet 安全章节 | https://documentation.espressif.com/esp32-c6_datasheet_cn.html |
| 安全启动指南 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/security/index.html |

---

> **文档总结**：ESP32-C6 安全在 C3 基础上全面升级：TEE 可信执行环境（硬件隔离关键软件）、AES-256-XTS Flash 加密（更强）、ECC 硬件加速（比 RSA 更高效）。适合高安全需求场景（支付/DRM/门锁）。配合 [01-Datasheet](01-ESP32-C6-Datasheet.md) 第 7 节使用。
