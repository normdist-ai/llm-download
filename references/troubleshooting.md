# 故障排除

下载和使用过程中常见问题的排查指南。

---

## 下载速度慢

| 检查项 | 排查方法 |
|--------|---------|
| aria2 多线程是否生效 | 确认命令中包含 `-x 16 -s 16`，观察 aria2 输出中的连接数 |
| 下载源选择 | ModelScope（国内 CDN）通常比 HuggingFace 快很多 |
| 代理线路 | 如果用了代理，尝试切换节点；如果没用代理，考虑是否需要代理加速 |
| 磁盘写入速度 | 机械硬盘可能是瓶颈，考虑下载到 SSD |
| 网络带宽 | 运行 `speedtest-cli` 或访问测速网站确认实际带宽 |

---

## 代理连接失败

排查步骤：

1. **确认代理软件已启动**
   ```bash
   # 测试代理是否可用
   curl --proxy http://127.0.0.1:7890 https://huggingface.co --connect-timeout 5
   ```

2. **确认端口正确**
   - 在代理软件界面查看实际监听端口
   - 常见端口：Clash 7890、V2Ray 10808/10809、SS 1080

3. **确认协议匹配**
   - 有些代理软件 HTTP 和 SOCKS5 端口不同
   - 尝试 HTTP 代理和 SOCKS5 代理互换

4. **防火墙**
   ```bash
   # Linux: 检查端口是否放行
   sudo ufw status
   # Windows: 检查防火墙规则
   netsh advfirewall firewall show rule name=all | findstr "7890"
   ```

5. **允许局域网连接**
   - 部分代理软件默认只监听 127.0.0.1
   - 确认「Allow LAN」或类似选项已开启

---

## 下载中断

aria2 默认启用断点续传（`continue=true`），处理方式：

1. **重新执行相同命令** — aria2 会自动从断点继续
2. **检查控制文件** — 确保 `.aria2` 控制文件还在（与下载文件同目录）
3. **控制文件丢失** — 删除部分下载的文件和 `.aria2` 文件，重新下载
4. **网络波动** — 配置重试参数：
   ```bash
   aria2c --max-tries=10 --retry-wait=15 --timeout=120 ...
   ```

---

## 磁盘空间不足

```bash
# Linux 检查剩余空间
df -h
du -sh ~/.ollama/models    # 查看 Ollama 已用空间
du -sh ~/.lmstudio/models  # 查看 LM Studio 已用空间

# Windows PowerShell 检查剩余空间
Get-PSDrive C
# 查看模型目录大小
"{0:N2} GB" -f ((Get-ChildItem "$env:USERPROFILE\.ollama\models" -Recurse | Measure-Object Length -Sum).Sum / 1GB)
```

**节省空间的方法：**
- 使用更低的量化级别（Q4_K_M 代替 Q8_0，体积减少约 40%）
- 将模型存放到其他磁盘（修改 `OLLAMA_MODELS` 环境变量或 LM Studio 模型目录）
- 删除不再使用的旧模型：`ollama rm <模型名>`

---

## Ollama 导入失败

| 问题 | 解决方法 |
|------|---------|
| GGUF 文件不完整 | 比对文件大小与源站标注是否一致 |
| Modelfile 路径错误 | 使用绝对路径而非相对路径 |
| 模型架构不支持 | 确认 Ollama 版本支持该模型架构（`ollama --version` 检查） |
| 权限问题 | Linux 下检查文件读写权限 |

**查看详细日志：**
```bash
OLLAMA_DEBUG=1 ollama serve
```

---

## LM Studio 找不到模型

| 检查项 | 说明 |
|--------|------|
| 目录结构 | 必须是 `<publisher>/<model-name>/<file>.gguf` |
| 文件扩展名 | 必须是 `.gguf` |
| 刷新模型列表 | 在 LM Studio 中点击刷新 |
| 手动导入 | `lms import <path/to/model.gguf>` |
| 符号链接导入 | `lms import --symbolic-link <path/to/model.gguf>` |

---

## 模型无法启动 / 推理失败

| 问题 | 排查方向 |
|------|---------|
| 内存不足 | 检查可用内存是否满足量化级别要求（见量化指南） |
| GGUF 格式损坏 | 重新下载，验证 SHA256 |
| 不支持的架构 | 确认模型服务器版本是否支持该架构 |
| GPU 兼容性 | 检查 CUDA / Metal / Vulkan 支持情况 |
