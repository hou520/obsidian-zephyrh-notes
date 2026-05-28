要彻底删除我们刚才配置的 `semble` 环境，主要涉及 **Cursor 配置清理**、**物理文件删除** 以及 **uv 缓存清理** 三个步骤。

按照以下顺序操作即可做到“无痕卸载”：

### 第一步：清理 Cursor 配置

首先让 Cursor 停止占用相关文件，否则 Windows 可能会提示“文件正在被使用”。

1. 打开 Cursor 的 **Settings -> Features -> MCP**。
    
2. 找到 `semble` 项目。
    
3. 点击右侧的 **垃圾桶图标** (Delete) 彻底删除该配置。
    

### 第二步：删除物理环境（核心步骤）

由于我们将环境安装在 `D:\Python\.semble_test`，直接删除该文件夹即可移除所有相关的 Python 解释器、库文件和可执行程序。

在 PowerShell 中执行：

PowerShell

```
# 删除虚拟环境文件夹
Remove-Item -Recurse -Force D:\Python\.semble_test
```

### 第三步：删除项目索引文件

`semble` 在运行期间会在你的代码项目根目录下创建一个隐藏文件夹来存储向量索引。如果你以后也不打算用它了，建议也删掉它。

1. 进入你的代码项目文件夹（例如 `D:\Visual Studio\Projects\aihelp`）。
    
2. 找到并删除 **`.semble`** 隐藏文件夹。
    

### 第四步：清理 uv 全局缓存（可选）

虽然你删除了虚拟环境，但 `uv` 的全局缓存中可能还保留着下载好的 `semble` 和 `numpy` 等原始包。如果你想彻底腾出硬盘空间，可以执行：

PowerShell

```
# 清理所有未被使用的 uv 缓存
D:\Python\uv\uv.exe cache clean
```

### 验证是否删除干净

1. 在终端输入 `D:\Python\.semble_test\Scripts\semble.exe`，如果提示 **“路径不存在”**，说明删除成功。
    
2. 检查 `D:\Python` 目录下不再有 `.semble_test` 文件夹。
    

**注意：** 如果你以后想重新安装，只需要再次运行你刚才整理的那份 MD 文档里的“创建环境”和“安装”命令即可。这套流程非常绿色，不会影响你 D 盘其他的 Python 环境。