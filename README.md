# ESP32 WiFi Demo

一个基于 ESP-IDF 开发的 ESP32 WiFi Captive Portal（强制门户）演示项目。该项目创建一个 WiFi 热点，当设备连接后会自动弹出 Web 页面。

## 功能特性

- **WiFi 热点（SoftAP）**: 自动生成唯一 SSID，支持多设备连接
- **DNS 劫持**: 将所有 DNS 查询重定向到 ESP32 IP 地址
- **Captive Portal**: 设备连接后自动弹出 Web 页面
- **嵌入式 Web 页面**: HTML 文件嵌入固件，无需外部存储
- **C++ 实现**: 主程序使用 C++ 编写

## 硬件要求

- ESP32 开发板（如 ESP32-DevKitC、ESP-WROVER-KIT 等）
- USB 数据线

## 软件要求

- [ESP-IDF](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/index.html) v5.0 或更高版本
- Python 3.8+
- CMake 3.16+

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/shenjingnan/esp32-wifi-demo.git
cd esp32-wifi-demo
```

### 2. 设置 ESP-IDF 环境

```bash
# 如果已安装 ESP-IDF，激活环境
. $HOME/esp/esp-idf/export.sh

# 或使用别名（如果已配置）
get_idf
```

### 3. 构建项目

```bash
idf.py build
```

### 4. 烧录到设备

```bash
# 连接 ESP32 开发板，然后执行
idf.py -p PORT flash

# 例如：
# idf.py -p /dev/ttyUSB0 flash   # Linux
# idf.py -p COM3 flash            # Windows
# idf.py -p /dev/tty.usbserial-110 flash  # macOS
```

### 5. 查看日志输出

```bash
idf.py -p PORT monitor

# 也可以一步完成构建、烧录和监控
idf.py -p PORT flash monitor
```

## 使用方法

1. 烧录完成后，ESP32 会创建一个 WiFi 热点
2. 使用手机或电脑搜索 WiFi，SSID 格式为 `ESP32_WIFI_XXXXXX`（后缀为 MAC 地址后3字节）
3. 连接该 WiFi（无需密码）
4. 大多数设备会自动弹出 Web 页面；如果没有，打开浏览器访问任意网站即可跳转

## 项目结构

```
esp32-wifi-demo/
├── CMakeLists.txt              # 根目录 CMake 配置
├── sdkconfig.defaults          # SDK 默认配置
├── README.md                   # 项目说明文档
├── main/
│   ├── CMakeLists.txt          # main 组件 CMake 配置
│   ├── main.cc                 # 主程序源代码（C++）
│   └── index.html              # 嵌入的 Web 页面
└── components/
    └── dns_server/             # 自定义 DNS 服务器组件
        ├── CMakeLists.txt
        ├── dns_server.c
        └── include/
            └── dns_server.h
```

## 技术架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         ESP32 设备                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   WiFi AP   │    │ DNS Server  │    │   HTTP Server       │ │
│  │  (SoftAP)   │    │  (Port 53)  │    │   (Port 80)         │ │
│  │             │    │             │    │                     │ │
│  │ SSID:       │    │ 拦截所有    │    │  ┌───────────────┐  │ │
│  │ ESP32_WIFI_ │───▶│ DNS 查询    │───▶│  │ GET /         │  │ │
│  │ XXXXXX      │    │ 返回本地 IP │    │  │ → index.html  │  │ │
│  │             │    │             │    │  ├───────────────┤  │ │
│  │ IP:         │    │             │    │  │ 404 Handler   │  │ │
│  │ 192.168.4.1 │    │             │    │  │ → 302 → /     │  │ │
│  └─────────────┘    └─────────────┘    │  └───────────────┘  │ │
│                                        └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 工作原理

### Captive Portal 流程

```mermaid
sequenceDiagram
    participant 用户设备
    participant WiFi AP
    participant DNS Server
    participant HTTP Server

    用户设备->>WiFi AP: 1. 连接 WiFi
    用户设备->>DNS Server: 2. DNS 查询（检测网络）
    DNS Server-->>用户设备: 3. 返回 192.168.4.1
    用户设备->>HTTP Server: 4. HTTP GET（检测 URL）
    HTTP Server-->>用户设备: 5. 302 重定向 → /
    用户设备->>HTTP Server: 6. GET /
    HTTP Server-->>用户设备: 7. 返回 HTML 页面
```

### 核心组件说明

| 组件 | 功能 |
|------|------|
| WiFi AP | 创建开放热点，SSID 基于 MAC 地址生成 |
| DNS Server | 将所有 A 记录查询重定向到 ESP32 IP |
| HTTP Server | 提供 Web 页面，404 重定向实现 Portal |

## 关键配置参数

| 参数 | 值 | 说明 |
|------|-----|------|
| WiFi 模式 | `WIFI_MODE_AP` | 接入点模式 |
| SSID 格式 | `ESP32_WIFI_XXXXXX` | MAC 地址后3字节 |
| 认证方式 | `WIFI_AUTH_OPEN` | 开放，无密码 |
| 最大连接数 | 4 | 同时最多4个设备 |
| 默认 IP | `192.168.4.1` | ESP-IDF 默认 AP IP |
| DNS 端口 | 53 | 标准 DNS 端口 |
| HTTP 端口 | 80 | 标准 HTTP 端口 |

## 自定义配置

### 修改 Web 页面

编辑 `main/index.html` 文件，修改后重新构建烧录即可。

### 修改 WiFi 配置

在 `main/main.cc` 中修改相关参数：

```cpp
wifi_config_t wifi_config = {};
strcpy(reinterpret_cast<char *>(wifi_config.ap.ssid), "你的SSID");
wifi_config.ap.ssid_len = strlen("你的SSID");
wifi_config.ap.max_connection = 8;  // 最大连接数
wifi_config.ap.authmode = WIFI_AUTH_WPA2_PSK;  // 使用密码
strcpy(reinterpret_cast<char *>(wifi_config.ap.password), "你的密码");
```

### 修改 DNS 重定向规则

在 `main/main.cc` 中修改 DNS 服务器配置：

```cpp
dns_server_config_t config = DNS_SERVER_CONFIG_SINGLE("*", "WIFI_AP_DEF");
// "*" 表示匹配所有域名
// 可以指定特定域名，如 "example.com"
```

## 常见问题

### 设备连接后没有自动弹出页面

- 部分设备需要手动打开浏览器访问任意网站
- iOS 设备可能需要等待几秒钟
- Android 设备可能需要点击"登录"或"连接"按钮

### 无法连接 WiFi

- 确认 ESP32 已正确烧录
- 检查日志输出确认 WiFi AP 是否正常启动
- 尝试重启 ESP32

### 编译错误

- 确认 ESP-IDF 版本 >= 5.0
- 执行 `idf.py fullclean` 后重新构建
- 检查 Python 依赖是否完整安装

## 参考资料

- [ESP-IDF 编程指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/)
- [ESP32 WiFi 驱动](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-reference/network/esp_wifi.html)
- [ESP HTTP Server](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-reference/protocols/esp_http_server.html)

## 许可证

本项目采用 MIT 许可证，详见 [LICENSE](LICENSE) 文件。