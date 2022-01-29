---
title: "LXC搭建开发环境"
categories: [it, pve, lxc, python]
tags: [pve, lxc, mount, python, vnc,jupyter,远程桌面]

---

- [LXC基础环境](#lxc基础环境)
  - [安装桌面环境](#安装桌面环境)
  - [Zsh 安装及配置](#zsh-安装及配置)
  - [GIT安装及配置](#git安装及配置)
    - [安装](#安装)
    - [配置](#配置)
- [VNC远程桌面](#vnc远程桌面)
  - [安装](#安装-1)
  - [配置](#配置-1)
- [IDE](#ide)
  - [VSCode](#vscode)
  - [pycharm-ce](#pycharm-ce)
  - [jupyter-lab](#jupyter-lab)
  

# LXC基础环境
lxc on pve 的安装过程：略
我使用的Linux发行版本为Archlinux，且配置好`archlinuxcn`软件源，参考🔗：[tuna](https://mirrors.tuna.tsinghua.edu.cn/help/archlinuxcn/) 
在 `/etc/pacman.conf` 文件末尾添加以下两行：
```bash
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
之后安装好`archlinuxcn-keyring`即可。

## 安装桌面环境
我使用的KDE，其他桌面安装请参考相关资料，安装的时候也一并安装了部分常用软件

```bash
sudo pacman -S plasma-desktop konsole dolphin ark kate kcalc spectacle krunner partitionmanager packagekit-qt5 kdialog gnome-keyring
```

## Zsh 安装及配置
```bash
sudo pacman -S zsh oh-my-zsh-git 
cp /usr/share/oh-my-zsh/zshrc ~/.zshrc
# 修改默认shell为zsh
chsh -s /usr/bin/zsh $(whoami)
```

## GIT安装及配置
### 安装
```bash
sudo pacman -S git
```
### 配置
1. ssh认证
略


# VNC远程桌面
## 安装
```bash
sudo pacman -S tigervnc
```
## 配置
1. 编辑`/etc/tigervnc/vncserver.users`，假设你用户远程登录的Linux用户名为`zhangsan`在文件末尾新增如下行：
    ```bash
    :1=zhangsan
   ```
2. 切换到`zhangsan`用户下，执行`vncpasswd`,按提示输入用户名密码。
    > ** tigervnc 只支持8位密码，超过8位会截取密码的前8位 **

3. 启用tigervnc服务
    ```bash
    sudo systemctl enable vncserver@:1.service
    sudo systemctl restart vncserver@:1.service
    sudo systemctl status vncserver@:1.service
    ```

# Python基础环境

基础环境基于Anaconda，Anaconda的优化加速也参考：[tuna](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/) 
```bash
sudo pacman -S anaconda

```
新增下面两行到`~/.zshrc`中，便于快速切换python虚拟环境
```
alias conda_active="source /opt/anaconda/bin/activate root"
alias conda_deactive="conda deactivate"
```
# IDE

## VSCode
```bash
yay vscode
```

## pycharm-ce
```bash
sudo pacman -S pycharm-community-edition
```

## jupyter-lab 

```bash
/home/fate/mambaforge/bin/jupyter-lab \
	--no-browser  \
	--notebook-dir=/workspace \ # 请改为自己的workspace
	--ip='*' --port=8888 \ #`optional`，可修改为自己喜欢的端口
	--NotebookApp.certfile=/mnt/data_ssd/ssl/cert.pem \ # 有证书会修改为自己的证书路径，否则删除此行
	--NotebookApp.keyfile=/mnt/data_ssd/ssl/key.pem # 有证书会修改为自己的证书路径，否则删除此行
```


