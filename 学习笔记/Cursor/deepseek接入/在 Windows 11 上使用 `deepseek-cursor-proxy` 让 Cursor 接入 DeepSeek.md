# 在 Windows 11 上使用 `deepseek-cursor-proxy` 让 Cursor 接入 DeepSeek

> 本指南将帮助您在 Windows 11 系统上，通过 `deepseek-cursor-proxy` 代理工具，将 Cursor 编辑器连接到 DeepSeek 的 V4 模型（如 `deepseek-v4-pro` 或 `deepseek-v4-flash`）。  
> 核心工具：`uv`（极速 Python 包管理器）+ `ngrok`（内网穿透）。  
> **不需要 Node.js 和 pm2**，所有服务均在前台运行，适合开发测试或简单使用场景。

---

## 📋 前置准备

在开始之前，请确保您已具备以下条件：

- **DeepSeek API Key**  
  访问 [DeepSeek 平台](https://platform.deepseek.com/) 注册并创建 API Key（需要充值或已有额度）。

- **Windows 11 系统**  
  已安装 Cursor 编辑器（可从 [cursor.com](https://cursor.com/) 下载安装）。

- **稳定的网络**  
  能正常访问外网，以下载工具和连接 ngrok 服务。

---

## 🔧 第一步：安装必备工具

### 1. 安装 `uv`

`uv` 是一个 Rust 编写的极速 Python 包管理器，以单个可执行文件运行，不依赖本地 Python 环境。

**安装步骤：**

1. 访问 [uv 的 GitHub Releases 页面](https://github.com/astral-sh/uv/releases)。
2. 下载适用于 Windows 的最新版压缩包，例如 `uv-x86_64-pc-windows-msvc.zip`。
3. 解压压缩包，将 `uv.exe` 放置到一个方便的位置，例如 `C:\tools\uv\`。
4. 将该路径添加到系统环境变量 `PATH`：
   - 右键点击“此电脑” → “属性” → “高级系统设置” → “环境变量”。
   - 在“系统变量”中找到 `Path`，双击编辑 → “新建” → 输入 `C:\tools\uv` → 确定。
5. 打开 **PowerShell** 或 **命令提示符**，输入以下命令验证安装成功：

```powershell
   uv --version
```
如果输出版本号（如 `uv 0.x.x`），则表示安装成功。

### 2. 安装 `ngrok`

`ngrok` 用于将本地代理服务暴露到公网，使 Cursor 能够访问（Cursor 默认无法直接访问 `localhost`）。

**安装步骤：**

1. 访问 [ngrok 官网](https://ngrok.com/)，注册一个免费账户。
    
2. 登录后，在 [Dashboard](https://dashboard.ngrok.com/) 上找到 **“Your Authtoken”**。
    
3. 下载 Windows 版本的 `ngrok.exe`（通常是一个单独的 `.exe` 文件）。
    
4. 将 `ngrok.exe` 放置到一个方便的位置，例如 `C:\tools\ngrok\`。
    
5. 同样将该路径添加到系统环境变量 `PATH`（步骤同上述 `uv`）。
    
6. 打开 **PowerShell** 或 **命令提示符**，执行以下命令配置 Authtoken（替换 `<YOUR_AUTHTOKEN>` 为您实际的值）：
```powershell
ngrok config add-authtoken <YOUR_AUTHTOKEN>
```

7. 验证安装：
```powershell
ngrok --version
```
输出版本号即表示成功。
### 3. （可选）安装 Git for Windows

如果您习惯使用 `git clone` 命令，可以安装 Git for Windows。  
从 [Git 官网](https://git-scm.com/downloads/win) 下载并安装即可（保持默认选项）。

如果您不想安装 Git，可以直接下载项目的 ZIP 压缩包。

---

## 📦 第二步：获取并启动代理服务

### 1. 克隆或下载 `deepseek-cursor-proxy` 项目

**方法一：使用 Git**

```powershell

git clone https://github.com/yxlao/deepseek-cursor-proxy.git
cd deepseek-cursor-proxy
```

**方法二：直接下载 ZIP**

- 访问项目主页：[https://github.com/yxlao/deepseek-cursor-proxy](https://github.com/yxlao/deepseek-cursor-proxy)
    
- 点击绿色的 **“Code”** 按钮 → **“Download ZIP”**
    
- 解压 ZIP 文件到您喜欢的目录，例如 `C:\deepseek-cursor-proxy`
    
- 在 PowerShell 中进入该目录：
    
    ```powershell
    cd C:\deepseek-cursor-proxy
    ```
    

### 2. 使用 `uv` 运行代理服务

在项目目录下，执行以下命令：

```powershell

uv run deepseek-cursor-proxy
```

- `uv run` 会自动创建项目的虚拟环境（`.venv` 文件夹）并安装所有依赖。
    
- **首次运行**会下载依赖包，可能需要一两分钟，请耐心等待。
    
- 启动成功后，您会看到类似以下的输出（其中包含 ngrok 生成的公网地址）：
    
    ```text
    
    Starting deepseek-cursor-proxy...
    ngrok tunnel started at: https://xxxx-xx-xx-xx-xx.ngrok-free.dev
    Proxy server listening on http://localhost:9000
    ```
    

**请复制 `https://xxxx-... .ngrok-free.dev` 这个地址**，稍后要在 Cursor 中配置。

> 💡 **提示**：  
> 如果您不想使用 ngrok（例如您有自定义域名或其他内网穿透方案），可以添加 `--no-ngrok` 参数：  
> `uv run deepseek-cursor-proxy --no-ngrok`  
> 但 Cursor 无法直接访问 `localhost`，因此通常仍需要 ngrok。

---

## ⚙️ 第三步：在 Cursor 中配置 DeepSeek 模型

1. **打开 Cursor 设置**
    
    - 启动 Cursor 编辑器
        
    - 点击左下角齿轮图标 → **“Settings”**（或按 `Ctrl + ,`）
        
2. **进入模型配置**
    
    - 在设置页面左侧菜单中，选择 **“Models”** 选项卡
        
3. **添加自定义模型**
    
    - 点击 **“Add Custom Model”** 按钮
        
    - 在弹出的对话框中，按照下表填写：
        

|配置项|填写内容|说明|
|---|---|---|
|**Model Name**|`deepseek-v4-pro` 或 `deepseek-v4-flash`|推荐使用 `deepseek-v4-pro`（更强的推理能力）|
|**API Key**|您的 DeepSeek API Key|从 DeepSeek 平台获取|
|**Base URL**|`https://xxxx-xx-xx-xx-xx.ngrok-free.dev/v1`|将 `xxxx...` 替换为第二步中复制的 ngrok 地址，**末尾必须加上 `/v1`**|
|**Model Provider**|选择 **“OpenAI”** 或 **“Custom”**|代理兼容 OpenAI API 格式|

4. **保存配置**
    
    - 点击 **“Add”** 或 **“Save”** 按钮
        
5. **切换模型（可选）**
    
    - 在 Cursor 的聊天窗口或 Agent 模式中，从模型下拉列表选择您刚才添加的模型（例如 `deepseek-v4-pro`）
        
    - 快捷键 `Ctrl + Shift + 0` 可以快速切换自定义 API 模型
        

---

## ✅ 第四步：验证使用

1. **确保代理服务正在运行**
    
    - 运行 `deepseek-cursor-proxy` 的终端窗口应保持打开，且显示 ngrok 地址无报错。
        
2. **在 Cursor 中发送一条消息**
    
    - 向 DeepSeek 模型提问，例如 `“Hello, what can you do?”`
        
    - 如果配置正确，您将收到模型的正常回复。
        
3. **测试工具调用（Tool Calls）**
    
    - 可以尝试让模型执行文件读写、代码运行等操作（Cursor 的 Agent 模式）。
        
    - 代理会正确处理 DeepSeek 的 `reasoning_content` 字段，**不会再出现 `reasoning_format` 错误**。
        

---

## 🛠️ 常见问题与故障排除

### ❓ 代理启动时提示端口 9000 被占用

**解决方法**：指定其他端口，例如 9001：

```powershell

uv run deepseek-cursor-proxy --port 9001
```

然后在 Cursor 的 Base URL 中也要对应修改端口号（ngrok 仍会映射该端口，无需修改 ngrok 地址）。

### ❓ 启动后没有显示 ngrok 地址

- 检查网络连接，确保能访问 `ngrok.io`。
    
- 确认已正确配置 ngrok Authtoken（执行 `ngrok config check` 可验证）。
    
- 如果长时间无输出，可以按 `Ctrl+C` 终止，然后重新运行。
    

### ❓ Cursor 中收到 `400` 或 `invalid_request_error`

可能原因：

- Base URL 末尾忘记加 `/v1`。
    
- DeepSeek API Key 无效或余额不足。
    
- 代理服务未正确运行（检查终端有无错误日志）。
    

### ❓ ngrok 重启后地址变了，Cursor 无法连接

- 免费版 ngrok 每次启动会分配一个**新的随机子域名**。
    
- 您需要**重新复制新的 ngrok 地址**，并在 Cursor 设置中**更新 Base URL**。
    
- 若想使用固定地址，可升级 ngrok 付费套餐（绑定保留域名），或使用 `--ngrok-url` 参数指定自定义域名（需自己配置 DNS）。
    

### ❓ 如何让代理在后台运行（不使用 pm2）？

如果您希望关闭终端后代理继续运行，可以使用 Windows 自带的 `start` 命令或创建快捷方式，但最简单的做法是**保持终端窗口最小化**。  
对于长期使用，建议使用 `pm2`（本指南不包含）或 Windows 服务包装工具（如 `winsw`）。

---

## 📖 补充说明：代理的工作原理

`deepseek-cursor-proxy` 解决了 Cursor 与 DeepSeek V4 模型之间的协议不兼容问题：

- **DeepSeek V4 的思考模式（thinking mode）** 要求在多轮对话中，客户端必须将上一轮模型返回的 `reasoning_content` 字段原样传回。
    
- **Cursor 会遗漏该字段**，导致 API 返回 `reasoning_format` 错误。
    
- 本代理会**自动存储** DeepSeek 返回的 `reasoning_content`，并在后续请求中**重新注入**到对话历史中，从而绕过 Cursor 的缺陷。
    
- 此外，代理还处理了 `functions`/`function_call` 到 `tools`/`tool_choice` 的转换，以及对 `reasoning_effort` 参数的兼容。
    

---

## 🔗 参考链接

- [deepseek-cursor-proxy GitHub 仓库](https://github.com/yxlao/deepseek-cursor-proxy)
    
- [DeepSeek 官方平台](https://platform.deepseek.com/)
    
- [uv 官方文档](https://docs.astral.sh/uv/)
    
- [ngrok 官方文档](https://ngrok.com/docs)
    

---

> **最后提醒**：本指南中使用的工具和服务均来自第三方。使用 DeepSeek API 会产生费用，请合理管理 API Key 和调用量。如果在使用过程中遇到问题，欢迎查阅上述参考链接或搜索相关社区讨论。

