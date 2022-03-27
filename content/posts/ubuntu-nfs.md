---
title: "Ubuntu Nfs"
date: 2022-01-18T21:42:21+08:00
tags: [""]
categories: [""]
series: [""]
summary: "Summary todo"
draft: true
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---



server

```shell
sudo apt update
sudo apt install nfs-kernel-server
```

client

```shell
sudo apt update
sudo apt install nfs-common
```

server

```shell
sudo mkdir /var/nfs/general -p
```

```shell
sudo chown nobody:nogroup /var/nfs/general
```

```shell
sudo nano /etc/exports
```

```shell
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
```

```shell
sudo systemctl restart nfs-kernel-server
```

client

```shell
sudo mkdir -p /nfs/general
sudo mkdir -p /nfs/home
```

```shell
sudo mount host_ip:/var/nfs/general /nfs/general
sudo mount host_ip:/home /nfs/home
```



```
sudo apt update ; sudo apt install nfs-common -y ; sudo mkdir -p /nfs/data ; sudo mount 114.116.9.65:/nfs/data /nfs/data
```

```
114.116.9.65:/nfs/data    /nfs/data   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```



## 参考

1. [How To Set Up an NFS Mount on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)