# Claude Code 对接第三方 API 缓存命中率优化环境变量配置

第三方 API 命中率低**首要原因：CCH 随机计费头（anthropic-attribution-header）每次请求随机变化，破坏 system 前缀哈希，第三方网关无法命中缓存**，其次默认缓存 TTL 只有 5 分钟、缓存策略未开启长效缓存。下面分**必配核心变量、长效缓存变量、辅助优化变量、配置方式、验证方法**。

## 一、必配环境变量（命中率从 20%→90%+ 核心）

### 1. 关闭随机 Attribution Header（最关键）

bash

```
# Linux/macOS
export CLAUDE_CODE_ATTRIBUTION_HEADER=false
# Windows CMD
set CLAUDE_CODE_ATTRIBUTION_HEADER=false
# Windows PowerShell
$env:CLAUDE_CODE_ATTRIBUTION_HEADER="false"
```

**作用**：不再在 system prompt 头部插入每次随机变化的`anthropic-attribution:xxx`防伪串，6.8w + 固定系统提示词、工具定义前缀内容完全一致，第三方缓存 Key 稳定。

> 别名兼容：`CLAUDECODEATTRIBUTIONHEADER=0`等效关闭。

## 二、缓存 TTL 时效优化（默认 5 分钟太短，跨会话极易失效）

第三方 API / 代理默认缓存生命周期**5min**，频繁间断使用直接缓存过期，开启 1 小时长效缓存：

bash

```
# 开启1小时TTL长效缓存（第三方API专用，官方订阅默认自带1h）
export ENABLE_PROMPT_CACHING_1H=1

# 调试时强制锁定5分钟缓存（排查用，日常不用）
# export FORCE_PROMPT_CACHING_5M=1
```

- `ENABLE_PROMPT_CACHING_1H=1`：请求自动携带`ttl:1h`缓存标记，缓存存活 60 分钟，间歇打开新会话也能命中前缀缓存。

## 三、附加优化环境变量（减少 prompt 动态变更、稳固缓存 key）

bash

```
# 关闭动态注入实时时间（新版已优化，老版本必备，防止system prompt日期变动）
export CLAUDE_CODE_DISABLE_DYNAMIC_DATE=1

# 固定Agent版本标识，避免版本号随机变更破坏前缀哈希
export CLAUDE_CODE_FIX_VERSION_TAG=1

# 开启自动前缀缓存标记（自动在system末尾添加cache_control:ephemeral）
export AUTO_ADD_PROMPT_CACHE_CONTROL=1
```

## 四、三种持久化配置方式（永久生效，不用每次终端 export）

### 方式 1：全局 shell 配置（~/.bashrc/ ~/.zshrc）

shell

```
# 写入配置文件
echo 'export CLAUDE_CODE_ATTRIBUTION_HEADER=false' >> ~/.zshrc
echo 'export ENABLE_PROMPT_CACHING_1H=1' >> ~/.zshrc
source ~/.zshrc
```

### 方式 2：Claude Code 配置文件～/.claude/settings.json（推荐）

json

```
{
  "env": {
    "CLAUDE_CODE_ATTRIBUTION_HEADER": "false",
    "ENABLE_PROMPT_CACHING_1H": "1",
    "AUTO_ADD_PROMPT_CACHE_CONTROL": "1"
  },
  "anthropic_attribution_header": false
}
```

### 方式 3：启动命令行临时传参

bash

```
claude --attribution-header=false
```

## 五、第三方 API 网关配套优化（客户端配置完后补充）

1. 代理服务拦截：剔除请求里`anthropic-attribution:`开头文本，不参与缓存哈希计算；
2. OpenAI 兼容入参固定`prompt_cache_key=session_id`，同会话复用缓存分组；
3. Anthropic 原生接口在`metadata.user_id`绑定会话 ID。

## 六、命中率验证方法

查看 API 返回`usage`字段：

- `cache_read_input_tokens > 0`：缓存命中
- 命中占比 = `cache_read_input_tokens/(cache_read_input_tokens+input_tokens)`
    
    优化达标：命中率≥85%~95%。

## 最简一键部署脚本

bash

```
# 一键配置全量优化环境变量
export CLAUDE_CODE_ATTRIBUTION_HEADER=false
export ENABLE_PROMPT_CACHING_1H=1
export AUTO_ADD_PROMPT_CACHE_CONTROL=1
export CLAUDE_CODE_DISABLE_DYNAMIC_DATE=1
```