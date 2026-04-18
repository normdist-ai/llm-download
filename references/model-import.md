# Ollama 模型导入

从 GGUF 文件导入到 Ollama 的详细步骤。

---

## 创建 Modelfile

```bash
cat > ./tmp_model/Modelfile << 'EOF'
FROM ./<下载的模型文件>.gguf

PARAMETER temperature 0.7
PARAMETER top_p 0.9

TEMPLATE """{{- if .System }}{{ .System }}{{ end }}
{{ .Prompt }}"""
EOF
```

### Modelfile 常用参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `temperature` | 0.7 | 生成随机性，越高越有创意 |
| `top_p` | 0.9 | 核采样范围 |
| `num_ctx` | 2048 | 上下文窗口大小 |
| `stop` | — | 停止生成的标记 |

### 常见模型的 TEMPLATE

**Gemma 系列**：
```
TEMPLATE """<start_of_turn>user
{{ .Prompt }}<end_of_turn>
<start_of_turn>model
"""
```

**Qwen 系列**：
```
TEMPLATE """{{- if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant
"""
```

**Llama 系列**：
```
TEMPLATE """{{- if .System }}<|start_header_id|>system<|end_header_id|>
{{ .System }}<|eot_id|>{{ end }}<|start_header_id|>user<|end_header_id|>
{{ .Prompt }}<|eot_id|><|start_header_id|>assistant<|end_header_id|>
"""
```

> 💡 如果不确定模板格式，可以从 Ollama 官方库拉取同系列模型后查看其 Modelfile：
> `ollama show {模型名} --modelfile`

## 导入

```bash
ollama create <自定义模型名> -f ./tmp_model/Modelfile
```

模型名示例：`my-gemma2-2b`、`qwen3-8b-custom`

## 清理临时文件

```bash
# Linux / Git Bash
rm -rf ./tmp_model

# Windows CMD
rmdir /s /q ".\tmp_model"
```

## 验证导入

```bash
ollama list                       # 确认模型已注册
ollama run <模型名> "你好"        # 测试对话
```

## 常见导入失败原因

| 错误 | 原因 | 解决 |
|------|------|------|
| `invalid model archive` | GGUF 文件不完整或损坏 | 重新下载，检查文件大小 |
| `unsupported architecture` | GGUF 架构不被 Ollama 支持 | 更新 Ollama 到最新版本 |
| `file not found` | Modelfile 中路径错误 | 使用绝对路径 |
| 权限错误 | 文件或目录权限不足 | `chmod` 修复权限 |
