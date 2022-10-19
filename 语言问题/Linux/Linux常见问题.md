# 控制台设置颜色

家目录下的配置文件

~/.bashrc

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```



# Linux编译运行C++程序

```C++
g++ -o test test.c
test.exe
```



# Linux 不能图形化问题

- 查看Xshell连接的时候 是否有问题 [链接](https://blog.csdn.net/wugenqiang/article/details/86554753)
- 然后检查Xmanager 是否安装 [链接](https://www.pianshen.com/article/80871587504/)
- 然后是 Xshell 和 Xmanager的配合使用 [链接1](https://blog.csdn.net/qq_25406563/article/details/82082273) [链接2](https://blog.csdn.net/qq_42283110/article/details/84889466)
- 打开 Xmanager - Passive
- xhost +无法打开的问题 [链接](https://blog.51cto.com/77jiayuan/2160835)

> 执行以下命令
> `vncserver`
> `export DISPLAY=localhost:1`
> `xhost +`

- make qemu 内存分配的问题 [链接](https://blog.csdn.net/szuhuanggang/article/details/88770938)

> 执行以下命令
> `sudo sysctl vm.overcommit_memory=1`

# Linux下查看环境变量

```shell
echo $PATH
```

# Git连接github进行提交

参考这两篇文章[Git上传文件代码到GitHub（超详细）](https://blog.csdn.net/weixin_43806049/article/details/124963415)、[手把手教你用git上传项目到GitHub](https://zhuanlan.zhihu.com/p/193140870)

这篇博客超级详细的解释了原理 [Git 原理详解及实用指南](https://blog.csdn.net/Aria_Miazzy/article/details/107782890)

首先安装 git，然后配置用户名和邮箱

```Shell
git config --global user.name "vaesong"
git config --global user.email “18365519973@163.com”
```

还需要绑定一个 `.ssh` 下面的公钥，如果没有，需要手动创建，然后**把公钥复制到`github`上面的 ssh 设置里面**

```Shell
ssh-keygen -t rsa -C "18365519973@163.com"
```



其中最主要的操作就是：

```shell
git init //把这个目录变成Git可以管理的仓库
git add README.md //文件添加到仓库
git add . //不但可以跟单一文件，还可以跟通配符，更可以跟目录。一个点就把当前目录下所有未追踪的文件全部add了 
git commit -m "first commit" //把文件提交到仓库
git remote add origin git@github.com:vaesong/Note.git //关联远程仓库
git push -u origin main //把本地库的所有内容推送到远程库上
```

如果出现`error: remote origin already exists.`这种情况，需要先删除关联的 origin 的远程仓库

```Shell
git remote rm origin
```

实际上在进行 git 上传的时候，github并没有完全把文件显示，是因为有些文件夹是空的，必须有东西时，才会上传成功，单个文件夹是无法上传成功的！！！

可能在 `pull` 的时候还会碰到冲突的情况

- 将本地修改放入**缓存区**(成功后本地**工作区间**的代码跟**本地仓库**代码会同步)

```shell
git stash 
```

- 从**远程仓库**获取最新代码

```Shell
git pull
```

- 然后， 取出本地修改的代码

```Shell
git stash pop
```

之后，再根据提示的冲突文件，进行手工修改



# 两台服务器之间传输文件夹

[参考这里](https://blog.csdn.net/qq_42224274/article/details/81735315?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-81735315-blog-125486150.pc_relevant_multi_platform_whitelistv5&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-81735315-blog-125486150.pc_relevant_multi_platform_whitelistv5&utm_relevant_index=1)

```shell
scp -P [目标服务器的端口] [要传输的文件路径] [目标用户名]@[目标IP]:[目标文件路径]
scp -P 13122 Matterport3D.tar lhui@166.111.138.88:/home/lhui
如果是文件夹的话，前面加上 -r 
# 这个是从服务器下载文件到本地，这里指定的端口是服务器的端口
scp -P 13122 zp_3c@166.111.138.88:~/Anaconda3-2020.02-Linux-x86_64.sh /home/vaesong
```

本地与服务器之间传输文件，[点击这里](https://www.cnblogs.com/doubleyue/p/14688181.html)



# 挂载U盘

首先查看所有的磁盘信息

```Shell
sudo fdisk -l
```

然后找到 U 盘，通常是 `/dev/sd*`

```Shell
mkdir /mount
sudo mount /dev/sdb1 /mount
```

最后取消挂载

```Shell
sudo umount /mount
```

期间可能会出现不能识别 U 盘格式的问题，需要安装两个东西

```Shell
sudo apt-get install exfat-fuse exfat-utils
```

再次尝试，如果还是不行，就执行下面的命令

```Shell
sudo mount.exfat-fuse /dev/sdb1 /mount
```











# 双系统安装Debian

## 分区大小的设置

```Linux
/
linux系统根目录
目录结构如下：
linux目录结构：

/bin

存放二进制可执行文件(ls,cat,mkdir等)，系统常规命令一般都在这里。

/sbin

存放二进制可执行文件，只有root才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如ifconfig等。

/dev

用于存放设备文件。

/home

存放所有用户文件的根目录，是用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示

/root

超级用户家目录。

/lib

存放跟文件系统中的程序运行所需要的共享库及内核模块。共享库又叫动态链接共享库，作用类似windows里的.dll文件，存放了根文件系统程序运行所需的共享文件。系统函数库。

/lib
系统64位函数库。

/proc

虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息。

/run
当前运行的硬件和系统模块。

/srv
系统数据（常量）

/var

用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件(系统启动日志等。)等。系统数据（变量）。

/tmp

用于存放各种临时文件，是公用的临时文件存储点。


/mnt

系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统。临时设备挂载点。

/media
cdrom（光驱）临时挂载点。

/etc

存放系统管理和配置文件。

/opt

第三方软件安装位置。额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把tomcat等都安装到这里。

/boot

存放用于系统引导时使用的各种文件。

/sys
关于内核设定目录。

/usr
（unix system resourse）仅次于/目录，多数系统重要资源都存放在此。
用于存放系统应用程序，比较重要的目录/usr/local 本地系统管理员软件安装目录(安装系统级的应用)。这是最庞大的目录，要用到的应用程序和文件几乎都在这个目录。

/usr/x11r6 存放x window的目录

/usr/bin 众多的应用程序

/usr/sbin 超级用户的一些管理程序

/usr/doc linux文档

/usr/include linux下开发和编译应用程序所需要的头文件

/usr/lib 常用的动态链接库和软件包的配置文件

/usr/man 帮助文档

/usr/src 源代码，linux内核的源代码就放在/usr/src/linux里

/usr/local/bin 本地增加的命令

/usr/local/lib 本地增加的库

```



# 查看自身的 IP

```Shell
curl cip.cc
```



# ssh免密登录

基本原理：ssh登录分为两种：

- 口令登录

> 客户端发起请求
>
> 服务端发送自己的公钥给客户端
>
> 客户端用 公钥加密密码，发给服务器
>
> 然后服务器用私钥解密，如果密码正确，登录成功

- 密钥登录

> 先把客户端的公钥发送给服务器
>
> 然后客户端发送请求，例如登录请求
>
> 服务器端在授权的密钥列表中查找，找到了，就用客户端的公钥加密一个 自己 随机生成的字符串，发送给客户端
>
> 然后客户端用自己的私钥进行解密，再把解密之后的字符串发给服务器端
>
> 服务器端验证成功后，即登录成功

可以参考这篇，不过注意公钥只能加密，私钥只能解密 [SSH原理与运用（一）：远程登录](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

下面是服务器配置免密过程

1、首先客户端生成公钥和私钥，如果没有 .ssh 的话，需要手动生成

```Shell
ssh-keygen -t rsa -C "18365519973@163.com"
```

2、然后把客户端的公钥复制到服务器，相应的用户名的 `.ssh` 下面，例如 `/home/zp_3c/.ssh`。注意不要复制错了，**先打开插入模式**

```Shell
# 如果没有授权文件，需要自己创建
vim ~/.ssh/authorized_keys
```

3、这时候，需要修改 `.ssh` 和 `authorized_keys` 的权限，这俩文件只能让所有者有权限操作，否则验证不通过

```Shell
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

4、如果不能登录的话，看一下配置文件的 `AuthorizedKeysFile` 是否是打开的

```Shell
sudo vim /etc/ssh/sshd_config
# AuthorizedKeysFile 修改成下面的样子
AuthorizedKeysFile      %h/.ssh/authorized_keys
```

5、最后，重启 `ssh` 服务

```Shell
service sshd restart
```



# frp实现通过公网ip访问内网

通过frp搭建隧道，使外网可以直接SSH连接校园网中的服务器，知乎的一篇文章，跟着教程做就行 [参考链接](https://www.zhihu.com/question/22568892)

0.42.0 版本是有 `systemd` 文件的

## 1、准备

1. 固定IP的公网服务器一台（阿里云服务器）
2. 校园网中可SSH连接的实验室服务器若干台（以下步骤需要重复N次）

## 2、步骤

公网服务器作为fcp服务器，校园网服务器作为frp客户端。部署成功后，访问者主机使用SSH命令可直接访问校园网服务器。

- 公网服务器操作

- - 在所需目录下，下载frp压缩包并解压
  - 编辑frp公网服务器配置文件frps.ini
  - 开放公网服务器入站端口（frp服务器服务端口和服务端口）
  - 配置环境，以服务运行
  - 网页登录frp管理服务

- 校园网服务器操作

- - 在所需目录下，下载frp压缩包并解压
  - 编辑frp客户端配置文件frpc.ini
  - 开放公网服务器入站端口（对应每一个frp客户端，frp服务器都需要监听一个对应的端口）
  - 配置环境，以服务运行

### 2.1 公网服务器操作

本人选择阿里云云服务器作为公网服务器，IP为 `120.76.100.14`，请注意替换。

frp工作原理为：

1. 服务端运行，监听一个主端口，等待客户端的连接；
2. 客户端连接到服务端的主端口，同时告诉服务端要监听的端口和转发类型；
3. 服务端fork新的进程监听客户端指定的端口；
4. 外网用户连接到客户端指定的端口，服务端通过和客户端的连接将数据转发到客户端；
5. 客户端进程再将数据转发到本地服务，从而实现内网对外暴露服务的能力。

### 下载frp并解压

在 [frp下载页面](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases) 选择合适版本下载。

在用户目录下创建frp目录，并在其中下载压缩包，并解压进入解压目录。

```Shell
mkdir ~/frp
cd ~/frp
wget https://github.com/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_amd64.tar.gz
tar -zxvf frp_0.38.0_linux_amd64.tar.gz
cd frp_0.38.0_linux_amd64
```

解压目录如下，其中frps系列文件主要用于frp服务器，frpc系列文件主要用于frp客户端（校园网服务器）。

稍后我们在校园服务器上也需要下载同样的软件包。

```Shell
frp_0.38.0_linux_amd64
├── frpc
├── frpc_full.ini
├── frpc.ini
├── frps
├── frps_full.ini
├── frps.ini
├── LICENSE
└── systemd
    ├── frpc.service
    ├── frpc@.service
    ├── frps.service
    └── frps@.service
```

### 编辑frp服务器配置文件frps.ini

```Shell
[common]
# frp监听的端口，默认是7000，可以改成其他的
bind_port = 7000
# 授权码，请改成更复杂的
token = 12345678

# frp管理后台端口，请按自己需求更改
dashboard_port = 7500
# frp管理后台用户名和密码，请改成自己的
dashboard_user = admin
dashboard_pwd = admin
enable_prometheus = true
```

### 开放公网服务器入站端口

在云服务器网络设置中，添加以上7000（frp服务器绑定端口）和7500（frp服务管理网页访问端口）TCP协议的入站规则。

### 配置环境，以服务运行

当前路径下，一般可以通过`./frps -c frps.ini`直接开启服务，但是SSH连接断开后就会失效，所以需要借助systemd服务运行。

```Shell
cd ~/frp/frp_0.38.0_linux_amd64/
sudo mkdir -p /etc/frp
sudo cp frps.ini /etc/frp
sudo cp frps /usr/bin
sudo cp systemd/frps.service /usr/lib/systemd/system/
sudo systemctl enable frps
sudo systemctl start frps
```

### 网页登录frp管理服务

顺利的话，可以登录`公网ip:frp服务管理网页访问端口`查看frp的管理界面。下图已经有一个客户端连接。

### 2.2 校园网服务器操作

校园网服务器下载frp并配置客户端文件，以服务运行。

### 下载frp压缩包并解压

同公网服务器。

```shell
mkdir ~/frp
cd ~/frp
wget https://github.com/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_amd64.tar.gz
tar -zxvf frp_0.38.0_linux_amd64.tar.gz
cd frp_0.38.0_linux_amd64
```

### 编辑frp客户端配置文件frpc.ini

```Shell
# 服务端配置
[common]
server_addr = 120.76.100.14
# 请换成设置的服务器端口
server_port = 7000
token = 12345678

# 配置ssh服务,ssh9009是frp连接名，每个frp客户端都要设置不一样的
[ssh9018]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 9018  #这个端口就对应了一台实验室服务器的SSH连接，这里把端口和校园网服务器编号保持一致方便记忆
```

### 开放公网服务器入站端口

### 配置环境，以服务运行

```shell
cd ~/frp/frp_0.38.0_linux_amd64/
sudo mkdir -p /etc/frp
sudo cp frpc.ini /etc/frp
sudo cp frpc /usr/bin
sudo mkdir /usr/lib/systemd/system/
sudo cp systemd/frpc.service /usr/lib/systemd/system/
sudo systemctl enable frpc

# Created symlink from /etc/systemd/system/multi-user.target.wants/frpc.service to /usr/lib/systemd/system/frpc.service.

sudo systemctl start frpc
```

### 网页登录frp管理服务验证

## 3、SSH远程登录

登录失败的话，如果报错

```shell
[W] [service.go:104] login to server failed: i/o deadline reached i/o deadli
```

在客户端的（也就是内网服务器）的 `frpc.ini` 的文件中，加上 `tls_enable = true`

```Shell
[common]
server_addr = 120.76.100.14
server_port = 7000
token=123456789
tls_enable=true

[ssh6001]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001
```



