# ESP-AT 指令集详解（ESP32-S3 MQTT 项目必读）

> 文档编号：19  
> 适用芯片：ESP32 / ESP32-S2 / ESP32-S3 / ESP32-C3 / ESP32-C5 / ESP32-C6 / ESP32-C61 等  
> 适用项目：基于 MQTT 协议的物联网通信、Wi-Fi 配网、远程控制  
> 来源：乐鑫官方 ESP-AT 文档、GitHub esp-at 仓库发行说明（v0.10 ~ v5.0）

---

## 目录

1. [ESP-AT 概述](#1-esp-at-概述)
2. [AT 指令通用规则](#2-at-指令通用规则)
3. [基础 AT 指令集](#3-基础-at-指令集)
4. [Wi-Fi AT 指令集](#4-wi-fi-at-指令集)
5. [TCP/IP AT 指令集](#5-tcpip-at-指令集)
6. [MQTT AT 指令集（核心）](#6-mqtt-at-指令集核心)
7. [BLE AT 指令集](#7-ble-at-指令集)
8. [HTTP AT 指令集](#8-http-at-指令集)
9. [WebSocket AT 指令集](#9-websocket-at-指令集)
10. [ESP32-S3 上使用 AT 固件的方法](#10-esp32-s3-上使用-at-固件的方法)
11. [MQTT 完整应用示例](#11-mqtt-完整应用示例)
12. [参考资源](#12-参考资源)

---

## 1. ESP-AT 概述

### 1.1 什么是 ESP-AT

ESP-AT 是乐鑫（Espressif）官方提供的一套 AT 指令固件，可烧录到 ESP 系列芯片中，使主控 MCU（如 STM32、树莓派等）通过 UART 串口发送简单的文本指令来控制 ESP32 实现 Wi-Fi 通信、蓝牙连接、MQTT 消息收发等功能。这种方式极大地降低了客户开发成本，无需深入研究 Wi-Fi/蓝牙协议栈，仅需几条 AT 指令即可完成复杂的网络通信。

### 1.2 ESP-AT 固件特色

- **内置 TCP/IP 协议栈**：芯片内部已经完整实现 TCP/IP 栈并自带数据缓冲
- **便捷集成**：能够非常方便地集成到资源受限的主控平台中
- **易于解析的响应**：主机端对指令回应格式清晰（`OK` / `ERROR`），便于程序解析
- **用户自定义指令**：支持用户通过源码扩展添加自定义 AT 指令
- **多协议支持**：Wi-Fi、TCP/UDP、SSL/TLS、MQTT、HTTP、WebSocket、BLE、BluFi 等

### 1.3 当前固件版本

| 版本 | 主要支持芯片 | 特点 |
|------|-------------|------|
| v2.0 | ESP32（原始） | 引入 MQTT、HTTP、BLE HID 指令 |
| v2.2 | ESP32-C3 | 新增文件系统操作 AT+FS |
| v2.4 | ESP32-S2 | 加入 MQTT 完整指令集 |
| v3.0 | ESP32 / ESP32-C3 | 新增 AT+CWINIT、AT+CWSTATE、AT+CWRECONNCFG |
| v3.2 | ESP32 / ESP32-C3 | WebSocket AT 命令、AT+SYSMSGFILTER |
| v3.3 | ESP32-C2 | 系统消息过滤、制造 NVS 分区 |
| v4.0 | ESP32-C6 | 支持 Wi-Fi 6 |
| v4.1 | ESP32 | WebSocket 证书配置 |
| v5.0 | ESP32-C61 | 支持新一代 ESP32-C61 芯片 |

---

## 2. AT 指令通用规则

### 2.1 指令四种类型

| 类型 | 命令格式 | 说明 |
|------|---------|------|
| 测试命令 | `AT+<命令名称>=?` | 查询设置命令的内部参数及其取值范围 |
| 查询命令 | `AT+<命令名称>?` | 返回当前参数值 |
| 设置命令 | `AT+<命令名称>=<...>` | 设置用户自定义的参数值，并运行命令 |
| 执行命令 | `AT+<命令名称>` | 运行无用户自定义参数的命令 |

> 注意：不是每条 AT 命令都具备上述四种类型。

### 2.2 语法约定

- **参数类型**：当前只支持字符串参数和整型数字参数
- **尖括号 `< >`**：内部参数不可省略
- **方括号 `[ ]`**：内部参数可省略，省略时使用默认值
- **字符串引号**：字符串参数用双引号表示，如 `AT+CWSAP="ESP756290","21030826",1,4`
- **逗号分隔**：省略参数后还有参数时必须用逗号占位，如 `AT+CWJAP="ssid","password",,1`

### 2.3 特殊字符转义

当前需要转义的字符有 `,`、`"`、`\`：

- `\\`：转义反斜杠本身
- `\,`：转义逗号（分隔参数的逗号无需转义）
- `\"`：转义双引号（表示字符串参数的双引号无需转义）
- `\<any>`：转义任意字符，即只使用 `<any>` 字符

> 注意：仅 AT **命令中** 的特殊字符需转义。当 AT 命令口打印 `>` 等待输入数据时，该数据不需要转义。

### 2.4 通信约定

- 默认波特率：**115200**
- 每条 AT 命令长度不超过 **256 字节**
- AT 命令以新行（CR-LF）结束，串口工具应设置为"新行模式"
- 命令以 `\r\n` 作为行尾结束符

---

## 3. 基础 AT 指令集

基础指令用于芯片初始化、系统管理、UART 配置等。

| 指令 | 功能说明 |
|------|---------|
| `AT` | 测试 AT 启动（最基本指令，返回 OK 表示 AT 系统正常） |
| `AT+RST` | 重启模块 |
| `AT+GMR` | 查询版本信息（AT 版本、SDK 版本、编译时间） |
| `AT+CMD` | 列出当前固件支持的所有 AT 命令及类型（v2.0 及以后） |
| `AT+GSLP` | 进入深度睡眠模式 |
| `ATE` | 配置 AT 回显（ATE0 关闭回显，ATE1 开启回显） |
| `AT+RESTORE` | 恢复出厂默认设置 |
| `AT+SAVETRANSLINK` | 设置开机是否进入透传模式 |
| `AT+TRANSINTVL` | 设置透传模式下的数据发送间隔 |
| `AT+UART_CUR` | 当前 UART 配置（不保存到 flash） |
| `AT+UART_DEF` | 默认 UART 配置（保存到 flash） |
| `AT+SLEEP` | 设置睡眠模式（0=禁用、1=轻度睡眠、2=最小模式） |
| `AT+SYSRAM` | 查询当前剩余堆大小和最小堆大小 |
| `AT+SYSMSG` | 查询/设置系统提示信息 |
| `AT+SYSMSGFILTER` | 启用或禁用系统消息过滤器 |
| `AT+SYSMSGFILTERCFG` | 查询/设置系统消息过滤器规则 |
| `AT+SYSFLASH` | 查询/设置 flash 中的用户分区 |
| `AT+SYSMFG` | 查询/设置制造 NVS 用户分区 |
| `AT+RFPOWER` | 查询/设置 RF 发射功率 |
| `AT+RFCAL` | RF 全量校准（v5.0 新增） |
| `AT+SYSROLLBACK` | 回滚到前一个固件版本 |
| `AT+SYSTIMESTAMP` | 查询/设置本地时间戳 |
| `AT+SYSLOG` | 启用或禁用 AT 错误代码提示 |
| `AT+SLEEPWKCFG` | 查询/设置轻度睡眠唤醒源和唤醒 GPIO |
| `AT+SYSSTORE` | 查询/设置参数存储模式 |
| `AT+SYSREG` | 读写寄存器 |
| `AT+FS` | 文件系统操作（部分芯片支持） |
| `AT+USERWKMCUCFG` | 配置唤醒 MCU（v3.2 新增） |
| `AT+USERMCUSLEEP` | 设置 MCU 进入睡眠（v3.2 新增） |

---

## 4. Wi-Fi AT 指令集

Wi-Fi 指令用于配置 Wi-Fi 模式、连接 AP、扫描热点、设置 SoftAP 等。

### 4.1 模式与连接

| 指令 | 功能说明 |
|------|---------|
| `AT+CWINIT` | 初始化/反初始化 Wi-Fi 驱动 |
| `AT+CWMODE` | 设置 Wi-Fi 模式（1=Station、2=SoftAP、3=Station+SoftAP） |
| `AT+CWSTATE` | 查询 Wi-Fi 状态和 Wi-Fi 信息 |
| `AT+CWJAP` | 连接到 AP（路由器） |
| `AT+CWRECONNCFG` | 查询/设置 Wi-Fi 重连配置 |
| `AT+CWQAP` | 断开与 AP 的连接 |
| `AT+CWAUTOCONN` | 设置上电时是否自动连接 AP |

**连接 AP 示例：**

```
AT+CWMODE=1
AT+CWJAP="your_ssid","your_password"
```

响应：
```
WIFI CONNECTED
WIFI GOT IP

OK
```

### 4.2 扫描与信息

| 指令 | 功能说明 |
|------|---------|
| `AT+CWLAP` | 列出可用 AP |
| `AT+CWLAPOPT` | 设置 AT+CWLAP 输出格式 |
| `AT+CWSAP` | 查询/设置 ESP SoftAP 配置 |
| `AT+CWLIF` | 获取连接到 ESP SoftAP 的 Station IP |
| `AT+CWQIF` | 断开连接到 ESP SoftAP 的 Station |
| `AT+CWDHCP` | 启用/禁用 DHCP |
| `AT+CWDHCPS` | 查询/设置 SoftAP DHCP 服务器分配的 IP 地址范围 |
| `AT+CWAPPROTO` | 查询/设置 SoftAP 模式的 802.11 协议标准 |
| `AT+CWSTAPROTO` | 查询/设置 Station 模式的 802.11 协议标准 |
| `AT+CIPSTAMAC` | 查询/设置 Station 的 MAC 地址 |
| `AT+CIPAPMAC` | 查询/设置 SoftAP 的 MAC 地址 |
| `AT+CIPSTA` | 查询/设置 Station 的 IP 地址 |
| `AT+CIPAP` | 查询/设置 SoftAP 的 IP 地址 |

### 4.3 高级功能

| 指令 | 功能说明 |
|------|---------|
| `AT+CWSTARTSMART` | 启动 SmartConfig（智能配网） |
| `AT+CWSTOPSMART` | 停止 SmartConfig |
| `AT+WPS` | 启用 WPS 功能 |
| `AT+MDNS` | 配置 mDNS 功能 |
| `AT+CWJEAP` | 连接 WPA2 企业级 AP |
| `AT+CWHOSTNAME` | 查询/设置 Station 的主机名 |
| `AT+CWCOUNTRY` | 查询/设置 Wi-Fi 国家代码 |
| `AT+CWBANDWIDTH` | 查询/设置 Wi-Fi 带宽（v5.0 新增） |
| `AT+CWCONFIG` | 查询/设置 Wi-Fi 不活跃时间和监听间隔（v5.0 新增） |

---

## 5. TCP/IP AT 指令集

TCP/IP 指令用于建立网络连接、发送和接收数据。

| 指令 | 功能说明 |
|------|---------|
| `AT+CIPSTATUS` | 获取连接状态 |
| `AT+CIPDOMAIN` | DNS 域名解析 |
| `AT+CIPSTART` | 建立 TCP 连接、UDP 传输或 SSL 连接 |
| `AT+CIPSTARTEX` | 建立连接并自动分配 ID |
| `AT+CIPSEND` | 发送数据 |
| `AT+CIPSENDEX` | 发送数据（遇到 `\0` 或达到指定长度时停止） |
| `AT+CIPCLOSE` | 关闭 TCP/UDP/SSL 连接 |
| `AT+CIFSR` | 获取本地 IP 地址 |
| `AT+CIPMUX` | 配置多连接模式（0=单连接、1=多连接） |
| `AT+CIPSERVER` | 删除/创建 TCP 或 SSL 服务器 |
| `AT+CIPSERVERMAXCONN` | 设置服务器允许的最大连接数 |
| `AT+CIPMODE` | 配置传输模式（0=普通模式、1=透传模式） |
| `AT+SAVETRANSLINK` | 保存透传链路到 flash |
| `AT+CIPSTO` | 设置 TCP 服务器超时时间 |
| `AT+CIPSNTPCFG` | 配置时区和 SNTP 服务器 |
| `AT+CIPSNTPTIME` | 查询 SNTP 时间 |
| `AT+CIUPDATE` | 通过 Wi-Fi 进行 OTA 升级 |
| `AT+CIPDINFO` | 在 +IPD 中显示远端 IP 和远端端口 |
| `AT+CIPSSLCCONF` | 配置 SSL 客户端 |
| `AT+CIPRECONNINTV` | 设置 Wi-Fi 透传模式下自动重连间隔 |
| `AT+CIPRECVMODE` | 设置 Socket 接收模式 |
| `AT+CIPRECVDATA` | 在被动接收模式下获取 Socket 数据 |
| `AT+CIPRECVLEN` | 在被动接收模式下获取 Socket 数据长度 |
| `AT+PING` | Ping 测试 |
| `AT+CIPDNS` | 配置 DNS 服务器 |
| `AT+CIPTCPOPT` | 配置 Socket 选项 |
| `AT+CIPFWVER` | 查询防火墙固件版本（v3.2 新增） |

---

## 6. MQTT AT 指令集（核心）

MQTT 指令集是 ESP-AT 固件中最重要的协议指令之一，对于 MQTT 项目开发至关重要。当前 ESP32 系列 AT 固件支持 **MQTT 3.1.1 版本**。

### 6.1 指令列表

| 指令 | 功能说明 |
|------|---------|
| `AT+MQTTUSERCFG` | 设置 MQTT 用户属性（包括 scheme、client_id、username、password、证书等） |
| `AT+MQTTLONGCLIENTID` | 设置 MQTT 客户端 ID（长字符串） |
| `AT+MQTTLONGUSERNAME` | 设置 MQTT 登录用户名（长字符串） |
| `AT+MQTTLONGPASSWORD` | 设置 MQTT 登录密码（长字符串） |
| `AT+MQTTCONNCFG` | 设置 MQTT 连接属性（keepalive、will 消息等） |
| `AT+MQTTALPN` | 设置 MQTT 应用层协议协商 (ALPN) |
| `AT+MQTTSNI` | 设置 MQTT 服务器名称指示 (SNI) |
| `AT+MQTTCONN` | 连接 MQTT Broker |
| `AT+MQTTPUB` | 发布 MQTT 消息（字符串格式） |
| `AT+MQTTPUBRAW` | 发布长 MQTT 消息（二进制/大数据） |
| `AT+MQTTSUB` | 订阅 MQTT Topic |
| `AT+MQTTUNSUB` | 取消订阅 MQTT Topic |
| `AT+MQTTCLEAN` | 断开 MQTT 连接，清理资源 |

### 6.2 AT+MQTTUSERCFG 参数详解

```
AT+MQTTUSERCFG=<LinkID>,<scheme>,<"client_id">,<"username">,<"password">,<cert_key_ID>,<CA_ID>,<"path">
```

| 参数 | 说明 |
|------|------|
| `<LinkID>` | 当前仅支持 Link ID 0 |
| `<scheme>` | 连接方案，详见下表 |
| `<client_id>` | MQTT 客户端 ID，最大长度 256 字节 |
| `<username>` | MQTT 用户名，最大长度 64 字节 |
| `<password>` | MQTT 密码，最大长度 64 字节 |
| `<cert_key_ID>` | 证书和私钥索引 |
| `<CA_ID>` | CA 证书索引 |
| `<path>` | 资源路径，最大长度 32 字节 |

**scheme 参数取值：**

| scheme 值 | 含义 |
|-----------|------|
| 1 | MQTT over TCP |
| 2 | MQTT over TLS（不校验证书） |
| 3 | MQTT over TLS（校验 server 证书） |
| 4 | MQTT over TLS（提供 client 证书） |
| 5 | MQTT over TLS（校验 server 证书且提供 client 证书） |
| 6 | MQTT over WebSocket（基于 TCP） |
| 7 | MQTT over WebSocket Secure（基于 TLS，不校验证书） |
| 8 | MQTT over WebSocket Secure（基于 TLS，校验 server 证书） |
| 9 | MQTT over WebSocket Secure（基于 TLS，提供 client 证书） |
| 10 | MQTT over WebSocket Secure（基于 TLS，校验 server 证书且提供 client 证书） |

### 6.3 AT+MQTTCONN 参数详解

```
AT+MQTTCONN=<LinkID>,<"host">,<port>,<reconnect>[,<timeout_ms>]
```

| 参数 | 说明 |
|------|------|
| `<LinkID>` | 当前仅支持 0 |
| `<host>` | MQTT broker 域名或 IP，最大 128 字节 |
| `<port>` | MQTT broker 端口 |
| `<reconnect>` | 0=不自动重连；1=自动重连（消耗较多内存） |
| `<timeout_ms>` | 超时时间，范围 [3000, 60000]，默认 15000 毫秒 |

**MQTT 连接状态值：**

| state 值 | 含义 |
|----------|------|
| 0 | MQTT 未初始化 |
| 1 | 已设置 AT+MQTTUSERCFG |
| 2 | 已设置 AT+MQTTCONNCFG |
| 3 | 连接已断开 |
| 4 | 已建立连接 |
| 5 | 已连接，未订阅 topic |
| 6 | 已连接，已订阅 topic |

### 6.4 订阅消息接收格式

当订阅者收到消息时，AT 会输出：

```
+MQTTSUBRECV:<LinkID>,<"topic">,<data_length>,<data>
```

### 6.5 禁用 MQTT 支持

如果不需要 MQTT 功能以节省 flash 空间，可在编译 ESP-AT 工程时配置：

```
Component config > AT > AT MQTT command support  [取消勾选]
```

---

## 7. BLE AT 指令集

蓝牙低功耗（Bluetooth Low Energy）指令集，用于 BLE 通信功能。

### 7.1 基础 BLE 指令

| 指令 | 功能说明 |
|------|---------|
| `AT+BLEINIT` | Bluetooth LE 初始化 |
| `AT+BLEADDR` | 查询/设置 BLE 设备地址 |
| `AT+BLENAME` | 查询/设置 BLE 设备名称 |
| `AT+BLESCANPARAM` | 查询/设置 BLE 扫描参数 |
| `AT+BLESCAN` | 使能 BLE 扫描 |
| `AT+BLESCANRSPDATA` | 设置 BLE 扫描响应数据 |
| `AT+BLEADVPARAM` | 查询/设置 BLE 广播参数 |
| `AT+BLEADVDATA` | 设置 BLE 广播数据 |
| `AT+BLEADVDATAEX` | 自动设置 BLE 广播数据 |
| `AT+BLEADVSTART` | 开始 BLE 广播 |
| `AT+BLEADVSTOP` | 停止 BLE 广播 |
| `AT+BLECONN` | 建立 BLE 连接 |
| `AT+BLECONNPARAM` | 查询/更新 BLE 连接参数 |
| `AT+BLEDISCONN` | 断开 BLE 连接 |
| `AT+BLEDATALEN` | 设置 BLE 数据包长度 |
| `AT+BLECFGMTU` | 设置 BLE MTU 长度 |

### 7.2 GATT 服务指令

| 指令 | 功能说明 |
|------|---------|
| `AT+BLEGATTSSRVCRE` | GATTS 创建服务 |
| `AT+BLEGATTSSRVSTART` | GATTS 开启服务 |
| `AT+BLEGATTSSRVSTOP` | GATTS 停止服务 |
| `AT+BLEGATTSSRV` | GATTS 发现服务 |
| `AT+BLEGATTSCHAR` | GATTS 发现服务特征 |
| `AT+BLEGATTSNTFY` | 服务器 notify 特征值给客户端 |
| `AT+BLEGATTSIND` | 服务器 indicate 特征值给客户端 |
| `AT+BLEGATTSSETATTR` | GATTS 设置特征值 |
| `AT+BLEGATTCPRIMSRV` | GATTC 发现基本服务 |
| `AT+BLEGATTCINCLSRV` | GATTC 发现包含服务 |
| `AT+BLEGATTCCHAR` | GATTC 发现服务特征 |
| `AT+BLEGATTCRD` | GATTC 读取特征值 |
| `AT+BLEGATTCWR` | GATTC 写特征值 |

### 7.3 BLE 其他指令

| 指令 | 功能说明 |
|------|---------|
| `AT+BLESPPCFG` | 查询/设置 BLE SPP 参数 |
| `AT+BLESPP` | 进入 BLE SPP 透传模式 |
| `AT+BLESECPARAM` | 查询/设置 BLE 加密参数 |
| `AT+BLEENC` | 发起 BLE 加密请求 |
| `AT+BLEENCRSP` | 回复对端设备的配对请求 |
| `AT+BLEKEYREPLY` | 给对方设备回复密钥 |
| `AT+BLECONFREPLY` | 给对方设备回复确认结果 |
| `AT+BLEENCDEV` | 查询绑定的 BLE 加密设备列表 |
| `AT+BLEENCCLEAR` | 清除 BLE 加密设备列表 |
| `AT+BLESETKEY` | 设置 BLE 静态配对密钥 |
| `AT+BLEHIDINIT` | BLE HID 协议初始化 |
| `AT+BLEHIDKB` | 发送 BLE HID 键盘信息 |
| `AT+BLEHIDMUS` | 发送 BLE HID 鼠标信息 |
| `AT+BLEHIDCONSUMER` | 发送 BLE HID consumer 信息 |
| `AT+BLUFI` | 开启或关闭 BluFi（蓝牙配网） |
| `AT+BLUFINAME` | 查询/设置 BluFi 设备名称 |
| `AT+BLUFISEND` | 发送 BluFi 用户自定义数据 |
| `AT+BLERDRSSI` | 查询当前连接的 RSSI |
| `AT+BLEWL` | 设置白名单 |
| `AT+BLEPERIODICDATA` | 设置 BLE 周期性广播数据 |
| `AT+BLEPERIODICSTART` | 开始 BLE 周期性广播 |
| `AT+BLEPERIODICSTOP` | 停止 BLE 周期性广播 |
| `AT+BLESYNCSTART` | 开始与周期性广播同步 |
| `AT+BLESYNCSTOP` | 停止与周期性广播同步 |
| `AT+BLEREADPHY` | 查询当前发射 PHY |
| `AT+BLESETPHY` | 设置当前发射 PHY |

---

## 8. HTTP AT 指令集

| 指令 | 功能说明 |
|------|---------|
| `AT+HTTPCLIENT` | 发送 HTTP 客户端请求（GET/POST 等） |
| `AT+HTTPGETSIZE` | 获取 HTTP 资源大小 |
| `AT+HTTPCHEAD` | 设置 HTTP 请求头（v3.2 新增） |
| `AT+HTTPCPUT` | PUT 指定长度的 HTTP 数据（v3.2 新增） |
| `AT+HTTPPOST` | POST HTTP 数据 |
| `AT+HTTPREAD` | 读取 HTTP 响应数据 |

---

## 9. WebSocket AT 指令集

> 注意：默认 AT 固件不支持 WebSocket，需自行编译 ESP-AT 工程并在配置中启用 `Component config > AT > AT WebSocket command support`。

| 指令 | 功能说明 |
|------|---------|
| `AT+WSCFG` | 配置 WebSocket 参数（Ping 间隔、超时、缓冲区、鉴权） |
| `AT+WSHEAD` | 设置/查询 WebSocket 请求头 |
| `AT+WSOPEN` | 查询/打开 WebSocket 连接（支持最多 3 个连接） |
| `AT+WSSEND` | 向 WebSocket 连接发送数据 |
| `AT+WSDATAFMT` | 设置 WebSocket 接收数据格式 |
| `AT+WSCLOSE` | 关闭 WebSocket 连接 |

**AT+WSCFG 参数：**

```
AT+WSCFG=<link_id>,<ping_intv_sec>,<ping_timeout_sec>[,<buffer_size>][,<auth_mode>,<pki_number>,<ca_number>]
```

- `<link_id>`：WebSocket 连接 ID，范围 [0, 2]
- `<ping_intv_sec>`：Ping 发送间隔，单位秒，默认 10
- `<ping_timeout_sec>`：Ping 超时，单位秒，默认 120
- `<buffer_size>`：缓冲区大小，默认 1024 字节
- `<auth_mode>`：0=不认证、1=提供客户端证书、2=校验服务器证书、3=相互认证

---

## 10. ESP32-S3 上使用 AT 固件的方法

### 10.1 获取 AT 固件

ESP32-S3 系列芯片可使用乐鑫官方发布的 AT 固件：

- **官方下载地址**：<https://www.espressif.com/zh-hans/support/download/at>
- **GitHub 仓库**：<https://github.com/espressif/esp-at>
- **ESP-AT 文档**：<https://docs.espressif.com/projects/esp-at/zh_CN/latest/>

### 10.2 硬件连接

ESP32-S3 通过 UART 与主控 MCU 通信：

| ESP32-S3 引脚 | 主控 MCU 引脚 | 说明 |
|--------------|-------------|------|
| GPIO43 (U0TXD) | RXD | AT 固件输出 |
| GPIO44 (U0RXD) | TXD | AT 固件输入 |
| GND | GND | 共地 |

默认波特率：115200，8 数据位，无校验，1 停止位。

### 10.3 烧录固件

使用 `esptool.py` 工具烧录：

```bash
esptool.py --chip esp32s3 --port COM3 --baud 921600 write_flash -z 0x0 bootloader.bin 0x10000 esp-at.bin 0x8000 partition-table.bin 0x60000 at_custom_ota.bin
```

### 10.4 自定义编译

如需自定义 AT 指令（例如增加 MQTT 证书、启用 WebSocket）：

```bash
git clone --recursive https://github.com/espressif/esp-at.git
cd esp-at

# 选择芯片目标
./build.py menuconfig
# 设置 target chip: ESP32-S3
# 配置组件：
#   - Component config > AT > AT MQTT command support
#   - Component config > AT > AT WebSocket command support
#   - Component config > AT > AT BLE command support

# 编译
./build.py build

# 烧录
./build.py -p COM3 flash
```

### 10.5 通信测试

烧录完成后，使用串口终端工具（如 PuTTY、Tera Term）连接 ESP32-S3：

```
AT                    # 测试连接
AT+GMR                # 查询版本信息
AT+CWMODE=1           # 设置为 Station 模式
AT+CWJAP="ssid","pwd" # 连接 Wi-Fi
AT+CIFSR              # 查询 IP 地址
```

---

## 11. MQTT 完整应用示例

以下示例演示了两块 ESP32-S3 开发板通过 AT 指令实现 MQTT 通信（发布/订阅）的完整流程。

### 11.1 MQTT 发布者（Publisher）

```
步骤 1：设置 Wi-Fi 模式
AT+CWMODE=1
OK

步骤 2：连接路由器
AT+CWJAP="your_ssid","your_password"
WIFI CONNECTED
WIFI GOT IP
OK

步骤 3：配置 MQTT 用户属性（scheme=1 表示基于 TCP）
AT+MQTTUSERCFG=0,1,"publisher_client","username","password",0,0,""
OK

步骤 4：连接 MQTT Broker
AT+MQTTCONN=0,"192.168.3.102",1883,1
+MQTTCONNECTED:0,1,"192.168.3.102","1883","",1
OK

步骤 5：发布消息
AT+MQTTPUB=0,"sensor/temperature","{\"temp\":25.6}",1,0
OK

步骤 6：断开 MQTT 连接
AT+MQTTCLEAN=0
OK
```

### 11.2 MQTT 订阅者（Subscriber）

```
步骤 1-3：与发布者相同的 Wi-Fi 连接和 MQTT 用户配置
AT+MQTTUSERCFG=0,1,"subscriber_client","username","password",0,0,""
OK

步骤 4：连接 MQTT Broker
AT+MQTTCONN=0,"192.168.3.102",1883,1
+MQTTCONNECTED:0,1,"192.168.3.102","1883","",1
OK

步骤 5：订阅 Topic
AT+MQTTSUB=0,"sensor/temperature",1
OK

步骤 6：接收消息（当发布者发布消息时，订阅者端输出）
+MQTTSUBRECV:0,"sensor/temperature",20,{"temp":25.6}

步骤 7：取消订阅
AT+MQTTUNSUB=0,"sensor/temperature"
OK

步骤 8：断开连接
AT+MQTTCLEAN=0
OK
```

### 11.3 使用 TLS 安全连接的 MQTT

```
AT+MQTTUSERCFG=0,5,"client_id","username","password",0,0,""
OK
AT+MQTTCONN=0,"broker.emqx.io",8883,1
+MQTTCONNECTED:0,5,"broker.emqx.io","8883","",1
OK
```

> scheme=5 表示 MQTT over TLS（校验 server 证书并提供 client 证书）

### 11.4 MQTT 连接云平台（AWS IoT）

连接 AWS IoT 需要预先烧录设备证书：

1. 使用 `AT+SYSMFG` 命令或编译时预烧录证书
2. 配置 scheme=5（MQTT over TLS，双向证书认证）
3. client_id 需要与 AWS IoT 中注册的 Thing 名称匹配

---

## 12. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-AT 官方文档（中文） | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/> |
| AT 指令集（ESP32） | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Set/index.html> |
| MQTT AT 命令集 | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Set/MQTT_AT_Commands.html> |
| MQTT AT 示例 | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Examples/MQTT_AT_Examples.html> |
| AT 固件下载 | <https://www.espressif.com/zh-hans/support/download/at> |
| ESP-AT GitHub | <https://github.com/espressif/esp-at> |
| ESP-AT 产品概述 | <https://www.espressif.com/zh-hans/products/sdks/esp-at/overview> |
| BLE AT 命令集 | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Set/BLE_AT_Commands.html> |
| WebSocket AT 命令集 | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Set/websocket_at_commands.html> |
| AT API 参考 | <https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/Compile_and_Develop/AT_API_Reference.html> |
| MQTT 3.1.1 协议规范 | <https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html> |

---

> **文档总结**：ESP-AT 指令集为 ESP32-S3 的 MQTT 项目提供了最简单的开发路径。通过 UART 串口发送文本指令即可完成 Wi-Fi 配网、MQTT 连接/发布/订阅、HTTP 请求、BLE 通信等功能，无需深入了解底层协议栈。对于 MQTT 项目而言，核心指令为 `AT+MQTTUSERCFG`、`AT+MQTTCONN`、`AT+MQTTPUB`、`AT+MQTTSUB`，配合 Wi-Fi 配网指令 `AT+CWJAP` 即可完成完整的物联网通信链路。
