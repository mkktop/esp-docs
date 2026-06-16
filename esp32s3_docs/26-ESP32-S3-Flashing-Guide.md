# ESP32-S3 固件烧录指南

> 本文档全面介绍 ESP32-S3 的固件烧录方法，包括下载模式、esptool 使用、UART 烧录、USB Serial/JTAG 烧录、分区表、Flash 加密、Secure Boot 烧录以及常见烧录错误排查。

---

## 目录

1. [烧录概述](#1-烧录概述)
2. [下载模式](#2-下载模式)
3. [硬件连接](#3-硬件连接)
4. [esptool 工具](#4-esptool-工具)
5. [通过 UART 烧录](#5-通过-uart-烧录)
6. [通过 USB Serial/JTAG 烧录](#6-通过-usb-serialjtag-烧录)
7. [通过 USB OTG (DFU) 烧录](#7-通过-usb-otg-dfu-烧录)
8. [分区表与 Flash 布局](#8-分区表与-flash-布局)
9. [Flash 加密烧录](#9-flash-加密烧录)
10. [Secure Boot 烧录](#10-secure-boot-烧录)
11. [量产烧录工具](#11-量产烧录工具)
12. [常见烧录错误与排查](#12-常见烧录错误与排查)
13. [Flash 操作命令](#13-flash-操作命令)

---

## 1. 烧录概述

ESP32-S3 支持三种固件下载方式：

| 下载方式 | 接口 | 速度 | 适用场景 |
|----------|------|------|---------|
| UART 下载 | UART0 (GPIO43/44) | 最高 921600 bps | 通用，需要 USB-UART 桥接 |
| USB Serial/JTAG | USB (GPIO19/20) | 数倍于 UART | 推荐，无需外部桥接芯片 |
| USB DFU | USB OTG (GPIO19/20) | 快速 | 需要 USB OTG 外设 |

---

## 2. 下载模式

### 2.1 启动模式选择

ESP32-S3 通过 Strapping 管脚决定启动模式：

| GPIO0 | GPIO46 | 启动模式 |
|-------|--------|---------|
| 高 (默认) | 低 (默认) | **SPI Flash Boot** (正常启动) |
| 低 | 低 | **Download Boot** (下载模式, USB/UART0) |

### 2.2 自动进入下载模式

当使用 USB Serial/JTAG 或带有自动复位电路的 UART 桥接芯片时，esptool 会自动通过 DTR/RTS 信号控制芯片复位并进入下载模式。用户无需手动操作。

### 2.3 手动进入下载模式

如果自动下载不工作，按以下步骤手动进入下载模式：

1. **按住 BOOT 按键**（连接 GPIO0 到 GND）
2. **短按 RST 按键**（拉低 CHIP_PU 然后释放）
3. **释放 BOOT 按键**
4. 此时芯片应处于下载模式，串口输出 `waiting for download`

### 2.4 验证下载模式

进入下载模式后，UART0 会输出：

```
ESP-ROM:esp32s3-20210327
Build:Mar 27 2021
rst:0x1 (POWERON),boot:0x0 (DOWNLOAD(USB/UART0))
waiting for download
```

---

## 3. 硬件连接

### 3.1 UART 下载接线

| ESP32-S3 | 3.3V 外部电源 | 串口调试工具 |
|----------|-------------|------------|
| 3V3 | VDD | |
| GND | GND | GND |
| EN (CHIP_PU) | VDD | |
| GPIO0 (拉低) | GND | |
| GPIO46 (拉低) | GND | |
| TXD0 (GPIO43) | | RXD |
| RXD0 (GPIO44) | | TXD |

> **Strapping 管脚 GPIO45** 用于选择 VDD_SPI Flash 电压：
> - 1.8V Flash 时拉高
> - 3.3V Flash 时拉低

### 3.2 USB 下载接线

| ESP32-S3 | 3.3V 外部电源 | USB 线 |
|----------|-------------|--------|
| 3V3 | VDD | |
| GND | GND | GND (黑) |
| EN (CHIP_PU) | VDD | |
| GPIO0 (拉低) | GND | |
| GPIO46 (拉低) | GND | |
| GPIO19 | | USB_D- (白) |
| GPIO20 | | USB_D+ (绿) |
| +5V (或外部提供) | | +5V (红) |

### 3.3 电源要求

- 工作电压：3.0V ~ 3.6V
- 建议供电电压：3.3V
- 输出电流要求：**500 mA 及以上**
- 峰值电流：Wi-Fi 发射时可达 340 mA

> **警告**：不要使用 FTDI FT232R 的 3.3V 输出供电，其电流不足以可靠驱动 ESP32-S3。

---

## 4. esptool 工具

### 4.1 esptool 简介

esptool 是乐鑫科技开源的串口烧录工具，支持所有 ESP 系列芯片。ESP32-S3 的芯片类型标识为 `esp32s3`。

### 4.2 安装

```bash
# 通过 pip 安装
pip install esptool

# ESP-IDF 已内置
# 直接使用 idf.py flash

# 验证安装
esptool.py version
```

### 4.3 基本命令格式

```bash
esptool.py --chip esp32s3 --port PORT --baud BAUD [全局选项] <命令> [命令参数]
```

### 4.4 常用全局选项

| 选项 | 说明 |
|------|------|
| `--chip esp32s3` | 指定芯片类型 |
| `--port PORT` | 指定串口端口 |
| `--baud BAUD` | 波特率 (默认 115200) |
| `--before default-reset` | 连接前复位方式 |
| `--after hard-reset` | 连接后复位方式 |
| `--no-stub` | 不使用 flasher stub |
| `--trace` | 启用详细跟踪输出 |

---

## 5. 通过 UART 烧录

### 5.1 使用 idf.py 烧录

```bash
# 自动检测端口
idf.py flash

# 指定端口和波特率
idf.py -p /dev/ttyACM0 -b 460800 flash

# 烧录并监视
idf.py -p /dev/ttyACM0 flash monitor
```

### 5.2 使用 esptool 直接烧录

```bash
esptool.py --chip esp32s3 -p /dev/ttyACM0 -b 460800 \
    --before default-reset --after hard-reset \
    write-flash \
    --flash-mode dio \
    --flash-size detect \
    --flash-freq 40m \
    0x0 build/bootloader/bootloader.bin \
    0x8000 build/partition_table/partition-table.bin \
    0x10000 build/hello_world.bin
```

### 5.3 烧录偏移地址说明

| 偏移地址 | 文件 | 说明 |
|----------|------|------|
| 0x0 | bootloader.bin | 二级引导加载程序 |
| 0x8000 | partition-table.bin | 分区表 |
| 0x10000 | app.bin | 应用程序固件 |

> 如果启用了 Flash 加密或 Secure Boot，分区表偏移可能需要调整为 0xF000。

### 5.4 成功烧录输出示例

```
esptool v5.0
Serial port /dev/tty.usbserial-0001:
Connecting...
Connected to ESP32-S3 on /dev/tty.usbserial-0001:
Chip type:          ESP32-S3 (revision v0.1)
Crystal frequency:  40MHz
MAC:                de:ad:be:ef:1d:ea

Uploading stub flasher...
Running stub flasher...
Stub flasher running.
Changing baud rate to 460800...
Configuring flash size...
Auto-detected flash size: 8MB

Compressed 25536 bytes to 15935...
Wrote 25536 bytes (15935 compressed) at 0x0 in 0.7 seconds
Hash of data verified.

Hard resetting via RTS pin...
```

### 5.5 使用 flash_args 批量烧录

ESP-IDF 构建后会生成 `flash_args` 文件：

```bash
cd build/
esptool.py --chip esp32s3 -b 460800 write-flash @flash_args
```

---

## 6. 通过 USB Serial/JTAG 烧录

### 6.1 USB Serial/JTAG 优势

- 无需外部 USB-UART 桥接芯片
- 支持固件烧录
- 支持串口控制台输出
- 支持 JTAG 调试（可同时进行）
- 自动进入下载模式

### 6.2 硬件连接

将 ESP32-S3 的 GPIO19 (D-) 和 GPIO20 (D+) 连接到 USB 接口。部分开发板已内置 USB 连接器。

### 6.3 端口识别

USB Serial/JTAG 控制器的串口端口：
- **Linux**: `/dev/ttyACM*`
- **macOS**: `/dev/cu*`
- **Windows**: `COM*`

### 6.4 烧录命令

```bash
# 使用 idf.py
idf.py -p /dev/ttyACM0 flash

# 使用 esptool
esptool.py --chip esp32s3 -p /dev/ttyACM0 write-flash @flash_args
```

### 6.5 配置控制台输出

```
# menuconfig
Component config → ESP System Settings → Channel for console output
→ USB Serial/JTAG Controller  (CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG)
```

### 6.6 USB Serial/JTAG 限制

以下情况 USB 自动下载功能将被禁用：
1. USB PHY 被应用程序关闭
2. USB 被用于其他 USB 功能（USB 主机、USB 标准设备）
3. USB 对应的 IO 管脚被用于其他外设功能

此时必须通过配置 Strapping 管脚手动进入 Download Boot 模式。

---

## 7. 通过 USB OTG (DFU) 烧录

ESP32-S3 的 USB OTG 外设支持通过 USB 设备固件升级 (DFU) 直接连接主机。

### 7.1 DFU 前提条件

- 默认情况下，USB_SERIAL_JTAG 连接到内部 USB PHY，DFU 不可用
- 需要烧录 `USB_PHY_SEL` eFuse 切换内部 USB PHY 为 USB OTG 模式
- 启用 Secure Boot 或 Flash 加密后，DFU 不可用

### 7.2 构建 DFU 镜像

```bash
idf.py dfu
```

生成 `build/dfu.bin` 文件。

### 7.3 烧录 DFU 镜像

```bash
# 安装 dfu-util
# macOS: brew install dfu-util
# Linux: sudo apt install dfu-util
# Windows: 参考 ESP-IDF 文档

# 烧录
idf.py dfu-flash

# 列出设备
idf.py dfu-list

# 指定设备路径
idf.py dfu-flash --path 1-10
```

### 7.4 Linux Udev 规则

创建文件 `/etc/udev/rules.d/40-dfuse.rules`：

```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="303a", ATTRS{idProduct}=="00??", GROUP="plugdev", MODE="0666"
```

---

## 8. 分区表与 Flash 布局

### 8.1 Flash 默认布局

```
偏移地址    内容
─────────────────────────────
0x0000      Bootloader (二级引导)
0x8000      分区表
0x9000      NVS (非易失性存储)
0xF000      PHY 初始化数据
0x10000     应用程序
```

### 8.2 分区表格式

ESP-IDF 使用 CSV 格式的分区表文件 (`partitions.csv`)：

```
# Name,   Type, SubType, Offset,   Size, Flags
nvs,      data, nvs,     0x9000,  0x6000,
phy_init, data, phy,     0xf000,  0x1000,
factory,  app,  factory, 0x10000, 1M,
ota_0,    app,  ota_0,   0x110000, 1M,
ota_1,    app,  ota_1,   0x210000, 1M,
otadata,  data, ota,     0x310000, 0x2000,
```

### 8.3 分区类型说明

| 类型 | 子类型 | 说明 |
|------|--------|------|
| app | factory | 出厂应用 |
| app | ota_0, ota_1 | OTA 升级分区 |
| data | nvs | 非易失性存储 |
| data | phy | PHY 校准数据 |
| data | ota | OTA 数据分区 |
| data | nvs_keys | NVS 加密密钥 |
| data | fat | FAT 文件系统 |
| data | spiffs | SPIFFS 文件系统 |

### 8.4 配置分区表

```
# menuconfig
Partition Table → Custom partition table CSV
```

### 8.5 查看分区表

```bash
idf.py partition-table
```

输出示例：
```
# Name,   Type, SubType,  Offset,  Size, Flags
nvs,      data, nvs,      0x9000,  0x6000,
phy_init, data, phy,      0xf000,  0x1000,
factory,  app,  factory,  0x10000, 0x100000,
```

### 8.6 加密分区标志

启用了 Flash 加密后，某些分区需要标记为 `encrypted`：

```
nvs_key,  data, nvs_keys, 0x320000, 0x1000, encrypted
```

Flash 加密默认加密的类型：
- 二级引导加载程序
- 分区表
- NVS 密钥分区
- Otadata
- 所有 `app` 类型分区

---

## 9. Flash 加密烧录

### 9.1 Flash 加密概述

ESP32-S3 使用 **AES-XTS** 算法进行 Flash 加密，支持 AES-128 (256 位密钥) 或 AES-256 (512 位密钥)。

### 9.2 加密模式

| 模式 | SPI_BOOT_CRYPT_CNT | 说明 |
|------|-------------------|------|
| 开发模式 | 0b001 | 允许重新烧录明文固件 |
| 量产模式 | 0b111 | **永久加密**，不可逆 |

### 9.3 开发模式配置

```
# menuconfig
Security features →
  Enable Flash Encryption on boot = y
  Flash Encryption Mode = Development (NOT SECURE)
  Generated XTS-AES Key Size = AES-128 (256-bit key)
  UART ROM download mode = Enabled
```

### 9.4 量产模式配置

```
# menuconfig
Security features →
  Enable Flash Encryption on boot = y
  Flash Encryption Mode = Release
  UART ROM download mode = Permanently switch to Secure mode (recommended)
```

### 9.5 启用 Flash 加密的步骤

```bash
# 1. 配置并构建
idf.py menuconfig  # 启用 Flash 加密
idf.py build

# 2. 首次烧录明文固件
idf.py flash

# 3. 首次启动时，bootloader 自动：
#    a. 生成 256/512 位加密密钥写入 eFuse
#    b. 就地加密 Flash 内容
#    c. 设置 SPI_BOOT_CRYPT_CNT
#    d. 重新启动
```

### 9.6 外部加密烧录

在量产环境中，可以在主机端预加密固件：

```bash
# 加密 bootloader
espsecure encrypt-flash-data --aes-xts \
    --keyfile my_flash_encryption_key.bin \
    --address 0x0 \
    --output bootloader-enc.bin build/bootloader/bootloader.bin

# 加密分区表
espsecure encrypt-flash-data --aes-xts \
    --keyfile my_flash_encryption_key.bin \
    --address 0x8000 \
    --output partition-table-enc.bin build/partition_table/partition-table.bin

# 加密应用程序
espsecure encrypt-flash-data --aes-xts \
    --keyfile my_flash_encryption_key.bin \
    --address 0x10000 \
    --output my-app-enc.bin build/my-app.bin
```

### 9.7 相关 eFuse

| eFuse | 描述 |
|-------|------|
| `BLOCK_KEYN` | AES 密钥存储 (N=0~5) |
| `KEY_PURPOSE_N` | 密钥用途：2=XTS_AES_256_KEY_1, 3=XTS_AES_256_KEY_2, 4=XTS_AES_128_KEY |
| `SPI_BOOT_CRYPT_CNT` | 3位，奇数位启用加密 |
| `DIS_DOWNLOAD_MANUAL_ENCRYPT` | 下载模式下禁用 Flash 加密 |

### 9.8 检查 Flash 加密状态

```bash
espefuse.py --port /dev/ttyACM0 summary
```

---

## 10. Secure Boot 烧录

### 10.1 Secure Boot V2 概述

ESP32-S3 支持 **Secure Boot V2**，使用 RSA-PSS 签名算法验证固件完整性。

### 10.2 生成签名密钥

```bash
# 生成 RSA-3072 私钥
espsecure generate_signing_key --version 2 --scheme rsa-pss secure_boot_signing_key.pem
```

### 10.3 配置 Secure Boot

```
# menuconfig
Security features →
  Enable hardware Secure Boot in bootloader = y
  Secure Boot V2 = y
  Secure boot signing key = secure_boot_signing_key.pem
```

### 10.4 Secure Boot + Flash 加密 + NVS 加密组合配置

这是最完整的安全方案：

1. **创建分区表**：包含 nvs_key 和 custom_nvs 分区
2. **启用 Secure Boot V2**：在 menuconfig → Security features 中配置
3. **启用 Flash Encryption**：选择 Release 模式
4. **启用 NVS Encryption**：Component config → NVS → Enable NVS encryption
5. **调整分区表偏移**：从 0x8000 调整为 0xF000

### 10.5 Secure Boot 烧录后行为

```
rst:0x1 (POWERON),boot:0x8 (SPI_FAST_FLASH_BOOT)
mode:DIO, clock div:1
Valid secure boot key blocks: 0
secure boot verification succeeded    ← 签名验证成功
...
secure_boot_v2: Signature verified successfully!
boot: Checking flash encryption...
flash_encrypt: flash encryption is enabled (0 plaintext flashes left)
```

### 10.6 Secure Boot 量产配置项

| 配置项 | 说明 |
|--------|------|
| `dis_usb_jtag` | 禁用 USB JTAG |
| `hard_dis_jtag` | 硬禁用 JTAG |
| `soft_dis_jtag` | 软禁用 JTAG |
| `dis_usb_otg_download_mode` | 禁用 USB OTG 下载 |
| `dis_direct_boot` | 禁用直接启动 |
| `dis_download_manual_encrypt` | 禁用下载模式手动加密 |

---

## 11. 量产烧录工具

### 11.1 Flash 下载工具 (Flash Download Tool)

乐鑫提供 Windows GUI 工具用于量产烧录：

1. **选择芯片类型**：ESP32-S3
2. **选择工作模式**：Develop 或 Factory
3. **选择加载模式**：UART 或 USB
4. **加载固件**：填入 bin 文件和烧录地址
5. **选择 COM 口和波特率**
6. **点击 START**

### 11.2 量产加密烧录

Flash 下载工具支持自动完成 Flash 加密和 Secure Boot 流程：

1. 打开配置文件 `./configure/[chip_name]/security.conf`
2. 配置加密选项
3. 工具自动：
   - 烧录明文固件
   - 芯片加密固件并写入 Flash
   - 自动生成并烧录密钥到 eFuse

配置项示例：
```ini
[SECURE BOOT]
secure_boot_en = True
public_key_digest_path = .\secure\public_key_digest.bin

[FLASH ENCRYPTION]
flash_encryption_en = True
reserved_burn_times = 1

[ESP32-S* DISABLE FUNC]
dis_usb_jtag = False
hard_dis_jtag = False
dis_usb_otg_download_mode = False
```

---

## 12. 常见烧录错误与排查

### 12.1 Failed to connect / 无法连接芯片

**可能原因**：
- 串口端口选择错误
- 端口被其他程序占用
- 电源不稳定
- GPIO 引脚连接不正确

**解决方案**：
```bash
# 1. 检查串口端口
ls /dev/ttyACM* /dev/ttyUSB*

# 2. 使用更低的波特率
esptool.py --chip esp32s3 -p PORT -b 9600 ...

# 3. 手动进入下载模式

# 4. 断开连接到 GPIO 引脚的其他设备

# 5. 指定芯片类型跳过自动检测
esptool.py --chip esp32s3 -p PORT ...
```

### 12.2 No serial data received

**原因**：硬件问题，RX/TX 未连接或连接错误。

**解决方案**：
- 检查 TX/RX 是否交叉连接
- 检查 USB 线是否支持数据传输
- 尝试手动进入下载模式

### 12.3 Wrong boot mode detected (0xXX)

**原因**：芯片未进入下载模式。

**解决方案**：
- 检查自动复位电路
- 尝试手动进入下载模式

### 12.4 Download mode successfully detected, but getting no sync reply

**原因**：TX 线路（主机到设备）有问题。

**解决方案**：检查主机 TX 到设备 RX 的线路。

### 12.5 Invalid head of packet (0xXX)

**可能原因**：
- USB 线质量差
- 面包板导致 SPI Flash 引脚短路
- 电源不稳定（掉电）

**解决方案**：
- 更换高质量 USB 线
- 从面包板上取下开发板
- 使用更稳定的电源

### 12.6 Writing to Flash fails part way through

**解决方案**：
- 降低波特率
- 检查电源稳定性

### 12.7 Flash 烧录成功但程序不运行

**可能原因**：

1. **Flash 模式错误**：
```bash
# 尝试使用 dio 模式
esptool.py --chip esp32s3 write-flash -fm dio ...
```

2. **Bootloader 缺失**：确保 bootloader 烧录到偏移 0x0

3. **电源不足**：固件运行时电流需求大于烧录时

4. **SPI 引脚被占用**：GPIO 7, 8, 9, 10 (QIO 模式) 或 7, 8 (DIO 模式) 用于读取 SPI Flash

### 12.8 Could not open /dev/ttyACM0, the port doesn't exist

**解决方案**：
1. 确认开发板已连接
2. 手动进入下载模式：
   - 按住 BOOT 按键
   - 短按 RST 按键
   - 释放 BOOT 按键
3. 检查 USB 驱动是否正确安装

### 12.9 OSError: Protocol error (USB 模式)

**原因**：应用程序重新配置了 USB 外设引脚。

**解决方案**：
1. 手动进入下载模式
2. 擦除 Flash：`esptool.py --chip esp32s3 erase_flash`
3. 修复应用程序代码
4. 重新烧录

### 12.10 eFuse 错误

如果出现 `esp_check_mac_and_efuse` 错误：
- 可能是待下载设备选择有误
- 或设备 eFuse 确有错误，需联系乐鑫技术支持

### 12.11 MD5 校验错误

**解决方案**：先擦除整片 Flash，再重新烧录：
```bash
esptool.py --chip esp32s3 -p PORT erase_flash
esptool.py --chip esp32s3 -p PORT write-flash @flash_args
```

### 12.12 COM 端口递增问题（Windows 量产）

**原因**：Windows 根据 USB 设备序列号递增 COM 编号。

**解决方案**：参考 [阻止 Windows 依据 USB 设备序列号递增 COM 编号](https://docs.espressif.com/projects/esp-iot-solution/zh_CN/latest/usb/usb_overview/usb_device_const_COM.html) 文档。

---

## 13. Flash 操作命令

### 13.1 擦除 Flash

```bash
# 擦除整个 Flash
esptool.py --chip esp32s3 -p PORT erase_flash

# 擦除指定区域
esptool.py --chip esp32s3 -p PORT erase_region OFFSET SIZE
```

### 13.2 读取 Flash

```bash
# 读取 Flash 内容到文件
esptool.py --chip esp32s3 -p PORT read-flash OFFSET SIZE filename.bin
```

### 13.3 验证 Flash

```bash
# 验证 Flash 内容
esptool.py --chip esp32s3 -p PORT verify-flash OFFSET filename.bin
```

> esptool v5 中，`write-flash` 后会自动验证，`--verify` 选项已被废弃。

### 13.4 查看 Flash 信息

```bash
# 查看 Flash ID 和容量
esptool.py --chip esp32s3 -p PORT flash_id
```

### 13.5 查看芯片信息

```bash
esptool.py --chip esp32s3 -p PORT chip_id
```

### 13.6 查看镜像信息

```bash
esptool.py --chip esp32s3 image_info build/hello_world.bin
```

### 13.7 合并固件

```bash
# 合并多个 bin 文件为单个文件
esptool.py --chip esp32s3 merge_bin -o merged.bin \
    --flash-mode dio --flash-size 8MB --flash-freq 40m \
    0x0 build/bootloader/bootloader.bin \
    0x8000 build/partition_table/partition-table.bin \
    0x10000 build/hello_world.bin
```

### 13.8 压缩烧录

```bash
# 使用压缩模式烧录（默认启用）
esptool.py --chip esp32s3 -p PORT -b 460800 \
    write-flash --compress 0x0 bootloader.bin
```

---

## 附录：快速烧录速查表

### A1: 最简烧录命令

```bash
# ESP-IDF
idf.py -p PORT flash

# esptool (ESP-IDF 构建后)
cd build && esptool.py --chip esp32s3 write-flash @flash_args
```

### A2: 常用端口

| 操作系统 | UART 端口 | USB Serial/JTAG 端口 |
|----------|----------|---------------------|
| Linux | /dev/ttyUSB* | /dev/ttyACM* |
| macOS | /dev/cu.usbserial* | /dev/cu.usbmodem* |
| Windows | COM* | COM* |

### A3: Flash 模式速查

| 模式 | 说明 | 命令参数 |
|------|------|---------|
| QIO | 四线 IO（最快） | `-fm qio` |
| QOUT | 四线输出 | `-fm qout` |
| DIO | 双线 IO（兼容性好） | `-fm dio` |
| DOUT | 双线输出 | `-fm dout` |

### A4: esptool v5 变更

- `--verify` 选项已废弃，写入后自动验证
- `image-info` 输出格式已更新
- 错误信息输出到 STDERR
- `ESP32-S3(beta2)` 等测试芯片不再支持

---

## 参考资料

- [Esptool 用户指南 (ESP32-S3)](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/index.html)
- [ESP32-S3 硬件设计指南 - 下载指导](https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32s3/download-guidelines.html)
- [ESP-IDF USB 串行/JTAG 控制器控制台](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/usb-serial-jtag-console.html)
- [ESP-IDF DFU 指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/dfu.html)
- [ESP-IDF Flash 加密](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/security/flash-encryption.html)
- [ESP-IDF 安全功能工作流程](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/security/security-features-enablement-workflows.html)
- [Flash 下载工具用户指南](https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32s3/production_stage/tools/flash_download_tool.html)
- [Flash 下载工具 FAQ](https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32s3/faq/flash_download_tool_faq.html)
- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)
- [USB-Serial-JTAG 外设介绍](https://docs.espressif.com/projects/esp-iot-solution/zh_CN/latest/usb/usb_overview/usb_serial_jtag.html)
- [Flash Encryption + Secure Boot 实战指南](https://developer.espressif.com/blog/flash-encryption-and-secure-boot/)
