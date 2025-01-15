# Git 的基本概念
Git 是一个分布式版本控制系统，广泛用于软件开发中来跟踪代码变更、协作开发和版本管理。以下是 Git 的基本使用指南，涵盖常见的操作和命令。

- **Repository（仓库）**：存储项目及其版本历史的地方。
- **Commit（提交）**：对文件或目录的快照。
- **Branch（分支）**：代码开发的独立分支。
- **Merge（合并）**：将分支的修改合并到主分支中。
- **Remote（远程）**：存储在服务器上的仓库。

# Git 的基本操作
##  安装 Git

- **Windows**：从 [Git 官网](https://git-scm.com/download/win) 下载并安装。
- **macOS**：使用 Homebrew 安装：`brew install git`
- **Linux**：使用包管理器安装，例如 Ubuntu：`sudo apt-get install git`

## 配置 Git

```sh
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

##  创建一个新的 Git 仓库

```sh
mkdir myproject
cd myproject
git init
```

## 克隆一个现有的仓库

```sh
git clone https://github.com/username/repository.git
```

## 检查仓库状态

```sh
git status
```

## 添加文件到仓库

```sh
git add filename
git add .
```

##  提交更改

```sh
git commit -m "Commit message"
```

## 查看提交历史

```sh
git log
```

## 创建和切换分支

```sh
git branch new-branch
git checkout new-branch
```

或者创建并切换到新分支

```sh
git checkout -b new-branch
```

## 合并分支

切换到你要合并到的分支（例如 `main`）

```sh
git checkout main
git merge new-branch
```

## 删除分支

```sh
git branch -d new-branch
```

## 添加远程仓库

下面的栗子是添加一个远程仓库，远程仓库的名字为origin，url为`https://github.com/username/repository.git`

```sh
git remote add origin https://github.com/username/repository.git
```

##  推送到远程仓库

首次推送需要先设置公钥

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

生成密钥之后查看密钥文件中的内容，将密钥文件中的内容复制到github的设置中的SSH中即可开始推送

```bash
cat ~/.ssh/id_rsa.pub 
```

可使用以下命令测试密钥是否配置成功

```bash
ssh -T git@github.com
```

推送的语法为

```bash
git push [添加的仓库名] [仓库的分支名]
```

下面的栗子是向origin仓库的branch-name分支推送代码

```bash
git push origin branch-name
```

## 从远程仓库拉取更新

```sh
git pull origin branch-name
```

##  解决冲突

在合并或拉取过程中，如果有冲突，Git 会提示冲突文件。你需要手动解决冲突并提交解决后的更改。

# 常用 Git 命令总结

- `git init`：初始化一个新的 Git 仓库
- `git clone <repo>`：克隆一个远程仓库
- `git status`：查看仓库状态
- `git add <file>`：添加文件到暂存区
- `git commit -m "message"`：提交更改
- `git log`：查看提交历史
- `git branch`：列出分支
- `git checkout <branch>`：切换到分支
- `git checkout -b <branch>`：创建并切换到新分支
- `git merge <branch>`：合并分支
- `git branch -d <branch>`：删除分支
- `git remote add origin <url>`：添加远程仓库
- `git push origin <branch>`：推送分支到远程仓库
- `git pull origin <branch>`：从远程仓库拉取分支更新

通过这些基本的 Git 操作和命令，你可以有效地管理和协作开发项目。

# .gitignore的使用规则

在Git中，`.gitignore` 文件用于告诉Git哪些文件和目录应该被忽略，不需要被版本控制。`.gitignore` 文件中的语法规则非常灵活，支持多种模式和通配符。以下是一些常用的语法规则和示例：

1. **注释**：以 `#` 开头的行是注释。

   ```plaintext
   # 这是一个注释
   ```

2. **忽略文件或目录**：直接写文件或目录的名称。

   ```plaintext
   # 忽略所有 .log 文件
   *.log
   
   # 忽略名为 temp 的目录
   temp/
   ```

3. **通配符 `*`**：匹配零个或多个字符。

   ```plaintext
   # 忽略所有 .log 文件
   *.log

   # 忽略所有的临时文件
   *~
   ```

4. **问号 `?`**：匹配任意一个字符。

   ```plaintext
   # 忽略所有扩展名为 .l?g 的文件（例如 .log 和 .l1g）
   *.l?g
   ```

5. **方括号 `[]`**：匹配括号内的任意一个字符。

   ```plaintext
   # 忽略所有扩展名为 .log 或 .lbg 的文件
   *.l[ob]g
   ```

6. **斜杠 `/`**：用于分隔目录。如果以 `/` 开头，表示相对于仓库根目录。如果以 `/` 结尾，表示一个目录。

   ```plaintext
   # 忽略根目录下的 temp 目录
   /temp/

   # 忽略所有子目录中的 temp 目录
   temp/
   ```

7. **否定模式 `!`**：将某些文件从忽略列表中排除。

   ```plaintext
   # 忽略所有的 .log 文件
   *.log

   # 但不要忽略特定的 debug.log 文件
   !debug.log
   ```

8. **双星号 `**`**：匹配任意目录深度。

   ```plaintext
   # 忽略任何子目录中的 temp 目录
   **/temp/

   # 忽略所有子目录中的 .log 文件
   **/*.log
   ```

9. **注释行中的 `\`**：用于转义字符。

   ```plaintext
   # 忽略所有以 # 开头的文件
   \#*.log
   ```

以下是一个示例 `.gitignore` 文件，结合了上述规则：

```plaintext
# 忽略所有的 .log 文件
*.log

# 忽略所有的临时文件
*~

# 忽略根目录下的 temp 目录
/temp/

# 忽略任何子目录中的 temp 目录
**/temp/

# 忽略所有子目录中的 .log 文件
**/*.log

# 但不要忽略特定的 debug.log 文件
!debug.log
```

