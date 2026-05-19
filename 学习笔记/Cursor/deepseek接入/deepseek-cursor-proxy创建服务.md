# 方案1 新建 .vbs 文件和任务计划程序——可行
### 第一步：编写静默启动脚本

为了让程序在后台运行且**不弹出那个黑色的 CMD 窗口**，我们需要一个简单的 `.vbs` 脚本。

1. 在项目目录 `D:\Documents\Projects\Python\deepseek-cursor-proxy\` 下新建一个文件，命名为 `run_proxy.vbs`。
    
2. 将以下内容粘贴进去（注意：路径一定要使用双引号包裹）：
    
```VBScript
Set shell = CreateObject("WScript.Shell")
' 设置工作目录
shell.CurrentDirectory = "D:\Documents\Projects\Python\deepseek-cursor-proxy\"
' 执行命令：0 代表隐藏窗口，False 代表脚本不需要等待程序结束
shell.Run ".\.venv\Scripts\python.exe -m deepseek_cursor_proxy --no-ngrok", 0, False
```

---

### 第二步：创建“伪服务”任务

1. 按下 `Win + S`，搜索并打开 **任务计划程序**。
    
2. 在右侧操作栏点击 **创建任务...**（注意：不是创建基本任务）。
    
3. **常规**选项卡：
    
    - **名称**: `DeepSeek-Cursor-Proxy`
        
    - 勾选 **不管用户是否登录都要运行**（这是实现“服务化”的关键，确保它在后台运行）。
        
    - 勾选 **使用最高权限运行**。
        
4. **触发器**选项卡：
    
    - 点击“新建”，选择 **启动时**（At startup）。
        
5. **操作**选项卡：
    
    - 点击“新建”，操作选择“启动程序”。
        
    - **程序或脚本**: `wscript.exe`
        
    - **添加参数**: `"D:\Documents\Projects\Python\deepseek-cursor-proxy\run_proxy.vbs"` (带上双引号)。
        
6. **条件**选项卡：
    
    - **取消勾选** “只有在计算机使用交流电源时才启动此任务”。
        
7. **设置**选项卡：
    
    - 勾选 “如果任务失败，按以下频率重新启动”，设置为 1 分钟。
        

---

### 第三步：保存并激活

1. 点击“确定”保存任务。
    
2. 系统会弹出一个对话框，要求输入你的 **Windows 账户和密码**。
    
    - _注：这是为了让任务能在你未登录时以你的权限在后台静默运行。_
        
3. 在列表中找到刚刚创建的任务，点击右侧的 **运行** 来测试一下。
    

---

### 如何验证是否成功？

1. **查看进程**: 打开“任务管理器” -> “详细信息”，找找看有没有 `python.exe` 正在运行。
    
2. **测试接口**: 访问你的 `http://127.0.0.1:9000/v1`（或者项目定义的其他端口），看是否有响应。
    
3. **日志查看**: 如果程序有报错，由于是静默运行，你可能看不见。建议在 `.vbs` 脚本测试成功后，再重启电脑验证它是否会自动随系统启动。
    

这种方法的优点是**纯原生、不依赖第三方、完美支持路径跳转、且彻底隐藏黑窗口**。

-----
-----

# 方案2 新建 .bat 文件和sc命令——不太可行
## 1、新建 .bat 文件
```run-proxy.bat
@echo off
d:
cd D:\Documents\Projects\Python\deepseek-cursor-proxy\
uv run deepseek-cursor-proxy --no-ngrok
```

## 2、添加服务
```powershell
PS C:\Users\aihelp> sc.exe create deepseek-proxy binPath= "D:\Documents\Projects\Python\start-deepseek-cursor-proxy\run_proxy.bat" start= auto
[SC] CreateService 成功
PS C:\Users\aihelp> sc.exe delete deepseek-proxy
[SC] DeleteService 成功
```

## 3、结果
![[Pasted image 20260519153901.png]]

原因：
**错误 1053** 是 Windows 服务开发中非常经典的报错。简单来说，Windows 服务启动时，系统会给程序约 30 秒的时间来反馈“我启动成功了”。由于 `uv run` 需要加载 Python 环境和各种依赖，耗时往往超过了系统的等待阈值，导致系统认为启动失败并强行关停。

----
----

# 方案3 新建 .bat 文件和sc命令——不太可行
## 1、新建 .bat 文件
```run-proxy.bat
@echo off
:: 1. 切换到脚本所在的盘符
d:
:: 2. 切换到项目具体目录
cd "D:\Documents\Projects\Python\deepseek-cursor-proxy"

:: 3. 执行启动命令
:: 使用绝对路径指向虚拟环境中的 python 解释器
".\.venv\Scripts\python.exe" -m deepseek_cursor_proxy --no-ngrok

:: 如果程序崩溃，暂停窗口以便查看报错（可选）
pause
```

## 2、添加服务
```powershell
PS C:\Users\aihelp> sc.exe create deepseek-proxy binPath= "D:\Documents\Projects\Python\deepseek-cursor-proxy\start_proxy.bat" start= auto
[SC] CreateService 成功
PS C:\Users\aihelp> sc.exe delete deepseek-proxy
[SC] DeleteService 成功
```

## 3、结果
还是报错：**错误 1053** 超时

