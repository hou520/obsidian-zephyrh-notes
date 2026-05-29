以下是基于 `uv tool` 方式整理的 **Windows 11 + Cursor 安装 Semble (MCP) 最终版指南**。

# Windows 11 + Cursor: Semble (MCP) 全局工具安装指南

这种方式利用 `uv` 的工具管理功能，为 `semble` 创建一个独立的、持久的全局环境，是目前在 Windows 上最稳定、配置最简单的方案。

## 1. 环境安装

打开 PowerShell，执行以下命令安装 `semble` 及其 MCP 插件：

PowerShell

```
# 将 semble 安装为全局工具
uv tool install "semble[mcp]"
```

安装完成后，`uv` 会自动将 `semble.exe` 放置在它的二进制目录中（通常是 `%LOCALAPPDATA%\uv\bin`）。

## 2. 首次运行与模型下载

由于 `semble` 依赖 AI 模型，建议在终端手动触发一次下载，以防 Cursor 后台连接超时：

PowerShell

```
# 运行 mcp 模式以触发模型下载
semble mcp
```

- **观察**：看到 `HTTP Request ... 200 OK` 证明正在下载。
    
- **结束**：等下载日志停止滚动后，按 `Ctrl + C` 退出。
    

## 3. 配置 Cursor MCP

在 Cursor 中，这种安装方式的配置最为简洁。

1. 进入 **Settings -> Features -> MCP**。
    
2. 点击 **+ Add Server**：
    
    - **Name**: `semble`
        
    - **Type**: `command`
        
    - **Command**: `semble` _(注：如果提示找不到命令，请使用绝对路径，见下方提示)_
        
    - **Args**: `["mcp"]`
        

> **路径提示**：如果输入 `semble` 无法启动，请在终端输入 `where.exe semble` 获取绝对路径（例如 `C:\Users\你的用户名\.local\bin\semble.exe`），并将其填入 **Command** 栏。

## 4. 日常维护

### 索引项目代码

在使用 `@semble` 搜索前，建议在项目根目录下先建立索引：

PowerShell

```
semble search "init"
```

### 升级工具

如果 `semble` 发布了新版本，只需一行命令即可平滑升级：

PowerShell

```
uv tool upgrade "semble[mcp]"
```

## 5. 如何彻底删除？

如果你决定不再使用 `semble`，请按照以下步骤彻底清理：

1. **移除工具**：
    
    PowerShell
    
    ```
    uv tool uninstall semble
    ```
    
2. **清理模型缓存**： 手动删除模型占用空间（约几百 MB）： `C:\Users\你的用户名\.cache\huggingface`
    
3. **清理项目索引**： 删除代码项目根目录下的 `.semble` 隐藏文件夹。
    
4. **移除 Cursor 配置**： 在 Cursor MCP 设置中点击垃圾桶图标。
    

### 为什么推荐 `uv tool` 方式？

- **持久性**：不像 `uvx` 每次运行可能需要重新检查环境，`uv tool` 安装后响应速度极快。
    
- **路径清晰**：它生成的 `.exe` 垫片（Shim）对 Windows 的路径解析非常友好，极少出现 `FileNotFoundError`。
    
- **环境隔离**：虽然它是全局命令，但它的依赖库完全锁定在独立的虚拟环境中，不会污染你其他的 Python 项目。

测试：
只要 Cursor 的那个圆点变**绿**且显示 **2 tools**：

1. 按 `Ctrl + L` 打开聊天。
    
2. 输入 `@semble search "test"`。
    
3. 如果它开始扫描你的索引，那么恭喜你，这个最难搞的 Windows MCP 环境终于打通了！