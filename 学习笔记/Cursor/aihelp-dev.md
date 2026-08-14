为了让大家更方便的使用定制的这套工作流，创建了一个插件，目前支持cursor、 codex和claude code。其他vibe coding环境会陆续添加，时间有限先给大家使用一下，遇到问题找我或者找Gary都行。

安装方式：

- 删除、卸载之前我在wiki中分发给大家的rule和subagent
    
- **cursor安装**：export AIHELP_AGENT_PLUGIN_REF=v0.1.3 && /bin/bash -c "$(curl -fsSL https://git.aihelp.net/randy/aihelp-cursor-plugin/-/raw/$AIHELP_AGENT_PLUGIN_REF/scripts/install.sh)"
    
- **Claude code安装**：export AIHELP_AGENT_PLUGIN_REF=v0.1.3 && /bin/bash -c "$(curl -fsSL https://git.aihelp.net/randy/aihelp-cursor-plugin/-/raw/$AIHELP_AGENT_PLUGIN_REF/scripts/install-claude.sh)"
    
- **Codex 安装**：export AIHELP_AGENT_PLUGIN_REF=v0.1.3 && /bin/bash -c "$(curl -fsSL https://git.aihelp.net/randy/aihelp-cursor-plugin/-/raw/$AIHELP_AGENT_PLUGIN_REF/scripts/install-codex.sh)"
    

使用方式：

- 手册：[AIHelp-Dev 工作流说明](https://git.aihelp.net/randy/aihelp-cursor-plugin/-/blob/v0.1.3/docs/aihelp-dev/AIHelp-Dev%E5%B7%A5%E4%BD%9C%E6%B5%81%E5%AE%9A%E5%88%B6%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E.md)
    
- 安装之后新开会话的第一次Agent交互会检查工程中的工作流schema、codegraph的状态等，遇到提问的就直接初始化就行，如果没初始化可以直接调用 “/using-aihelp”这个skill手动初始化一下工作流自定义的schema
    
![[学习笔记/Cursor/attachments/b37593026ce2125efc9106c42b49b12a.png]]

----

这个插件大家尽快安装，windows系统的先用下面的命令安装cursor的插件，macos的直接用上面的命令不变，直接版本升级到v0.1.4。支持zcode、kimi-code、修改了几个skill的bug。

windows的安装命令

```

$env:AIHELP_AGENT_PLUGIN_REF = "v0.1.4"

$env:FORCE = "1"

irm [https://git.aihelp.net/randy/aihelp-cursor-plugin/-/raw/v0.1.4/scripts/install.ps1](https://git.aihelp.net/randy/aihelp-cursor-plugin/-/raw/v0.1.4/scripts/install.ps1) | iex

```

----

关于这个aihelp-dev的工作流跟大家统一说一下

1. 插件正确安装之后首次在一个工程中执行的时候会提示大家初始化工程的工作流schema和codegraph，大家开发哪个工程如果初始化了直接提交即可，其他人就不会初始化了。
    
2. 如果工程没初始化，大家发现`openspec/config.yaml`这个文件中不是schema: aihelp-dev，也没提示，可以手动执行 “/using-aihelp 初始化工程” 这个命令。会主动初始化内容。
    

其他的使用方式都在手册：[AIHelp-Dev 工作流说明](https://git.aihelp.net/randy/aihelp-cursor-plugin/-/blob/v0.1.3/docs/aihelp-dev/AIHelp-Dev%E5%B7%A5%E4%BD%9C%E6%B5%81%E5%AE%9A%E5%88%B6%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E.md) 中写清楚了