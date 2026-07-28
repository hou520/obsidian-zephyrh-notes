迁移前
```cmd

C:\Users\aihelp>dotnet nuget locals all --list
http-cache: C:\Users\aihelp\AppData\Local\NuGet\v3-cache
global-packages: C:\Users\aihelp\.nuget
temp: C:\Users\aihelp\AppData\Local\Temp\NuGetScratch
plugins-cache: C:\Users\aihelp\AppData\Local\NuGet\plugins-cache
```

对于 .NET 开发者来说：

```
C:\Users\aihelp\.nuget
```

通常是 C 盘占用空间最大的目录之一，几十 GB 甚至上百 GB 都很常见。

---

# 一、这个目录是什么？

`.nuget` 是 **NuGet（.NET 包管理器）** 的用户目录。

一般结构如下：

```
C:\Users\aihelp\.nuget
│
├── packages          ← 最大（几GB~几十GB）
│
├── plugins
│
├── NuGet
│     NuGet.Config
│
└── v3-cache          ← HTTP缓存
```

其中最重要的是：

## ① packages（最大的目录）

例如：

```
packages
    newtonsoft.json
    microsoft.extensions.logging
    grpc.core
    xunit
    stackexchange.redis
    ...
```

这里保存的是：

> **所有项目下载过的 NuGet 包。**

例如：

你的项目：

```
Project A
```

需要：

```
Newtonsoft.Json 13.0.3
```

下载后：

```
C:\Users\aihelp\.nuget\packages\
```

以后其它项目就不用重新下载了。

所以这里会越来越大。

---

## ② v3-cache

这里是：

NuGet HTTP 请求缓存。

可以删除。

重新 Restore 时会重新生成。

---

## ③ plugins

NuGet 插件。

通常很小。

---

# 二、可以迁移吗？

**完全可以。**

微软官方支持修改：

```
NUGET_PACKAGES
```

环境变量。

这是推荐方式。

---

# 三、推荐方法（官方支持）

例如：

准备：

```
D:\NuGetPackages
```

然后：

## 第一步

创建目录：

```
D:\NuGetPackages
```

---

## 第二步

复制过去

例如：

```
robocopy "C:\Users\aihelp\.nuget\packages" "D:\NuGetPackages" /E
```

或者直接资源管理器复制。

不要剪切。

---

## 第三步

添加环境变量

新增：

```
变量：

NUGET_PACKAGES
```

值：

```
D:\NuGetPackages
```

位置可以是：

用户变量（推荐）

或者：

系统变量。

---

## 第四步

重新打开：

Visual Studio

Cursor

Terminal

执行：

```
dotnet nuget locals all --list
```

会看到：

例如：

```
global-packages: D:\NuGetPackages
```

说明已经生效。

---

# 四、HTTP Cache 也可以迁移

还有：

```
NUGET_HTTP_CACHE_PATH
```

例如：

```
D:\NuGetHttpCache
```

不过一般：

HTTP Cache 不大。

迁移意义没有 Packages 大。

---

# 五、如何查看当前 NuGet 使用的位置？

执行：

```
dotnet nuget locals all --list
```

例如：

```
http-cache:
C:\Users\aihelp\AppData\Local\NuGet\v3-cache

global-packages:
C:\Users\aihelp\.nuget\packages

temp:
C:\Users\aihelp\AppData\Local\Temp\NuGetScratch

plugins-cache:
C:\Users\aihelp\AppData\Local\NuGet\plugins-cache
```

迁移后：

```
global-packages:
D:\NuGetPackages
```

---

# 六、迁移完成后可以删除旧目录吗？

可以。

确认：

```
dotnet restore
```

正常。

Visual Studio 能编译。

然后：

删除：

```
C:\Users\aihelp\.nuget\packages
```

即可。

其它：

```
plugins
NuGet.Config
```

建议保留。

---

# 七、还有一种方法（不太推荐）

使用：

Windows：

```
mklink /J
```

例如：

```
mklink /J "C:\Users\aihelp\.nuget\packages" "D:\NuGetPackages"
```

这样：

程序仍认为：

```
C:\Users\aihelp\.nuget\packages
```

存在。

实际上数据在：

```
D:\NuGetPackages
```

优点：

几乎所有工具都兼容。

缺点：

以后维护稍微麻烦一点。

现在微软推荐直接使用：

```
NUGET_PACKAGES
```

所以没必要再用 Junction。

---

# 八、结合你的开发环境，我建议统一迁移这些缓存

根据我们之前的交流，你使用 **Go、.NET、Java、Node.js、Cursor** 开发，建议把所有大型缓存统一放到 D 盘，例如：

|缓存类型|默认位置|建议迁移位置|是否推荐|
|---|---|---|---|
|NuGet Packages|`C:\Users\aihelp\.nuget\packages`|`D:\DevCache\NuGet\packages`|⭐⭐⭐⭐⭐|
|Go Module Cache|`go env GOMODCACHE`（通常在 `GOPATH\pkg\mod`）|`D:\DevCache\Go\pkg\mod`|⭐⭐⭐⭐⭐|
|npm Cache|用户缓存目录|`D:\DevCache\npm`|⭐⭐⭐⭐☆|
|Gradle Cache|`C:\Users\aihelp\.gradle`|`D:\DevCache\Gradle`|⭐⭐⭐⭐☆|
|Maven Repository|`C:\Users\aihelp\.m2\repository`|`D:\DevCache\Maven\repository`|⭐⭐⭐⭐☆|

统一管理后，重装系统或清理 C 盘都会方便很多，也能显著减少系统盘的占用。