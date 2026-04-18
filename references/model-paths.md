# 模型存放路径规范

本文档详细说明 Ollama 和 LM Studio 在不同操作系统上的模型存放路径、目录结构，以及**自动检测实际路径**的方法。

> ⚠️ **不要假设默认路径**。用户可能已自定义路径，必须通过检测获取实际路径。

---

## 自动检测实际路径

### Ollama 路径检测

```bash
# Linux：检查环境变量，再检查 systemd 服务配置，最后用默认值
echo "${OLLAMA_MODELS:-$HOME/.ollama/models}"

# 如果以 systemd 服务运行，还需检查：
systemctl show ollama --property=Environment 2>/dev/null | grep OLLAMA_MODELS

# Windows（PowerShell）：依次检查会话/用户/系统级环境变量
$env:OLLAMA_MODELS
[Environment]::GetEnvironmentVariable("OLLAMA_MODELS", "User")
[Environment]::GetEnvironmentVariable("OLLAMA_MODELS", "Machine")
# 都为空则默认: "$env:USERPROFILE\.ollama\models"
```

### LM Studio 路径检测

检测分两步：先找 home 目录，再从配置文件读取模型存储路径。

```bash
# === Linux / macOS ===
# 步骤 1: 读取 home 指针文件
LM_HOME=$(cat ~/.lmstudio-home-pointer 2>/dev/null || echo "$HOME/.lmstudio")

# 步骤 2: 从 settings.json 提取 downloadsFolder
# 用 grep 提取，无需 Python 依赖
LM_PATH=$(grep -o '"downloadsFolder"[[:space:]]*:[[:space:]]*"[^"]*"' "$LM_HOME/settings.json" 2>/dev/null | grep -o '"[^"]*"$' | tr -d '"' | sed 's|\\\\|/|g')
echo "${LM_PATH:-$LM_HOME/models}"

# === Windows (Git Bash) ===
# 步骤 1: 读取 home 指针文件
LM_HOME=$(cat "$USERPROFILE/.lmstudio-home-pointer" 2>/dev/null || echo "$USERPROFILE/.lmstudio")

# 步骤 2: 从 settings.json 提取 downloadsFolder
LM_PATH=$(grep -o '"downloadsFolder"[[:space:]]*:[[:space:]]*"[^"]*"' "$LM_HOME/settings.json" 2>/dev/null | grep -o '"[^"]*"$' | tr -d '"' | sed 's|\\\\|/|g')
echo "${LM_PATH:-$LM_HOME/models}"
```

> 如果 grep 解析 JSON 不可靠（如字段值包含引号），可改用 Python：
> ```bash
> python3 -c "import json; d=json.load(open('$LM_HOME/settings.json')); print(d.get('downloadsFolder','') or '$LM_HOME/models')"
> ```

**检测优先级：**
1. `.lmstudio-home-pointer` 文件 → 自定义 home 目录
2. `settings.json` 中的 `downloadsFolder` → 自定义模型路径
3. 都没有 → 默认 `<home>/models/`

---

## Ollama

### 默认路径

| 操作系统 | 默认路径 |
|---------|---------|
| Windows | `C:\Users\<用户名>\.ollama\models\` |
| Linux（用户安装） | `~/.ollama/models/` |
| Linux（系统服务） | `/usr/share/ollama/.ollama/models` |

### 目录结构（OCI 格式）

Ollama 不直接存放 GGUF 文件，而是使用 Docker 风格的内容寻址存储：

```
.ollama/models/
├── blobs/                  # 实际模型权重（GGUF 封装在 blob 中）
│   ├── sha256-abc123...
│   ├── sha256-def456...
│   └── sha256-ghi789...
└── manifests/              # 模型元数据（JSON）
    └── registry.ollama.ai/
        └── library/
            ├── qwen3/
            │   └── latest          # JSON manifest
            ├── llama3.2/
            └── mistral/
```

- **blobs/**：按 SHA-256 哈希命名，多个模型共享相同基础权重时会自动去重
- **manifests/**：模型"配方卡"，描述架构、使用的 blob、量化类型等

> ⚠️ **不要手动往 Ollama 模型目录放文件。** 使用 `ollama pull` 或 `ollama create` 来管理模型。

### 自定义路径

通过 `OLLAMA_MODELS` 环境变量设置：

**Windows（PowerShell，管理员）：**
```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "D:\OllamaModels", "Machine")
```

**Windows（图形界面）：** 系统属性 → 高级 → 环境变量 → 新建 `OLLAMA_MODELS`

**Linux（systemd 服务）：**
```bash
sudo systemctl edit ollama.service
# 添加：
[Service]
Environment="OLLAMA_MODELS=/mnt/data/ollama-models"
# 然后：
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

**Linux（用户会话）：**
```bash
export OLLAMA_MODELS=/custom/path/to/models
ollama serve
```

**备选方案：符号链接**
```bash
# Linux
ln -s /mnt/data/ollama-models ~/.ollama/models

# Windows（管理员 CMD）
mklink /D "%USERPROFILE%\.ollama\models" "D:\OllamaModels"
```

---

## LM Studio

### 默认路径

| 操作系统 | 默认路径 |
|---------|---------|
| Windows | `C:\Users\<用户名>\.lmstudio\models\` |
| Linux | `~/.lmstudio/models/` |

> LM Studio v0.3.16+ 还会在 `~/.lmstudio/hub/models/` 存放较小的元数据/配置文件。

### 目录结构（直接 GGUF）

LM Studio 镜像 Hugging Face 的 publisher/model 层级：

```
.llmstudio/models/
└── <publisher>/
    └── <model-name>/
        └── <model-file>.gguf
```

**实际示例：**
```
.llmstudio/models/
├── Qwen/
│   └── Qwen3-8B-GGUF/
│       └── Qwen3-8B-Q4_K_M.gguf
├── bartowski/
│   └── Meta-Llama-3.1-8B-Instruct-GGUF/
│       ├── Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf
│       └── Meta-Llama-3.1-8B-Instruct-Q8_0.gguf
└── TheBloke/
    └── Mistral-7B-Instruct-v0.2-GGUF/
        └── mistral-7b-instruct-v0.2.Q4_K_M.gguf
```

- 每个量化版本是独立的 GGUF 文件（不像 Ollama 会去重）
- `<publisher>` 和 `<model-name>` 应与源站（ModelScope/HuggingFace）一致

### 自定义路径

**方法一：应用内设置（推荐）**

在 LM Studio 中：My Models 页面 → 顶部路径 → 点击 "Change"

**方法二：home 指针文件**

创建/编辑 `~/.lmstudio-home-pointer` 文件，内容为新路径：
```
D:\AI_Models\LMStudio
```

步骤：退出 LM Studio → 移动 `.lmstudio` 文件夹 → 编辑指针文件 → 重启

**方法三：符号链接**

```bash
# Linux
ln -s /mnt/data/lmstudio-models ~/.lmstudio/models

# Windows（管理员 CMD）
mklink /D "%USERPROFILE%\.lmstudio\models" "D:\AI_Models\LMStudio\models"
```

**方法四：导入已有 GGUF**

```bash
# 复制导入
lms import <path/to/model.gguf>

# 符号链接导入（不占额外空间）
lms import --symbolic-link <path/to/model.gguf>
```

---

## 快速对照

| 对比项 | Ollama | LM Studio |
|-------|--------|-----------|
| 存储格式 | OCI（blobs + manifests） | 直接 GGUF 文件 |
| Windows 默认路径 | `C:\Users\<用户>\.ollama\models\` | `C:\Users\<用户>\.lmstudio\models\` |
| Linux 默认路径 | `~/.ollama/models/` | `~/.lmstudio/models/` |
| 自定义路径方式 | `OLLAMA_MODELS` 环境变量 | 应用内设置 / 指针文件 / 符号链接 |
| 是否去重 | 是（共享 blob） | 否（独立文件） |
| 能否手动放文件 | ❌ 不建议 | ✅ 可以 |
| API 端口 | 11434 | 1234 |
