# Xray-core for HarmonyOS

> Xray-core 原生鸿蒙 ARM64 编译版本，支持 VLESS、VMess、Trojan、Shadowsocks、REALITY、XTLS Vision 等全部协议。
>
> 基于 [XTLS/Xray-core](https://github.com/XTLS/Xray-core) 源码编译，感谢 [Harmonybrew](https://harmonybrew.atomgit.com) 社区提供的 Go 鸿蒙适配方案。

## 适用设备

- HarmonyOS 6.1+ / OpenHarmony 6.1+
- ARM64 架构（华为 MateBook Pro、鸿蒙平板等）

## 快速开始

### 方式一：下载预编译版本

1. 下载 `xray-harmonyos-arm64.tar.gz`
2. 解压并运行安装脚本：

```bash
tar xzf xray-harmonyos-arm64.tar.gz
cd xray-harmonyos
./install.sh
```

脚本会自动安装签名工具、签名 xray、下载 geoip/geosite 数据文件。

3. 编辑配置文件：

```bash
nano config.json
```

4. 启动：

```bash
./xray run -c config.json
```

5. 浏览器设置 SOCKS5 代理 `127.0.0.1:10808` 即可。

### 方式二：自行编译

#### 1. 安装 Harmonybrew

在鸿蒙终端执行：

```bash
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)"
echo >> /storage/Users/currentUser/.zshrc
echo 'eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"' >> /storage/Users/currentUser/.zshrc
eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"
```

#### 2. 安装 Go 和签名工具

```bash
brew install go ohos-sdk llvm-gcc-compat
```

#### 3. 下载源码

```bash
curl -L -o xray.tar.gz https://github.com/XTLS/Xray-core/archive/refs/tags/v26.3.27.tar.gz
tar xzf xray.tar.gz
mv Xray-core-26.3.27 Xray-core
cd Xray-core
```

#### 4. 编译

```bash
CGO_ENABLED=0 go build -o xray -trimpath -ldflags="-s -w" ./main/
```

#### 5. 签名

```bash
binary-sign-tool ./xray
```

#### 6. 下载数据文件

```bash
curl -L -o geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
curl -L -o geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat
```

#### 7. 运行

```bash
./xray version
./xray run -c config.json
```

## 配置文件模板

```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [
    {
      "tag": "socks",
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": { "udp": true }
    },
    {
      "tag": "http",
      "port": 10809,
      "listen": "127.0.0.1",
      "protocol": "http"
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vless",
      "settings": {
        "vnext": [{
          "address": "你的服务器地址",
          "port": 443,
          "users": [{
            "id": "你的UUID",
            "encryption": "none",
            "level": 0,
            "flow": "xtls-rprx-vision"
          }]
        }]
      },
      "streamSettings": {
        "security": "reality",
        "realitySettings": {
          "serverName": "目标网站域名",
          "publicKey": "服务器公钥",
          "shortId": "ShortID",
          "fingerprint": "chrome"
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      { "type": "field", "ip": ["geoip:private"], "outboundTag": "direct" }
    ]
  }
}
```

## 浏览器代理设置

Xray 启动后，在浏览器或系统中设置代理：

| 协议 | 地址 | 端口 |
|------|------|------|
| SOCKS5 | 127.0.0.1 | 10808 |
| HTTP | 127.0.0.1 | 10809 |

### 终端临时代理

```bash
export http_proxy=http://127.0.0.1:10809
export https_proxy=http://127.0.0.1:10809
export all_proxy=socks5://127.0.0.1:10808
```

## 支持的协议

| 协议 | 传输 | 安全 | 状态 |
|------|------|------|------|
| VLESS | TCP | REALITY | ✅ |
| VLESS | TCP | TLS + XTLS Vision | ✅ |
| VLESS | WebSocket | TLS | ✅ |
| VLESS | gRPC | TLS | ✅ |
| VMess | TCP/WebSocket | TLS/None | ✅ |
| Trojan | TCP | TLS | ✅ |
| Shadowsocks | TCP | - | ✅ |

## 常见问题

### Q: 运行报错 "not signed" 或 "Permission denied"

需要签名：`binary-sign-tool ./xray`

### Q: 运行报错 "geoip.dat not found"

确保 `geoip.dat` 和 `geosite.dat` 与 `xray` 在同一目录，或配置文件中使用绝对路径。

### Q: Go 编译时网络超时

```bash
export GOPROXY=https://goproxy.cn,direct
export HTTPS_PROXY=http://127.0.0.1:你的代理端口
```

### Q: curl 下载 GitHub 文件超时

使用 GitHub tarball 方式下载源码，比 git clone 更稳定：

```bash
curl -L -o xray.tar.gz https://github.com/XTLS/Xray-core/archive/refs/tags/v26.3.27.tar.gz
```

## 致谢

- [XTLS/Xray-core](https://github.com/XTLS/Xray-core) — Xray-core 原始项目
- [Harmonybrew](https://harmonybrew.atomgit.com) — 鸿蒙 Homebrew 社区
- [ohos-sdk](https://www.openharmony.cn/) — 鸿蒙开发工具包

## License

[Mozilla Public License Version 2.0](LICENSE)
