本文档指导你如何在 Cursor 编辑器中手动添加 **DeepSeek Chat** 模型（`deepseek-chat`），并通过官方 API 进行调用，无需任何代理。

## 1. 打开 Cursor 设置

- 启动 Cursor 编辑器。
    
- 点击左下角的齿轮图标 → **Settings**（或使用快捷键 `Ctrl + ,`）。
    

## 2. 进入模型配置

在设置页面的左侧菜单中，选择 **Models** 选项卡。

## 3. 添加自定义模型

- 点击 **Add Custom Model** 按钮。
    
- 在弹出的对话框中，按下表填写配置项：
    

| 配置项                | 填写内容                      | 说明                                                   |
| ------------------ | ------------------------- | ---------------------------------------------------- |
| **Model Name**     | `deepseek-chat`           | 使用 DeepSeek 官方的对话模型（非 V4 Flash 版本）                   |
| **API Key**        | 你的 DeepSeek API Key       | 从 [DeepSeek 开放平台](https://platform.deepseek.com/) 获取 |
| **Base URL**       | `https://api.deepseek.com | DeepSeek 官方 API 端点，**无需任何代理**                        |
| **Model Provider** | `OpenAI` 或 `Custom`       | DeepSeek API 兼容 OpenAI 格式，选择任一均可                     |

## 4. 保存配置

点击 **Add**（或 **Save**）按钮，完成模型添加。

## 5. 切换模型（可选）

- 在 Cursor 的聊天窗口或 Agent 模式中，从模型下拉列表中选择你刚才添加的模型（`deepseek-chat`）。
    
- 快捷键 `Ctrl + Shift + 0` 可以快速切换自定义 API 模型。
    

---

### 补充说明

- 该配置使用 DeepSeek 官方 API，不需要任何本地代理或 ngrok 转发。
    
- 模型 `deepseek-chat` 当前对应 DeepSeek-V3.2 的非思考模式，上下文 128K，输入价格为 ￥1 / 百万 tokens，输出价格为 ￥2 / 百万 tokens。
    
- 如果你希望使用更强的推理能力，可以将 Model Name 改为 `deepseek-reasoner`（需注意价格和速率限制）。
    

配置完成后，即可在 Cursor 中直接调用 DeepSeek 模型进行代码辅助和对话。