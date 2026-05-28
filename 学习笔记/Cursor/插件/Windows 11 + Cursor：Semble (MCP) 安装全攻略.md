这份文档总结了我们在 Windows 11 环境下，针对 **D 盘自定义路径**以及 **Cursor 路径解析特性**整理出的最佳实践流程。

# Windows 11 + Cursor：Semble (MCP) 安装全攻略

本指南适用于希望在非系统盘（如 D 盘）安装 `semble` 并集成到 Cursor 的开发者，解决了 Windows 路径识别及依赖缺失等常见坑点。

## 1. 环境准备

确保你的系统中已安装 **Python 3.10+** 和 **uv**（高性能 Python 包管理器）。

- **uv 安装路径**（示例）：`D:\Python\uv\uv.exe`
    
- **基础 Python 路径**（示例）：`D:\Python\Python312\python.exe`
    

## 2. 纯净环境构建

在 PowerShell 中执行以下命令，建议手动指定路径以确保环境隔离。

PowerShell

```
# 1. 创建专用的虚拟环境
D:\Python\uv\uv.exe venv D:\Python\.semble_test

# 2. 精准安装 semble 及其 MCP 扩展（核心步骤：必须指向该环境的 python.exe）
D:\Python\uv\uv.exe pip install "semble[mcp]" --python D:\Python\.semble_test\Scripts\python.exe
```

## 3. 首次启动与模型预热

`semble` 在首次运行时需要从 HuggingFace 下载语义模型（约数百 MB）。建议在终端手动触发下载，避免 Cursor 连接超时。

PowerShell

```
# 运行以下命令，直到看到 "HTTP 200 OK" 且不再有大量下载进度条
D:\Python\.semble_test\Scripts\semble.exe mcp
```

_下载完成后，按 `Ctrl + C` 退出。_

## 4. 配置 Cursor MCP

打开 Cursor，进入 **Settings -> Features -> MCP**，点击 **+ Add Server**。

- **Name**: `semble`
    
- **Type**: `command`
    
- **Command**: `D:\Python\.semble_test\Scripts\semble.exe`
    
- **Args**: `["mcp"]`
    

> **注意**：如果 Cursor 提示 `FileNotFoundError`，请将 `args` 修改为 `["mcp", "."]` 以强制指定当前目录。

## 5. 故障排查与日志解读

### Q: Cursor Output 窗口显示一堆红色 `[error]` 日志？

- **原因**：Cursor 将所有 `stderr`（标准错误流）输出都标红，而 Python 工具习惯将正常的 `INFO` 运行日志发送到该流。
    
- **对策**：只要 Cursor MCP 界面显示 **绿灯** 且出现 **`search`**、**`find_related`** 工具，请直接忽略红色日志。
    

### Q: 提示 `No module named semble`？

- **原因**：`semble.exe` 没能正确关联到同目录下的库。
    
- **对策**：在 `mcp.json` 中尝试将 `command` 改为 Python 引导模式：
    
    - **Command**: `D:\Python\.semble_test\Scripts\python.exe`
        
    - **Args**: `["-m", "semble", "mcp"]`
        

## 6. 日常使用

- **语义搜索**：在 Chat 窗口输入 `@semble search "你的问题"`。
    
- **手动索引**：如果发现搜不到新代码，在项目根目录运行： `D:\Python\.semble_test\Scripts\semble.exe search "init"` 强制刷新索引。
    

**一句话秘籍**：在 Windows 上，**绝对路径**和**环境一致性**高于一切。只要 `uv pip` 装对了地方，Cursor 就能变绿。