# 启动 Vim

在终端中输入 `vim` 启动 Vim，或者输入 `vim filename` 打开特定文件。

```sh
vim
vim filename
```

#  基本模式

Vim 有三种主要模式：

- **Normal 模式**：默认模式，用于执行命令。
- **Insert 模式**：用于编辑文本。
- **Visual 模式**：用于选择文本。

切换模式

- **Normal 模式**：启动 Vim 后默认进入 Normal 模式。按 `Esc` 可以从其他模式切换到 Normal 模式。
- **Insert 模式**：在 Normal 模式下，按 `i` 进入 Insert 模式，按 `Esc` 退出到 Normal 模式。
- **Visual 模式**：在 Normal 模式下，按 `v` 进入 Visual 模式，按 `Esc` 退出到 Normal 模式。

# 常用命令

## Normal 模式命令

- **移动光标**：
  - `h`：左移
  - `j`：下移
  - `k`：上移
  - `l`：右移
  - `w`：移动到下一个单词的开头
  - `b`：移动到前一个单词的开头
  - `0`：移动到行首
  - `$`：移动到行尾

- **编辑文本**：
  - `i`：在光标前插入
  - `a`：在光标后插入
  - `o`：在当前行下方插入新行
  - `dd`：删除当前行
  - `d$`：删除从光标到行尾的内容
  - `x`：删除光标下的字符
  - `yy`：复制当前行
  - `p`：粘贴
  - `u`：撤销
  - `Ctrl + r`：重做

- **保存和退出**：
  - `:w`：保存文件
  - `:q`：退出 Vim
  - `:wq` 或 `ZZ`：保存并退出
  - `:q!`：不保存退出

## Insert 模式命令

- 在 Insert 模式下，你可以像普通文本编辑器一样输入文本。
- 使用 `Esc` 返回到 Normal 模式。

## Visual 模式命令

- `v`：进入 Visual 模式并选择字符
- `V`：进入 Visual 模式并选择行
- `Ctrl + v`：进入 Visual 模式并选择块
- 在 Visual 模式下，你可以使用 `y` 复制，`d` 删除，`p` 粘贴选中的内容。

## 键位图

![VIM键位图](https://i-blog.csdnimg.cn/blog_migrate/9d1351469929da16936b11c946ecd2c4.jpeg)

# 搜索和替换

- **搜索**：
  - `/pattern`：向下搜索 `pattern`
  - `?pattern`：向上搜索 `pattern`
  - `n`：跳到下一个匹配
  - `N`：跳到上一个匹配

- **替换**：
  - `:%s/old/new/g`：全局替换所有 `old` 为 `new`
  - `:s/old/new/g`：替换当前行所有 `old` 为 `new`
  - `:s/old/new/gc`：替换当前行所有 `old` 为 `new`，并确认每次替换

# 配置 Vim

你可以通过编辑 `~/.vimrc` 文件来配置 Vim。例如：

```vim
syntax on              " 启用语法高亮
set number             " 显示行号
set tabstop=4          " 设置 Tab 长度为 4
set shiftwidth=4       " 设置自动缩进为 4
set expandtab          " 将 Tab 转换为空格
```

# 插件管理

Vim 有丰富的插件生态，可以使用插件管理器来安装和管理插件。常见的插件管理器有：

- **Vundle**：在 `~/.vimrc` 中添加插件配置，然后运行 `:PluginInstall`。

```vim
set nocompatible              " 必须
filetype off                  " 必须

" 设置 runtime path
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
" 在这里添加更多的插件
Plugin 'tpope/vim-sensible'
call vundle#end()            " 必须
filetype plugin indent on    " 必须
```

- **Pathogen**：将插件克隆到 `~/.vim/bundle/` 目录。

```sh
mkdir -p ~/.vim/autoload ~/.vim/bundle && \
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```

在 `~/.vimrc` 中添加：

```vim
execute pathogen#infect()
syntax on
filetype plugin indent on
```

通过这些基本操作，你可以高效地使用 Vim 进行文本编辑和代码开发。随着使用经验的增加，你可以进一步探索 Vim 的高级功能和插件生态，提升工作效率。