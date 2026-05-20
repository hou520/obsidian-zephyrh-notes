下面给你一套 **Windows + Obsidian + GitHub Token + 自动同步** 的完整流程。  
按这个做，最后效果就是：

> ✅ Obsidian 自动提交  
> ✅ 自动 push 到 GitHub  
> ✅ 多电脑同步  
> ✅ 不再需要输密码

---

# 一、准备工作

需要：

|工具|作用|
|---|---|
|Git|Git 命令|
|GitHub 账号|远程仓库|
|Obsidian|笔记|
|Obsidian Git 插件|自动同步|

---

# 二、安装 Git

下载安装：

## [Git for Windows](https://git-scm.com/download/win?utm_source=chatgpt.com)

安装完成后打开 CMD：

```
git --version
```

看到版本号即可。

---

# 三、创建 GitHub 仓库

打开：

## [GitHub](https://github.com?utm_source=chatgpt.com)

点击：

```
New repository
```

例如：

```
obsidian-notes
```

推荐：

|选项|建议|
|---|---|
|Private|✅|
|Add README|❌ 不勾|

创建完成。

---

# 四、生成 GitHub Token（最关键）

打开：

## [GitHub Token 页面](https://github.com/settings/tokens?utm_source=chatgpt.com)

---

## 1️⃣ 进入 Fine-grained tokens

点击：

```
Fine-grained personal access tokens
```

然后：

```
Generate new token
```

---

## 2️⃣ 填写基本信息

### Token name

例如：

```
obsidian-sync
```

---

### Expiration

建议：

```
No expiration
```

或者：

```
90 days
```

---

## 3️⃣ Repository access（重点）

选择：

```
Only select repositories
```

然后勾选：

```
obsidian-notes
```

---

## 4️⃣ Repository permissions（最关键）

找到：

# Contents

设置为：

```
Read and write
```

其他都不用改。

---

## 5️⃣ 生成 Token

点击：

```
Generate token
```

会得到：

```
github_pat_xxxxxxxx
```

⚠️ 只显示一次，立刻保存。

---

# 五、创建 Obsidian Vault

例如：

```
D:\Obsidian\MyNotes
```

在 Obsidian 中：

```
Open folder as vault
```

---

# 六、初始化 Git 仓库

打开 CMD：

```
cd D:\Obsidian\MyNotes
```

执行：

```
git init
```

---

# 七、关联 GitHub 仓库

复制 GitHub 仓库地址：

例如：

```
https://github.com/你的用户名/obsidian-notes.git
```

执行：

```
git remote add origin https://github.com/你的用户名/obsidian-notes.git
```

验证：

```
git remote -v
```

---

# 八、配置 Git 用户信息

执行：

```
git config --global user.name "你的GitHub用户名"
```

```
git config --global user.email "你的GitHub邮箱"
```

---

# 九、让 Git 记住 Token（推荐）

执行：

```
git config --global credential.helper manager
```

---

# 十、第一次提交

执行：

```
git add .
```

```
git commit -m "init vault"
```

---

# 十一、推送到 GitHub（第一次会要求认证）

执行：

```
git branch -M main
```

```
git push -u origin main
```

---

# 十二、输入 GitHub Token

会弹：

## Username

输入：

```
你的 GitHub 用户名
```

---

## Password

⚠️ 这里：

> ❌ 不是 GitHub 密码  
> ✅ 粘贴刚才生成的 Token

例如：

```
github_pat_xxxxxxx
```

---

# 十三、成功后效果

以后：

```
git push
```

不会再要求输入密码。

---

# 十四、安装 Obsidian Git 插件（自动同步）

在 Obsidian：

```
设置 → 社区插件 → 关闭安全模式
```

搜索：

```
Obsidian Git
```

安装：

### Obsidian Git

---

# 十五、推荐插件配置（非常实用）

进入插件设置：

---

## 自动备份

### Vault backup interval

例如：

```
5
```

表示：

```
每 5 分钟自动 commit
```

---

## 自动 push

开启：

```
Push on backup
```

---

## 自动 pull

开启：

```
Pull updates on startup
```

---

# 十六、添加 .gitignore（推荐）

创建文件：

```
.gitignore
```

内容：

```
.obsidian/workspace.json.obsidian/cache.trash/
```

---

# 十七、以后日常使用方式

## 单电脑

基本什么都不用管：

```
写笔记 → 自动 commit → 自动 push
```

---

## 多电脑

打开前：

```
git pull
```

关闭前：

```
git push
```

或者直接让插件自动处理。

---

# 十八、最容易踩坑的问题

## ❌ git push 403

原因：

```
Token 权限不对
```

必须：

```
Contents → Read and write
```

---

## ❌ push 要密码

原因：

```
没用 token
```

GitHub 已经不支持账号密码。

---

## ❌ Obsidian Git 不自动 push

检查：

```
Push on backup
```

是否开启。

---

# 十九、一句话总结

> GitHub Token 的核心就是：给仓库 `Contents → Read and write` 权限，然后把 Token 当 Git 密码使用，最后配合 Obsidian Git 插件实现自动同步。