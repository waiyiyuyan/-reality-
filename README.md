
# 🚀 VLESS + REALITY 节点完整部署与配置教程（Xray）

> **零域名 · 高隐蔽性 · 纯手工配置 · 可直接复制使用**

---

## 🧩 一、准备环境

### 1️⃣ 检查系统架构

运行以下命令，查看你的 VPS CPU 架构（决定下载哪个 Xray 版本）：

```bash
uname -m
```

常见输出：

| 架构      | 说明               |
| ------- | ---------------- |
| x86_64  | 64 位 CPU（主流 VPS） |
| aarch64 | ARM 架构（如部分轻量服务器） |

---

### 2️⃣ 下载 Xray 核心

访问 Xray 官方发布页：
👉 [https://github.com/XTLS/Xray-core/releases](https://github.com/XTLS/Xray-core/releases)

根据系统架构选择对应文件（例如 Linux 64 位）：

```bash
wget https://github.com/XTLS/Xray-core/releases/download/v25.10.15/Xray-linux-64.zip
unzip Xray-linux-64.zip
chmod +x xray
```

---

## 🔑 二、生成关键信息

REALITY 模式需要三个核心数据：

| 项目              | 命令                    | 说明                |
| --------------- | --------------------- | ----------------- |
| **VLESS 用户 ID** | `./xray uuid`         | 客户端识别用            |
| **REALITY 密钥对** | `./xray x25519`       | 生成公钥（客户端）与私钥（服务器） |
| **Short ID**    | `openssl rand -hex 8` | 二次验证用             |

示例输出（请自行保存）：

```bash
Private key: KKskeZ3BJ-ZroZ6w9dMEP-aM1ZA0GQasvZXFJ6ckR04
Public key:  CYXTxXy_FfOhiJ9-4XlUeb6bX5Xt-3zVzZ2yB2B1dTo
Short ID:    fe4f754a504e7cac
UUID:        f76f4404-73a0-439e-b449-65ddca9d8614
```

> ⚠️ 注意：
>
> * 私钥仅服务端使用，**绝对不要泄露**。
> * 公钥填入客户端中。

---

## 🌐 三、选择伪装域名（REALITY 核心）

REALITY 流量伪装依赖一个**高质量 HTTPS 域名**，建议选择：

- 支持 TLS 1.3  
- 不跳转  
- 有真实网页内容  
- 与 VPS 地理位置接近  

### 🔹 推荐工具

1. **扫描目标域名/IP 段**

```bash
./RealiTLScanner -addr VPS_IP -port 443 -thread 100 -timeout 5 -out file.csv
````
> 🔗 [RealiTLScanner GitHub 下载地址](https://github.com/XTLS/RealiTLScanner/releases/tag/v0.2.1)
说明：

* `-addr VPS_IP` → 你的 VPS 公网 IP
* `-port 443` → HTTPS 端口
* `-thread` → 并发线程数
* `-timeout` → 超时时间（秒）
* `-out file.csv` → 扫描结果保存文件

2. **筛选可用域名**

```bash
./reality-checker csv file.csv
```

说明：

* 解析 CSV 文件，找出支持 TLS 1.3 / HTTP/2 的可用域名
* 只保留国内访问速度快、不重定向的网站

> 🔗 [RealityChecker GitHub 下载地址](https://github.com/V2RaySSR/RealityChecker/)

### 🔹 示例

```
server8.webhostmost.com:443
```

* 可作为 `dest` 和 `serverNames` 填入 Xray 配置。

---

## ⚙️ 四、Xray 服务端配置

将以下内容保存为 `/home/container/server.json`（或任意位置）：

```json
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
			"listen": "0.0.0.0",
            "port": 3190,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "cfbd986e-dab5-48c7-ab8a-7d18b1487037"
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
                    "privateKey": "wMgw3c2SIhK6Wc4oZOX7b4vOPzhALSkpw5gMJU6sQ0Y",
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
                        "88d5e2f99bba521c"
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

---

### 📘 字段解释（重点）

| 字段                      | 说明                                 |
| ----------------------- | ---------------------------------- |
| `"protocol": "vless"`   | 使用无加密轻量协议 VLESS                    |
| `"security": "reality"` | 开启 Reality 流量伪装                    |
| `"privateKey"`          | 服务端私钥（仅此服务器保存）                     |
| `"serverNames"`         | 模拟握手时的 SNI 域名                      |
| `"shortIds"`            | 短 ID，用于客户端匹配验证                     |
| `"fingerprints"`        | 浏览器指纹伪装类型（chrome/firefox/safari 等） |
| `"dest"`                | 实际伪装的 HTTPS 网站                     |

---

## ▶️ 五、启动服务

后台运行：

```bash
nohup ./xray run -c /home/container/server.json > xray.log 2>&1 &
```

实时查看日志：

```bash
tail -f xray.log
```

如无报错，即代表启动成功。

---

## 📱 六、V2RayN 客户端配置

打开 V2RayN → 右键 → “添加[VLESS]服务器”，按下表填写：

| 中文名称     | 英文字段        | 示例                                          |
| -------- | ----------- | ------------------------------------------- |
| 地址       | Address     | VPS 公网 IP（例：158.69.xx.xx）                   |
| 端口       | Port        | 14559                                       |
| 用户 ID    | UUID        | f76f4404-73a0-439e-b449-65ddca9d8614        |
| 传输协议     | Network     | tcp                                         |
| 伪装类型     | Type        | none                                        |
| 底层安全     | Security    | reality                                     |
| SNI      | Server Name | server8.webhostmost.com                     |
| 指纹       | Fingerprint | chrome                                      |
| 公钥       | Public Key  | CYXTxXy_FfOhiJ9-4XlUeb6bX5Xt-3zVzZ2yB2B1dTo |
| Short ID | Short ID    | fe4f754a504e7cac                            |
| 路径       | SpiderX     | /                                           |

---

## ✅ 七、测试连接

1. 在 V2RayN 中点击 “连接”
2. 打开 [https://www.google.com](https://www.google.com)
3. 若日志提示：

```
connected successfully
```

即配置完成 🎉

---

## 🧠 八、常见问题

| 问题                  | 原因             | 解决办法                           |
| ------------------- | -------------- | ------------------------------ |
| 启动报错 "invalid JSON" | JSON 格式错误或多余逗号 | 用 `jq . server.json` 检查语法      |
| 客户端连接失败             | 公钥/短 ID 不匹配    | 确认 PublicKey 与 PrivateKey 一致配对 |
| 打不开网站               | 伪装目标不合适        | 换成能直连 HTTPS 的网站                |
| 显示 handshake 超时     | VPS 网络或端口未开放   | 检查防火墙、端口号                      |

---

## 🧾 九、小结

| 项目    | 内容                            |
| ----- | ----------------------------- |
| 协议    | VLESS + REALITY               |
| 端口    | 14559                         |
| 加密    | 无需额外加密（REALITY 已加密）           |
| 优点    | 无需域名 · 高伪装 · 无 CDN 限制         |
| 核心程序  | Xray-core                     |
| 推荐客户端 | V2RayN / Nekoray / Clash.Meta |

---
