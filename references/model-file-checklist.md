# 模型文件清单

一个模型要正常运行，通常需要多个文件。本文档说明如何判断和下载完整的模型文件集。

---

## 文件类型说明

| 文件类型 | 文件名模式 | 是否必需 | 说明 |
|---------|-----------|---------|------|
| 主模型 | `*-Q4_K_M.gguf` 等 | ✅ 必需 | 模型权重文件，按量化级别有不同版本 |
| 多模态投影器 | `mmproj-*.gguf` | 视情况 | 多模态模型（视觉、音频）需要，纯文本模型没有 |
| 配置文件 | `configuration.json` / `config.json` | ⚠️ 推荐 | 模型架构和参数配置 |
| 分词器 | `tokenizer.json` / `tokenizer.model` | ⚠️ 推荐 | 分词器配置 |
| 特殊 tokens | `added_tokens.json` / `special_tokens.json` | 可选 | 补充分词配置 |
| 生成配置 | `generation_config.json` | 可选 | 生成参数默认值 |
| 许可证 | `LICENSE*` / `README.md` | 可选 | 使用条款 |

---

## 如何判断需要哪些文件

### 方法一：查看模型仓库中的说明

在 ModelScope / HuggingFace 的模型页面，通常会有 README 说明该模型需要哪些文件。

### 方法二：根据模型类型判断

| 模型类型 | 必需文件 | 示例 |
|---------|---------|------|
| 纯文本 LLM | 主模型 GGUF + configuration.json | Qwen3-8B、Llama-3.2 |
| 多模态（视觉） | 主模型 GGUF + mmproj GGUF + configuration.json | Qwen3-VL、LLaVA |
| 多模态（音频） | 主模型 GGUF + mmproj GGUF + configuration.json | Whisper |

### 方法三：查看同仓库中的其他文件

在模型仓库页面，查找所有非 GGUF 文件。如果存在 `mmproj-*.gguf`，说明这是多模态模型，需要额外下载。

---

## 下载策略

### 必需文件：逐个指定下载

只下载需要的文件，避免浪费时间下载用不到的量化版本：

```bash
# 下载特定模型需要的所有文件
modelscope download --model '<ModelID>' \
  'Qwen3.5-4B-Q4_K_M.gguf' \
  'mmproj-Qwen3.5-4B-BF16.gguf' \
  'configuration.json' \
  --local_dir '<目标路径>'
```

### 不要用 `--include '*.gguf'` 盲目下载

这会下载仓库中所有 GGUF 文件（包括所有量化版本），可能下载几十 GB 的不需要的文件。

---

## 常见模型文件示例

### Qwen3.5-4B（多模态）

```
Qwen3.5-4B-Q4_K_M.gguf          # 主模型（2.4 GB）— 必需
mmproj-Qwen3.5-4B-BF16.gguf     # 视觉投影器（180 MB）— 必需
configuration.json               # 模型配置 — 推荐
```

### Qwen3-8B（纯文本）

```
Qwen3-8B-Q4_K_M.gguf            # 主模型 — 必需
configuration.json               # 模型配置 — 推荐
```

### Llama-3.2-3B-Instruct（纯文本）

```
Llama-3.2-3B-Instruct-Q4_K_M.gguf   # 主模型 — 必需
```

---

## 下载前预检查

下载前应检查：

1. **目标路径是否已存在同名文件** — 避免重复下载
2. **磁盘剩余空间** — 确保能放下所有文件（主模型 + 配套文件）
3. **模型仓库中文件列表** — 确认哪些文件是必需的

```bash
# 检查目标路径现有文件
ls -la "<目标路径>" 2>/dev/null

# 检查磁盘剩余空间
df -h "<目标磁盘>"              # Linux
Get-PSDrive <盘符>              # Windows PowerShell

# 检查磁盘剩余空间（Windows CMD）
wmic logicaldisk get size,freespace,caption
```
