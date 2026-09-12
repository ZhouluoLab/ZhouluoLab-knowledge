# Git 与 GitHub 使用笔记

Git 是分布式版本控制系统，用来记录文件的修改历史；GitHub 是托管 Git 仓库并支持协作的平台。Git 可以完全在本地使用，只有在需要备份或协作时才需要连接 GitHub 等远程平台。

## Git 的核心概念

一次修改通常会依次经过以下区域：

1. **工作区（Working Tree）**：当前正在编辑的文件。
2. **暂存区（Staging Area / Index）**：通过 `git add` 选入下一次提交的内容。
3. **本地仓库（Local Repository）**：通过 `git commit` 保存的版本历史。
4. **远程仓库（Remote Repository）**：通过 `git push` 上传到 GitHub 等平台的版本历史。

常见流程可以概括为：

```text
编辑文件 -> git add -> git commit -> git push
工作区      暂存区       本地仓库       远程仓库
```

## 创建本地仓库

进入项目根目录后，初始化仓库并直接指定初始分支名为 `main`：

```bash
git init -b main
```

`git init` 会在当前目录中创建 `.git` 目录。这里保存着版本历史和仓库配置，不应手动修改或删除。

如果使用的 Git 版本不支持 `git init -b main`，可以改用：

```bash
git init
git branch -M main
```

### 配置用户名和邮箱

提交记录会保存作者姓名与邮箱。为当前用户的所有仓库设置默认身份：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

检查配置：

```bash
git config --global user.name
git config --global user.email
```

如果某个仓库需要使用不同身份，在该仓库内省略 `--global`：

```bash
git config user.name "Your Name"
git config user.email "you@example.com"
```

## 使用 `.gitignore`

仓库根目录中的 `.gitignore` 用来声明不应被 Git 跟踪的文件，例如编辑器配置、缓存、构建产物和本地环境文件：

```gitignore
.idea/
.vscode/
__pycache__/
*.pyc
.env
```

不要把密码、令牌、私钥等敏感信息提交到仓库。仅将文件加入 `.gitignore`，不能清除已经进入 Git 历史的敏感信息。

检查某个文件匹配了哪条忽略规则：

```bash
git check-ignore -v path/to/file
```

`.gitignore` 只影响尚未被跟踪的文件。如果文件已经被 Git 跟踪，需要把它从 Git 索引中移除，再提交这次变更：

```bash
git rm -r --cached .idea/
git add .gitignore
git commit -m "Stop tracking IDE settings"
```

`--cached` 表示只停止跟踪，保留工作区中的本地文件。

## 第一次提交

先检查仓库状态，再把准备提交的内容加入暂存区：

```bash
git status
git add -A
git status
```

`git add -A` 会暂存仓库中的新增、修改和删除。也可以只暂存指定文件：

```bash
git add README.md .gitignore
```

提交前检查差异：

```bash
# 查看尚未暂存的修改
git diff

# 查看已暂存、即将提交的修改
git diff --staged
```

确认无误后提交：

```bash
git commit -m "Initial commit"
```

提交信息应简洁说明这次修改做了什么，例如 `Add Git usage notes` 或 `Fix image path`。

### 修正刚刚的提交

如果遗漏了文件，可以补充暂存后修改最近一次提交：

```bash
git add README.md .gitignore
git commit --amend --no-edit
```

`--amend` 会重写最近一次提交并产生新的提交 ID。它适合修正尚未推送、没有被他人使用的提交；已经共享的提交不要随意改写。

## 连接 GitHub 远程仓库

如果本地已经存在提交，建议在 GitHub 新建一个**空仓库**，不要同时初始化 README、`.gitignore` 或 LICENSE。然后执行：

```bash
git remote add origin <remote-url>
git remote -v
git push -u origin main
```

`origin` 是远程仓库的常用别名。`-u` 会建立本地 `main` 与远程 `origin/main` 的跟踪关系，以后通常可以直接执行 `git push` 和 `git pull`。

修改已有的远程地址：

```bash
git remote set-url origin <new-remote-url>
```

### 远程仓库已经包含 README 或 LICENSE

最简单的做法是在开始工作前直接克隆远程仓库，再把本地文件复制进去，这样不会产生两套互不相关的历史。

如果本地与远程都已经分别产生了提交，并且确认需要保留两边的历史，可以显式合并：

```bash
git fetch origin
git merge origin/main --allow-unrelated-histories
git push -u origin main
```

`--allow-unrelated-histories` 会绕过 Git 对“没有共同祖先的两段历史”的安全检查，可能产生冲突，只应在确认两边确实属于同一个项目时使用。

如果想取消尚未完成的合并：

```bash
git merge --abort
```

## 日常提交流程

多人协作或远程仓库可能有更新时，可以采用以下流程：

```bash
git status
git pull --ff-only

# 编辑文件

git status
git diff
git add -A
git diff --staged
git commit -m "Describe the change"
git push
```

`git pull --ff-only` 只接受快进更新，可以避免拉取时意外创建合并提交。如果本地与远程已经分叉，它会停止并提示，需要根据团队约定选择 merge 或 rebase。

提交前应检查 `git status` 和 `git diff --staged`，避免把临时文件或敏感信息一并上传。

## 查看仓库状态与历史

```bash
# 查看文件状态
git status

# 查看未暂存的修改
git diff

# 查看已暂存的修改；--cached 与 --staged 等价
git diff --staged

# 以紧凑图形查看所有分支的提交历史
git log --oneline --decorate --graph --all

# 查看 Git 当前跟踪的文件
git ls-files

# 查看本地分支
git branch

# 查看本地与远程分支
git branch -a

# 获取远程更新但暂不合并
git fetch origin
```

注意选项是 `--oneline`，不是 `--online`。

## 文件重命名、移动和删除

使用 Git 命令移动文件：

```bash
git mv old-name.md new-name.md
```

也可以用文件管理器或编辑器完成移动，再执行：

```bash
git add -A
git status
```

Git 在底层记录的是文件快照，而不是“重命名”动作；查看差异时，它会根据内容相似度推断文件是否被重命名。

删除文件：

```bash
git rm path/to/file
```

随后正常提交即可。

## 换行符

Windows 常用 CRLF，Linux 和 macOS 常用 LF。Git 出现换行符转换提示时通常不是错误，但不一致的设置可能造成整份文件看起来都被修改。

团队项目更适合在仓库根目录提交 `.gitattributes`，统一不同平台的行为：

```gitattributes
* text=auto
*.sh text eol=lf
*.bat text eol=crlf
```

也可以配置个人环境中的默认转换策略：

```bash
# Windows
git config --global core.autocrlf true

# Linux 或 macOS
git config --global core.autocrlf input
```

修改换行符策略前，应先确认项目规范，避免产生大范围无意义差异。

## 克隆已有仓库

```bash
git clone <repository-url>
cd <repository-directory>
git remote -v
git branch -a
git log --oneline --decorate --graph --all
```

获取远程更新并合并到当前分支：

```bash
git pull --ff-only
```

只获取远程信息而不修改当前分支：

```bash
git fetch origin
```

## 分支管理

分支适合隔离新功能、学习笔记和实验修改，让 `main` 保持相对稳定。

从最新的 `main` 创建并切换到新分支：

```bash
git switch main
git pull --ff-only
git switch -c study-notes
```

旧版本 Git 也可以使用：

```bash
git checkout -b study-notes
```

在新分支上修改并提交：

```bash
git add -A
git commit -m "Add study notes"
git push -u origin study-notes
```

切换回主分支：

```bash
git switch main
```

删除已经合并的本地分支：

```bash
git branch -d study-notes
```

`git branch -D <branch>` 会强制删除尚未合并的分支，可能使提交难以找回，使用前必须确认其中的工作不再需要。

### 实验分支示例

```bash
git switch main
git switch -c exp/dice-loss

# 修改代码并完成实验
git add -A
git commit -m "Try Dice loss"
```

如果不采用该实验，可以回到 `main`，再创建另一个分支：

```bash
git switch main
git switch -c exp/focal-loss
```

## 保留或撤销部分修改

查看尚未提交的修改：

```bash
git status
git diff
```

丢弃某个文件中**尚未暂存**的修改：

```bash
git restore path/to/file
```

该命令会用暂存区中的版本覆盖工作区，未提交的修改通常无法通过 Git 找回，执行前必须仔细确认。

取消暂存但保留工作区修改：

```bash
git restore --staged path/to/file
```

如果只想提交文件中的一部分修改，可以交互式选择差异块：

```bash
git add -p
```

常用选项包括：`y` 暂存当前块、`n` 跳过当前块、`s` 拆分当前块、`e` 手动编辑、`q` 退出。选择完成后，用 `git diff --staged` 检查将要提交的内容。

## 合并分支

如果希望把功能分支的全部提交合并到 `main`：

```bash
git switch main
git pull --ff-only
git merge feature-branch
git push
```

如果发生冲突：

1. 打开冲突文件并决定保留哪些内容。
2. 删除 `<<<<<<<`、`=======`、`>>>>>>>` 等冲突标记。
3. 使用 `git add <file>` 标记冲突已经解决。
4. 使用 `git commit` 完成合并，或者按 Git 的提示继续。

不想继续合并时，可以执行 `git merge --abort`。开始合并前最好保持工作区干净。

## 只应用某个提交

如果不想合并整个分支，只想把其中一个提交应用到当前分支，可以使用 `cherry-pick`。

先查找源分支上的提交 ID：

```bash
git log --oneline source-branch
```

切换到目标分支并应用该提交：

```bash
git switch target-branch
git cherry-pick <commit-id>
```

`cherry-pick` 会把指定提交引入的修改应用到当前分支，并通常创建一个新提交，因此新提交的 ID 与原提交不同。

发生冲突时，解决并暂存文件后继续：

```bash
git cherry-pick --continue
```

放弃本次操作：

```bash
git cherry-pick --abort
```

## 为开源仓库贡献代码

没有原仓库写入权限时，常见流程是 Fork 和 Pull Request：

1. 在 GitHub 上 Fork 原仓库到自己的账号。
2. 克隆自己的 Fork。
3. 将原仓库添加为 `upstream`，方便同步更新。
4. 从最新主分支创建独立分支。
5. 修改、检查并提交。
6. 推送到自己的 Fork。
7. 在 GitHub 上向原仓库发起 Pull Request。

示例：

```bash
git clone <your-fork-url>
cd <repository-directory>
git remote add upstream <original-repository-url>

git fetch upstream
git switch main
git merge --ff-only upstream/main
git switch -c fix/dataloader

# 修改并测试
git add -A
git commit -m "Fix data loader"
git push -u origin fix/dataloader
```

推送后，在 GitHub 页面上创建 Pull Request。不要直接在自己的 `main` 上堆积实验修改，这会增加与上游同步的难度。

## README 建议

一个易于理解的仓库通常应在 `README.md` 中说明：

- 项目简介与目标。
- 主要功能或内容。
- 目录结构。
- 安装或构建环境。
- 使用、构建和运行方法。
- 当前进度与维护规范。
- 许可证或版权说明。

## 参考资料

- 个人 Git 笔记
- [Git 官方文档](https://git-scm.com/docs)
- [Git 官方速查表](https://git-scm.com/cheat-sheet.pdf)
- [GitHub Docs：将本地代码添加到 GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github)
- [GitHub Docs：配置 Git 处理换行符](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)
