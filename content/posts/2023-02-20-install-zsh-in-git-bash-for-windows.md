---
title: "Windows 安装 Zsh in Git Bash"
date: 2023-02-20T22:17:36+08:00
draft: false
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

## 为啥要用 `Zsh`
最近搞网络和这个博客，Windows 的终端越来越不顺手，常用工具还是 Linux 强，刚开始用 [Scoop](https://scoop.sh/) 安装了 [Git Bash](https://gitforwindows.org/)，自带了很多工具，真的不够也可以用 Scoop 安装。

但是最近玩 [Kali Linux](https://www.kali.org/) 才发现它自带 [Zsh](https://zsh.sourceforge.io/) 超级顺手，最心动的当然是[历史命令自动补全/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)，上网一查发现支持这个功能的 Shell 屈指可数，就三个：Zsh, Fish, PowerShell 安装 PSReadLine 模块。

PowerShell 自带的那些命令，真的是难记，也用 Scoop 安装过 [Oh My Posh](https://ohmyposh.dev/), 算了，放弃。

然后发现网上 Windows 安装 Zsh, 要么是 WSL, 要么是 [MSYS2](https://www.msys2.org/), WSL 访问本地文件还是有点繁琐```cd /mnt/c/Users/```, MSYS2 需要安装 Zsh 插件 ```pacman -S zsh git vim```, 最后一个办法就是本文想介绍的。

![img](/images/Zsh.png "Zsh Shell")

## 安装 Git Bash 和 Zsh

1. 下载 `Git For Windows`, 从下面选择适合你的:
    * [Scoop](https://scoop.sh/) ```scoop install git```
    * [Git官网](https://git-scm.com/)下载 <https://git-scm.com/download/win>, 下载点击安装，安装过程一切默认。
    * [Windows10/11 商店](https://www.microsoft.com/store/productId/9NBLGGH4NNS1)自带 [winget](https://aka.ms/getwinget), ```winget install -e --id Git.Git```

2. 下载 `Zsh` 插件
    * 打开 <https://packages.msys2.org/package/zsh?repo=msys&variant=x86_64>
    * 点击下载 File: <https://mirror.msys2.org/msys/x86_64/zsh-5.9-2-x86_64.pkg.tar.zst>

    > Git Bash(Git For Windows) 是基于 MSYS2 MINGW64 环境的，所以我们只要手动下载 MSYS2 的 Zsh 插件手动解压到 Git 目录即可。
3. 将 `*.tar.zst` 解压缩成 `*.tar`, 这样可以用任意压缩软件打开 `*.tar`. 有下面两个办法：
    * 下载 Windows 解压工具 [PeaZip](https://peazip.github.io/) 或者 [7-Zip 插件](https://github.com/mcmilk/7-Zip-zstd/releases)
    * WSL 安装 ztsd 解压，`sudo apt update && sudo apt install zstd -y` 然后 `unzstd filename.tar.zst`

4. 将压缩包所有文件 (主要是 `etc` `usr`) 解压到你的 Git 安装目录，注意是合并文件夹，不会提示覆盖文件。
    * `%USERPROFILE%\scoop\apps\git\current\`
    * `C:\Program Files\Git\`

5. 打开 Git Bash 运行 `Zsh`
    ```bash
    zsh
    ```

6. **重要** 如果你不小心跳过了 `Zsh` 配置页面，你可以使用下面命令重新配置。
    ```bash
    autoload -U zsh-newuser-install
    zsh-newuser-install -f
    ```
    * 按 `1` "Continue to the main menu." 进入配置页面
    * 按 `1` 配置命令历史，按 `1-3` 来修改命令历史大小和位置，按 `0` 返回。
    * 按 `2` 配置补全，按 `1` 选择 "Use the new completion system", 有提示等待确认就按`回车`，按 `0` 返回，再次提示是否保存按 `y`
    * 按 `0` 保存，完成
7. `Zsh` 设置为默认的 Shell, 有两种办法:
    1. **推荐**，在 `%USERPROFILE%\scoop\apps\git\current\usr\bin\bash.exe -i -l -c zsh`, 末尾加入 ` -i -l -c zsh`.
    2. 不推荐，新建 `.profile` 到**用户文件夹** `%USERPROFILE%`, 使用文本编辑器 `nano ~/.profile` 粘贴下面的内容，`Crtl+S` 保存。
        > 也可添加到以下文件 `.profile` > `.bash_profile` > `.bashrc`, 优先级越高，兼容越好，还能手动切换到 bash.
    ```bash
    if [ -x /usr/bin/zsh.exe]; then
    exec zsh
    fi
    ```

## 安装Zsh插件

1. 先 clone 所有库，到你电脑本地磁盘上，如**用户文件夹** `%USERPROFILE%`

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
3. 如果你修改了 `Git Clone` 目录，请自己修改上面的路径，比如 `/c/Users/xxxx/xxxx`

## 自定义`.zshrc`，并添加 Git Branch.

1. 修改PROMPT，里面详细含义请看这里<https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html>

    ```bash
    PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
    ```

2. 修改Title，为文件路径，某些老的终端软件不支持\e，所以要替换，\033=\e, \007=\a, \012=\n, \015=\r.
    ```bash
    case $TERM in xterm*)
        precmd () {print -Pn "\e]0;%~\a"}
        ;;
    esac
    ```

3. 添加 Git Branch 信息-方法1，使用 Git 命令

    > ~~和**Windows Terminal**一起使用时，Zsh输入框为空时[改变窗口大小会导致卡死](https://github.com/msys2/MSYS2-packages/issues/2820)。~~

    > 重新修改了代码，使用[这位兄弟的代码](https://code.mendhak.com/simple-bash-prompt-for-developers-ps1-git/),我增加了`if [ -d ./.git ]; then`，再让 *New Bing* 帮我合并两个function为一个。bash 和 zsh 通用。

    ```bash
    function parse_git_branch {
        if [ -d ./.git ]; then
            local branch=$(git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e "s/* \(.*\)/\1/")
            local dirty=$(git status --porcelain 2> /dev/null | wc -l)
            if [ $dirty -gt 0 ]; then
                echo " on $branch*"
            else
                echo " on $branch"
            fi
        fi
    }
    setopt prompt_subst
    RPROMPT=%F{yellow}\$(parse_git_branch)%f
    ```

4. 添加 Git Branch 信息-方法2，使用 `Zsh` 自带的 `vcs_info` 模块。同时包含了 1-3 的修改。
    > 因为引用 `vcs_info` 会导致响应速度下降，推荐使用方法1.

    ```bash
    autoload -Uz vcs_info
    case $TERM in xterm*)
        precmd () {vcs_info && print -Pn "\e]0;%~\a"}
        ;;
    esac

    zstyle ':vcs_info:git:*' formats 'on %b'

    setopt prompt_subst
    PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
    RPROMPT=%F{yellow}\$vcs_info_msg_0_%f
    ```
5. 强制关闭 `Zsh`时, 导致生成了 `.zsh-histfile.lock` 会使得 `Zsh` 打开卡住，添加下面这个解决。
    ```
    setopt HIST_FCNTL_LOCK
    ```

## 添加 Zsh 至 Windows Terminal
> 设置 -> 新增空配置 -> 填入命令行`bash.exe -i -l -c zsh`，其他可填可不填，看你心情。

> -i = --interactive, -l = --login, -c = execute command

```
{
    "closeOnExit": "always",
    "commandline": "%USERPROFILE%\\scoop\\apps\\git\\current\\usr\\bin\\bash.exe -i -l -c zsh",
    "guid": "{9e15491f-01ea-44bf-878c-c90f7749c9a8}",
    "hidden": false,
    "icon": "%USERPROFILE%\\scoop\\apps\\git\\current\\usr\\share\\git\\git-for-windows.ico",
    "name": "Git Bash",
    "startingDirectory": "%USERPROFILE%"
},
```
> 其实上面可以填 `zsh.exe -i -l`, 但是在初始化 `/etc/profile` 和 `/mingw64/share/git/completion/git-completion.bash`，脚本呢有句话 `ERROR: this script is obsolete, please see git-completion.zsh`
```bash
if [[ -n ${ZSH_VERSION-} && -z ${GIT_SOURCING_ZSH_COMPLETION-} ]]; then
	echo "ERROR: this script is obsolete, please see git-completion.zsh" 1>&2
	return
fi
```
## 美化Zsh

- 安装 oh-my-zsh，可以美化Zsh，且包含了上面很多功能，但是因为功能多，导致速度不是很快。我是重实用，不中美观的。

- 但是你想安装就使用这个命令安装：
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```


## 高大全.zshrc模板

> 复制在右下角

```bash
# History configurations
# https://manpages.debian.org/bullseye/zsh-common/zshparam.1.en.html
HISTFILE=~/.zsh-history
HISTSIZE=1000
SAVEHIST=2000
HISTORY_IGNORE='(history*|h|ls|ll|la|l|cd|pwd|exit|cd ..)'
# configure key keybindings
# https://manpages.debian.org/bullseye/zsh-common/zshzle.1.en.html
bindkey -e                                        # emacs key bindings
bindkey ' ' magic-space                           # do history expansion on space
bindkey '^U' backward-kill-line                   # ctrl + U
bindkey '^[[3;5~' kill-word                       # ctrl + Supr
bindkey '^[[3~' delete-char                       # delete
bindkey '^[[1;5C' forward-word                    # ctrl + ->
bindkey '^[[1;5D' backward-word                   # ctrl + <-
bindkey '^[[5~' beginning-of-buffer-or-history    # page up
bindkey '^[[6~' end-of-buffer-or-history          # page down
bindkey '^[[H' beginning-of-line                  # home
bindkey '^[[F' end-of-line                        # end
bindkey '^[[Z' undo                               # shift + tab undo last action

# zsh options, case insensitive and underscores are ignored
# https://zsh.sourceforge.io/Doc/Release/Options.html
#setopt APPEND_HISTORY      # append their history list to the history file, rather than replace it.
setopt INC_APPEND_HISTORY   # history lines are added to the $HISTFILE incrementally
setopt HIST_FCNTL_LOCK      # add zsh history file lock avoid corruption
setopt HIST_IGNORE_ALL_DUPS # history list duplicates an older one, the older command is removed from the list
# setopt hist_ignore_dups   # ignore duplicated commands history list
setopt HIST_IGNORE_SPACE    # ignore commands that start with space
setopt HIST_VERIFY          # show command with history expansion to user before running it
setopt AUTO_CD              # change directory just by typing its name
setopt INTERACTIVE_COMMENTS # allow comments in interactive mode
setopt NUMERIC_GLOB_SORT    # sort filenames numerically when it makes sense
setopt PROMPT_SUBST         # prompt extend function, for git branch display

# xterm set the title
# \033=\e, \007=\a, \012=\n, \015=\r, https://manpages.debian.org/bullseye/manpages/ascii.7.en.html
# https://manpages.debian.org/bullseye/xterm/xterm.1.en.html
# https://web.archive.org/web/20221206072000/https://tldp.org/HOWTO/Xterm-Title-4.html
case $TERM in xterm*)
    precmd () {print -Pn "\e]0;%~\a"}
    ;;
esac

# Shows Git branch name in prompt.
# https://code.mendhak.com/simple-bash-prompt-for-developers-ps1-git/
function parse_git_branch {
    if [ -d ./.git ]; then
        local branch=$(git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e "s/* \(.*\)/\1/")
        local dirty=$(git status --porcelain 2> /dev/null | wc -l)
        if [ $dirty -gt 0 ]; then
            echo " on $branch*"
        else
            echo " on $branch"
        fi
    fi
}
# https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html
RPROMPT=%F{yellow}\$(parse_git_branch)%f
PROMPT=$'\n%F{%(#.blue.green)}┌──(%B%F{%(#.red.blue)}%*'$'%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '

# enable color support of ls, less and man, and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:" # fix ls color for folders with 777 permissions

    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'

    export LESS_TERMCAP_mb=$'\E[1;31m'     # begin blink
    export LESS_TERMCAP_md=$'\E[1;36m'     # begin bold
    export LESS_TERMCAP_me=$'\E[0m'        # reset bold/blink
    export LESS_TERMCAP_so=$'\E[01;33m'    # begin reverse video
    export LESS_TERMCAP_se=$'\E[0m'        # reset reverse video
    export LESS_TERMCAP_us=$'\E[1;32m'     # begin underline
    export LESS_TERMCAP_ue=$'\E[0m'        # reset underline

    # Take advantage of $LS_COLORS for completion as well
    zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
    zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
fi

# some useful aliases
alias ll='ls -laFh'
alias la='ls -A'
alias l='ls -AF1'
alias pwd='pwd -W'

# be paranoid
alias cp='cp -i'
alias mv='mv -i'
alias rm='rm -i'

function extract () {
   if [ -f $1 ] ; then
       case $1 in
           *.tar.bz2)   tar xvjf $1    ;;
           *.tar.gz)    tar xvzf $1    ;;
           *.bz2)       bunzip2 $1     ;;
           *.rar)       unrar x $1     ;;
           *.gz)        gunzip $1      ;;
           *.tar)       tar xvf $1     ;;
           *.tbz2)      tar xvjf $1    ;;
           *.tgz)       tar xvzf $1    ;;
           *.zip)       unzip $1       ;;
           *.Z)         uncompress $1  ;;
           *.7z)        7z x $1        ;;
           *)           echo "don't know how to extract '$1'..." ;;
       esac
   else
       echo "'$1' is not a valid file!"
   fi
 }


# enable syntax-highlighting
if [ -f ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
    . ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
    ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern)
    ZSH_HIGHLIGHT_STYLES[default]=none
    ZSH_HIGHLIGHT_STYLES[unknown-token]=fg=white,underline
    ZSH_HIGHLIGHT_STYLES[reserved-word]=fg=cyan,bold
    ZSH_HIGHLIGHT_STYLES[suffix-alias]=fg=green,underline
    ZSH_HIGHLIGHT_STYLES[global-alias]=fg=green,bold
    ZSH_HIGHLIGHT_STYLES[precommand]=fg=green,underline
    ZSH_HIGHLIGHT_STYLES[commandseparator]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[autodirectory]=fg=green,underline
    ZSH_HIGHLIGHT_STYLES[path]=bold
    ZSH_HIGHLIGHT_STYLES[path_pathseparator]=
    ZSH_HIGHLIGHT_STYLES[path_prefix_pathseparator]=
    ZSH_HIGHLIGHT_STYLES[globbing]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[history-expansion]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[command-substitution]=none
    ZSH_HIGHLIGHT_STYLES[command-substitution-delimiter]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[process-substitution]=none
    ZSH_HIGHLIGHT_STYLES[process-substitution-delimiter]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[single-hyphen-option]=fg=green
    ZSH_HIGHLIGHT_STYLES[double-hyphen-option]=fg=green
    ZSH_HIGHLIGHT_STYLES[back-quoted-argument]=none
    ZSH_HIGHLIGHT_STYLES[back-quoted-argument-delimiter]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[single-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[double-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[dollar-quoted-argument]=fg=yellow
    ZSH_HIGHLIGHT_STYLES[rc-quote]=fg=magenta
    ZSH_HIGHLIGHT_STYLES[dollar-double-quoted-argument]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[back-double-quoted-argument]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[back-dollar-quoted-argument]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[assign]=none
    ZSH_HIGHLIGHT_STYLES[redirection]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[comment]=fg=black,bold
    ZSH_HIGHLIGHT_STYLES[named-fd]=none
    ZSH_HIGHLIGHT_STYLES[numeric-fd]=none
    ZSH_HIGHLIGHT_STYLES[arg0]=fg=cyan
    ZSH_HIGHLIGHT_STYLES[bracket-error]=fg=red,bold
    ZSH_HIGHLIGHT_STYLES[bracket-level-1]=fg=blue,bold
    ZSH_HIGHLIGHT_STYLES[bracket-level-2]=fg=green,bold
    ZSH_HIGHLIGHT_STYLES[bracket-level-3]=fg=magenta,bold
    ZSH_HIGHLIGHT_STYLES[bracket-level-4]=fg=yellow,bold
    ZSH_HIGHLIGHT_STYLES[bracket-level-5]=fg=cyan,bold
    ZSH_HIGHLIGHT_STYLES[cursor-matchingbracket]=standout
fi

# enable auto-suggestions based on the history
if [ -f ~/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
    . ~/zsh-autosuggestions/zsh-autosuggestions.zsh
    # change suggestion color
    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=#9e9e9e'
fi
# enable extend completion
fpath=(~/zsh-completions/src $fpath)

# old way add funtion
#source ~/zsh-autosuggestions/zsh-autosuggestions.zsh
#source ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
#source ~/zsh-completions/zsh-completions.plugin.zsh

# compinit https://manpages.debian.org/bullseye/zsh-common/zshcompsys.1.en.html
# zstyle https://manpages.debian.org/bullseye/zsh-common/zshmodules.1.en.html
zstyle :compinstall filename '~/.zshrc'
autoload -Uz compinit
compinit -d ~/.zcompdump

zstyle ':completion:*:*:*:*:*' menu select
zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' rehash true
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'
```

> 顺便放出我的 `.bashrc`

```bash
# export LANG=C.UTF-8
# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
# https://manpages.debian.org/bullseye/bash/bash.1.en.html
HISTFILE=~/.bash-history
HISTSIZE=1000
HISTFILESIZE=2000
HISTCONTROL=ignorespace:erasedups
HISTIGNORE="history*:h:exit:ls:ll:la:l:cd:pwd:cd .."

# https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html
# https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html
# append to the history file, don't overwrite it
shopt -s histappend
# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# https://unix.stackexchange.com/a/18443
function historymerge {
    history -n; history -w; history -c; history -r;
}
trap historymerge EXIT
PROMPT_COMMAND="history -a; $PROMPT_COMMAND"

# Shows Git branch name in prompt.
# https://code.mendhak.com/simple-bash-prompt-for-developers-ps1-git/
#function parse_git_dirty {
#    if [ -d ./.git ]; then
#        [[ $(git status --porcelain 2> /dev/null) ]] && echo "*"
#    fi
#}
#function parse_git_branch {
#    if [ -d ./.git ]; then
#        git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e "s/* \(.*\)/ on \1$(parse_git_dirty)/"
#    fi
#}
function parse_git_branch {
    if [ -d ./.git ]; then
        local branch=$(git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e "s/* \(.*\)/\1/")
        local dirty=$(git status --porcelain 2> /dev/null | wc -l)
        if [ $dirty -gt 0 ]; then
            echo " on $branch*"
        else
            echo " on $branch"
        fi
    fi
}

# Define prompt colors
prompt_color='\[\033[;35m\]'
info_color='\[\033[1;32m\]'

# \033=\e, \007=\a, \012=\n, \015=\r, https://manpages.debian.org/bullseye/manpages/ascii.7.en.html
# https://manpages.debian.org/bullseye/xterm/xterm.1.en.html
# https://web.archive.org/web/20221206072000/https://tldp.org/HOWTO/Xterm-Title-4.html
# https://manpages.debian.org/bullseye/bash/bash.1.en.html
PS1='\[\033]0;\w\007\]\n'$prompt_color'┌──('$info_color'\t'$prompt_color')-[\[\033[0;1m\]\w'$prompt_color']\[\033[;33m\]$(parse_git_branch)\012'$prompt_color'└─'$info_color'\$\[\033[0m\] '
#PS1=$prompt_color'\[\033]0;\w\007\]\n┌──('$info_color'\t'$prompt_color')-[\[\033[0;1m\]\w'$prompt_color']\[\033[;33m\]$(GIT_PS1_SHOWUNTRACKEDFILES=1 GIT_PS1_SHOWDIRTYSTATE=1 __git_ps1)\012'$prompt_color'└─'$info_color'\$\[\033[0m\] '

unset prompt_color
unset info_color

# enable color support of ls, less and man, and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:" # fix ls color for folders with 777 permissions

    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'

    export LESS_TERMCAP_mb=$'\E[1;31m'     # begin blink
    export LESS_TERMCAP_md=$'\E[1;36m'     # begin bold
    export LESS_TERMCAP_me=$'\E[0m'        # reset bold/blink
    export LESS_TERMCAP_so=$'\E[01;33m'    # begin reverse video
    export LESS_TERMCAP_se=$'\E[0m'        # reset reverse video
    export LESS_TERMCAP_us=$'\E[1;32m'     # begin underline
    export LESS_TERMCAP_ue=$'\E[0m'        # reset underline
fi

# some more ls aliases
alias ll='ls -laFh'
alias la='ls -A'
alias l='ls -AF1'
alias pwd='pwd -W'

# be paranoid
alias cp='cp -i'
alias mv='mv -i'
alias rm='rm -i'

function extract () {
   if [ -f $1 ] ; then
       case $1 in
           *.tar.bz2)   tar xvjf $1    ;;
           *.tar.gz)    tar xvzf $1    ;;
           *.bz2)       bunzip2 $1     ;;
           *.rar)       unrar x $1     ;;
           *.gz)        gunzip $1      ;;
           *.tar)       tar xvf $1     ;;
           *.tbz2)      tar xvjf $1    ;;
           *.tgz)       tar xvzf $1    ;;
           *.zip)       unzip $1       ;;
           *.Z)         uncompress $1  ;;
           *.7z)        7z x $1        ;;
           *)           echo "don't know how to extract '$1'..." ;;
       esac
   else
       echo "'$1' is not a valid file!"
   fi
 }

```