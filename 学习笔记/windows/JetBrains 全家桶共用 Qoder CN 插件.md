下面是一份按你**当前机器实际路径**整理的完整方案，可直接按步骤执行。

---

# JetBrains 全家桶共用 Qoder CN 插件 — 完整方案

## 一、目标

- 全机只保留 **一份** Qoder CN JetBrains 插件（约 287 MB）
- PyCharm / GoLand / Rider / WebStorm 等通过 **junction** 共用
- **不**共用各 IDE 的 `config` / `system`（避免配置冲突）
- **不**动 `D:\JetBrains\plugins\.lingma\qoder-cn`（那是运行时数据，不是插件本体）

---

## 二、目录职责（先搞清楚）

```text
【要共享的 — JetBrains 插件本体】
D:\JetBrains\shared-plugins\qoder-cn-jetbrains\   ← 推荐最终放这里（唯一实体）
    └── lib\*.jar

【各 IDE 的 plugins 目录 — 只放 junction 链接】
D:\JetBrains\UserData\<产品>\config\plugins\qoder-cn-jetbrains  → 指向上面的实体

【不要 junction、不要删 — Qoder 运行时数据】
D:\JetBrains\plugins\.lingma\qoder-cn\            ← cache / index / logs 等

【各 IDE 独立 — 不要共享】
D:\JetBrains\UserData\<产品>\config\               ← 设置、键位等
D:\JetBrains\UserData\<产品>\system\              ← 索引、缓存等
```

---

## 三、你当前的 IDE 清单

| IDE | 插件目录 (`idea.plugins.path`) | 纳入共享 |
|---|---|---|
| IntelliJ IDEA 2024.2 | `D:\JetBrains\UserData\IntelliJIdea2024.2\config\plugins` | ✅ 已安装，作源 |
| PyCharm 2024.2 | `D:\JetBrains\UserData\PyCharm2024.2\config\plugins` | ✅ |
| GoLand 2024.3 | `D:\JetBrains\UserData\GoLand2024.3\config\plugins` | ✅ |
| Rider 2024.2 | `D:\JetBrains\UserData\Rider2024.2\config\plugins` | ✅ |
| WebStorm 2024.2 | `D:\JetBrains\UserData\WebStorm2024.2\config\plugins` | ✅ |
| DataGrip 2025.2 | `D:\JetBrains\UserData\DataGrip 2025.2.4\config\plugins` | ⚠️ 版本不同，单独试 |

---

## 四、执行步骤

### 阶段 0：准备

1. **完全退出**所有 JetBrains IDE（任务管理器确认无 `java.exe` / `idea64.exe` 等）
2. 以**管理员身份**打开 PowerShell（`mklink` 有时需要）
3. 确认 IDEA 里 Qoder 已能正常使用

---

### 阶段 1：把插件抽到独立共享目录（推荐）

当前插件在 IDEA 目录下，建议先抽到统一位置，再让所有 IDE（含 IDEA）都 junction 过来。

```powershell
# 变量定义
$SharedPlugin = "D:\JetBrains\shared-plugins\qoder-cn-jetbrains"
$IdeaPlugin   = "D:\JetBrains\UserData\IntelliJIdea2024.2\config\plugins\qoder-cn-jetbrains"

# 1. 创建共享目录
New-Item -ItemType Directory -Force -Path "D:\JetBrains\shared-plugins" | Out-Null

# 2. 若共享目录还没有实体，从 IDEA 移过去
if (-not (Test-Path $SharedPlugin)) {
    if (Test-Path $IdeaPlugin) {
        $item = Get-Item $IdeaPlugin -Force
        if ($item.LinkType -eq "Junction") {
            # 已是 junction：先删链接，再移动目标（少见）
            cmd /c rmdir "$IdeaPlugin"
            Move-Item $item.Target $SharedPlugin
        } else {
            Move-Item $IdeaPlugin $SharedPlugin
        }
        Write-Host "已移动插件到共享目录"
    } else {
        Write-Error "找不到 IDEA 中的插件目录，请先确认 Qoder 已安装"
        exit 1
    }
}

# 3. IDEA 建 junction（若尚不存在）
if (-not (Test-Path $IdeaPlugin)) {
    cmd /c mklink /J "$IdeaPlugin" "$SharedPlugin"
    Write-Host "IDEA junction 已创建"
} else {
    $item = Get-Item $IdeaPlugin -Force
    if ($item.LinkType -ne "Junction") {
        Write-Warning "IDEA 下仍是实体目录，请手动检查是否重复"
    } else {
        Write-Host "IDEA junction 已存在"
    }
}
```

**验证：**

```powershell
cmd /c "dir /AL D:\JetBrains\UserData\IntelliJIdea2024.2\config\plugins"
# 应看到 qoder-cn-jetbrains <JUNCTION>
```

---

### 阶段 2：给其他 IDE 建 junction

```powershell
$SharedPlugin = "D:\JetBrains\shared-plugins\qoder-cn-jetbrains"

$targets = @(
    "D:\JetBrains\UserData\PyCharm2024.2\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\GoLand2024.3\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\Rider2024.2\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\WebStorm2024.2\config\plugins\qoder-cn-jetbrains"
)

foreach ($link in $targets) {
    $parent = Split-Path $link -Parent
    if (-not (Test-Path $parent)) {
        New-Item -ItemType Directory -Force -Path $parent | Out-Null
    }
    if (Test-Path $link) {
        Write-Host "跳过（已存在）: $link"
    } else {
        cmd /c mklink /J "$link" "$SharedPlugin"
        Write-Host "已创建: $link"
    }
}
```

**DataGrip（可选，单独试）：**

```powershell
$dgLink = "D:\JetBrains\UserData\DataGrip 2025.2.4\config\plugins\qoder-cn-jetbrains"
if (-not (Test-Path $dgLink)) {
    cmd /c mklink /J "$dgLink" "D:\JetBrains\shared-plugins\qoder-cn-jetbrains"
}
```

> DataGrip 是 **2025.2**，其余多为 **2024.2/2024.3**。若启动报插件不兼容，删掉 DataGrip 的 junction，单独用 zip 安装对应版本。

---

### 阶段 3：各 IDE 启用插件

对每个 IDE（一次开一个）：

1. 启动 IDE  
2. **Settings → Plugins → Installed**  
3. 找到 **Qoder CN (Formerly Lingma)**，勾选 **Enable**  
4. **Restart IDE**  
5. 若列表里没有：**File → Invalidate Caches → Invalidate and Restart**

---

### 阶段 4：登录 Qoder CN

每个 IDE 首次需要登录（账号相同，登录态可能不自动同步）：

1. 右侧边栏点 **Qoder CN**（没有则右键侧边栏勾选）  
2. 点 **登录**  
3. 浏览器完成授权  
4. 回 IDE 确认已连接  

登录方式取决于你的订阅：

- **Qoder CN 全家桶** → `qoder.com.cn` 账号  
- **原灵码线** → 阿里云账号  

---

### 阶段 5：功能验证清单

在每个 IDE 里确认：

- [ ] Plugins 中 Qoder CN 为 **Enabled**  
- [ ] 侧边栏 Qoder 面板可打开  
- [ ] 能发起一次简单对话  
- [ ] 打开一个项目，无插件报错  

---

## 五、日常维护规则

| 场景 | 做法 |
|---|---|
| **更新 Qoder** | 只在 **IDEA** 里更新；更新前关闭其他 JetBrains IDE |
| **新装 JetBrains IDE** | 在新 IDE 的 `config\plugins` 下对 `shared-plugins\qoder-cn-jetbrains` 建 junction |
| **卸载 Qoder** | 先关所有 IDE，删各 IDE 下的 junction（`rmdir` 链接名），再删 `shared-plugins\qoder-cn-jetbrains` |
| **清理缓存** | 可删 `D:\JetBrains\plugins\.lingma\qoder-cn\cache` 等；**不要删** `shared-plugins` 里的 `lib` |
| **磁盘占用** | 插件约 287 MB × 1 份；运行时数据另计，通常较小 |

---

## 六、一键脚本（阶段 1 + 2 合并）

保存为 `setup-qoder-shared.ps1`，**管理员 PowerShell**、**所有 IDE 已关闭** 后执行：

```powershell
#Requires -RunAsAdministrator
$ErrorActionPreference = "Stop"

$SharedPlugin = "D:\JetBrains\shared-plugins\qoder-cn-jetbrains"
$IdeaPlugin   = "D:\JetBrains\UserData\IntelliJIdea2024.2\config\plugins\qoder-cn-jetbrains"

function New-JunctionIfMissing([string]$Link, [string]$Target) {
    if (Test-Path $Link) {
        Write-Host "[跳过] $Link"
        return
    }
    $parent = Split-Path $Link -Parent
    New-Item -ItemType Directory -Force -Path $parent | Out-Null
    cmd /c mklink /J "$Link" "$Target" | Out-Null
    Write-Host "[创建] $Link -> $Target"
}

# 阶段 1：归集到共享目录
New-Item -ItemType Directory -Force -Path "D:\JetBrains\shared-plugins" | Out-Null

if (-not (Test-Path $SharedPlugin)) {
    if (-not (Test-Path $IdeaPlugin)) {
        throw "未找到 IDEA 插件目录: $IdeaPlugin"
    }
    $item = Get-Item $IdeaPlugin -Force
    if ($item.LinkType -eq "Junction") {
        cmd /c rmdir "$IdeaPlugin"
        Move-Item $item.Target $SharedPlugin
    } else {
        Move-Item $IdeaPlugin $SharedPlugin
    }
    Write-Host "[移动] 插件 -> $SharedPlugin"
}

New-JunctionIfMissing $IdeaPlugin $SharedPlugin

# 阶段 2：其他 IDE
@(
    "D:\JetBrains\UserData\PyCharm2024.2\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\GoLand2024.3\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\Rider2024.2\config\plugins\qoder-cn-jetbrains",
    "D:\JetBrains\UserData\WebStorm2024.2\config\plugins\qoder-cn-jetbrains"
) | ForEach-Object { New-JunctionIfMissing $_ $SharedPlugin }

Write-Host ""
Write-Host "完成。请依次打开各 IDE：启用插件 -> 登录 Qoder CN"
Write-Host "DataGrip 2025.2 请单独测试后再决定是否 junction。"
```

---

## 七、常见问题

| 现象 | 处理 |
|---|---|
| `mklink` 拒绝访问 | 用管理员 PowerShell；确认目标 IDE 已完全退出 |
| Plugins 里看不到 Qoder | 检查 junction 目录名是否为 `qoder-cn-jetbrains`；Invalidate Caches |
| 插件启用后启动失败 | 看 `Help → Show Log in Explorer`；DataGrip 先单独排除版本问题 |
| 更新后其他 IDE 异常 | 关闭全部 IDE 后只开 IDEA 更新，再逐个重启其他 IDE |
| 和 `D:\JetBrains\plugins` 搞混 | 那是 Qoder **运行时数据**，与 junction 无关，一般不用管 |

---

## 八、最终目录结构（目标状态）

```text
D:\JetBrains\
├── shared-plugins\
│   └── qoder-cn-jetbrains\          ← 唯一实体（~287 MB）
│       └── lib\*.jar
│
├── plugins\
│   └── .lingma\qoder-cn\          ← 运行时数据（自动生成，不共享）
│
└── UserData\
    ├── IntelliJIdea2024.2\config\plugins\qoder-cn-jetbrains  → junction
    ├── PyCharm2024.2\config\plugins\qoder-cn-jetbrains       → junction
    ├── GoLand2024.3\config\plugins\qoder-cn-jetbrains        → junction
    ├── Rider2024.2\config\plugins\qoder-cn-jetbrains         → junction
    ├── WebStorm2024.2\config\plugins\qoder-cn-jetbrains       → junction
    └── DataGrip 2025.2.4\config\plugins\qoder-cn-jetbrains  → junction（可选）
```

---

## 九、建议执行顺序（今天）

1. 跑 **阶段 1** 脚本（或一键脚本）  
2. 打开 **IDEA**，确认 Qoder 仍正常  
3. 依次打开 **PyCharm → Rider → GoLand → WebStorm**，启用并登录  
4. 最后试 **DataGrip**；不兼容则跳过  
5. 以后只在 **IDEA** 里更新 Qoder  

如果你希望，我可以把上面的 `setup-qoder-shared.ps1` 直接写到你的项目或 `D:\JetBrains\` 下，你本地双击/执行即可。