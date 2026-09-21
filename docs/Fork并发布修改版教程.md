# GitHub Fork 项目并发布修改版完整教程

## 一、整体流程概览

```
GitHub 原仓库 → Fork 到自己账号 → Clone 到本地 → 修改代码 → 提交 → Push 到自己仓库 → 使用
```

## 二、详细步骤

### 步骤 1：Fork 原仓库到自己的 GitHub 账号

1. 打开原项目地址：`https://github.com/soulqw/Curtain`
2. 点击右上角的 **Fork** 按钮
3. 选择你自己的 GitHub 账号作为目标
4. 等待 Fork 完成，你会得到一个 `https://github.com/你的用户名/Curtain` 仓库

> **为什么要 Fork？** Fork 会在你的账号下创建一个原仓库的完整副本，你可以自由修改而不影响原项目。

---

### 步骤 2：将你 Fork 的仓库 Clone 到本地

打开终端（Git Bash 或 CMD），执行：

```bash
# 进入你想存放项目的目录
cd D:/Android/workspace

# Clone 你自己 Fork 的仓库（注意替换成你的用户名）
git clone https://github.com/你的用户名/Curtain.git

# 进入项目目录
cd Curtain
```

> ⚠️ **注意**：一定要 Clone **你自己账号下**的仓库，而不是原作者的仓库，否则你没有 push 权限。

---

### 步骤 3：关联原仓库（可选但推荐）

为了后续能同步原作者的更新，添加一个 `upstream` 远程地址：

```bash
# 查看当前远程地址
git remote -v
# 你会看到 origin 指向你自己的仓库

# 添加原仓库为 upstream
git remote add upstream https://github.com/soulqw/Curtain.git

# 再次确认
git remote -v
# origin    https://github.com/你的用户名/Curtain.git (fetch)
# origin    https://github.com/你的用户名/Curtain.git (push)
# upstream  https://github.com/soulqw/Curtain.git (fetch)
# upstream  https://github.com/soulqw/Curtain.git (push)
```

**同步原仓库更新的方法**（以后用得到）：

```bash
# 拉取原仓库最新代码
git fetch upstream

# 合并到你的主分支
git checkout master
git merge upstream/master

# 推送到你自己的 GitHub
git push origin master
```

---

### 步骤 4：修改代码

根据 bug 分析，修改 `curtain/src/main/java/com/qw/curtain/lib/GuideDialogFragment.java` 中的 `onCreateDialog` 方法。

修改完成后，查看变更：

```bash
git status
git diff
```

---

### 步骤 5：提交修改

```bash
# 添加修改的文件
git add curtain/src/main/java/com/qw/curtain/lib/GuideDialogFragment.java

# 提交，写清楚修改说明
git commit -m "fix: 修复 Fragment 重建时 onCreateDialog 空指针崩溃

- 原因：Activity 重建（如屏幕旋转、进程被杀后恢复）时，
  GuideDialogFragment 被系统重新实例化，param 和 contentView 为 null
- 修复：onCreateDialog 中增加 null 防御，状态丢失时优雅 dismiss"
```

---

### 步骤 6：推送到你自己的 GitHub 仓库

```bash
git push origin master
```

如果是第一次 push，可能需要输入 GitHub 用户名和密码（现在密码需要用 **Personal Access Token**，见下文）。

> 🔑 **关于 GitHub 登录认证**：
> GitHub 已不再支持账号密码 push，需要用 **Personal Access Token (PAT)** 或 **SSH Key**。
>
> **推荐用 SSH（一次配置永久使用）：**
> ```bash
> # 生成 SSH Key（一路回车即可）
> ssh-keygen -t ed25519 -C "你的邮箱"
>
> # 查看公钥，复制内容
> cat ~/.ssh/id_ed25519.pub
> ```
> 然后到 GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴保存。
>
> 最后把远程地址改成 SSH 格式：
> ```bash
> git remote set-url origin git@github.com:你的用户名/Curtain.git
> ```

---

### 步骤 7：在你的项目中使用修改后的版本

有几种方式，按推荐程度排序：

#### 方式 A：发布到 JitPack（推荐，和原库使用方式一致）

1. 到 [jitpack.io](https://jitpack.io) 用 GitHub 账号登录
2. 输入你的仓库地址：`你的用户名/Curtain`，点击 Look up
3. 选择一个版本（可以用 commit hash 或 release 版本号），点击 Get it
4. 按 JitPack 页面提示，在你的项目中添加依赖：

```gradle
// 项目根目录 build.gradle
allprojects {
    repositories {
        // ...
        maven { url 'https://jitpack.io' }
    }
}

// app/build.gradle
dependencies {
    implementation 'com.github.你的用户名:Curtain:版本号'
}
```

> 💡 **版本号怎么来？** 你可以在 GitHub 上打一个 tag + release，比如 `v1.0.0-fix`：
> ```bash
> git tag v1.0.0-fix
> git push origin v1.0.0-fix
> ```
> 然后在 GitHub 仓库页面 → Releases → Create a new release → 选择这个 tag → 发布。
> 之后 JitPack 就能用 `v1.0.0-fix` 作为版本号了。

#### 方式 B：本地 module 依赖（调试阶段最方便）

把修改后的 `curtain` module 直接拷贝到你的项目中：

1. 将 `Curtain-master/curtain/` 文件夹复制到你项目的根目录
2. 在 `settings.gradle` 中添加：
   ```gradle
   include ':curtain'
   ```
3. 在 `app/build.gradle` 中添加依赖：
   ```gradle
   implementation project(':curtain')
   ```

#### 方式 C：用 JitPack + commit hash（无需打 tag）

```gradle
implementation 'com.github.你的用户名:Curtain:最新commit的hash值'
```

commit hash 可以在 `git log` 或 GitHub 仓库页面找到。

---

## 三、常见问题

### Q1：push 时提示 `permission denied`？
确认你 Clone 的是**自己账号下**的仓库，不是原作者的。可以用 `git remote -v` 检查。

### Q2：原作者更新了代码，我怎么同步？
参考步骤 3 中的 `git fetch upstream` + `git merge upstream/master`。

### Q3：JitPack 构建失败怎么办？
- 检查项目根目录是否有正确的 `build.gradle`
- 到 JitPack 对应的构建日志页面看具体错误
- 确保 library module 配置了 `android-library` 插件

### Q4：我想给原作者提 PR 怎么办？
在你自己的仓库修改并 push 后，到原仓库页面 → Pull requests → New pull request → 选择你的分支 → 填写描述 → 创建 PR。

---

## 四、快速命令速查表

| 操作 | 命令 |
|------|------|
| Fork | 在 GitHub 网页点 Fork 按钮 |
| 克隆自己的仓库 | `git clone https://github.com/你的用户名/Curtain.git` |
| 添加原仓库上游 | `git remote add upstream https://github.com/soulqw/Curtain.git` |
| 查看状态 | `git status` |
| 提交修改 | `git add . && git commit -m "描述"` |
| 推送到自己仓库 | `git push origin master` |
| 打 tag | `git tag v1.0.0-fix && git push origin v1.0.0-fix` |
| 同步原仓库更新 | `git fetch upstream && git merge upstream/master` |
