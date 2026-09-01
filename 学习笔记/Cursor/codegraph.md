codegraph。更新了。现在新版本快速使用是这个
npm install -g @optave/codegraph
cd your-project
codegraph build

https://git.aihelp.net/randy/aihelp-cursor-plugin/-/blob/v0.1.3/docs/aihelp-dev/AIHelp-Dev%E5%B7%A5%E4%BD%9C%E6%B5%81%E5%AE%9A%E5%88%B6%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E.md
现在开发流程示例使用的是旧版codegraph，codegraph init -i命令在新版本已经不存在，如果更新了可以看一下新命令。

---------
Windows 上全局装的是 **CLI**，包名是 `@colbymchenry/codegraph`。装完还要把 MCP 接到 Cursor，并在每个仓库建索引。

## 1. 全局安装 CLI（二选一）

**推荐（不依赖本机 Node）**，在 PowerShell 里：

```powershell
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex
```

**已有 Node** 时：

```powershell
npm i -g @colbymchenry/codegraph
```

装完 **新开一个终端**，再验证：

```powershell
codegraph --version
```

官方安装脚本会把 `codegraph` 加到 PATH，但不会刷新当前窗口。

以后升级：

```powershell
codegraph upgrade
```

不要装 npm 上的 `codegraph-ai`，那是另一个项目。

## 2. 接到 Cursor（全局一次）

CLI 装好 **不会** 自动出现 MCP 工具。再执行：

```powershell
codegraph install --yes
```

这会检测 Cursor / Claude Code 等，并写入 MCP 配置。一台电脑做一次即可，不用每个仓库再跑。

只要 Cursor：

```powershell
codegraph install --target=cursor --yes
```

然后 **重启 Cursor**。

## 3. 每个项目建索引（不是全局）

在仓库根目录：

```powershell
cd D:\Documents\Projects\DotNet\aihelp\Server_DotNetCore
codegraph init -i
```

这会生成 `.codegraph/`。没有这个目录时，Agent 会报 not initialized。AIHelp 侧用的就是 `codegraph init -i`。

---

三步对应三件事：`npm i -g` / `install.ps1` → 电脑上有 `codegraph` 命令；`codegraph install` → Cursor 能调 MCP；`codegraph init` → 当前仓库有图谱。