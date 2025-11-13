---

# 🚀 VLESS + REALITY 纯手工部署与客户端配置终极指南

> **零代码配置｜Xray 核心｜高隐蔽性伪装方案**

本指南将详细介绍如何在 VPS 上使用 **Xray 核心** 部署基于 **REALITY** 伪装的 **VLESS** 协议节点，并配置 **V2RayN 客户端** 实现安全、高效的代理连接。

---

## 🖥️ 一、服务器环境准备与必要信息生成

### 1. 确认 VPS 架构

运行以下命令，确定您的系统架构（以便下载正确版本的 Xray 核心）：

| 命令         | 作用          | 示例输出     |
| :--------- | :---------- | :------- |
| `uname -m` | 查看系统 CPU 架构 | `x86_64` |

---

### 2. 下载 Xray 核心

根据系统架构，从 [Xray 官方发布页](https://github.com/XTLS/Xray-core/releases) 下载对应版本。

> 示例（x86_64 架构）：

```bash
https://github.com/XTLS/Xray-core/releases/download/v25.10.15/Xray-linux-64.zip
```

---

### 3. 生成密钥与标识符

这些信息是 **REALITY + VLESS** 认证必需的参数。

| 步骤      | 命令                    | 说明                                     | 示例结果                                                      |
| :------ | :-------------------- | :------------------------------------- | :-------------------------------------------------------- |
| 生成密钥对   | `./xray x25519`       | 生成 REALITY 的 **私钥（服务器）** 与 **公钥（客户端）** | PrivateKey: `KKskeZ3BJ-ZroZ6w9dMEP-aM1ZA0GQasvZXFJ6ckR04` |
| 生成短 ID  | `openssl rand -hex 8` | 生成 **Short ID**（REALITY 二次认证）          | Short ID: `fe4f754a504e7cac`                              |
| 生成 UUID | `./xray uuid`         | 生成 **VLESS 用户 ID**                     | UUID: `f76f4404-73a0-439e-b449-65ddca9d8614`              |

> ⚠️ **注意**：
>
> * **公钥 (Public Key)** 必须在客户端中填写。
> * 请妥善保存 `x25519` 命令的完整输出内容。

---

### 4. 选择优质伪装域名 (`dest`)

REALITY 的关键在于伪装一个高质量目标域名。
请选取一个与 VPS **地理位置接近**、**HTTPS 流量大**、**不重定向** 的境外网站。

#### 工具推荐：

1. **扫描伪装目标：**

   ```bash
   ./RealiTLScanner -addr VPS_IP -port 443 -thread 100 -timeout 5 -out file.csv
   ```
2. **筛选可用站点：**

   ```bash
   ./reality-checker csv file.csv
   ```

   👉 [RealityChecker 下载地址](https://github.com/V2RaySSR/RealityChecker/)

#### 示例选择：

```
伪装目标：server8.webhostmost.com:443
```

---

## ⚙️ 二、服务器配置（`config.json`）

将以下配置保存为 `server.json`（或任意文件名），并确保路径正确。

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
          "privateKey": "KKskeZ3BJ-ZroZ6w9dMEP-aM1ZA0GQasvZXFJ6ckR04",
          "minClientVer": "",
          "maxClientVer": "",
          "maxSha256Time": 60,
          "fingerprints": ["chrome", "firefox"],
          "serverNames": ["server8.webhostmost.com"],
          "shortIds": ["fe4f754a504e7cac"],
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

---

### 🧩 字段详解

#### 日志设置

* `loglevel: "warning"` → 仅记录警告及错误，减少日志体积。

#### 入站连接

* `listen: "0.0.0.0"` → 监听所有网络接口。
* `port: 14559` → 连接端口。
* `protocol: "vless"` → 使用 VLESS 协议。

#### 客户端验证

* `id` → 客户端 UUID。
* `decryption: "none"` → 不使用额外加密（REALITY 已负责安全）。

#### 传输层

* `network: "tcp"` → REALITY 必须基于 TCP。
* `security: "reality"` → 使用 REALITY 流量伪装。
* `header.type: "none"` → 不额外伪装 TCP 头。

#### REALITY 设置

| 字段             | 含义          | 示例                            |
| :------------- | :---------- | :---------------------------- |
| `dest`         | 伪装访问的目标网站   | `server8.webhostmost.com:443` |
| `privateKey`   | 服务器私钥（务必保密） | `KKskeZ3BJ-ZroZ6...`          |
| `fingerprints` | 浏览器指纹伪装类型   | `["chrome", "firefox"]`       |
| `serverNames`  | 伪装的 SNI 名称  | `["server8.webhostmost.com"]` |
| `shortIds`     | 客户端验证短 ID   | `["fe4f754a504e7cac"]`        |
| `spiderX`      | 伪装路径        | `/`                           |

---

### ▶️ 启动服务

在后台运行 Xray：

```bash
nohup ./xray run -c server.json > xray.log 2>&1 &
```

查看运行日志：

```bash
tail -f xray.log
```

---

## 📱 三、V2RayN 客户端配置指南

在 V2RayN 中选择 **添加 VLESS 服务器**，并填写如下参数：

| 字段 (中文/英文)        | 对应配置                           | 示例                                     |
| :---------------- | :----------------------------- | :------------------------------------- |
| 备注名称 (Remarks)    | -                              | 我的 VLESS-REALITY 节点                    |
| 地址 (Address)      | VPS 公网 IP                      | 例如 `158.69.xx.xx`                      |
| 端口 (Port)         | `inbounds.port`                | `14559`                                |
| 用户 ID (UUID / ID) | `clients.id`                   | `f76f4404-73a0-439e-b449-65ddca9d8614` |
| AlterId           | 固定值                            | `0`                                    |
| 传输协议 (Network)    | `streamSettings.network`       | `tcp`                                  |
| 伪装类型 (Type)       | `tcpSettings.header.type`      | `none`                                 |
| 流控 (Flow)         | -                              | **留空**                                 |
| 底层传输安全 (Security) | `streamSettings.security`      | `reality`                              |
| SNI               | `realitySettings.serverNames`  | `server8.webhostmost.com`              |
| 指纹 (Fingerprint)  | `realitySettings.fingerprints` | `chrome`                               |
| 公钥 (Public Key)   | `x25519` 生成结果                  | `填写对应的 Public Key`                     |
| 短 ID (Short ID)   | `realitySettings.shortIds`     | `fe4f754a504e7cac`                     |
| 跳转目标 (SpiderX)    | `realitySettings.spiderX`      | `/`                                    |

---

## ✅ 测试连接

完成配置后，点击 **[连接]** → 查看日志，如提示：

```
connected successfully
```

则表示配置成功，REALITY 节点已成功运行！

---

## 📚 小结

| 项目        | 说明                            |
| :-------- | :---------------------------- |
| **协议**    | VLESS + REALITY               |
| **核心**    | Xray-core                     |
| **伪装原理**  | 利用真实网站的 TLS 握手伪装流量            |
| **优势**    | 无需域名、强隐蔽、高性能                  |
| **客户端推荐** | V2RayN / Nekoray / Clash.Meta |

---
