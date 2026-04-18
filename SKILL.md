---
name: llm-download
description: 使用专业多线程下载工具为本机大模型服务器（Ollama / LM Studio）加速下载大模型，支持代理服务器配置。当用户说"下载模型"、"帮我装个模型"、"pull 个模型"、"下载 Qwen"、"装个 llama"、"给 ollama 装模型"、"给 lmstudio 加模型"、"从 modelscope 下载"、"从 huggingface 下载模型"、"模型下载太慢"、"加速下载模型"时自动触发。即使用户只是提到想要在本地运行某个模型，也应触发此技能。
compatibility: Windows (Git Bash) and Linux. Requires Python with modelscope package. Optional: aria2, HuggingFace CLI, Ollama or LM Studio.
metadata:
  author: cherryclaw
  version: "1.2"
allowed-tools: Bash(aria2c:*) Bash(pip:*) Bash(modelscope:*) Bash(hf:*) Bash(ollama:*) Bash(lms:*) Bash(mkdir:*) Bash(rm:*) Bash(rmdir:*) Bash(curl:*) Bash(ls:*) Bash(dir:*) Bash(wmic:*) Bash(certutil:*) Bash(nvidia-smi:*) Bash(sha256sum:*) Bash(systeminfo:*) Bash(python3:*) Bash(python:*) Read
---

# 大模型下载技能

用专业多线程下载工具替代 Ollama / LM Studio 自带的下载器，**大幅提升下载速度**，并支持**代理服务器**配置以应对特殊网络环境。

支持 **Ollama** 和 **LM Studio**，支持 **Windows** 和 **Linux**。

## 核心特性

- **数据源三级优先级**：模型服务软件官方库 → 模型官方仓库 → 第三方仓库，兼容性优先
- **aria2 多线程下载**：16 线程并行，充分利用带宽
- **ModelScope 优先**：国内 CDN 速度远超 HuggingFace
- **灵活代理配置**：解决网络受限环境下的访问问题
- **断点续传**：大文件下载中断后可继续，不用重头开始

## 工作流程

```
分析需求 → 准备工具 → 配置代理（可选）→ 查找数据源 → 准备路径 → 下载 → 导入 → 验证 → 报告
```

---

## 第一步：分析需求与环境检测

确认以下信息（未提供的主动询问）：**目标模型**、**量化格式**、**模型服务器**（Ollama / LM Studio）、**网络环境**（是否需要代理）。

然后按顺序检测环境，详细命令见 `references/environment-check.md`：

1. **检测模型服务器**：`ollama list` / `lms status`，都没检测到则询问用户
2. **检测实际模型存放路径**（⚠️ 不要假设默认路径，必须检测实际配置）
   - Ollama：检查 `OLLAMA_MODELS` 环境变量
   - LM Studio：读 `~/.lmstudio-home-pointer` → `<home>/settings.json` 的 `downloadsFolder`
3. **检测硬件**（GPU 显存 + 内存）→ 推荐量化级别（默认 Q4_K_M）
4. **磁盘空间预检查** → 不足 1.1 倍预估下载量则预警
5. **重复下载检查** → 已存在则跳过

---

## 第二步：准备工具与代理

### 下载工具选择

| 下载源 | 推荐工具 | 原因 |
|--------|---------|------|
| **ModelScope** | modelscope CLI / Python API | 国内 CDN 直连，内置多线程（30-45 MB/s） |
| **HuggingFace** | aria2 | 多线程 + 代理，突破单线程限速 |
| **Ollama 原生** | `ollama pull` | 自动处理下载、校验、断点续传 |

安装命令：`pip install modelscope`、`pip install "huggingface_hub[cli]"`。aria2 安装详见 `references/proxy-and-mirrors.md`。

> Windows 下 modelscope CLI 可能因 python 重定向不可用，用 Python API 更可靠（实际 Python 路径见 `references/environment-check.md`）。

### 代理配置（按需）

检测 `curl -sI https://huggingface.co --connect-timeout 5`，超时或极慢则建议配置代理：

```bash
aria2c --all-proxy="http://127.0.0.1:7890" ...     # aria2 用代理
export HF_ENDPOINT=https://hf-mirror.com            # HF 用国内镜像
```

完整代理配置、镜像站地址、直链格式见 `references/proxy-and-mirrors.md`。

---

## 第三步：查找模型数据源

按**数据源权威性**分三级优先级查找，同一级内 **ModelScope 优先**，找不到再查 HuggingFace。

```
Level 1 → 模型服务软件提供商的官方模型库（兼容性最佳）
Level 2 → 模型官方提供商的 GGUF 仓库（官方出品）
Level 3 → 第三方量化仓库（量化选项最全）
兜底    → 征求用户意见
```

详细的数据源清单、命名规律、检测方法见 `references/data-sources.md`。

### 需要下载的文件

**不要用 `--include '*.gguf'` 盲目下载所有文件**，逐个指定。常见文件类型：
- ✅ **主模型 GGUF** — 必需
- ⚠️ **mmproj GGUF** — 多模态模型必需（纯文本模型没有）
- ⚠️ **configuration.json** — 推荐

完整判断方法见 `references/model-file-checklist.md`。

### 搜索流程

#### Level 1 — 模型服务软件提供商

**LM Studio**：检测 `lmstudio-community/{模型名}-GGUF` 是否存在（先 ModelScope 后 HuggingFace）。

```bash
curl -sI "https://modelscope.cn/models/lmstudio-community/{模型名}-GGUF" -o /dev/null -w '%{http_code}'
```

`200` → 仓库存在，列出量化版本并选择。`404` → Level 2。

> 💡 `lmstudio-community` 的 GGUF 经过 LM Studio 团队验证，兼容性最佳，通常提供 4-8 个量化级别。

**Ollama**：如果只需本地运行，直接 `ollama pull {模型名}:{tag}` 即可（这是最快的方式）。需要 GGUF 文件则继续 Level 2/3。

#### Level 2 — 模型官方提供商

查找模型原始作者的 GGUF 仓库。**注意**：官方仓库通常只有 F32/FP16（文件很大），没有量化版本。

```bash
curl -sI "https://modelscope.cn/models/{官方组织}/{模型名}-GGUF" -o /dev/null -w '%{http_code}'
```

有合适量化 → 使用。只有 F32/FP16 或无 GGUF → Level 3。

官方组织映射表见 `references/data-sources.md`。

#### Level 3 — 第三方量化仓库

按顺序尝试（每个先查 ModelScope 再查 HuggingFace）：

1. **bartowski/{模型名}-GGUF** — 量化最全
2. **unsloth/{模型名}-GGUF** — Dynamic 2.0 量化
3. **QuantFactory/{模型名}-GGUF** — 标准量化
4. **NexaAIDev/{模型名}-GGUF** — ModelScope 活跃

```bash
# 批量并行检测
for repo in "bartowski/{模型名}-GGUF" "unsloth/{模型名}-GGUF" "QuantFactory/{模型名}-GGUF" "NexaAIDev/{模型名}-GGUF"; do
    (code=$(curl -sI "https://modelscope.cn/models/$repo" -o /dev/null -w '%{http_code}')
     echo "$code $repo") &
done
wait
```

找到第一个有目标量化版本的仓库 → 使用。

#### 兜底

所有级别都没找到 → 报告已尝试的仓库，建议用户手动搜索或指定仓库地址。

### 列出量化版本

找到仓库后列出可用文件，API 调用方法见 `references/data-sources.md` 的「列出量化版本」章节。

如果用户没指定量化级别，**默认推荐 Q4_K_M**。详细推荐逻辑见 `references/quantization-guide.md`。

---

## 第四步：准备输出路径

使用第一步检测到的**实际模型存放路径**。

- **Ollama**：下载到临时目录（`~/tmp_model`），用 `ollama create` 导入
- **LM Studio**：直接下载到 `<实际路径>/<publisher>/<model-name>/`

```bash
mkdir -p "<路径>/<publisher>/<model-name>"
```

---

## 第五步：下载模型

根据数据源选择下载方式。

### 场景 A：Ollama 原生拉取

Ollama 用户且只需本地运行 → 直接 `ollama pull {模型名}:{tag}`，无需额外工具。

### 场景 B：ModelScope 下载（国内最快）

适用于所有 Level 1/2/3 的 ModelScope 镜像。

```bash
# modelscope CLI（Linux）
modelscope download --model '<ModelID>' '<文件>' --local_dir '<目标路径>'

# Python API（Windows 推荐）
python3 -c "
from modelscope.hub.file_download import model_file_download
for f in ['<文件1>.gguf', 'configuration.json']:
    print(f'Downloading {f}...')
    path = model_file_download('<ModelID>', f, local_dir='<目标路径>')
    print(f'  Done: {path}')
"
```

### 场景 C：HuggingFace 下载（aria2 加速）

ModelScope 没有目标模型时使用。直链格式见 `references/proxy-and-mirrors.md`。

```bash
aria2c -x 16 -s 16 -k 1M \
  [--all-proxy="http://127.0.0.1:7890"] \
  -d "<目标目录>" -o "<文件名>.gguf" \
  "<下载链接>"
```

### 断点续传

两种工具都支持：重新执行相同命令即可继续。

---

## 第六步：导入模型服务器

### Ollama

下载 GGUF 后创建 Modelfile 并导入，模板和详细步骤见 `references/model-import.md`：

```bash
ollama create <模型名> -f ./tmp_model/Modelfile
rm -rf ./tmp_model    # 清理临时文件
```

### LM Studio

LM Studio 自动发现模型目录中的新 GGUF 文件。未识别时用 `lms import <path/to/model.gguf>`。

---

## 第七步：验证与报告

### 验证

1. **文件大小** — `ls -lh` 对比源站，误差 < 1%
2. **临时文件** — `.aria2` 应已清除；`._____temp/` 残留不影响使用
3. **SHA256 校验**（可选）— `sha256sum` / `certutil -hashfile`

### 测试

```bash
# Ollama
ollama run <模型名> "你好，请简单介绍一下你自己。"

# LM Studio（需要先启动服务器）
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"<文件名>","messages":[{"role":"user","content":"你好"}],"max_tokens":200}'
```

### 报告

报告模板见 `references/report-template.md`。必须包含五项信息：

| 必填项 | 示例 |
|--------|------|
| 模型服务器 | Ollama / LM Studio |
| 存放路径 | `E:\.lmstudio\models\<publisher>\<model-name>\` |
| 下载工具 | modelscope Python API / aria2 |
| 下载源 | ModelScope `lmstudio-community/...` |
| 文件清单 | `gemma-2-2b-it-Q8_0.gguf`（2.6 GB） |

---

## 故障排除

见 `references/troubleshooting.md`：下载速度慢、代理连接失败、下载中断、磁盘空间不足、导入失败、模型无法启动。

---

## 参考文件

| 文件 | 内容 | 何时查阅 |
|------|------|---------|
| `references/data-sources.md` | 三级数据源优先级、仓库命名规律、搜索检测 API、官方组织映射 | 查找模型下载源时 |
| `references/environment-check.md` | 硬件检测、路径检测、磁盘检查的详细命令 | 执行环境检测时 |
| `references/model-import.md` | Ollama Modelfile 模板、导入步骤、临时文件清理 | 导入模型到 Ollama 时 |
| `references/model-file-checklist.md` | 模型文件类型判断、下载前预检查 | 确定需要下载哪些文件时 |
| `references/proxy-and-mirrors.md` | aria2 安装、代理配置、镜像站、直链格式 | 安装 aria2 或网络受限时 |
| `references/quantization-guide.md` | 硬件推荐逻辑、量化级别对照表 | 选择量化版本时 |
| `references/report-template.md` | 成功/失败/部分成功报告模板 | 向用户报告结果时 |
| `references/troubleshooting.md` | 下载速度、代理、断点续传、导入等问题排查 | 遇到错误时 |

## 支持矩阵

| 特性 | Ollama | LM Studio |
|------|--------|-----------|
| Level 1 数据源 | `ollama pull`（原生注册表） | `lmstudio-community`（HF/ModelScope） |
| 下载方式 | 原生 pull 或 GGUF → `ollama create` | 下载 GGUF 到模型目录 |
| 模型格式 | GGUF（OCI 封装存储） | 直接 GGUF 文件 |
| API 端口 | 11434 | 1234 |
| ModelScope | modelscope 下载 GGUF | modelscope 下载 GGUF |
| HuggingFace | aria2 多线程下载 GGUF | aria2 多线程下载 GGUF |
