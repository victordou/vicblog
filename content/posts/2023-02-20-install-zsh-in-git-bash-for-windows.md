---
title: "Windows 安装 Zsh in Git Bash"
date: 2023-02-20T22:17:36+08:00
draft: true
toc: false
images:
tags: 
  - Github
  - zsh
slug: install-zsh-in-git-bash-for-windows
---
参考：

<https://dominikrys.com/posts/zsh-in-git-bash-on-windows/>

<http://i.lckiss.com/?p=6268>

<https://zhuanlan.zhihu.com/p/455925403>

### 为啥要用 `Zsh`
最近搞网络和这个博客，Windows 的终端越来越不顺手，常用工具还是 Linux 强，刚开始用 [Scoop](https://scoop.sh/) 安装了 [Git Bash](https://gitforwindows.org/)，自带了很多工具，真的不够也可以用 Scoop 安装。

但是最近玩 [Kali Linux](https://www.kali.org/) 才发现它自带 [Zsh](https://zsh.sourceforge.io/) 超级顺手，最心动的当然是[历史命令自动补全/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)，上网一查发现支持这个功能的 Shell 屈指可数，就三个：Zsh, Fish, PowerShell 安装 PSReadLine 模块。

PowerShell 自带的那些命令，真的是难记，也用 Scoop 安装过 [Oh My Posh](https://ohmyposh.dev/), 算了，放弃。

然后发现网上 Windows 安装 Zsh, 要么是 WSL, 要么是 [MSYS2](https://www.msys2.org/), WSL 访问本地文件还是有点繁琐```cd /mnt/c/Users/```, MSYS2 需要安装 Zsh 插件 ```pacman -S zsh git vim```, 最后一个办法就是本文想介绍的。


### 安装 Git Bash 和 Zsh
Git Bash(Git For Windows) 是基于 MSYS2 MINGW64 环境的，所以我们只要手动下载 MSYS2 的 Zsh 插件手动解压到 Git 目录即可。

1. 下载 `Git For Windows`, 从下面选择适合你的:
    * [Scoop](https://scoop.sh/) ```scoop install git```
    * [Git官网](https://git-scm.com/)下载 <https://git-scm.com/download/win>, 下载点击安装，安装过程一切默认。
    * [Windows10/11 商店](https://www.microsoft.com/store/productId/9NBLGGH4NNS1)自带 [winget](https://aka.ms/getwinget), ```winget install -e --id Git.Git```

2. 下载 `Zsh` 插件
    * 打开 <https://packages.msys2.org/package/zsh?repo=msys&variant=x86_64>
    * 点击下载 File: <https://mirror.msys2.org/msys/x86_64/zsh-5.9-2-x86_64.pkg.tar.zst>

3. 将 `*.tar.zst` 解压缩成 `*.tar`, 这样可以用任意压缩软件打开 `*.tar`. 解压有了两个办法：
    * 下载 Windows 解压工具 [PeaZip](https://peazip.github.io/) 或者 [7-Zip 插件](https://github.com/mcmilk/7-Zip-zstd/releases)
    * WSL 安装 ztsd 解压，`sudo apt update && sudo apt install zstd -y` 然后 `unzstd filename.tar.zst`

4. 将压缩包所有文件 (主要是 `etc` `usr`) 解压到你的 Git 安装目录，注意是合并文件夹，不会提示覆盖文件。
    * `%USERPROFILE%\scoop\apps\git\current\`
    * `C:\Program Files\Git\`

5. 打开 Git Bash 运行
    ```bash
    zsh
    ```

6. ***重要*** 如果你不小心跳过了 `Zsh` 配置页面，你可以使用下面命令重新配置。
    ```bash
    autoload -U zsh-newuser-install
    zsh-newuser-install -f
    ```
    * 按 `1` "Continue to the main menu." 进入配置页面
    * 按 `1` 配置命令历史，按 `1-3` 来修改命令历史大小和位置，按 `0` 返回。
    * 按 `2` 配置补全，按 `1` 选择 "Use the new completion system", 有提示等待确认就按`回车`，按 `0` 返回，再次提示是否保存按 `y`
    * 按 `0` 保存，完成
7. 将 `Zsh` 设置为默认 Shell, 新建 `.profile` 到你的用户文件夹 `%USERPROFILE%`, 在 `Git Bash` 输入`nano ~/.profile`, 粘贴下面的内容，`Crtl+S` 保存，再次打开 `Git Bash`就是 `Zsh` 了。

    ```bash
    if [ -t 1 ]; then
    exec zsh
    fi
    ```
### 安装Zsh插件

1. 先 clone 所有库，到你电脑本地磁盘上，如用户文件夹`%USERPROFILE%`

    ```bash
    git clone https://github.com/zsh-users/zsh-completions
    git clone https://github.com/zsh-users/zsh-syntax-highlighting
    git clone https://github.com/zsh-users/zsh-autosuggestions
    ```
2. 使用你顺手的编辑器打开 `.zshrc`, 添加下面这些东西，到第一行

    ```bash
    source ~/zsh-autosuggestions/zsh-autosuggestions.zsh
    source ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
    source ~/zsh-completions/zsh-completions.plugin.zsh
    ```
3. **如果自定义了 `Git Clone` 目录，自己修改上面的路径，比如 `/c/Users/xxxx/xxxx`

### 自定义`.zshrc`，添加 Git Branch.

1. 修改PROMPT，里面的定义请看这里<https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html>

```bash
PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
```
这个是从 Kali 抄的

2. 修改Title
    ```bash
    case $TERM in xterm*)
        precmd () {print -Pn "\e]0;%~\a"}
        ;;
    esac
    ```

3. 添加 Git Branch信息

    ```bash
    function git_branch_name()
    {
    branch=$(git symbolic-ref HEAD 2> /dev/null | awk 'BEGIN{FS="/"} {print $NF}')
    if [[ $branch == "" ]];
    then
        :
    else
        echo '%F{yellow}on '$branch'%f'
    fi
    }
    setopt prompt_subst
    RPROMPT=\$(git_branch_name)
    ```

4. 使用 `Zsh` 的 `vcs_info` 重新写一编，包含上面所有修改，但是因为引用 `vcs_info` 响应慢，耗资源，但是它支持显示 git 当前 tag，上面只能显示 Branch。

    ```bash
    autoload -Uz vcs_info
    case $TERM in xterm*)
        precmd () {vcs_info && print -Pn "\e]0;%~\a"}
        ;;
    esac

    zstyle ':vcs_info:git:*' formats '%F{yellow}on %b%f'

    setopt prompt_subst
    PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
    RPROMPT=\$vcs_info_msg_0_
    ```
5. 上次强制关闭 `Zsh`, 导致生成 `.zsh-histfile.lock` 使得 `Zsh` 打不开
    ```
    setopt HIST_FCNTL_LOCK
    ```

## 高大全.zshrc模板

```bash
case $TERM in xterm*)
    precmd () {print -Pn "\e]0;%~\a"}
    ;;
esac

function git_branch_name()
{
  branch=$(git symbolic-ref HEAD 2> /dev/null | awk 'BEGIN{FS="/"} {print $NF}')
  if [[ $branch == "" ]];
  then
    :
  else
    echo '%F{yellow}on '$branch'%f'
  fi
}
setopt prompt_subst
RPROMPT=\$(git_branch_name)

PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '

source ~/zsh-autosuggestions/zsh-autosuggestions.zsh
source ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source ~/zsh-completions/zsh-completions.plugin.zsh

# The following lines were added by compinstall
zstyle :compinstall filename '~/.zshrc'

autoload -Uz compinit
compinit
# End of lines added by compinstall
# Lines configured by zsh-newuser-install
HISTFILE=~/.histfile
HISTSIZE=1000
SAVEHIST=1000
# End of lines configured by zsh-newuser-install

setopt HIST_FCNTL_LOCK
setopt HIST_IGNORE_ALL_DUPS
setopt hist_ignore_dups       # ignore duplicated commands history list
setopt hist_ignore_space      # ignore commands that start with space
setopt hist_verify            # show command with history expansion to user before running it

alias ls='ls --color=auto'
alias dir='dir --color=auto'
alias vdir='vdir --color=auto'
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias diff='diff --color=auto'
alias ip='ip --color=auto'
```

### 要不要安装oh-my-zsh
我是重实用，不中美观的，你想安装就使用这个命令安装
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```