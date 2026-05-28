根据 GitHub 项目 rtk-ai/rtk 的官方文档，RTK 是一个专为 AI Agent（如 Cursor, Claude Code）设计的 CLI 代理工具，通过过滤和压缩终端输出（如 git status、npm test 等），可减少 60-90% 的 Token 消耗，显著降低 API 费用并提升 AI 的处理效率。

https://github.com/rtk-ai/rtk

  

以下是在 Windows 环境下，配合 Cursor 使用 RTK 的完整下载、安装及配置指南。

  

一、 下载 RTK (Windows 版)

访问 RTK Releases 页面。

  

在最新版本（Latest）的 Assets 列表中，下载后缀为 -x86_64-pc-windows-msvc.zip 的压缩包。

  

解压文件：将解压得到的 rtk.exe 放置在一个固定的文件夹中（例如：C:\Tools\rtk\）。

  

二、 安装与环境变量配置

为了在任何终端（以及 Cursor 内置终端）中都能运行 rtk 命令，需要将其添加到系统环境变量。

  

按下 Win + S，搜索“编辑系统环境变量”并打开。

  

点击“环境变量”按钮。

  

在“系统变量”栏找到 Path，点击“编辑” -> “新建”。

  

输入你存放 rtk.exe 的完整路径（例如 C:\Tools\rtk\）。
![[image-1.png]]

点击“确定”保存。

  

验证安装：打开 PowerShell 或 cmd，输入以下命令。如果显示版本号，则安装成功：

```

rtk --version

```

  

三、 在 Cursor 中配置使用

RTK 提供了针对 Cursor 的专门优化。虽然 RTK 在 Windows 原生环境（cmd/PowerShell）下无法像 Linux/macOS 那样实现全自动透明拦截，但可以通过以下两种方式在 Cursor 中获得最佳体验：

  

1. 自动配置方式（推荐）

在 Cursor 的内置终端中运行以下初始化命令：

```

rtk init --global --cursor

```

2. 手动提速（Cursor 常用命令）

在 Cursor 的终端或对话框（Ctrl+K / Ctrl+L）中，你可以让 Cursor 直接运行带有 rtk 前缀的命令，从而减少它读取到的上下文 Token。

  ![[image.png]]

![alt text](image.png)

  

四、 进阶：Windows WSL 用户 (最佳体验)

RTK 在 Windows 原生环境下由于系统限制（缺乏 Bash 钩子机制），自动重写（Auto-rewrite）功能较弱。如果你在 Cursor 中配合 WSL (Windows Subsystem for Linux) 使用，体验会大幅提升：

  

在 WSL 终端运行安装脚本：

```

curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

```

  

运行初始化：

```

rtk init --global --cursor

```

  

效果：在 WSL 下，当你告诉 Cursor “检查 git 状态”时，Cursor 发出的 git status 会在底层被自动重写为 rtk git status。AI 只能看到过滤后的简洁输出，从而避免因输出过长导致对话崩溃或费用激增。

  

五、 使用小贴士

查看省了多少钱：运行 rtk gain。它会显示通过 RTK 过滤节省的总 Token 数和预估节省的金额。

  

不要直接双击：rtk.exe 是命令行工具，双击会闪退。必须在 Cursor 终端、PowerShell 或 CMD 中通过指令调用。

  

Cursor 提示词优化：你可以在 Cursor 的 .cursorrules 文件（如果有）中加入一句话：“Always prefer using rtk prefixed commands for file reading, searching, and git operations to save tokens.” 引导 AI 主动使用 RTK。

  

----

## 如果在CMD中可以运行rtk命令，但是cursor中无法运行

在 CMD 中能运行，说明 rtk.exe 的路径已经正确添加到了系统的 Path 变量中。但由于你的 Cursor 是在修改环境变量之前就已经打开的，或者它内置的 PowerShell 终端没有同步最新的系统变量，导致了“找不到命令”的报错。

  

请按照以下步骤依次尝试，通常第一步就能解决：

  

1. 彻底重启 Cursor（最快解决）

Windows 的环境变量在修改后，已经启动的软件（如 Cursor 及其内置终端）无法实时感知。

  

请保存好代码，彻底关闭 Cursor。

  

重新打开 Cursor，在内置 PowerShell 中再次运行：

```

rtk init --global --cursor

```

报错的话执行

```

rtk init --global

```

  

在 Cursor 中的正确打开方式

由于我们无法通过 init 自动重写 Cursor 内部的命令，接下来在 Cursor 中使用时，你可以直接通过手动前缀或提示词约束来享受它的 Token 压缩功能。

  

在 Cursor 的 Ctrl + K（行内生成）或 Ctrl + L（侧边栏 Chat）中，直接让它用 rtk 执行命令。例如：

  

让 Cursor 检查你的 .NET 项目状态：

  

“帮我用 rtk git status 看看当前仓库状态，并清理未追踪的文件。”

  

让 Cursor 帮你跑测试：

  

“在终端运行 rtk dotnet test，如果报错了帮我分析原因。”

  

----

既然要配合 Cursor 和 RTK 使用，我们直接安装最稳定、生态最好的 Ubuntu。请在当前的 PowerShell 窗口中，按照以下两步操作即可搞定：

  

## 第一步：安装 Ubuntu 系统

在你的 PowerShell 窗口中，直接复制并运行以下命令：

```

PowerShell

wsl --install -d Ubuntu

```

会发生什么：系统会自动开始下载并安装 Ubuntu。由于是从微软服务器下载，根据网络情况可能需要等待 1 到几分钟。

  

完成的标志：安装完成后，系统通常会弹出一个新的独立黑色终端窗口。

  

## 第二步：设置 Linux 的用户名和密码

在新弹出的 Ubuntu 窗口中（或者提示你需要重启后打开的窗口中），你会看到如下提示：

Enter new UNIX username:

  

输入一个用户名：比如输入 aihelp，然后按回车。

  

设置密码：提示 New password: 时，输入你的密码（注意：Linux 输入密码时屏幕上不会显示任何星号或字符，这是正常的，盲打完直接回车即可）。

  

再次确认密码：提示 Retype new password: 时，再次输入相同的密码并回车。

  

当看到类似 aihelp@XXXX:~$ 的绿色或白色提示符时，说明你的 WSL 终端已经彻底安装并启动成功了！

  

🎉 回到 Cursor 使用 WSL 的最佳姿势

搞定之后，你可以享受最完美的 RTK 体验了：

  

在 Cursor 中切到 WSL：打开 Cursor，按 Ctrl + `  打开内置终端。点击终端右上角的 + 号下拉菜单，现在里面会多出一个 Ubuntu (WSL)，点击它。

  

一键安装 WSL 版 RTK：在 Cursor 的 WSL 终端中，直接复制运行 RTK 的 Linux 官方脚本：

```

Bash

curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

```

全局初始化：

```

Bash

rtk init --global

```

接下来，只要你在这个 WSL 终端里做开发、跑测试，或者用 .cursorrules 让 AI 在 WSL 环境下运行，RTK 就会在底层自动拦截所有的命令（比如你打 git status，它自动帮你跑 rtk git status），真正做到无感省钱，把 Token 消耗降到最低！