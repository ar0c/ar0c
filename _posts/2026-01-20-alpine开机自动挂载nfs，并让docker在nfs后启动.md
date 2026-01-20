---
share: true
sharetitle: 2026-01-20-alpine开机自动挂载nfs，并让docker在nfs后启动
date modified: 2026-01-20
title: alpine开机自动挂载nfs，并让docker在nfs后启动
tags:
  - system/linux/alpine
  - nfs
  - docker
date created:
  - 2026-01-12
---
自动挂载nfs的要点在于，启动顺序需要保证networking服务优先运行

对于alpine，/etc/fstab中没有启动顺序控制功能，因此需要使用启动脚本执行

# /etc/fstab

```
192.168.0.10:/volume1/appdata  /mnt/appdata  nfs  rw,vers=4.1,_netdev,noatime,hard,timeo=600,retrans=2,nofail  0  0
```

# 服务启动顺序

```sh
rc-update add nfsmount boot
rc-update add networking boot
rc-update add netmount boot
```

# 控制docker顺序

vim /etc/init.d/docker

```
# /etc/init.d/docker
depend() {
    need sysfs cgroups networking
    use dns logger
    # 添加netmount
    after firewall netmount
}
```

```config
# /etc/conf.d/docker
rc_need="netmount"
```
