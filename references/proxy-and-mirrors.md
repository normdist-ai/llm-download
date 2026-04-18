# 代理与镜像配置

特殊网络环境下的代理设置和镜像站信息。配置代理可以解决 HuggingFace 无法访问、ModelScope 速度慢等问题。

---

## aria2 安装来源

### 官方地址

| 资源 | 地址 |
|------|------|
| GitHub 仓库 | https://github.com/aria2/aria2 |
| 最新 Release | https://github.com/aria2/aria2/releases/latest |

### Windows 直接下载

```
https://github.com/aria2/aria2/releases/download/release-1.37.0/aria2-1.37.0-win-64bit-build1.zip
```

下载后解压，将包含 `aria2c.exe` 的目录加入系统 PATH 即可。

### Linux 安装

| 发行版 | 命令 |
|--------|------|
| Debian/Ubuntu | `sudo apt update && sudo apt install aria2` |
| Fedora/RHEL | `sudo dnf install aria2` |
| Arch/Manjaro | `sudo pacman -S aria2` |
| openSUSE | `sudo zypper install aria2` |

---

## 国内镜像站

| 镜像站 | 用途 | 地址 |
|--------|------|------|
| HF-Mirror | HuggingFace 国内镜像 | `https://hf-mirror.com` |
| ModelScope | 国内模型平台（本身就是国内源） | `https://modelscope.cn` |

### 使用 HF-Mirror

**临时使用：**
```bash
export HF_ENDPOINT=https://hf-mirror.com
```

**Windows PowerShell：**
```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

**持久化：** 写入 shell 配置文件（`~/.bashrc` / `~/.zshrc`）或系统环境变量。

设置后 `hf download` 命令会自动使用镜像站，无需额外参数。

---

## 常见代理软件端口

| 代理软件 | 默认端口 | 协议 |
|---------|---------|------|
| Clash / Clash Verge | 7890 | HTTP |
| Clash / Clash Verge | 7891 | SOCKS5 |
| V2RayN / V2RayNG | 10808 | SOCKS5 |
| V2RayN | 10809 | HTTP |
| Shadowsocks | 1080 | SOCKS5 |
| SSR | 1087 | HTTP |

> 以上是默认值，实际端口以用户代理软件设置为准。务必向用户确认。

---

## aria2 代理配置

### 命令行参数（临时）

```bash
# HTTP 代理
aria2c --all-proxy="http://127.0.0.1:7890" ...

# SOCKS5 代理
aria2c --all-proxy="socks5://127.0.0.1:7890" ...

# 带认证的代理
aria2c --all-proxy="http://user:password@127.0.0.1:7890" ...
```

### 配置文件（持久化，推荐）

创建 `~/.aria2/aria2.conf`（Linux）或 `%USERPROFILE%\.aria2\aria2.conf`（Windows）：

```ini
# ===== 代理设置 =====
# 根据实际代理软件修改地址和端口
all-proxy=http://127.0.0.1:7890

# 不走代理的地址（国内源直连）
no-proxy=localhost,127.0.0.1,modelscope.cn,*.modelscope.cn

# ===== 下载优化 =====
max-connection-per-server=16
split=16
min-split-size=1M
continue=true
max-tries=5
retry-wait=10

# ===== 日志 =====
# log-level=warn
# log=/path/to/aria2.log
```

---

## modelscope CLI 代理配置

```bash
# Linux/macOS
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export NO_PROXY=modelscope.cn,*.modelscope.cn

# Windows PowerShell
$env:HTTP_PROXY = "http://127.0.0.1:7890"
$env:HTTPS_PROXY = "http://127.0.0.1:7890"
$env:NO_PROXY = "modelscope.cn,*.modelscope.cn"
```

> ModelScope 是国内平台，通常**不需要代理**。仅在特殊环境（如企业内网）下配置。

---

## Hugging Face CLI 代理配置

```bash
# 方式一：使用国内镜像（推荐，不需要代理）
export HF_ENDPOINT=https://hf-mirror.com

# 方式二：使用代理
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

---

## 下载源直链格式

获取文件直链后可使用 aria2 多线程下载。

### ModelScope 直链

```
https://modelscope.cn/models/<ModelID>/resolve/<文件名>
```

示例：
```
https://modelscope.cn/models/Qwen/Qwen3-8B-GGUF/resolve/qwen3-8b-q4_k_m.gguf
```

### HuggingFace 直链

```
https://huggingface.co/<RepoID>/resolve/main/<文件名>
```

HF-Mirror 直链（替换域名即可）：
```
https://hf-mirror.com/<RepoID>/resolve/main/<文件名>
```

示例：
```
https://huggingface.co/Qwen/Qwen3-8B-GGUF/resolve/main/qwen3-8b-q4_k_m.gguf
```

---

## 代理排查清单

1. ✅ 代理软件是否已启动
2. ✅ 端口是否正确（在代理软件界面确认）
3. ✅ 协议是否匹配（HTTP vs SOCKS5）
4. ✅ 防火墙是否放行代理端口
5. ✅ 代理软件是否开启了「允许局域网连接」（Allow LAN）
6. ✅ 用 `curl --proxy <代理地址> https://huggingface.co` 测试连通性
