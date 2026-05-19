从 `ngrok` 切换到 `Cpolar` 来为 `deepseek-cursor-proxy` 服务，步骤很直接。你只需要在配置内网穿透时，用 `Cpolar` 创建一条指向本地 `9000` 端口的隧道即可。

这个方案的核心操作就是，在 `Cpolar` 的 Web 管理界面创建一个全新的隧道，将它映射到你本地的代理服务上。

下面是一份完整的 Windows 11 配置指南。

---

### 📝 配置步骤详解

#### 第一步：准备工作

1. **获取 `Cpolar` 认证令牌 (Authtoken)**：用邮箱 [注册](https://dashboard.cpolar.com/signup) 并登录 `Cpolar` 官网[](https://www.cpolar.com/blog/serv-u-sftp-client)。登录后，在后台仪表盘里找到你的**认证令牌 (Authtoken)**，复制保存好，后面会用到[](https://blog.csdn.net/qq_63911508/article/details/150354722)。
    
2. **安装 Cpolar**：在官网下载 Windows 版本安装包（`.msi` 文件）。双击安装包，一路点击“Next”即可完成安装[](https://blog.csdn.net/weixin_73134956/article/details/156204340)。
    
3. **保持代理服务运行**：确保 `deepseek-cursor-proxy` 正在前台运行，并且会占用 `9000` 端口。如果端口不同，可自行修改。
    

#### 第二步：配置并启动 Cpolar

1. **认证与初始化**：
    
    - 打开一个命令提示符 (cmd) 或 PowerShell。
        
    - 输入以下命令，将 `YOUR_AUTHTOKEN` 替换为你从官网复制的认证令牌，完成客户端认证[](https://blog.csdn.net/weixin_43309426/article/details/150554508)。
        
        powershell
        
        cpolar authtoken YOUR_AUTHTOKEN
        
2. **通过 Web 界面创建隧道**：
    
    - 在浏览器中访问 `http://localhost:9200`，这是 `Cpolar` 的 Web 管理界面。
        
    - 用你的 `Cpolar` 账号登录。
        
    - 在左侧菜单栏找到 **“隧道管理”** -> **“创建隧道”**[](https://blog.csdn.net/qq_63911508/article/details/150354722)。
        
    - 按以下信息填写（端口可根据实际情况调整）：
        
        - **隧道名称**: `deepseek-proxy` （可自定义，但建议使用英文）
            
        - **协议**: 选择 **HTTP**[](https://www.cpolar.com/blog/say-goodbye-to-local-area-networks-enable-free-remote-access-to-nascab-cloud-2)。因为代理服务提供了标准的 HTTP 接口。
            
        - **本地地址**: 填写 **`9000`**，这是代理服务的默认端口。
            
        - **域名类型**: **“随机域名”** 即可满足测试和使用。
            
        - **地区**: 选择 **China VIP** 以获得更稳定的连接[](https://post.smzdm.com/p/am9nkgz4/)。
            
    - 点击 **“创建”** 按钮。
        
3. **获取公网地址**：
    
    - 隧道创建成功后，点击左侧菜单栏的 **“状态”** -> **“在线隧道列表”**[](https://blog.csdn.net/qq_63911508/article/details/150354722)。
        
    - 在列表中找到你刚刚创建的 `deepseek-proxy` 隧道，复制它对应的 **“公网地址”**，格式类似 `https://xxxxx.cpolar.cn`。
        
    - **这个地址就是 Cursor 需要配置的 Base URL 地址。**
        

#### 第三步：配置 Cursor

1. 打开 Cursor 编辑器，进入 `Settings` -> `Models` 页面。
    
2. 点击 `Add Custom Model`。
    
3. 按照以下信息填写：
    
    - **Model Name**: `deepseek-v4-pro` (或其他你想用的模型名称)
        
    - **API Key**: 填入你的 DeepSeek API Key。
        
    - **Base URL**: 填入上一步复制的 **`Cpolar 公网地址`**，并加上 `/v1` 路径。例如，如果你的公网地址是 `https://xxxxx.cpolar.cn`，那么这里应填写 `https://xxxxx.cpolar.cn/v1`[](https://blog.csdn.net/weixin_42557537/article/details/160936312)。
        
    - **Model Provider**: 选择 `OpenAI` 或 `Custom`。
        
4. 点击 `Add` 保存配置。
    

#### 第四步：验证

在 Cursor 的聊天框中选择你刚添加的模型并提问。如果配置正确，你应该能收到模型的正常回复。

---

### 🔧 故障排查与进阶

#### 常见问题

- **Cursor 报错 `reasoning_format`?**
    
    - **原因**: 协议转换问题。
        
    - **解决**: 确保 `deepseek-cursor-proxy` 代理服务已启动，且 Cursor 中配置的 Base URL 指向的是 `Cpolar` 的公网地址（而非 `localhost`）。
        
- **Cpolar 公网地址无法访问?**
    
    - **原因**: 代理服务未运行，或 Cpolar 隧道配置的端口不匹配。
        
    - **解决**: 检查代理服务是否正常运行，并确认 Cpolar 隧道中的“本地地址”是否为 `9000`（或你设置的代理端口）。
        
- **免费版 Cpolar 公网地址不稳定?**
    
    - **原因**: 免费版生成的公网地址是随机的，并且会在 24 小时后或 `Cpolar` 服务重启后发生变化[](https://blog.csdn.net/weixin_42878111/article/details/159670196)。
        
    - **影响**: 一旦地址变化，Cursor 将无法连接。届时，你需要回到 `Cpolar` 的“在线隧道列表”重新获取新的公网地址，并更新 Cursor 中的 Base URL 配置。
        
    - **解决**: 若需长期稳定使用，可考虑升级至 `Cpolar` 的付费套餐，以**保留一个固定的二级子域名**或**绑定自己的域名**。
        

---

### 💎 总结

总的来说，整个配置流程包括以下几个关键步骤：

1. 保持 `deepseek-cursor-proxy` 后台运行。
    
2. 安装并登录 `Cpolar`。
    
3. 在 `Cpolar` Web 界面创建一条 HTTP 隧道，指向你本地的代理服务端口（默认是 `9000`）。
    
4. 将获取到的 `Cpolar` 公网地址，作为 Base URL 配置到 Cursor 的自定义模型中。