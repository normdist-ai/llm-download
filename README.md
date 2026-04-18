# LLM Download Skill

[中文文档](#llm-下载技能)

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
