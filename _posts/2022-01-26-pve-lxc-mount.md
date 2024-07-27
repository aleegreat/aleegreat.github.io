---
title: "在基于proxmox上的LXC中挂载远程目录"
last_modified_at: 2022-01-27
categories: [IT, PVE] 
tags: [PVE, LXC, Mount] 
---


# 背景
我们有一台PVE，PVE上有个LXC容器（我的是Archlinux），现在我需要把远程目录挂载到Archlinux中。

# 思路
1. LXC是特权容器：按Linux常规挂载即可
2. LXC不是特权容器：远程目录挂载到PVE，通过PVE和LXC目录映射

# 方案（仅思路2）
1. 获取Archllinux中用户的uid、gid（一般为1000，1000）
2. 获取PVE与LXC中uid、gid映射起始值
    ```bash
    cat /var/lib/lxc/your_lxc_id/config | grep idmap
    lxc.idmap = u 0 100000 65536
    lxc.idmap = g 0 100000 65536
    ```

3. mount命令中的uid、gid 等于 lxc 容器中uid、gid （即1000，1000）加上 pve与lxc映射值（即100000，100000）等于 101000，101000
4. 在pve中mount好远程目录
    ```bash
    sudo mount.cifs -o "rw,dir_mode=0777,file_mode=0777,username=yoursmbusername,password=yoursmbpassword,uid=101000,gid=101000" //remote_ip //mount_point
    ```

5.  在LXC容器中创建好映射目录
6. 修改LXC容器的配置文件 `nano /etc/pve/lxc/yourlxcid.conf` ，新增一行
    ```bash
    lxc.mount.entry: /pve_mount_point  lxc_mount_point none rw,bind 0 0
    ```
注意，lxc_mount_point 是相对根目录路径


