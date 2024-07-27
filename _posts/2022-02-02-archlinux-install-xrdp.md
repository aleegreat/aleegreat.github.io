---
title: "Archlinux安装xrdp"
last_modified_at: 2022-02-02
categories: [IT, Linux]
tags: [LXC, Linux, 远程桌面]
---

- [基础软件安装](#基础软件安装)
- [配置](#配置)
  - [xinit](#xinit)
  - [xrdp](#xrdp)
- [错误排查](#错误排查)

# 基础软件安装
通过aur安装`xrdp、xorgxrdp`

```bash 
yay xrdp
```

我是基于LXC安装的，因此还额外多一步桌面的安装

```bash
# 两行命令选其一即可，区别在于plasma-desktop 安装的程序会少一点
sudo pacman -S plasma 
sudo pacman -S plasma-desktop 
```

# 配置

## xinit

```bash
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

编辑` ~/.xinitrc`， 按下面示例修改文件末尾（示例为plasma，气体桌面环境按需调整启动命令）
```bash
#twm &
#xclock -geometry 50x50-1+1 &
#xterm -geometry 80x50+494+51 &
#xterm -geometry 80x20+494-0 &
#exec xterm -geometry 80x66+0+0 -name login
exec dbus-run-session -- startplasma-x11
```

## xrdp
使普通用户可以启动X，`sudo nano /etc/X11/Xwrapper.config`，添加下面配置
```bash
allowed_users=anybody
```

启动xrdp服务
```bash
sudo systemctl restart xrdp.service xrdp-sesman.service
sudo systemctl status xrdp.service xrdp-sesman.service
```

# 错误排查
日志文件路径
```bash
sudo tail -100f /var/log/xrdp-sesman.log
sudo tail -100f /var/log/xrdp.log
```


参考资料：
[archlinux_xrdp]
[archlinux_forums]


[archlinux_xrdp]:https://wiki.archlinux.org/title/Xrdp
[archlinux_forums]:https://bbs.archlinux.org/viewtopic.php?id=255583

