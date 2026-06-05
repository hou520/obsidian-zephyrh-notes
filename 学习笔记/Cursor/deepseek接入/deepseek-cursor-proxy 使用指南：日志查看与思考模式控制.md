# deepseek-cursor-proxy 使用指南：日志查看与思考模式控制

> 适用于 Windows 环境下使用 `deepseek-cursor-proxy` 代理 Cursor 接入 DeepSeek 的场景。  
> 重点说明如何通过配置文件控制思考模式（`thinking`）以及如何查看详细请求/响应日志。

---

## 📋 前提条件

- 已安装 `uv` 并克隆 `deepseek-cursor-proxy` 项目。
- 代理服务能够正常启动并转发请求到 DeepSeek API。
- Cursor 中已添加自定义模型，Base URL 指向代理地址（如 `http://127.0.0.1:9000/v1`）。

---

## 🔍 一、查看详细日志（请求/响应体及 Token 消耗）

代理默认输出 INFO 级别日志，若要查看完整的请求体、响应体以及 Token 使用情况，需要使用 **`--verbose`** 参数启动。

### 启动命令示例

```powershell
cd D:\Documents\Projects\Python\deepseek-cursor-proxy
uv run deepseek-cursor-proxy --no-ngrok --verbose
```
### 日志输出内容说明

- **启动信息**：显示当前配置（思考模式状态、缓存路径、上游地址等）。
    
- **`incoming POST`**：收到来自 Cursor 或 Postman 的请求。
    
- **`cursor request body`**：Cursor 发送的原始请求体（不含 `thinking` 等额外参数）。
    
- **`upstream request body`**：代理实际转发给 DeepSeek API 的请求体（包含 `thinking` 字段）。
    
- **`upstream response status`**：API 返回状态码及耗时（`elapsed_ms`）。
    
- **`cursor response body`**：DeepSeek 返回的完整响应，包含 `usage`（Token 统计）和 `choices`（最终答案）。
    
- **`stats` 行**：汇总本次请求的 `prompt_tokens`、`output_tokens`、`reasoning_tokens`（如果开启思考模式）以及缓存命中率。
- ### 示例日志片段（思考模式关闭）

text

2026-05-19 19:22:32,009 INFO default_model: deepseek-v4-pro (no thinking, max)
...
2026-05-19 19:22:37,512 INFO upstream request body:
{
  "messages": [...],
  "model": "deepseek-v4-flash",
  "thinking": {
    "type": "disabled"
  }
}
...
2026-05-19 19:22:38,341 INFO └ stats   prompt=5 output=9 reasoning=? cache_hit=0.0%

### 日志重定向到文件

如果希望保存日志以供后续分析，可将输出重定向到文件：

powershell

uv run deepseek-cursor-proxy --no-ngrok --verbose > proxy.log 2>&1

---

## 🧠 二、控制思考模式（`thinking`）

思考模式开启时，模型会先进行内部推理（生成 `reasoning_content`），再输出最终答案。这会显著增加 Token 消耗和响应时间。  
可通过修改**用户级配置文件**来永久关闭或开启思考模式。

### 配置文件位置

- **Windows**：`C:\Users\<你的用户名>\.deepseek-cursor-proxy\config.yaml`
    
- 如果文件不存在，首次运行代理时会自动生成。
    

### 关键配置项

yaml

# 思考模式总开关：disabled / enabled / auto
default_thinking: disabled
# 仅在 thinking 为 enabled 或 auto 时生效，控制推理强度：low / medium / high / max
default_reasoning_effort: max

|`default_thinking` 值|效果|
|---|---|
|`disabled`|**关闭思考模式**，速度最快，Token 最少（推荐日常使用）|
|`enabled`|强制开启，每次请求都进行内部推理|
|`auto`|模型根据复杂度自动决定是否思考|

### 修改步骤

1. 关闭正在运行的代理服务（`Ctrl + C`）。
    
2. 用任何文本编辑器打开 `C:\Users\aihelp\.deepseek-cursor-proxy\config.yaml`（路径中的 `aihelp` 替换为你的用户名）。
    
3. 找到 `default_thinking:` 行，将其值改为 `disabled`（若已为 `disabled` 则无需修改）。
    
4. 保存文件。
    
5. 重新启动代理：`uv run deepseek-cursor-proxy --no-ngrok --verbose`
    

### 验证修改是否生效

- 查看启动日志：应显示 `default_model: deepseek-v4-pro (no thinking, max)`（括号内为 `no thinking`）。
    
- 发送测试请求后，查看 `upstream request body` 应包含 `"thinking": {"type": "disabled"}`。
    
- 响应中不应出现 `reasoning_content` 字段，且 `completion_tokens` 数量显著减少。
    

---

## ⚠️ 常见问题

### 1. 修改 `config.yaml` 后未生效？

- 确保**重启了代理服务**（配置只在启动时读取一次）。
    
- 检查 YAML 语法是否正确（冒号后必须有空格，如 `default_thinking: disabled`）。
    
- 确认修改的是正确的用户目录（可使用 `echo %USERPROFILE%` 查看当前用户路径）。
    

### 2. 即使设置了 `disabled`，日志中仍出现 `thinking: enabled`？

- 可能是代码中硬编码覆盖了配置。请检查 `deepseek_cursor_proxy/proxy.py` 是否直接从配置读取 `thinking` 字段。建议从官方仓库拉取最新代码，或者手动修改代码中的逻辑。
    

### 3. 关闭思考模式后，模型回答质量会下降吗？

- 对于简单问题（如问候、基础知识问答）几乎无影响。
    
- 对于复杂推理（如数学证明、多步逻辑分析），关闭思考模式可能导致回答不够严谨。此时可临时切换模型或在 Cursor 中另建一个开启思考模式的模型供特殊使用。
    

---

## 📌 总结

- **日志查看**：使用 `--verbose` 启动代理，观察请求/响应体及 Token 统计。
    
- **控制思考模式**：修改 `~/.deepseek-cursor-proxy/config.yaml` 中的 `default_thinking` 字段为 `disabled`，可实现默认关闭，大幅节省 Token 并提高响应速度。
    

按照本文档操作，即可在 Windows 11 上灵活管理 `deepseek-cursor-proxy` 的行为，获得更高效、低成本的使用体验。