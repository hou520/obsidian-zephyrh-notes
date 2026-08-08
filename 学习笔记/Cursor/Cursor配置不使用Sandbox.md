![[Pasted image 20260808114408.png]]
你这个截图已经很明确了。**你现在的 `Run Mode` 是 `Auto-Review`，而这正是导致部分 Go 命令进入 Sandbox、从而使用 `cursor-sandbox-cache` 的原因。**

Cursor 官方对 `Auto-review` 的执行顺序说明是：先检查 Allowlist → 能 Sandbox 的命令进入 Sandbox → 其他命令交给 classifier。也就是说，**Auto-Review 并不等于“全部使用本机环境”**。([Cursor](https://cursor.com/changelog/auto-review?utm_source=chatgpt.com "Auto-review Run Mode · Cursor"))

结合你前面得到的：

```text
GOPATH     = D:\Go\workspace
GOMODCACHE = C:\Users\aihelp\AppData\Local\Temp\cursor-sandbox-cache\...\go-mod
GOCACHE    = C:\Users\aihelp\AppData\Local\Temp\cursor-sandbox-cache\...\go-build
```

基本可以确定：**Go build 被 Cursor 放进 Sandbox 执行了。**

## 你应该怎么改

你截图里的下拉菜单有三个选项：

```text
Allowlist
Auto-Review    ← 当前
Run Everything
```

### 推荐：选择 `Allowlist`

直接把：

```text
Auto-Review
```

改成：

```text
Allowlist
```

也就是：

**Cursor Settings → Agents → Execution and Approvals → Run Mode → Allowlist**

这是我最推荐你的配置。

Cursor 官方社区也明确说明，在 Windows 上如果希望 Agent 命令运行在主机环境，而不是 Sandbox，可以切换到 Allowlist；Allowlist 中的命令会自动执行，其他命令需要确认。([Cursor - Community Forum](https://forum.cursor.com/t/auto-review-sandbox-and-the-docker/164769/4?utm_source=chatgpt.com "Auto-review, sandbox and the Docker - #4 by deanrie - Help - Cursor - Community Forum"))

---

# 为什么不是 `Run Everything`？

你可能会想：

> 那我直接选 Run Everything，不就最方便？

确实，**从执行效率和避免 Sandbox 的角度，Run Everything 最彻底**。

但它相当于：

```text
Agent
  ↓
直接执行命令
  ↓
没有 Allowlist 限制
  ↓
没有人工确认
```

对于你的开发环境，我**不建议**。

比如 Agent 如果判断执行：

```powershell
git reset --hard
```

或者：

```powershell
Remove-Item ...
```

可能就直接执行了。

所以：

```text
Allowlist       ⭐⭐⭐⭐⭐ 推荐
Auto-Review     ⭐⭐⭐ 当前不适合你的 Go 场景
Run Everything  ⭐⭐ 不建议
```

---

# Allowlist 怎么配置才好？

你截图下面还有：

```text
Allowlist Options
You can configure Shell, MCP and Fetch allowlists for Auto mode.
```

这里你可以展开。

对于 Go 项目，我建议至少允许这些**只读/常用开发命令**：

```text
go
git
pwd
cd
dir
Get-ChildItem
Get-Content
Select-String
findstr
```

尤其是：

```text
go
```

这样 Agent 执行：

```powershell
go test ./...
go build ./...
go vet ./...
go fmt ./...
go mod tidy
go mod download
```

时，就可以直接运行。

---

# 但是这里有一个非常重要的区别

**Allowlist 下的命令应该是在 Host 上执行，而不是 Sandbox。**

所以你的目标应该变成：

```text
Cursor Agent
      │
      ▼
Allowlist
      │
      ▼
Windows Host
      │
      ├── GOROOT
      │    D:\Go\go1.25.12
      │
      ├── GOPATH
      │    D:\Go\workspace
      │
      ├── GOMODCACHE
      │    D:\Go\workspace\pkg\mod
      │
      └── GOCACHE
           D:\Go\cache\build
```

而不是现在：

```text
Cursor Agent
      │
      ▼
Auto-Review
      │
      ▼
Sandbox
      │
      ├── go-mod
      └── go-build
```

---

# 还有一个设置你最好一起检查

在：

```text
Cursor Settings
→ Agents
```

继续往下找：

```text
Inline Editing & Terminal
```

看看有没有：

```text
Legacy Terminal Tool
```

如果有，**建议打开**。

Cursor 官方社区目前针对 Windows Sandbox 的问题，明确建议：

> Switch from Sandbox to Allowlist，或者启用 Legacy Terminal Tool。

启用 Legacy Terminal Tool 后，Terminal 会直接使用 Host 环境，而不是 Sandbox 的那套环境。([Cursor - Community Forum](https://forum.cursor.com/t/windows-agent-shell-tool-returns-no-exit-status-and-produces-no-terminal-output-files-integrated-terminal-works-perfectly/163565?utm_source=chatgpt.com "[Windows] Agent Shell tool returns 'no exit status' and produces no terminal output files — integrated terminal works perfectly - Bug Reports - Cursor - Community Forum"))

所以你可以设置成：

```text
Run Mode
    ↓
Allowlist

Legacy Terminal Tool
    ↓
ON
```

这是我比较推荐你当前 Windows + Go 开发环境的组合。

---

# 改完后不要马上相信设置生效

你需要重新开一个 Agent 会话，然后让 Agent 执行：

```powershell
go env GOPATH
go env GOMODCACHE
go env GOCACHE
go env GOROOT
```

正常情况下应该变成：

```text
D:\Go\workspace
D:\Go\workspace\pkg\mod
D:\Go\cache\build
D:\Go\go1.25.12
```

**尤其看这两个：**

```text
GOMODCACHE
GOCACHE
```

不能再出现：

```text
C:\Users\aihelp\AppData\Local\Temp\cursor-sandbox-cache\
```

---

## 如果改成 Allowlist 后还是出现 Sandbox

这种情况也有可能。

Cursor 最近版本的 Run Mode 行为经历过变化，而且 Windows 上曾存在旧的 `Auto-Run in Sandbox` 设置残留在后台、但 UI 已经不再显示的情况。Cursor 官方社区对此有确认。([Cursor - Community Forum](https://forum.cursor.com/t/windows-agent-shell-tool-returns-no-exit-status-and-produces-no-terminal-output-files-integrated-terminal-works-perfectly/163565?utm_source=chatgpt.com "[Windows] Agent Shell tool returns 'no exit status' and produces no terminal output files — integrated terminal works perfectly - Bug Reports - Cursor - Community Forum"))

如果你改成：

```text
Allowlist
```

之后，Agent 仍然返回：

```text
GOMODCACHE=C:\Users\aihelp\AppData\Local\Temp\cursor-sandbox-cache\...
```

**先不要继续改 Go 配置。**

直接把：

```text
Settings → Agents → Inline Editing & Terminal
```

里面的 **Legacy Terminal Tool** 打开，然后**完全退出 Cursor，再重新启动**。

再测试：

```powershell
go env GOMODCACHE
go env GOCACHE
```

---

### 所以你现在就按这个顺序做

**第一步：**

```text
Run Mode
Auto-Review
     ↓
Allowlist
```

**第二步：**

找到：

```text
Agents
→ Inline Editing & Terminal
→ Legacy Terminal Tool
```

如果存在，打开。

**第三步：**

彻底退出 Cursor，再启动。

**第四步：**

让 Agent 执行：

```powershell
go env GOPATH
go env GOMODCACHE
go env GOCACHE
go env GOROOT
```

如果得到：

```text
D:\Go\workspace
D:\Go\workspace\pkg\mod
D:\Go\cache\build
D:\Go\go1.25.12
```

那么就成功了。

**另外，你现在截图中的 `Auto-Review` 不需要为了“更安全”而保留。对于你这个明确希望 Agent 使用本机 Go 环境、复用本机 Module/Build Cache 的场景，`Allowlist + Legacy Terminal Tool` 更合适。**