# LLM Download Skill

[🇨🇳 中文文档](#llm-下载技能) | [🇺🇸 English Documentation](#llm-download-skill)

---

<a id="llm-download-skill"></a>
## 🇺🇸 English Documentation

A professional multi-threaded download skill for large language models, accelerating downloads for local model servers (Ollama / LM Studio) with proxy support.


## Overview

This skill follows the Agent Skills open standard and provides a systematic approach to downloading large language models using professional multi-threaded download tools instead of built-in downloaders from Ollama or LM Studio. It significantly improves download speeds and supports proxy configuration for restricted network environments.

When triggered by phrases like "download model", "install a model", "pull Qwen", "add model to lmstudio", "download from modelscope", or "accelerate model download", the agent automatically uses this skill to optimize the download process.


## Compatibility


### Supported Platforms

| Platform | Support Status | Description |
| --- | --- | --- |
| Windows | ✅ Fully Supported | Git Bash / PowerShell with Python |
| Linux | ✅ Fully Supported | Bash with Python |
| macOS | ✅ Compatible | Bash with Python |


### Supported Model Servers

| Server | Support Status | Import Method |
| --- | --- | --- |
| Ollama | ✅ Fully Supported | Native pull or GGUF → `ollama create` |
| LM Studio | ✅ Fully Supported | Direct GGUF file placement |


### Supported Agent Platforms

This skill runs on the following Agent platforms:


| Platform | Support Status | Description |
| --- | --- | --- |
| OpenClaw | ✅ Fully Supported | Recommended Platform - Open-source self-hosted Agent platform |
| Claude Code | ✅ Fully Supported | Anthropic's official CLI tool |
| Cherry Studio | ✅ Fully Supported | Integrated AI assistant platform |
| Other Agent Frameworks | ✅ Compatible | Any framework supporting SKILL.md specification |


## Technical Features


- ✅ **Three-Level Data Source Priority**: Official model server libraries → Official model repositories → Third-party repositories for maximum compatibility
- ✅ **Multi-threaded Downloads**: aria2 with 16 parallel threads to maximize bandwidth utilization
- ✅ **ModelScope Priority**: Domestic CDN delivers speeds far exceeding HuggingFace (30-45 MB/s)
- ✅ **Flexible Proxy Configuration**: Solves access issues in restricted network environments
- ✅ **Resume Support**: Large file downloads can resume after interruption without restarting
- ✅ **Automatic Hardware Detection**: GPU VRAM + RAM detection for optimal quantization recommendations
- ✅ **Smart File Selection**: Downloads only necessary files, avoiding unnecessary bloat
- ✅ **Cross-Platform**: Works on Windows (Git Bash/PowerShell) and Linux/macOS (Bash)


## Core Workflow

```
Analyze Requirements → Prepare Tools → Configure Proxy (Optional) → Find Data Sources → Prepare Paths → Download → Import → Verify → Report
```


## Installation


### Prerequisites

Before using this skill, ensure you have:

1. **Python 3.x** with pip package manager
2. **Git Bash** (Windows) or **Bash** (Linux/macOS)
3. **Model Server**: Ollama or LM Studio installed
4. **Optional Tools**:
   - `aria2`: For HuggingFace downloads with multi-threading
   - `modelscope` Python package: For ModelScope downloads
   - `huggingface_hub[cli]`: For HuggingFace CLI access


### Installation Steps

When a user requests to install this skill, follow these steps:

1. Clone the repository (must use git clone, do not download ZIP):
```bash
git clone https://github.com/YOUR_USERNAME/llm-download.git
```

2. Detect current IDE/Agent platform: Check for config folders (e.g., `.claude/`, `.lingma/`, etc.)

3. Copy to skills directory: Copy the cloned `llm-download` folder to the appropriate skills directory

### IDE/Agent Skills Directories

| Platform | Skills Directory | Detection Flag |
| --- | --- | --- |
| Claude Code | ~/.config/claude-code/skills/llm-download | Claude Code CLI environment |
| Cherry Studio | [CherryStudio Data]/Agents/.claude/skills/llm-download | Cherry Studio installation |
| Lingma | .lingma/skills/llm-download | .lingma/ folder exists |
| Trae | .trae/skills/llm-download | .trae/ folder exists |
| Cursor | .cursor/skills/llm-download | .cursor/ folder exists |
| VS Code | .vscode/skills/llm-download | .vscode/ folder exists |


### Installation Verification

After installation, verify the skills directory structure:

```
llm-download/
├── SKILL.md              # Skill definition file (required)
├── README.md             # Project documentation
└── references/           # Reference documents
    ├── data-sources.md
    ├── environment-check.md
    ├── model-file-checklist.md
    ├── model-import.md
    ├── model-paths.md
    ├── proxy-and-mirrors.md
    ├── quantization-guide.md
    ├── report-template.md
    └── troubleshooting.md
```


## Usage


### Trigger Phrases

The skill is automatically triggered when you mention:

- Download model, install a model, pull a model
- Download Qwen, install llama, add model to ollama
- Add model to lmstudio, download from modelscope
- Download from huggingface, accelerate model download
- Model download too slow, speed up download


### Example Interactions

```
User: Download Qwen2.5-7B for Ollama

Agent: I'll help you download Qwen2.5-7B using the LLM Download skill.

Step 1: Analyzing requirements and detecting environment...
✓ Detected Ollama installation
✓ Checking actual model storage path...
✓ Hardware detected: 16GB RAM, RTX 3060 (12GB VRAM)
✓ Recommended quantization: Q4_K_M

Step 2: Finding optimal data source...
✓ Found at Level 1: lmstudio-community/Qwen2.5-7B-Instruct-GGUF
✓ Available on ModelScope (faster domestic CDN)

Step 3: Preparing download...
✓ Using modelscope Python API for multi-threaded download
✓ Target path: E:\.ollama\models\...

Step 4: Downloading model files...
[Download progress display]

Step 5: Importing to Ollama...
✓ Model imported successfully as qwen2.5-7b:latest

Step 6: Verification complete!
✓ File size matches source (2.4 GB)
✓ Test run successful

📊 Download Report:
- Model Server: Ollama
- Storage Path: E:\.ollama\models\...
- Download Tool: modelscope Python API
- Data Source: ModelScope lmstudio-community/Qwen2.5-7B-Instruct-GGUF
- Files: qwen2.5-7b-instruct-q4_k_m.gguf (2.4 GB)
```

```
User: Help me install Gemma-2-2B for LM Studio with Q8 quantization

Agent: [Starts LLM download workflow for LM Studio with Q8_0 quantization]
```


## Three-Level Data Source Strategy

The skill uses a hierarchical approach to find the best download source:

### Level 1 - Model Server Provider Libraries (Highest Priority)

**Best compatibility**, verified by model server teams.

- **LM Studio**: `lmstudio-community/{model-name}-GGUF`
  - Usually provides 4-8 quantization levels
  - Tested and verified for LM Studio compatibility
  
- **Ollama**: Native registry via `ollama pull {model}:{tag}`
  - Fastest method for Ollama users
  - Automatic handling of download, verification, and import


### Level 2 - Official Model Provider Repositories

**Official releases** from model creators.

- Search for `{official-org}/{model-name}-GGUF`
- Note: Often only has F32/FP16 (very large files), limited quantization options
- Examples: `google/gemma-2-gguf`, `meta-llama/llama-3-gguf`


### Level 3 - Third-Party Quantization Repositories

**Most quantization options**, community-maintained.

Search order (check ModelScope first, then HuggingFace):

1. **bartowski/{model-name}-GGUF** - Most comprehensive quantization
2. **unsloth/{model-name}-GGUF** - Dynamic 2.0 quantization
3. **QuantFactory/{model-name}-GGUF** - Standard quantization
4. **NexaAIDev/{model-name}-GGUF** - Active on ModelScope


## Download Methods


### Method A: Ollama Native Pull

For Ollama users who just need local execution:

```bash
ollama pull {model-name}:{tag}
```

**Advantages**: No additional tools needed, automatic handling of everything.


### Method B: ModelScope Download (Fastest in China)

Uses ModelScope's domestic CDN for all levels.

```bash
# modelscope CLI (Linux)
modelscope download --model '<ModelID>' '<file>' --local_dir '<target-path>'

# Python API (Recommended for Windows)
python3 -c "
from modelscope.hub.file_download import model_file_download
for f in ['<file1>.gguf', 'configuration.json']:
    print(f'Downloading {f}...')
    path = model_file_download('<ModelID>', f, local_dir='<target-path>')
    print(f'  Done: {path}')
"
```

**Speed**: 30-45 MB/s typical, no proxy needed.


### Method C: HuggingFace Download (aria2 Accelerated)

Used when ModelScope doesn't have the target model.

```bash
aria2c -x 16 -s 16 -k 1M \
  [--all-proxy="http://127.0.0.1:7890"] \
  -d "<target-directory>" -o "<filename>.gguf" \
  "<download-link>"
```

**Features**: 16-thread parallel download, proxy support, resume capability.


## Smart Quantization Recommendations

The skill automatically recommends optimal quantization based on hardware:

| Hardware | Recommended Quantization | Model Size Range |
| --- | --- | --- |
| ≤ 4GB VRAM | Q4_K_S / Q4_0 | Small models only |
| 4-8GB VRAM | Q4_K_M | 2-7B models |
| 8-12GB VRAM | Q5_K_M / Q6_K | 7-13B models |
| 12-16GB VRAM | Q6_K / Q8_0 | 13-30B models |
| 16+GB VRAM | Q8_0 / FP16 | 30B+ models |
| CPU Only | Q4_K_M | Depends on RAM |

**Default**: Q4_K_M (best balance of quality and speed).


## File Selection Strategy

The skill intelligently selects which files to download:

- ✅ **Main Model GGUF** - Required
- ⚠️ **mmproj GGUF** - Required for multimodal models (not present for text-only models)
- ⚠️ **configuration.json** - Recommended
- ❌ **All other files** - Skipped to save bandwidth

**Never uses `--include '*.gguf'`** to blindly download all files.


## Proxy Configuration

For restricted network environments, the skill supports flexible proxy configuration:

### Auto-Detection

Automatically detects network issues:
```bash
curl -sI https://huggingface.co --connect-timeout 5
```

If timeout or very slow → suggests proxy configuration.

### Proxy Setup

```bash
# aria2 with proxy
aria2c --all-proxy="http://127.0.0.1:7890" ...

# HuggingFace mirror
export HF_ENDPOINT=https://hf-mirror.com

# Environment variables
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

Complete proxy configuration details available in `references/proxy-and-mirrors.md`.


## Verification & Reporting


### Verification Steps

1. **File Size Check** - Compare with source, error < 1%
2. **Temporary Files** - Ensure `.aria2` files cleared; `._____temp/`残留 don't affect usage
3. **SHA256 Verification** (Optional) - `sha256sum` / `certutil -hashfile`
4. **Test Run** - Execute sample query to verify model works

### Success Report Template

Every successful download generates a standardized report:

```
✅ Model Download Complete!

📊 Summary:
- Model Server: Ollama / LM Studio
- Storage Path: <actual-path>
- Download Tool: modelscope Python API / aria2
- Data Source: ModelScope/HuggingFace <repository>
- Files Downloaded:
  • <filename>.gguf (<size>)
  
⚙️ Hardware:
- GPU: <GPU model> (<VRAM>)
- RAM: <RAM size>
- Recommended Quantization: <level>

✓ Verification:
- File size matches source
- Test run successful
- Model ready for use
```


## Troubleshooting

Common issues and solutions documented in `references/troubleshooting.md`:

- **Slow download speed** - Switch to ModelScope or configure proxy
- **Proxy connection failure** - Verify proxy address and port
- **Download interrupted** - Resume supported, just re-run command
- **Insufficient disk space** - Pre-check warns if < 1.1× estimated size
- **Import failure** - Check Modelfile format (Ollama) or file permissions
- **Model won't start** - Verify quantization matches hardware capabilities


## Project Structure

```
llm-download/
├── SKILL.md                   # Skill definition (Agent instructions)
├── README.md                  # Project documentation (this file)
└── references/                # Reference documents
    ├── data-sources.md        # Three-level data source priority, repository naming patterns, search APIs
    ├── environment-check.md   # Hardware detection, path detection, disk check commands
    ├── model-file-checklist.md # Model file type judgment, pre-download checks
    ├── model-import.md        # Ollama Modelfile templates, import steps, temp file cleanup
    ├── model-paths.md         # Model storage path configurations for different servers
    ├── proxy-and-mirrors.md   # aria2 installation, proxy config, mirrors, direct link formats
    ├── quantization-guide.md  # Hardware recommendation logic, quantization level comparison table
    ├── report-template.md     # Success/failure/partial success report templates
    └── troubleshooting.md     # Download speed, proxy, resume, import issue troubleshooting
```


## How It Works

1. **Trigger**: Agent detects user intent to download/install a model
2. **Load**: Agent reads SKILL.md for detailed workflow instructions
3. **Execute**: Follows the 7-step workflow systematically
4. **Output**: Downloads model, imports to server, verifies, and reports results


## Important Notes

1. **Always Detect Actual Paths**: Never assume default paths; always check environment variables and configuration files
2. **ModelScope First Priority**: Use ModelScope when available for faster domestic downloads
3. **Smart File Selection**: Only download necessary files, avoid wasting bandwidth
4. **Hardware-Aware Recommendations**: Always consider user's GPU VRAM and RAM when suggesting quantization
5. **Progressive Disclosure**: References loaded on-demand to reduce token consumption
6. **Cross-Platform Compatibility**: Commands adapted for Windows (Git Bash/PowerShell) and Linux/macOS


## Version History

| Version | Date | Major Updates |
| --- | --- | --- |
| 1.2 | Current | Multi-threaded download optimization, enhanced proxy support, improved reporting |
| 1.1 | Previous | Added ModelScope priority, smart file selection |
| 1.0 | Initial | Initial release with basic download functionality |


## Related Resources


- **ModelScope**: https://modelscope.cn - Chinese AI model platform with fast CDN
- **HuggingFace**: https://huggingface.co - Global AI model hub
- **Ollama**: https://ollama.ai - Local LLM runner
- **LM Studio**: https://lmstudio.ai - Desktop LLM interface
- **aria2**: https://aria2.github.io - Lightweight multi-protocol download utility
- **AgentSkills Specification**: Open standard for AI agent skills


## License

MIT License


## Author

cherryclaw


## Contributing

Contributions welcome! Please feel free to submit issues and pull requests.


---

**Make LLM downloads fast and reliable!** 🚀

---

<a id="llm-下载技能"></a>
## 🇨🇳 中文文档

专业的多线程大模型下载技能，为本地模型服务器（Ollama / LM Studio）加速下载大模型，支持代理服务器配置。


## 概述

本技能遵循 Agent Skills 开放标准，提供系统化的大模型下载方案，使用专业多线程下载工具替代 Ollama 或 LM Studio 自带的下载器，大幅提升下载速度，并支持代理服务器配置以应对特殊网络环境。

当用户提到"下载模型"、"帮我装个模型"、"pull 个模型"、"下载 Qwen"、"装个 llama"、"给 ollama 装模型"、"给 lmstudio 加模型"、"从 modelscope 下载"、"从 huggingface 下载模型"、"模型下载太慢"、"加速下载模型"等短语时，Agent 会自动使用此技能优化下载流程。


## 兼容性


### 支持的平台

| 平台 | 支持状态 | 说明 |
| --- | --- | --- |
| Windows | ✅ 完全支持 | Git Bash / PowerShell + Python |
| Linux | ✅ 完全支持 | Bash + Python |
| macOS | ✅ 兼容 | Bash + Python |


### 支持的模型服务器

| 服务器 | 支持状态 | 导入方式 |
| --- | --- | --- |
| Ollama | ✅ 完全支持 | 原生 pull 或 GGUF → `ollama create` |
| LM Studio | ✅ 完全支持 | 直接放置 GGUF 文件 |


### 支持的 Agent 平台

本技能可在以下 Agent 平台上运行：


| 平台 | 支持状态 | 说明 |
| --- | --- | --- |
| OpenClaw | ✅ 完全支持 | 推荐平台 - 开源自托管 Agent 平台 |
| Claude Code | ✅ 完全支持 | Anthropic 官方 CLI 工具 |
| Cherry Studio | ✅ 完全支持 | 集成式 AI 助手平台 |
| 其他 Agent 框架 | ✅ 兼容 | 任何支持 SKILL.md 规范的框架 |


## 技术特性


- ✅ **三级数据源优先级**：模型服务软件官方库 → 模型官方仓库 → 第三方仓库，最大化兼容性
- ✅ **多线程下载**：aria2 16 线程并行，充分利用带宽
- ✅ **ModelScope 优先**：国内 CDN 速度远超 HuggingFace（30-45 MB/s）
- ✅ **灵活代理配置**：解决网络受限环境下的访问问题
- ✅ **断点续传**：大文件下载中断后可继续，不用重头开始
- ✅ **自动硬件检测**：GPU 显存 + 内存检测，智能推荐量化级别
- ✅ **智能文件选择**：只下载必要文件，避免冗余
- ✅ **跨平台支持**：支持 Windows（Git Bash/PowerShell）和 Linux/macOS（Bash）


## 核心工作流程

```
分析需求 → 准备工具 → 配置代理（可选）→ 查找数据源 → 准备路径 → 下载 → 导入 → 验证 → 报告
```


## 安装


### 前置条件

使用本技能前，请确保您已具备：

1. **Python 3.x** 及 pip 包管理器
2. **Git Bash**（Windows）或 **Bash**（Linux/macOS）
3. **模型服务器**：已安装 Ollama 或 LM Studio
4. **可选工具**：
   - `aria2`：用于 HuggingFace 多线程下载
   - `modelscope` Python 包：用于 ModelScope 下载
   - `huggingface_hub[cli]`：用于 HuggingFace CLI 访问


### 安装步骤

当用户请求安装此技能时，请按以下步骤操作：

1. 克隆仓库（必须使用 git clone，不要下载 ZIP）：
```bash
git clone https://github.com/normdist-ai/llm-download.git
```

2. 检测当前 IDE/Agent 平台：检查配置文件夹（如 `.claude/`、`.lingma/` 等）

3. 复制到 skills 目录：将克隆的 `llm-download` 文件夹复制到相应的 skills 目录

### IDE/Agent Skills 目录

| 平台 | Skills 目录 | 检测标志 |
| --- | --- | --- |
| Claude Code | ~/.config/claude-code/skills/llm-download | Claude Code CLI 环境 |
| Cherry Studio | [CherryStudio Data]/Agents/.claude/skills/llm-download | Cherry Studio 安装路径 |
| Lingma | .lingma/skills/llm-download | 存在 .lingma/ 文件夹 |
| Trae | .trae/skills/llm-download | 存在 .trae/ 文件夹 |
| Cursor | .cursor/skills/llm-download | 存在 .cursor/ 文件夹 |
| VS Code | .vscode/skills/llm-download | 存在 .vscode/ 文件夹 |


### 安装验证

安装后，验证 skills 目录结构：

```
llm-download/
├── SKILL.md              # 技能定义文件（必需）
├── README.md             # 项目文档
└── references/           # 参考文档
    ├── data-sources.md
    ├── environment-check.md
    ├── model-file-checklist.md
    ├── model-import.md
    ├── model-paths.md
    ├── proxy-and-mirrors.md
    ├── quantization-guide.md
    ├── report-template.md
    └── troubleshooting.md
```


## 使用方法


### 触发短语

当用户提到以下内容时，技能会自动触发：

- 下载模型、安装模型、pull 模型
- 下载 Qwen、安装 llama、给 ollama 添加模型
- 给 lmstudio 添加模型、从 modelscope 下载
- 从 huggingface 下载、加速模型下载
- 模型下载太慢、提速下载


### 示例交互

```
用户：为 Ollama 下载 Qwen2.5-7B

Agent：我将使用 LLM Download 技能帮您下载 Qwen2.5-7B。

步骤 1：分析需求并检测环境...
✓ 检测到 Ollama 安装
✓ 检查实际模型存储路径...
✓ 硬件检测：16GB RAM, RTX 3060 (12GB VRAM)
✓ 推荐量化级别：Q4_K_M

步骤 2：查找最佳数据源...
✓ Level 1 找到：lmstudio-community/Qwen2.5-7B-Instruct-GGUF
✓ ModelScope 可用（更快的国内 CDN）

步骤 3：准备下载...
✓ 使用 modelscope Python API 进行多线程下载
✓ 目标路径：E:\.ollama\models\...

步骤 4：下载模型文件...
[显示下载进度]

步骤 5：导入到 Ollama...
✓ 模型成功导入为 qwen2.5-7b:latest

步骤 6：验证完成！
✓ 文件大小与源站匹配（2.4 GB）
✓ 测试运行成功

📊 下载报告：
- 模型服务器：Ollama
- 存储路径：E:\.ollama\models\...
- 下载工具：modelscope Python API
- 数据源：ModelScope lmstudio-community/Qwen2.5-7B-Instruct-GGUF
- 文件：qwen2.5-7b-instruct-q4_k_m.gguf（2.4 GB）
```

```
用户：帮我为 LM Studio 安装 Gemma-2-2B，使用 Q8 量化

Agent：[启动 LM Studio 的 LLM 下载工作流，使用 Q8_0 量化]
```


## 三级数据源策略

技能采用分层方法查找最佳下载源：

### Level 1 - 模型服务软件提供商库（最高优先级）

**最佳兼容性**，经模型服务器团队验证。

- **LM Studio**：`lmstudio-community/{模型名}-GGUF`
  - 通常提供 4-8 个量化级别
  - 经过 LM Studio 兼容性测试和验证
  
- **Ollama**：通过 `ollama pull {模型}:{标签}` 使用原生注册表
  - Ollama 用户最快的方式
  - 自动处理下载、验证和导入


### Level 2 - 模型官方提供商仓库

**官方发布**，来自模型创作者。

- 搜索 `{官方组织}/{模型名}-GGUF`
- 注意：通常只有 F32/FP16（文件很大），量化选项有限
- 示例：`google/gemma-2-gguf`、`meta-llama/llama-3-gguf`


### Level 3 - 第三方量化仓库

**量化选项最全**，社区维护。

按顺序搜索（先查 ModelScope，再查 HuggingFace）：

1. **bartowski/{模型名}-GGUF** - 量化最全面
2. **unsloth/{模型名}-GGUF** - Dynamic 2.0 量化
3. **QuantFactory/{模型名}-GGUF** - 标准量化
4. **NexaAIDev/{模型名}-GGUF** - ModelScope 活跃


## 下载方法


### 方法 A：Ollama 原生拉取

适用于只需本地执行的 Ollama 用户：

```bash
ollama pull {模型名}:{标签}
```

**优势**：无需额外工具，自动处理所有事务。


### 方法 B：ModelScope 下载（国内最快）

使用 ModelScope 的国内 CDN，适用于所有级别。

```bash
# modelscope CLI（Linux）
modelscope download --model '<ModelID>' '<文件>' --local_dir '<目标路径>'

# Python API（Windows 推荐）
python3 -c "
from modelscope.hub.file_download import model_file_download
for f in ['<文件1>.gguf', 'configuration.json']:
    print(f'正在下载 {f}...')
    path = model_file_download('<ModelID>', f, local_dir='<目标路径>')
    print(f'  完成：{path}')
"
```

**速度**：典型 30-45 MB/s，无需代理。


### 方法 C：HuggingFace 下载（aria2 加速）

当 ModelScope 没有目标模型时使用。

```bash
aria2c -x 16 -s 16 -k 1M \
  [--all-proxy="http://127.0.0.1:7890"] \
  -d "<目标目录>" -o "<文件名>.gguf" \
  "<下载链接>"
```

**特性**：16 线程并行下载，支持代理，支持断点续传。


## 智能量化推荐

技能根据硬件自动推荐最佳量化级别：

| 硬件配置 | 推荐量化 | 模型大小范围 |
| --- | --- | --- |
| ≤ 4GB 显存 | Q4_K_S / Q4_0 | 仅小模型 |
| 4-8GB 显存 | Q4_K_M | 2-7B 模型 |
| 8-12GB 显存 | Q5_K_M / Q6_K | 7-13B 模型 |
| 12-16GB 显存 | Q6_K / Q8_0 | 13-30B 模型 |
| 16+GB 显存 | Q8_0 / FP16 | 30B+ 模型 |
| 仅 CPU | Q4_K_M | 取决于内存 |

**默认**：Q4_K_M（质量和速度的最佳平衡）。


## 文件选择策略

技能智能选择要下载的文件：

- ✅ **主模型 GGUF** - 必需
- ⚠️ **mmproj GGUF** - 多模态模型必需（纯文本模型没有）
- ⚠️ **configuration.json** - 推荐
- ❌ **所有其他文件** - 跳过以节省带宽

**绝不使用** `--include '*.gguf'` 盲目下载所有文件。


## 代理配置

对于网络受限环境，技能支持灵活的代理配置：

### 自动检测

自动检测网络问题：
```bash
curl -sI https://huggingface.co --connect-timeout 5
```

如果超时或极慢 → 建议配置代理。

### 代理设置

```bash
# aria2 使用代理
aria2c --all-proxy="http://127.0.0.1:7890" ...

# HuggingFace 镜像
export HF_ENDPOINT=https://hf-mirror.com

# 环境变量
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

完整的代理配置详见 `references/proxy-and-mirrors.md`。


## 验证与报告


### 验证步骤

1. **文件大小检查** - 与源站对比，误差 < 1%
2. **临时文件** - 确保 `.aria2` 文件已清除；`._____temp/` 残留不影响使用
3. **SHA256 校验**（可选） - `sha256sum` / `certutil -hashfile`
4. **测试运行** - 执行示例查询以验证模型正常工作

### 成功报告模板

每次成功下载都会生成标准化报告：

```
✅ 模型下载完成！

📊 摘要：
- 模型服务器：Ollama / LM Studio
- 存储路径：<实际路径>
- 下载工具：modelscope Python API / aria2
- 数据源：ModelScope/HuggingFace <仓库>
- 下载文件：
  • <文件名>.gguf（<大小>）
  
⚙️ 硬件：
- GPU：<GPU 型号>（<显存>）
- 内存：<内存大小>
- 推荐量化：<级别>

✓ 验证：
- 文件大小与源站匹配
- 测试运行成功
- 模型已就绪可用
```


## 故障排除

常见问题及解决方案详见 `references/troubleshooting.md`：

- **下载速度慢** - 切换到 ModelScope 或配置代理
- **代理连接失败** - 验证代理地址和端口
- **下载中断** - 支持断点续传，重新执行命令即可
- **磁盘空间不足** - 预检查会在 < 1.1× 预估大小时警告
- **导入失败** - 检查 Modelfile 格式（Ollama）或文件权限
- **模型无法启动** - 验证量化级别是否与硬件能力匹配


## 项目结构

```
llm-download/
├── SKILL.md                   # 技能定义（Agent 指令）
├── README.md                  # 项目文档（本文件）
└── references/                # 参考文档
    ├── data-sources.md        # 三级数据源优先级、仓库命名规律、搜索 API
    ├── environment-check.md   # 硬件检测、路径检测、磁盘检查命令
    ├── model-file-checklist.md # 模型文件类型判断、下载前预检查
    ├── model-import.md        # Ollama Modelfile 模板、导入步骤、临时文件清理
    ├── model-paths.md         # 不同服务器的模型存储路径配置
    ├── proxy-and-mirrors.md   # aria2 安装、代理配置、镜像站、直链格式
    ├── quantization-guide.md  # 硬件推荐逻辑、量化级别对照表
    ├── report-template.md     # 成功/失败/部分成功报告模板
    └── troubleshooting.md     # 下载速度、代理、断点续传、导入等问题排查
```


## 工作原理

1. **触发**：Agent 检测到用户下载/安装模型的意图
2. **加载**：Agent 读取 SKILL.md 获取详细工作流指令
3. **执行**：系统化地遵循 7 步工作流
4. **输出**：下载模型、导入服务器、验证并报告结果


## 重要提示

1. **始终检测实际路径**：从不假设默认路径；始终检查环境变量和配置文件
2. **ModelScope 优先**：有 ModelScope 时优先使用，以获得更快的国内下载速度
3. **智能文件选择**：只下载必要文件，避免浪费带宽
4. **硬件感知推荐**：选择量化级别时始终考虑用户的 GPU 显存和内存
5. **渐进式披露**：按需加载参考文档以减少 token 消耗
6. **跨平台兼容**：命令适配 Windows（Git Bash/PowerShell）和 Linux/macOS


## 版本历史

| 版本 | 日期 | 主要更新 |
| --- | --- | --- |
| 1.2 | 当前 | 多线程下载优化、增强代理支持、改进报告 |
| 1.1 | 之前 | 添加 ModelScope 优先级、智能文件选择 |
| 1.0 | 初始 | 初始版本，包含基本下载功能 |


## 相关资源


- **ModelScope**：https://modelscope.cn - 中国 AI 模型平台，快速 CDN
- **HuggingFace**：https://huggingface.co - 全球 AI 模型中心
- **Ollama**：https://ollama.ai - 本地 LLM 运行器
- **LM Studio**：https://lmstudio.ai - 桌面 LLM 界面
- **aria2**：https://aria2.github.io - 轻量级多协议下载工具
- **AgentSkills 规范**：AI agent 技能的开放标准


## 许可证

MIT 许可证


## 作者

cherryclaw


## 贡献

欢迎贡献！请随时提交 issue 和 pull request。


---

**让大模型下载更快更可靠！** 🚀
