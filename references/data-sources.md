# 数据源优先级与搜索方法

## 核心原则

按**数据源权威性**分三级优先级查找模型，同一级内 **ModelScope 优先**（国内 CDN 快），找不到再查 HuggingFace。

```
Level 1: 模型服务软件提供商的官方模型库 → 兼容性最佳
Level 2: 模型官方提供商的 GGUF 仓库 → 官方出品
Level 3: 第三方量化仓库 → 量化选项最全
兜底：   征求用户意见
```

---

## Level 1 — 模型服务软件提供商

### LM Studio：`lmstudio-community`

LM Studio 在 HuggingFace 上维护 `lmstudio-community` 组织，提供经过验证的 GGUF 量化文件。ModelScope 有同步镜像。

**仓库命名规律**：`lmstudio-community/{原始模型名}-GGUF`

| 原始模型 | Level 1 仓库 |
|---------|-------------|
| `google/gemma-2-2b-it` | `lmstudio-community/gemma-2-2b-it-GGUF` |
| `meta-llama/Llama-3.3-70B-Instruct` | `lmstudio-community/Llama-3.3-70B-Instruct-GGUF` |
| `Qwen/Qwen3-4B` | `lmstudio-community/Qwen3-4B-GGUF` |
| `microsoft/phi-4` | `lmstudio-community/phi-4-GGUF` |

**量化版本**（通常 4-8 个）：IQ3_M, IQ4_XS, Q3_K_L, Q4_K_M, Q5_K_M, Q6_K, Q8_0, f32

**检测仓库是否存在**：

```bash
# ModelScope（国内快）
curl -sI "https://modelscope.cn/models/lmstudio-community/{模型名}-GGUF" | head -1

# HuggingFace
curl -sI "https://huggingface.co/api/models/lmstudio-community/{模型名}-GGUF" | head -1
```

**搜索特定模型**（HuggingFace API）：

```bash
curl -s "https://huggingface.co/api/models?author=lmstudio-community&search={关键词}" | python3 -c "
import sys, json
data = json.load(sys.stdin)
for m in data:
    print(m['id'])
"
```

### Ollama：原生注册表

Ollama 有自己的模型注册表 `registry.ollama.ai`，支持直接 `ollama pull`。

**模型命名**：`{模型名}:{tag}`，如 `gemma2:2b`、`qwen3:4b`、`llama3.3:70b`

**可用 tag**：`latest`(默认 Q4_0)、`q4_0`、`q4_1`、`q5_0`、`q5_1`、`q8_0`、`f16`

> ⚠️ Ollama 原生下载是 blob 分片格式，不是标准 GGUF 文件。如果需要 GGUF 文件（如跨平台使用），应从 Level 2/3 下载。

**Ollama 的特殊处理**：
- 如果用户使用 Ollama 且只需要本地运行 → 直接 `ollama pull {模型名}:{tag}`，这是最简单的方式
- 如果用户需要 GGUF 文件（跨服务器使用、备份等） → 按 Level 2/3 查找 GGUF

---

## Level 2 — 模型官方提供商

模型原始作者发布的 GGUF 仓库。**注意**：官方仓库通常只提供 F32/FP16 未量化版本，文件很大。

**常见官方 GGUF 仓库**：

| 模型家族 | 官方 GGUF 仓库 |
|---------|--------------|
| Google Gemma | `google/gemma-2-2b-it-GGUF`、`google/gemma-2-2b-GGUF` |
| Meta Llama | `meta-llama/Llama-3.3-70B-Instruct-GGUF` |
| Qwen | `Qwen/Qwen3-4B-GGUF`（Qwen 官方同时提供量化版） |
| Microsoft Phi | （通常没有官方 GGUF，需从 Level 1/3 获取） |

**判断是否有量化版本**：
- Google Gemma：通常只有 F32，体积巨大（2B 约 10GB）
- Meta Llama：通常提供多个量化版本
- Qwen：官方提供完整量化（Qwen 官方 GGUF 仓库就是最终来源）

**检测仓库是否存在**：

```bash
# ModelScope
curl -sI "https://modelscope.cn/models/{官方组织名}/{模型名}-GGUF" | head -1

# HuggingFace
curl -sI "https://huggingface.co/api/models/{官方组织名}/{模型名}-GGUF" | head -1
```

---

## Level 3 — 第三方量化仓库

当 Level 1/2 找不到或没有合适的量化版本时，从知名第三方量化仓库获取。这些仓库由社区维护，量化选项最全。

### bartowski — 量化最全

- **仓库格式**：`bartowski/{模型名}-GGUF`
- **特点**：量化版本最多（IQ 系列 + Q 系列全覆盖），社区活跃
- **ModelScope 镜像**：有（如 `bartowski/Qwen2.5-1.5B-Instruct-GGUF`）

### unsloth — Dynamic 2.0 量化

- **仓库格式**：`unsloth/{模型名}-GGUF`
- **特点**：使用 Dynamic 2.0 量化方法，提供独有格式如 `UD-Q4_K_XL`、`UD-Q6_K_XL`
- **ModelScope 镜像**：有，覆盖范围广

### QuantFactory — 标准量化

- **仓库格式**：`QuantFactory/{模型名}-GGUF`
- **特点**：提供标准 Q 系列量化（Q2_K → Q8_0），全面但下载量较小

### NexaAIDev — 标准量化

- **仓库格式**：`NexaAIDev/{模型名}-GGUF`
- **特点**：ModelScope 上较为活跃，提供完整的标准量化版本
- **ModelScope 镜像**：有

**检测仓库是否存在**（以 bartowski 为例）：

```bash
# ModelScope
curl -sI "https://modelscope.cn/models/bartowski/{模型名}-GGUF" | head -1

# HuggingFace
curl -sI "https://huggingface.co/api/models/bartowski/{模型名}-GGUF" | head -1
```

---

## 搜索流程

### 流程图

```
用户请求模型 X
    │
    ├─ LM Studio 用户？
    │   ├─ 构造 lmstudio-community/{X}-GGUF
    │   ├─ 检查 ModelScope → 有 → 列出量化版本 → 选择下载
    │   ├─ 检查 HuggingFace → 有 → 选择下载
    │   └─ 没有 → 进入 Level 2
    │
    ├─ Ollama 用户？
    │   ├─ 只需本地运行？→ ollama pull {X} → 完成
    │   └─ 需要 GGUF 文件？→ 进入 Level 2
    │
    ├─ Level 2：构造 {官方组织}/{X}-GGUF
    │   ├─ 检查 ModelScope → 有且有所需量化 → 选择下载
    │   ├─ 检查 HuggingFace → 有且有所需量化 → 选择下载
    │   └─ 没有或无合适量化 → 进入 Level 3
    │
    ├─ Level 3：搜索第三方仓库
    │   ├─ bartowski/{X}-GGUF → ModelScope → HuggingFace
    │   ├─ unsloth/{X}-GGUF → ModelScope → HuggingFace
    │   ├─ QuantFactory/{X}-GGUF → ModelScope → HuggingFace
    │   ├─ NexaAIDev/{X}-GGUF → ModelScope
    │   └─ 都没有 → 征求用户意见
    │
    └─ 兜底：告知用户搜索结果，让用户指定仓库
```

### 仓库存在性检测方法

用 HTTP HEAD/GET 请求判断仓库是否存在，避免逐个尝试的延迟：

```bash
# 批量检测 ModelScope（并行）
for repo in "lmstudio-community/gemma-2-2b-it-GGUF" "google/gemma-2-2b-it-GGUF" "bartowski/gemma-2-2b-it-GGUF"; do
    (code=$(curl -sI "https://modelscope.cn/models/$repo" -o /dev/null -w '%{http_code}')
     echo "$code $repo") &
done
wait

# 批量检测 HuggingFace（并行）
for repo in "lmstudio-community/gemma-2-2b-it-GGUF" "google/gemma-2-2b-it-GGUF" "bartowski/gemma-2-2b-it-GGUF"; do
    (code=$(curl -sI "https://huggingface.co/api/models/$repo" -o /dev/null -w '%{http_code}')
     echo "$code $repo") &
done
wait
```

**HTTP 状态码判断**：
- `200`：仓库存在
- `404`：仓库不存在
- 其他：网络问题，需要重试

### 列出量化版本（ModelScope API）

```python
import requests, json

def list_quantized_files(repo_id):
    """列出 ModelScope 上 GGUF 仓库的量化版本"""
    resp = requests.get(f'https://modelscope.cn/api/v1/models/{repo_id}', timeout=10)
    if resp.status_code != 200:
        return []

    data = resp.json()
    gguf_info = data.get('Data', {}).get('ModelInfos', {}).get('gguf', {})
    file_list = gguf_info.get('gguf_file_list', [])

    results = []
    for item in file_list:
        fi = item.get('file_info', [{}])[0]
        name = fi.get('name', '')
        size = fi.get('size', 0)
        quant = item.get('quantized', '')
        if size > 0:
            results.append({'name': name, 'size_mb': size / (1024*1024), 'quant': quant})

    return results

# 用法
for f in list_quantized_files('lmstudio-community/gemma-2-2b-it-GGUF'):
    print(f"{f['quant']:10s} | {f['name']:40s} | {f['size_mb']:.0f} MB")
```

### 列出量化版本（HuggingFace API）

```bash
curl -s "https://huggingface.co/api/models/{repo_id}" | python3 -c "
import sys, json
data = json.load(sys.stdin)
siblings = data.get('siblings', [])
for f in siblings:
    rfilename = f.get('rfilename', '')
    if rfilename.endswith('.gguf'):
        print(rfilename)
"
```

---

## 完整数据源清单

| 优先级 | 来源 | 平台 | 仓库格式 | 特点 |
|--------|------|------|---------|------|
| L1 | LM Studio 官方 | ModelScope / HF | `lmstudio-community/{model}-GGUF` | 兼容性最佳，4-8 个量化 |
| L1 | Ollama 官方 | registry.ollama.ai | `{model}:{tag}` | 原生支持，blob 格式 |
| L2 | 模型官方 | ModelScope / HF | `{官方组织}/{model}-GGUF` | 官方出品，通常仅 F32/FP16 |
| L3 | bartowski | ModelScope / HF | `bartowski/{model}-GGUF` | 量化最全 |
| L3 | unsloth | ModelScope / HF | `unsloth/{model}-GGUF` | Dynamic 2.0 量化 |
| L3 | QuantFactory | ModelScope / HF | `QuantFactory/{model}-GGUF` | 标准量化 |
| L3 | NexaAIDev | ModelScope | `NexaAIDev/{model}-GGUF` | ModelScope 活跃 |

## 常见模型的官方组织映射

用于构造 Level 2 仓库路径：

| 模型名关键词 | 官方组织（HuggingFace） | 官方组织（ModelScope） |
|-------------|----------------------|---------------------|
| gemma | `google` | `google` / `LLM-Research` |
| llama | `meta-llama` | `LLM-Research` |
| qwen | `Qwen` | `Qwen` |
| phi | `microsoft` | — |
| mistral | `mistralai` | `mistral` |
| deepseek | `deepseek-ai` | `deepseek-ai` |
| glm | `THUDM` | `ZhipuAI` |
| yi | `01-ai` | `01ai` |
| phi | `microsoft` | — |
| stablelm | `stabilityai` | — |
| command-r | `CohereForAI` | — |
