---
title: "使用Docker安装Samba共享服务 "
categories: [IT, PVE]
tags: [PVE, Docker, Samba]
---
<!-- TOC -->

- [Docker安装Samba](#docker%E5%AE%89%E8%A3%85samba)
- [Samba参数配置](#samba%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE)
- [参考资料](#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

<!-- /TOC -->


# Docker安装Samba
samba的docker镜像目前使用较广泛的是[dperson/samba](https://hub.docker.com/r/dperson/samba)的镜像,但是该镜像已经很长时间未更新。因此我选用了[dockur/samba](https://github.com/dockur/samba)。
我是用的CLI命令如下,请更具自己需要调整下面命令中参数：
```
docker run  -d --name samba --restart unless-stopped \
  -p 445:445 \
  -e "USER=${username}" -e "PASS=${password}" \
  -v /path_to_share:/mount/path_to_share \
  -v /path_to_samba/smb.conf:/etc/samba/smb.conf \
  dockurr/samba
```

# Samba参数配置
默认参数下，在MacOS挂载响应速度较慢，传输速度也不是很快。参考网上资料，对`SMb.conf`中参数进行了调整。
完整板如下
```conf
[global]
    security = user
    passdb backend = smbpasswd
    idmap config * : range = 3000-7999
    log file = /dev/stdout
    max log size = 50
    #map to guest = bad user
    #usershare allow guests = no
    create mask = 0664
    force create mode = 0664
    directory mask = 0775
    force directory mode = 0775
    #vfs objects = catia fruit recycle streams_xattr
    recycle:keeptree = yes
    recycle:maxsize = 0
    recycle:repository = .deleted
    recycle:versions = yes

    # Security SMB2
    client ipc max protocol = SMB3
    client ipc min protocol = SMB2_10
    client max protocol = SMB3
    client min protocol = SMB2_10
    server max protocol = SMB3
    server min protocol = SMB2_10
    server smb encrypt = desired
    server signing = desired
    # default/auto,desired,required,off
    smb encrypt = auto

    #idmap config * : backend = tdb
    #idmap config * : range = 1000-99999
    #winbind enum groups = no
    #winbind enum users = no
    # disable printing services
    load printers = no
    printing = bsd
    printcap name = /dev/null
    disable spoolss = yes

    # 优化参数
    follow symlinks = no
    wide links = no
    use sendfile = yes
    aio read size = 0
    aio write size = 1
    max xmit = 65535
    dead time = 15
    #max open files = 65535
    case sensitive = false

    # disable locking, because only 2 share can be written.
    strict locking = no
    fake oplocks = yes
    oplocks = no

[share]
    path = /mount/path_to_share
    comment = path_to_share
    valid users = @smb
    browseable = yes
    writable = yes
    read only = no
    guest ok = no
    force user = root
    force group = root
    veto files = /.apdisk/.DS_Store/.TemporaryItems/.Trashes/desktop.ini/ehthumbs.db/Network Trash Folder/Temporary Items/Thumbs.db/
    delete veto files = yes
```
上述部分参数说明如下：
`aio read/write size`，If this integer parameter is set to a non-zero value, Samba will read from files asynchronously when the request size is bigger than this value. Note that it happens only for non-chained and non-chaining reads and when not using write cache.
The only reasonable values for this parameter are 0 (no async I/O) and 1 (always do async I/O).

`aio write size`: If this integer parameter is set to a non-zero value, Samba will write to files asynchronously when the request size is bigger than this value. Note that it happens only for non-chained and non-chaining reads and when not using write cache.
The only reasonable values for this parameter are 0 (no async I/O) and 1 (always do async I/O).

`use sendfile`: If this parameter is yes, and the sendfile() system call is supported by the underlying operating system, then some SMB read calls (mainly ReadAndX and ReadRaw) will use the more efficient sendfile system call for files that are exclusively oplocked. This may make more efficient use of the system CPU's and cause Samba to be faster. Samba automatically turns this off for clients that use protocol levels lower than NT LM 0.12 and when it detects a client is Windows 9x (using sendfile from Linux will cause these clients to fail).

``


# 参考资料
[https://github.com/dockur/samba](https://github.com/dockur/samba)
[https://www.cnblogs.com/tcicy/p/8465071.html](https://www.cnblogs.com/tcicy/p/8465071.html)
[https://www.cnblogs.com/longchang/p/10734193.html](https://www.cnblogs.com/longchang/p/10734193.html)
[https://www.cnblogs.com/kevingrace/p/8662088.html](https://www.cnblogs.com/kevingrace/p/8662088.html)
[https://wiki.amahi.org/index.php/Make_Samba_Go_Faster](https://wiki.amahi.org/index.php/Make_Samba_Go_Faster)
[https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)

