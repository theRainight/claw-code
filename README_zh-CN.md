# Windows 平台运行与模型更换指南

本文档介绍了如何在 Windows 平台上运行 Claw Code 以及如何更换不同的模型供应商。

## 一、 在 Windows 平台上运行

在 Windows 系统上，官方推荐使用 **PowerShell** 进行操作。

### 方式 1：从源码编译（推荐开发者使用）

1. **安装 Rust**：前往 [Rust 官网 (https://rustup.rs/)](https://rustup.rs/) 下载并运行安装程序。安装完成后，请务必**关闭并重新打开终端**以加载环境变量。
2. **克隆并编译代码**：在 PowerShell 中依次运行以下命令：
   ```powershell
   git clone https://github.com/ultraworkers/claw-code
   cd claw-code\rust
   cargo build --workspace
   ```
3. **验证安装**：编译完成后，执行 `doctor` 健康检查命令确认是否成功：
   ```powershell
   .\target\debug\claw.exe doctor
   ```

### 方式 2：使用已编译的发布版本

如果你不想自己配置编译环境，可以直接在 GitHub 的 Release 页面下载 `claw-windows-x64.exe` 文件。下载后将其存放在固定文件夹并重命名为 `claw.exe`，然后即可在终端中直接调用。

---

## 二、 如何更换模型

Claw Code 支持动态切换多个模型供应商（如 Anthropic、OpenAI、Ollama 本地模型、阿里云 DashScope 等）。更换模型的核心步骤是：**设置正确的环境变量**并在命令中使用 `--model` 参数。

> **小提示**：为了防止系统路由错误，在切换供应商时，建议先清除其他供应商的环境变量。每次配置完毕后，都可以运行 `.\target\debug\claw.exe doctor` 来检查配置是否生效。

### 1. 使用默认的 Anthropic (Claude 模型)
你需要提供 `$env:ANTHROPIC_API_KEY`。可以通过模型别名（如 `sonnet`, `opus`, `haiku`）直接调用。
```powershell
# 清除可能干扰的 OpenAI 变量
Remove-Item Env:\OPENAI_BASE_URL -ErrorAction SilentlyContinue
Remove-Item Env:\OPENAI_API_KEY -ErrorAction SilentlyContinue

# 设置你的 Claude 密钥并运行
$env:ANTHROPIC_API_KEY = "sk-ant-你的密钥"
.\target\debug\claw.exe --model "sonnet" prompt "你好，请回复就绪"
```

### 2. 使用 OpenAI 兼容接口或 OpenRouter
需要将 `OPENAI_BASE_URL` 指向对应的服务，设置 API 密钥，且模型名称**必须带有 `openai/` 或 `gpt-` 前缀**（以便系统识别正确的路由）。
```powershell
# 移除 Anthropic 密钥
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue

# 设置 OpenRouter 环境变量
$env:OPENAI_BASE_URL = "https://openrouter.ai/api/v1"
$env:OPENAI_API_KEY = "sk-or-v1-你的密钥"
.\target\debug\claw.exe --model "openai/gpt-4.1-mini" prompt "你好，请回复就绪"
```

### 3. 使用本地模型 (例如 Ollama)
如果你在本地运行 Ollama，不需要真实的 API 密钥，只需将 Base URL 指向本地的地址即可。
```powershell
# 移除其他云端密钥
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
Remove-Item Env:\OPENAI_API_KEY -ErrorAction SilentlyContinue

# 指向本地 Ollama 服务
$env:OPENAI_BASE_URL = "http://127.0.0.1:11434/v1"
.\target\debug\claw.exe --model "llama3.2" prompt "你好，请回复就绪"
```

### 4. 使用阿里云 DashScope (通义千问 Qwen / Kimi)
对于 Qwen 或 Kimi 模型，程序会自动识别名称前缀（如 `qwen-` 或 `kimi-`）并路由到 DashScope 接口，你只需要设置对应的专属密钥。
```powershell
# 移除 Anthropic 密钥
Remove-Item Env:\ANTHROPIC_API_KEY -ErrorAction SilentlyContinue

# 设置阿里云密钥
$env:DASHSCOPE_API_KEY = "sk-你的阿里云密钥"
.\target\debug\claw.exe --model "qwen-plus" prompt "你好，请回复就绪"
```
