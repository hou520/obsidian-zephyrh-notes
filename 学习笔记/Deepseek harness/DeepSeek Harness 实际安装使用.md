npx @deepseek-ai/dsh web
和
git clone 从源码运行的区别：
版本来源
### npm 方式

你运行的是：

npm registry 中已经发布的版本

例如：

@deepseek-ai/dsh@1.2.3

具体版本取决于当前发布版本和 npm 的解析规则。

----
如果你的目的是：

> **研究 DeepSeek Harness 的源码，修改它，调试它，甚至给它增加功能**

那么才建议使用本地源码

| 项目               | `npx @deepseek-ai/dsh web` | 源码运行 |
| ---------------- | -------------------------- | ---- |
| Git              | ❌ 不需要                      | ✅ 需要 |
| pnpm             | ❌ 不需要                      | ✅ 需要 |
| 下载源码             | ❌                          | ✅    |
| `pnpm install`   | ❌                          | ✅    |
| `pnpm run build` | ❌                          | ✅    |
| 使用 npm 已发布版本     | ✅                          | ❌    |
| 使用 GitHub 最新源码   | ❌                          | ✅    |
| 修改源码             | ❌ 不方便                      | ✅    |
| 调试源码             | ❌                          | ✅    |
| 自定义功能            | ❌                          | ✅    |
| 使用简单程度           | ⭐⭐⭐⭐⭐                      | ⭐⭐⭐  |
| 适合普通使用           | ✅                          | ❌    |
| 适合开发             | ⚠️                         | ✅    |

----
**如果你只是偶尔使用 DeepSeek Harness，我反而建议继续用 `npx`；如果你会每天使用，建议全局安装。**

你现在已经验证：

```powershell
npx @deepseek-ai/dsh web
```

可以正常运行，那么可以根据使用频率选择。

### 方案一：继续用 `npx` —— 我更推荐

```powershell
npx @deepseek-ai/dsh web
```

优点：

- 不需要全局安装
    
- 不需要自己管理 dsh 版本
    
- npm package 更新后比较容易获取新版本
    
- 不污染全局 `node_global`
    
- 你当前 npm cache 已经迁移到 `D:\DevCache\npm`
    

缺点就是每次输入稍微长一点。

如果只是**偶尔打开 Harness**，没必要全局安装。

---

### 方案二：全局安装 —— 如果你以后经常用

你可以：

```powershell
npm install -g @deepseek-ai/dsh
```

你当前：

```powershell
npm config get prefix
```

已经是：

```text
D:\Fore\nodejs\node_global
```

所以这个包不会装到 C 盘，而会进入你的 D 盘全局 npm 环境。

安装后：

```powershell
dsh --version
```

然后：

```powershell
dsh web
```

以后就非常方便：

```powershell
dsh web
```

---

### 你的情况我会这样选

|使用场景|建议|
|---|---|
|偶尔测试一下|`npx`|
|每周用几次|`npx`|
|每天使用|**全局安装**|
|经常开发/调试 Harness|**源码方式**|
|修改 Harness 源码|源码方式|
|只想把 Web UI 当工具使用|**全局安装**|

如果你后面准备**把 DeepSeek Harness 当成日常开发工具长期使用**，那我建议你现在直接：

```powershell
npm install -g @deepseek-ai/dsh
```

然后验证：

```powershell
where.exe dsh
dsh --version
```

正常情况下 `dsh` 应该来自：

```text
D:\Fore\nodejs\node_global
```

之后直接：

```powershell
dsh web
```

即可。

**不过有一点需要注意：全局安装意味着版本由你本机管理。** 如果官方发布了新版本，不一定自动升级，需要你主动：

```powershell
npm update -g @deepseek-ai/dsh
```

所以如果你比较在意“每次都使用最新发布版本”，`npx` 其实更省心；如果你更在意**启动快、命令短、长期稳定使用**，全局安装更合适。