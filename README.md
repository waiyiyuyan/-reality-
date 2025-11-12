# -reality-
纯手工，零代码搭建 reality 节点

**服务器部署与客户端配置指南**。

-----

# 🚀 VLESS + REALITY 部署与 V2RayN 配置终极指南

这份指南将详细介绍如何配置和部署 Xray 核心，以实现基于 REALITY 伪装的 VLESS 协议。

## 🖥️ 步骤一：服务器环境准备与必要信息生成

### 1\. 确认 VPS 架构

在您的 VPS 上运行以下命令，确定您需要下载的 Xray 核心版本：

| 命令 | 目的 | 示例输出 (x86\_64) |
| :--- | :--- | :--- |
| `uname -m` | 查看系统 CPU 架构 | `x86_64` |

### 2\. 下载 Xray 核心

根据上一步确认的架构，下载对应的 Xray 核心文件。

> **示例：** 针对 `x86_64` 架构：
> `https://github.com/XTLS/Xray-core/releases/download/v25.10.15/Xray-linux-64.zip`

### 3\. 生成安全密钥与标识符

这些是 REALITY 认证和 VLESS 协议必需的安全参数。

| 步骤 | 命令 | 目的与用途 | 结果示例 |
| :--- | :--- | :--- | :--- |
| **生成密钥对** | `./xray x25519` | 生成 REALITY **私钥**（用于服务器）和 **公钥**（用于客户端）。 | PrivateKey: `KKskeZ3BJ-ZroZ6w9dMEP-aM1ZA0GQasvZXFJ6ckR04` |
| **生成短 ID** | `openssl rand -hex 8` | 生成 **Short ID**，用于 REALITY 流量的二次认证。 | Short ID: `fe4f754a504e7cac` |
| **生成 UUID** | `./xray uuid` | 生成 **VLESS 用户 ID**，用于 VLESS 协议认证。 | UUID: `f76f4404-73a0-439e-b449-65ddca9d8614` |

> **⚠️ 注意：** 您的 **公钥**（`Public Key`）是客户端连接时必须使用的。请务必保存好 `x25519` 命令的完整输出。在您的示例中，`Password` 字段可能是指公钥的别名，但**客户端需要的是 `Public key`**。

### 4\. 寻找优质伪装域名 (Dest)

REALITY 伪装效果的关键在于选择一个与您 VPS **相邻**、且具有大量 HTTPS 流量的网站作为伪装目标（`dest`）。

1.  **扫描：** 使用 `RealiTLScanner` 工具扫描与您 VPS IP 地址相近的 IP 段。
    > **命令示例：** `./RealiTLScanner -addr <您的VPS IP地址>`
2.  **筛选：** 从扫描结果中选择一个支持 TLS 1.3/HTTP/2，且在中国大陆地区访问速度快、不重定向的境外网站。
3.  **确定：** 假设您选择的伪装目标是 `server8.webhostmost.com:443`。

## ⚙️ 步骤二：服务器配置 (`config.json`)

请将以下配置保存为 `config.json` 或 `server.json`，并**移除所有 `"_comment_"` 开头的注释字段**后再运行。

```json
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "listen": "0.0.0.0",
            "port": 14559,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "f76f4404-73a0-439e-b449-65ddca9d8614"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "tcpSettings": {
                    "header": {
                        "type": "none"
                    }
                },
                "realitySettings": {
                    "show": false,
                    "dest": "server8.webhostmost.com:443",
                    "handshake": null,
                    "session": null,
                    "privateKey": "KKskeZ3BJ-ZroZ6w9dMEP-aM1ZA0GQasvZXFJ6ckR04",
                    "minClientVer": "",
                    "maxClientVer": "",
                    "maxSha256Time": 60,
                    "fingerprints": [
                        "chrome",
                        "firefox"
                    ],
                    "serverNames": [
                        "server8.webhostmost.com"
                    ],
                    "shortIds": [
                        "fe4f754a504e7cac"
                    ],
                    "spiderX": "/"
                }
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {}
        }
    ]
}
```

### 启动服务

使用 `nohup` 命令让 Xray 服务在后台持续运行，并将日志输出到 `xray.log`：

```bash
nohup ./xray run -c server.json > xray.log 2>&1 &
```

## 📱 步骤三：V2RayN 客户端配置指南

在 V2RayN 客户端中，选择 **【添加 VLESS 服务器】**，然后对照以下表格填写参数。请确保您的**公钥**正确无误。

| V2RayN 字段名 (中文/英文) | 对应服务器配置字段 | 填写值/说明 |
| :--- | :--- | :--- |
| **备注名称 (Remarks)** | (自定义) | `我的VLESS-REALITY节点` |
| **地址 (Address)** | (VPS IP 地址) | **您的 VPS 的公网 IP 地址** |
| **端口 (Port)** | `inbounds[0].port` | `14559` |
| **用户 ID (UUID / ID)** | `clients[0].id` | `f76f4404-73a0-439e-b449-65ddca9d8614` |
| **别名/用户 (AlterId)** | (默认) | `0` (VLESS 协议固定) |
| **传输协议 (Network)** | `streamSettings.network` | `tcp` |
| **伪装类型 (Type)** | `tcpSettings.header.type` | `none` (保持默认) |
| **流控 (Flow)** | (VLESS flow) | **留空！** 避免 `xtls-rprx-vision` 报错 |
| **底层传输安全 (Security)** | `streamSettings.security` | `reality` |
| **SNI** | `realitySettings.serverNames` | `server8.webhostmost.com` |
| **指纹 (Fingerprint)** | `realitySettings.fingerprints` | **选择 `chrome`** 或 `firefox` |
| **公钥 (Public Key)** | (根据 `privateKey` 生成) | **填写您在步骤 3 中生成的 `Public key`。** |
| **短 ID (Short ID)** | `realitySettings.shortIds` | `fe4f754a504e7cac` |
| **跳转目标 (SpiderX)** | `realitySettings.spiderX` | `/` |

完成以上步骤后，保存配置并尝试连接。如果一切顺利，您就可以通过 VLESS + REALITY 协议开始使用代理服务了。
