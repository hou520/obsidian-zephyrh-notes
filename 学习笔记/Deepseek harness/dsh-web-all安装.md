打开  
 `C:\Users\aihelp\AppData\Roaming\dsh-desktop\harness\profiles\web\pnpm-workspace.yaml`  
 改成：

```
allowBuilds:
  cloudflared: false
```

`false` 表示明确忽略 postinstall，pnpm 不会因下载失败整单失败。然后在 Desktop 里再装一次 `dsh-web-all`。

要用远程隧道：给能访问 GitHub 的代理，或先手动装好 `cloudflared.exe`，再保持 `cloudflared: true` 重试。

## 实际失败点

`cloudflared` 的 postinstall 会去 GitHub 拉二进制：

`https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe`

脚本已经获准执行（Desktop 的 `pnpm-workspace.yaml` 里是 `cloudflared: true`），但下载失败，整个安装 `exit=1`。国内访问 GitHub Releases 超时/被拦时很常见。

`@linxin666/dsh-web-all` 依赖远程 Web / 隧道（`dsh-remote-web-ui`），所以会带上这个包。