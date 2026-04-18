# 环境检测详细命令

SKILL.md 第一步的详细检测命令参考。

---

## 检测模型服务器

```bash
ollama list       # 检查 Ollama 是否运行
lms status        # 检查 LM Studio 是否运行
```

两者都没检测到 → 询问用户打算用哪个。

## 检测实际模型存放路径

> ⚠️ **不要假设默认路径**。用户可能已自定义路径，必须检测实际配置。

### Ollama

检查 `OLLAMA_MODELS` 环境变量：

```bash
# Linux
echo $OLLAMA_MODELS

# Windows Git Bash — 检查三级环境变量
echo $OLLAMA_MODELS                              # 当前会话
cat ~/.bashrc 2>/dev/null | grep OLLAMA_MODELS   # 用户级
# 系统级需要 PowerShell
powershell -c "[Environment]::GetEnvironmentVariable('OLLAMA_MODELS', 'User')"
powershell -c "[Environment]::GetEnvironmentVariable('OLLAMA_MODELS', 'Machine')"
```

为空则用默认路径：Linux `~/.ollama/models`，Windows `C:\Users\<用户>\.ollama\models`

### LM Studio

先读 home 指针文件，再读配置：

```bash
cat ~/.lmstudio-home-pointer    # 如 "C:\Users\ZhangJing\.lmstudio"
cat "<home>/settings.json" | grep downloadsFolder
```

`settings.json` 中 `downloadsFolder` 字段即为实际模型路径（如 `E:\.lmstudio\models`）。

## 检测硬件

### GPU 显存

```bash
nvidia-smi --query-gpu=memory.total --format=csv,noheader
```

### 系统内存

```bash
# === Linux ===
free -h

# === Windows (Git Bash) ===
# wmic 在 Git Bash 中输出 GBK 编码会乱码，用以下方式代替：
cat /proc/meminfo 2>/dev/null | grep -i "MemTotal\|MemAvailable"
# 或通过 Python（更可靠）
python3 -c "
import ctypes
kernel32 = ctypes.windll.kernel32
free = ctypes.c_ulonglong(0)
total = ctypes.c_ulonglong(0)
kernel32.GetDiskFreeSpaceExW(None, ctypes.pointer(free), ctypes.pointer(total), None)
print(f'Total: {total.value / (1024**3):.1f} GB, Free: {free.value / (1024**3):.1f} GB')
"
```

## 磁盘空间预检查

```bash
# Linux
df -h "<目标路径>"

# Windows (Git Bash)
df -h /e/    # E 盘示例

# Windows PowerShell
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free
```

剩余空间不足预估下载量的 1.1 倍 → 预警并建议使用更低的量化级别或更换磁盘。

## 重复下载检查

```bash
ls -lh "<目标路径>/<publisher>/<model-name>/" 2>/dev/null
```

已存在且文件大小与源站一致 → 跳过下载，报告"模型已存在"。

## Windows Python 路径问题

`python` 命令可能指向 Windows Store 重定向（exit code 49）。实际路径：

```bash
where python                                    # 排除 WindowsApps 的结果
# 常见实际路径：/c/Users/<用户>/AppData/Local/Python/bin/python.exe
```

用实际路径替代 `python3` 命令中的 `python3`。
