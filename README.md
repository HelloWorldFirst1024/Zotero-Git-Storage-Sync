# Zotero Git Storage Sync 使用说明

> 一个用于将 Zotero 本地 `storage/` 附件目录同步到 Git 远程仓库的 Zotero 7 插件。  
> 推荐搭配 Zotero 官方数据同步使用：Zotero 同步文献条目、笔记、标签、分类；本插件通过 Git 同步 PDF 附件。

---

## 1. 插件用途

Zotero 默认的云端附件存储空间有限。如果文献库中 PDF 很多，`storage/` 文件夹会很快变大。本插件的目标是：

```text
Zotero 官方同步：文献条目、笔记、标签、分类、引用信息
Git 远程仓库：PDF 附件与补充材料，即 Zotero 数据目录下的 storage/ 文件夹
```

也就是说，本插件**只同步 Zotero 数据目录中的 `storage/` 文件夹**，不同步 Zotero 数据库文件。

插件不会同步：

```text
zotero.sqlite
zotero.sqlite-journal
cache/
locate/
styles/
translators/
插件配置目录
```

这样可以避免直接用 Git 同步 Zotero 数据库造成数据库冲突或损坏。

---

## 2. 重要提醒

### 2.1 不要把整个 Zotero 数据目录都提交到 Git

Zotero 的条目、笔记、标签、分类等信息主要保存在：

```text
~/Zotero/zotero.sqlite
```

这个文件是 SQLite 数据库，不适合用 Git 多端同步。多台电脑同时修改数据库后，Git 无法像合并文本文件一样合并数据库内容，容易造成冲突甚至损坏。

因此，本插件只同步：

```text
~/Zotero/storage/
```

### 2.2 建议使用私有仓库

PDF 附件可能涉及版权、批注、下载来源或个人阅读记录，强烈建议把 GitHub 仓库设置为 **Private**。

### 2.3 建议只同步 100 MiB 以下文件

GitHub 对单个文件有大小限制：超过 50 MiB 会警告，超过 100 MiB 会拒绝 push。插件会尽量检查接近或超过 100 MiB 的文件，但仍建议用户自行控制附件大小。

---

## 3. 使用前准备

### 3.1 系统与软件

当前文档以 macOS + Zotero 7 为例，已知测试环境：

```text
macOS
Zotero 7.0.x
GitHub HTTPS remote
GitHub Personal Access Token
```

需要提前安装 Git。打开终端检查：

```bash
git --version
```

如果没有 Git，可以在 macOS 终端中执行：

```bash
xcode-select --install
```

### 3.2 备份 Zotero 附件目录

第一次使用前，建议先备份 `storage/`：

```bash
cp -R ~/Zotero/storage ~/Desktop/storage-backup
```

确认 GitHub 同步成功后，再自行决定是否删除备份。

---

## 4. Zotero 同步设置

推荐使用以下同步结构：

```text
Zotero 官方账号：同步文献数据、笔记、标签、分类
本插件 + Git：同步 PDF 附件
```

在 Zotero 中进入：

```text
Zotero → Settings... → Sync
```

设置如下：

### 数据同步

保留开启：

```text
自动同步
同步全文内容，可选
```

这一部分负责同步：

```text
文献条目
笔记
标签
分类
引用信息
附件记录
```

### 文件同步

关闭 Zotero 自带附件同步：

```text
取消勾选：“我的文库”附件同步方式
取消勾选：使用 Zotero 云存储同步群组文献库中的附件
```

这样 Zotero 官方账号不会再上传 PDF 附件，PDF 附件改由 Git 仓库同步。

如果你的 Zotero 云端空间已经被之前上传的 PDF 占满，可以在关闭文件同步后，到 Zotero 网页端的 Storage 设置中执行 **Purge Storage in My Library**，清理云端附件。这个操作只清理 Zotero 云端附件，不会删除本地 `storage/` 中的 PDF。

---

## 5. 创建 GitHub 仓库

在 GitHub 新建一个仓库，例如：

```text
Zotero-PDF
```

建议：

```text
Visibility: Private
不要勾选 Add a README
不要添加 .gitignore
不要添加 license
```

也就是创建一个空仓库。

仓库创建后，会得到一个远程地址，例如：

```text
https://github.com/HelloWorldFirst1024/Zotero-PDF.git
```

本文档推荐优先使用 HTTPS remote，因为它更容易和 GitHub Personal Access Token 配合。

---

## 6. 创建 GitHub Personal Access Token

GitHub 现在不支持用账号密码进行 HTTPS Git push。终端中提示输入 password 时，应输入 **Personal Access Token**，不是 GitHub 登录密码。

推荐使用 Fine-grained token。

进入：

```text
GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
```

创建 token 时建议这样设置：

```text
Repository access:
Only select repositories
选择你的仓库，例如 HelloWorldFirst1024/Zotero-PDF

Repository permissions:
Metadata: Read-only，默认已有
Contents: Read and write
```

只需要 `Contents: Read and write`，不需要 Actions、Administration、Codespaces 等权限。

生成 token 后立即复制保存。token 只显示一次。

---

## 7. 安装插件

下载插件 `.xpi` 文件，例如：

```text
zotero-git-storage-sync-v0.2.6.xpi
```

安装步骤：

```text
Zotero → Tools / 工具 → Plugins / 插件
右上角齿轮按钮
Install Add-on From File...
选择 .xpi 文件
安装后重启 Zotero
```

重启后，在以下位置查看插件入口：

```text
Tools / 工具 → Zotero Git Sync
```

部分版本也可能在 Zotero 顶部工具栏显示同步按钮。如果工具栏按钮没有出现，可以先使用 `Tools / 工具` 菜单中的入口。

---

## 8. 第一次 Push：主电脑上传 storage/

第一台电脑应该是本地 Zotero 附件最完整的电脑。

插件第一次 Push 时，会把以下目录作为 Git 仓库根目录：

```text
~/Zotero/storage
```

插件会尝试执行：

```bash
git init
git checkout -B main
git remote add origin <你的 GitHub 仓库 URL>
git add -A
git commit
git push
```

### 8.1 推荐先在终端完成第一次认证

因为 Zotero 插件调用 Git 时不一定适合交互式输入 GitHub 用户名和 token，推荐第一次先在终端中完成 push 和凭据保存。

进入 `storage/`：

```bash
cd ~/Zotero/storage
```

如果插件已经初始化过仓库，可以查看状态：

```bash
git status
git remote -v
git log --oneline --decorate -5
```

如果还没有设置远程仓库，可以设置：

```bash
git remote add origin https://github.com/HelloWorldFirst1024/Zotero-PDF.git
```

如果已经存在 origin 但地址不对，可以修改：

```bash
git remote set-url origin https://github.com/HelloWorldFirst1024/Zotero-PDF.git
```

然后 push：

```bash
git push -u origin main
```

终端会提示：

```text
Username for 'https://github.com':
```

输入你的 GitHub 用户名，例如：

```text
HelloWorldFirst1024
```

然后提示：

```text
Password for 'https://HelloWorldFirst1024@github.com':
```

这里粘贴刚才生成的 **Personal Access Token**，不要输入 GitHub 登录密码。

成功时会看到类似输出：

```text
To https://github.com/HelloWorldFirst1024/Zotero-PDF.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

这说明第一次上传成功。

### 8.2 后续可以使用插件按钮 Push

第一次终端 push 成功后，macOS Keychain 通常会保存 GitHub 凭据。之后回到 Zotero，点击插件的 Push，就可以自动提交和推送新的附件变化。

---

## 9. 代理设置，可选

如果你的网络环境访问 GitHub 不稳定，可以给当前仓库设置 Git 代理。

以本地代理端口 `7890` 为例：

```bash
cd ~/Zotero/storage

git config --local http.proxy http://127.0.0.1:7890
git config --local https.proxy http://127.0.0.1:7890
```

检查代理是否写入：

```bash
git config --local --get http.proxy
git config --local --get https.proxy
```

取消代理：

```bash
git config --local --unset http.proxy
git config --local --unset https.proxy
```

注意：

```text
在终端里 export http_proxy=... 只对当前终端有效。
如果 Zotero 是从 Dock 或 Finder 启动的，通常读不到终端里的 export 环境变量。
```

所以更推荐使用 `git config --local` 为 `~/Zotero/storage` 仓库单独配置代理。

---

## 10. 第二台电脑如何同步

第二台电脑建议按以下步骤操作。

### 10.1 安装 Zotero 并登录账号

先安装 Zotero，登录同一个 Zotero 账号，开启数据同步，关闭文件同步。

让 Zotero 先同步：

```text
文献条目
笔记
标签
分类
附件记录
```

### 10.2 安装插件

和第一台电脑一样安装 `.xpi` 插件并重启 Zotero。

### 10.3 用 GitHub 仓库替换 storage/

最稳妥的第一次同步方式是先退出 Zotero，然后在终端执行：

```bash
cd ~/Zotero
mv storage storage.backup
git clone https://github.com/HelloWorldFirst1024/Zotero-PDF.git storage
```

然后重新打开 Zotero。

这样第二台电脑的目录结构应为：

```text
~/Zotero/
├── zotero.sqlite        # Zotero 官方数据同步生成
└── storage/             # GitHub clone 下来的附件目录
```

Zotero 条目中的附件记录和 `storage/` 文件夹名对应后，就可以正常打开 PDF。

---

## 11. 日常使用流程

推荐习惯：

```text
在电脑 A 添加或修改 PDF 后：
1. Zotero 自动同步文献数据
2. 点击插件 Push，同步 storage/

切换到电脑 B 使用前：
1. Zotero 自动同步文献数据
2. 点击插件 Pull，拉取 storage/
```

简单记忆：

```text
离开一台电脑前：Push
开始使用另一台电脑前：Pull
```

尽量避免在多台电脑上同时大量添加或修改 PDF 后再同步，否则可能产生 Git 冲突。

---

## 12. 常见问题与解决方法

### 问题 1：插件安装失败，提示“无法与该版本 Zotero 兼容”

可能原因：插件 manifest 兼容信息不匹配，或使用了旧版插件包。

解决方法：

```text
1. 确认 Zotero 版本为 Zotero 7.x
2. 使用 v0.2.4 或更新版本的插件包
3. 安装后重启 Zotero
```

如果仍失败，可以查看 Zotero 错误日志：

```text
Help / 帮助 → Report Errors
```

或启用 Debug Output Logging 后重新安装。

---

### 问题 2：插件提示 Git Push 失败，只显示 observe@jar:file...

这是旧版插件错误提示不够友好，实际错误来自底层 Git 命令。

解决方法：

```text
1. 升级到 v0.2.6 或更新版本
2. 在终端中手动执行 git push 查看真实错误
```

终端检查：

```bash
cd ~/Zotero/storage
git status
git remote -v
git push -u origin main
```

---

### 问题 3：GitHub 仓库是空的，但插件提示“没有需要提交的变化”

可能原因：本地已经 commit，但尚未 push；旧版插件在没有新变化时直接返回，没有继续执行 push。

解决方法：

```text
1. 升级到 v0.2.5 或更新版本
2. 或者手动执行：
```

```bash
cd ~/Zotero/storage
git push -u origin main
```

---

### 问题 4：终端提示 Password authentication is not supported

错误示例：

```text
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed
```

原因：GitHub 不支持用账号密码进行 HTTPS Git push。

解决方法：

```text
Username 输入 GitHub 用户名
Password 粘贴 Personal Access Token
```

不要输入 GitHub 登录密码，也不要输入邮箱密码。

---

### 问题 5：终端提示 Write access to repository not granted 或 403

错误示例：

```text
remote: Write access to repository not granted.
fatal: unable to access ... The requested URL returned error: 403
```

可能原因：

```text
1. 输入了错误的 GitHub 用户名
2. 使用了错误账号生成的 token
3. token 没有当前仓库权限
4. token 没有 Contents: Read and write 权限
5. macOS Keychain 缓存了旧凭据
```

解决方法：

```bash
printf "protocol=https\nhost=github.com\n\n" | git credential-osxkeychain erase
```

然后重新 push：

```bash
cd ~/Zotero/storage
git push -u origin main
```

输入：

```text
Username: 你的 GitHub 用户名
Password: 具有 Contents: Read and write 权限的 Personal Access Token
```

---

### 问题 6：大文件警告 GH001 / Large files detected

错误或警告示例：

```text
warning: File xxx.pdf is 68.69 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
warning: GH001: Large files detected.
```

这只是警告。50 MiB 以上 GitHub 会提醒，但只要低于 100 MiB，通常仍可 push。

查找超过 99 MiB 的文件：

```bash
find ~/Zotero/storage -type f -size +99M -print
```

如果超过 100 MiB，需要移出同步目录、压缩、替换为较小版本，或改用 Git LFS。

---

### 问题 7：Git push 时提示 src refspec main does not match any

可能原因：本地还没有任何 commit。

解决方法：

```bash
cd ~/Zotero/storage
git add -A
git commit -m "Initial Zotero storage sync"
git push -u origin main
```

---

### 问题 8：GitHub 仓库默认分支不是 main

查看本地分支：

```bash
git branch
```

如果需要统一为 main：

```bash
git checkout -B main
git push -u origin main
```

如果 GitHub 上已有 master 分支或 README，可能需要先 pull：

```bash
git pull --rebase origin main
```

更简单的做法是：创建 GitHub 仓库时不要添加 README、.gitignore 或 license。

---

### 问题 9：Pull 时出现冲突

PDF 是二进制文件，Git 不能像文本一样自动合并。

建议原则：

```text
不要在多台电脑上同时修改同一个 PDF 后再同步。
切换电脑前先 Push。
开始使用另一台电脑前先 Pull。
```

如果已经冲突，先查看：

```bash
cd ~/Zotero/storage
git status
```

简单处理方式是保留两个版本，例如把其中一个冲突 PDF 改名为：

```text
paper.conflict-macbook.pdf
```

然后：

```bash
git add -A
git rebase --continue
```

如果不熟悉 Git 冲突处理，建议先完整备份 `storage/` 再操作。

---

### 问题 10：第二台电脑打开 Zotero 后找不到 PDF

检查几点：

```text
1. Zotero 官方数据同步是否已经完成
2. Git 仓库是否 clone 到了 ~/Zotero/storage，而不是 ~/Zotero/storage/Zotero-PDF
3. storage/ 下是否有类似 ABCD1234/paper.pdf 的目录结构
4. 两台电脑使用的是同一个 Zotero 账号和同一个 GitHub 仓库
```

正确结构应类似：

```text
~/Zotero/storage/5GSUS8EX/s41586-025-09446-5.pdf
```

而不是：

```text
~/Zotero/storage/Zotero-PDF/5GSUS8EX/s41586-025-09446-5.pdf
```

---

### 问题 11：代理设置后仍然无法访问 GitHub

确认当前仓库代理：

```bash
cd ~/Zotero/storage
git config --local --get http.proxy
git config --local --get https.proxy
```

如果没有输出，重新设置：

```bash
git config --local http.proxy http://127.0.0.1:7890
git config --local https.proxy http://127.0.0.1:7890
```

如果你使用的是 SSH remote：

```text
git@github.com:用户名/仓库.git
```

那么 `http.proxy` / `https.proxy` 不一定生效。建议改为 HTTPS remote：

```bash
git remote set-url origin https://github.com/用户名/仓库.git
```

---

### 问题 12：如何清除错误的 GitHub 登录缓存

macOS 上可以清除 GitHub 凭据：

```bash
printf "protocol=https\nhost=github.com\n\n" | git credential-osxkeychain erase
```

然后重新 push，让系统重新询问用户名和 token。

---

## 13. 推荐工作流总结

### 第一台电脑

```bash
# 备份
cp -R ~/Zotero/storage ~/Desktop/storage-backup

# 安装插件
# 创建 GitHub 私有空仓库
# 创建 GitHub fine-grained token，授予 Contents: Read and write

cd ~/Zotero/storage
git push -u origin main
```

之后日常在 Zotero 中点击插件 Push。

### 第二台电脑

```bash
# 先让 Zotero 官方同步条目、笔记、标签
# 退出 Zotero
cd ~/Zotero
mv storage storage.backup
git clone https://github.com/用户名/仓库名.git storage
```

之后日常在 Zotero 中点击插件 Pull / Push。

---

## 14. 插件定位

本插件适合：

```text
个人文献库
多台电脑之间同步 PDF 附件
希望绕开 Zotero 云端附件容量限制
熟悉或愿意使用 Git/GitHub
```

不适合：

```text
完全不了解 Git 的用户
需要同步 Zotero 数据库本身的用户
需要同步群组库附件的复杂团队场景
希望在 Zotero iOS 端直接依赖 Git 附件同步的用户
```

如果只想要最稳定、最接近 Zotero 官方体验的附件同步方案，WebDAV 或 Zotero Storage 仍然是更标准的选择。本插件是为明确希望使用 Git 同步 `storage/` 的用户设计的实验性方案。
