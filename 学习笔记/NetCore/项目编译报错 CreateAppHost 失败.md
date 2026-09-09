已经亲自验证：
1、AspectInjector 依赖包降级到 2.8.2 可以解决并行报错问题；
2、添加环境变量 `AspectInjector_Enabled=false`（本机环境变量），能过编译，功能缺一块 HeapMap


-----------------
这三条报错是**同一条构建链上的根因 + 级联**，不是三个独立业务编译错误。根因在 **AspectInjector 2.9.0 的织入任务** 和 **UnitTests 过大的项目引用图**；后两条「未能找到元数据文件」是上游没产出 `obj\...\ref\*.dll` 后的连锁失败。

---

## 错误分别是什么

| 报错 | 含义 |
|---|---|
| `Reference not found: ...\AIHelp.Internal.Domain.dll` + 指向 aspect-injector | **织入器**在解析程序集时去 `bin\Debug\net6.0\` 找**实现 DLL**，文件不存在或尚未拷过去 |
| `未能找到元数据文件 ...\obj\Debug\net6.0\ref\AIHelp.ETLClickHouse.dll` | CSC 要引用程序集，但这两个工程**没有成功产出** SDK 的 ref DLL |
| 同上，`ThirdPartyWebHookPush.Consumer.dll` | 同上 |

后两条几乎总是：**被引用项目先失败（或没编完）→ 测试项目再报找不到元数据**。真正要看的是第一条，以及那两个 `OutputType=Exe` 工程为何没产出 ref。

---

## 根因

### 1. AspectInjector 挂在 Common 上，织入任务会扩散到几乎整棵依赖树

`AIHelp.Common.csproj` 是全仓库**唯一**的 AspectInjector 引用：

```29:29:AIHelp.Common/AIHelp.Common.csproj
    <PackageReference Include="AspectInjector" Version="2.9.0" />
```

实际织入代码（`[Aspect]` / `[Injection]` / `[HeatMap]`）在 `AIHelp.WebApiFramework` 和 Elva 等 Web 控制器里，**不在 Common、也不在 Internal.Domain**。

这个包默认会带上 `build` + analyzer，且**没有** `PrivateAssets="all"`。凡是引用 Common 的项目（Core、Internal.Domain、两个 Consumer Exe、UnitTests 自己……）都会跑同一套 **2.9.0 MSBuild 织入任务**，哪怕项目里一个 Aspect 都没有。

Internal.Domain 自己很干净，只引用了 Application：

```1:12:Hosts/InternalHost/AIHelp.Internal.Domain/AIHelp.Internal.Domain.csproj
<Project Sdk="Microsoft.NET.Sdk">
  ...
    <ProjectReference Include="..\..\..\Application\AIHelp.CommonApplication\AIHelp.CommonApplication.csproj" />
    <ProjectReference Include="..\..\..\Application\AIHelp.TicketApplication\AIHelp.TicketApplication.csproj" />
```

它仍会间接触发织入器，于是去找自己的  
`...\AIHelp.Internal.Domain\bin\Debug\net6.0\AIHelp.Internal.Domain.dll`。

### 2. 2.9.0 织入器认 `bin\`，SDK 编译认 `obj\...\ref\`

.NET 6 SDK 给下游的是引用程序集：

`obj\Debug\net6.0\ref\Xxx.dll`

实现程序集之后才拷到：

`bin\Debug\net6.0\Xxx.dll`

AspectInjector 2.9.0 起改成 **MSBuild Task 织入**（不再是旧的 compiler 插件），解析的是 **bin 里的实现 DLL**。并行生成时常见两种情况：

- 下游（Internal.WebApi / UnitTests）已经开始织入，Internal.Domain 的 bin 还没拷好  
- Clean / 失败残留后 bin 根本不存在，织入器按 `ReferencePath` 去打开就报 `Reference not found`

这和官方 issue 文案一致（[aspect-injector #131](https://github.com/pamidur/aspect-injector/issues/131) 等同款 “Reference not found … Please submit an issue”）。

### 3. 两个失败工程都是 Exe，踩中 2.9.0 的已知缺陷

`AIHelp.ETLClickHouse`、`AIHelp.ThirdPartyWebHookPush.Consumer` 都是：

- `<OutputType>Exe</OutputType>`
- 直接引用 Common → 继承 2.9.0 织入任务

官方未关闭缺陷：[aspect-injector #244](https://github.com/pamidur/aspect-injector/issues/244)  
**2.9.0 + 控制台 Exe**：织入任务锁住中间 DLL / apphost，`CreateAppHost` 失败。2.8.2 无此问题。Exe 一失败就不会写 `obj\...\ref\*.dll`，UnitTests 就会报你看到的两条「未能找到元数据文件」。

### 4. 为什么偏偏是 UnitTests「启动生成」爆

`AIHelp.UnitTests.csproj` 一次拉了 25+ 个 ProjectReference，其中包括：

- `AIHelp.WebElva.WebApi`（大量 `[HeatMap]`，必须织入）
- `AIHelp.Internal.WebApi` + `AIHelp.Internal.Domain`
- 两个 Exe：`ETLClickHouse`、`ThirdPartyWebHookPush.Consumer`

测试代码里**没有** `using AIHelp.Internal.Domain`，Internal.Domain 对测试是多余的直接引用（Internal.WebApi 已经引用它）。  
VS「启动生成」会按**整棵图并行编**，把 weaver 的 bin 解析竞态和 Exe 文件锁放大；单独编某个 Web/API 时依赖少、顺序碰巧对，往往就不报。

---

## 因果链（启动生成 UnitTests 时）

```text
UnitTests 并行拉起整棵图
        │
        ├─ Common 的 AspectInjector 2.9.0 织入任务扩散到下游
        │
        ├─ 织入 Internal.Domain / Internal.WebApi / UnitTests
        │     → 去 bin 找 Internal.Domain.dll
        │     → 文件未就绪 → Reference not found（根因报错）
        │
        └─ 织入两个 Exe（或它们依赖失败）
              → 2.9.0 锁 DLL / CreateAppHost 失败
              → 不产出 obj\...\ref\*.dll
              → UnitTests：未能找到元数据文件（级联）
```

---

## 解决方案（不改仓库代码）

按优先级，都可在本机/VS 里做，无需改 csproj：

**1. 先编出缺失 DLL，再编测试（最快验证）**

在解决方案里**按顺序单独生成**（不要一开始就生成 UnitTests）：

1. `AIHelp.Common`  
2. `AIHelp.Internal.Domain`  
3. `AIHelp.ETLClickHouse`  
4. `AIHelp.ThirdPartyWebHookPush.Consumer`  
5. 再生成 `AIHelp.UnitTests`

若第 3/4 步仍失败，基本就坐实是 2.9.0 + Exe 的 #244，而不是测试代码问题。

**2. 关掉并行生成**

VS：`工具 → 选项 → 项目和解决方案 → 生成并运行` → 最大并行项目数设为 **1**。  
命令行等价：`msbuild /m:1`。用时间换稳定，避开 weaver 抢 bin。

**3. 清残留后再按依赖从下往上编**

删掉这三个目录的 `bin`、`obj`（以及必要时 Common 的）：

- `Hosts\InternalHost\AIHelp.Internal.Domain\`
- `AIHelp.ETLClickHouse\`
- `AIHelp.ThirdPartyWebHookPush.Consumer\`

不要用「启动 UnitTests」当启动项目去整树 Rebuild。跑测试用测试资源管理器，或 `dotnet test --filter` 指定类。

**4. 看完整生成输出，不要只看最后三条**

把输出窗口详细程度开到「详细」，搜：

- `AspectInjector`
- `CreateAppHost`
- `being used by another process`

若 Exe 上还有文件锁，就和 #244 对上了；后面的「元数据文件」可以忽略。

---

## 若以后允许改工程（建议，本次未改）

1. **收窄织入范围（最根本）**  
   Common 上：`PrivateAssets="all"`，只保留 Broker 给真正织入的项目；`PackageReference` 只加在 `AIHelp.WebApiFramework` / 带 `[HeatMap]` 的 Web 工程。Internal.Domain 和两个 Exe 不再跑 weaver。

2. **降级到 2.8.2**  
   避开 2.9.0 的 MSBuild Task / Exe 文件锁（社区对 #244 的常规处理）。

3. **测试不要直接引用 Exe**  
   把 ETL / WebHook 可测逻辑抽到 class library，UnitTests 只引用库。

4. **去掉 UnitTests 对 Internal.Domain 的多余 ProjectReference**  
   测试只用 Internal.WebApi 即可。

---

**有影响**：现有「Common 全局带 AspectInjector 2.9.0 + UnitTests 引用整棵宿主/Exe 图」会让测试工程启动生成不稳定，并行或 Clean 后更容易在 Internal.Domain 织入和两个 Exe 的 ref 产出上失败。  
**没有影响**：本次只做只读分析，未改代码、未在本地编译运行。

----
**并行数改成 1 无效是正常的。** 这次不是项目之间抢文件，而是**同一个 Web/Exe 工程内部**：2.9.0 织入任务还占着 `obj\...\Xxx.dll`，紧接着 SDK 的 `CreateAppHost` 要读同一份 DLL 去做原生宿主，于是报「正由另一进程使用」。

`AIHelp.ThirdPartyCommon.WebApi` 是 `Microsoft.NET.Sdk.Web`，一定会跑 `CreateAppHost`。`AIHelp.UnitTests` 直接引用了它，所以只要生成测试项目，这条路径就会走到。2.8.2 走旧织入方式、不长时间锁 DLL，所以一次能过。

保持 2.9.0、又不改仓库代码，只有下面几条路，**没有和降级一样稳的办法**。

---

## 1. 本机环境变量关掉 AppHost（最贴近这条报错）

用户/系统环境变量：

```text
UseAppHost=false
```

改完后**重开 VS**（MSBuild 只在进程启动时读环境变量）。

效果：跳过 `CreateAppHost`，织入仍会跑，`[HeatMap]` 还在。  
代价：这些 Web/Exe 的 `bin` 里不再生成 `Xxx.exe`，本地用 `dotnet run` / VS 调试 Web 一般仍可用；若你习惯双击 exe 或做自包含发布，会受影响。只在本机设即可，不要带到发布机。

---

## 2. 本机环境变量关掉织入

```text
AspectInjector_Enabled=false
```

同样要重开 VS。

效果：2.9.0 Weaver 不跑，锁文件消失。  
代价：本机编出来的 Elva **不会织入热力图**，行为采集在本地是空的。测 HeatMap 织入时不能用这个。

---

## 3. 操作上减轻，治不了根

- 不要把 UnitTests 当启动项目去「启动生成」整棵图；用测试资源管理器跑指定类。只要点了生成 UnitTests，仍会编 `ThirdPartyCommon.WebApi`。
- 官方 #244 现象：第一次失败、再编一次有时能过。不稳定，不适合当日常流程。
- 杀残留 `testhost` / 旧站点进程：对「同一次生成里 Weaver 锁 DLL」基本没用。

---

## 对照

| 做法 | 改仓库？ | 保 2.9.0？ | HeatMap 织入 | 稳定性 |
|---|---|---|---|---|
| 降级 2.8.2 | 改 csproj | 否 | 有 | 你已验证，最稳 |
| `UseAppHost=false`（本机环境变量） | 否 | 是 | 有 | 能避开这条 CreateAppHost |
| `AspectInjector_Enabled=false`（本机环境变量） | 否 | 是 | 本地无 | 能过编译，功能缺一块 |
| 并行数 = 1 | 否 | 是 | 有 | **无效**（同项目内锁） |

---

**结论：** 不改代码又坚持 2.9.0，只能在本机用 `UseAppHost=false`（保埋点）或 `AspectInjector_Enabled=false`（弃本地织入）。要织入和 AppHost 都正常，目前仍只有降级 2.8.2，或以后改工程把 Weaver 限制在真正带 `[HeatMap]` 的项目上。上游 [issue #244](https://github.com/pamidur/aspect-injector/issues/244) 还开着。

**没有影响：** 只说明可选做法，未改仓库代码。