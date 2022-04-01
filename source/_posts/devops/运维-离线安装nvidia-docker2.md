---
title: 运维-离线安装nvidia-docker2
date: 2022-03-31 16:36:56
index_img: /img/cover/DevOps.png
cover: /img/cover/DevOps.png
tags:
- Ubuntu
categories:
- 运维
- nvidia-docker2
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---
# 离线安装nvidia-docker2

> 容器中要用GPU资源，就需要安装NVIDIA Container Toolkit，按照官网的步骤很容易安装，但如果不能连外网，就需要离线安装
> 
- [ ]  安装好Docker
- [ ]  安装好nvidia driver
### 查看系统版本

```bash
root@dncloud-suat:/home/dntech/tools#  lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.6 LTS
Release:	18.04
Codename:	bionic
```

### 下载安装包

#### 下载安装包libnvidia-container1_1.9.0-1_amd64.deb

[libnvidia-container/stable at gh-pages · NVIDIA/libnvidia-container](https://github.com/NVIDIA/libnvidia-container/tree/gh-pages/stable)

#### 下载安装包libnvidia-container-tools_1.9.0-1_amd64.deb

[libnvidia-container/stable at gh-pages · NVIDIA/libnvidia-container](https://github.com/NVIDIA/libnvidia-container/tree/gh-pages/stable)

#### 下载安装包nvidia-container-runtime_3.5.0-1_amd64.deb

[nvidia-container-runtime/stable/ubuntu18.04/amd64 at gh-pages · NVIDIA/nvidia-container-runtime](https://github.com/NVIDIA/nvidia-container-runtime/tree/gh-pages/stable/ubuntu18.04/amd64)

#### 下载安装包nvidia-container-toolkit_1.5.1-1_amd64.deb

[nvidia-container-runtime/stable/ubuntu18.04/amd64 at gh-pages · NVIDIA/nvidia-container-runtime](https://github.com/NVIDIA/nvidia-container-runtime/tree/gh-pages/stable/ubuntu18.04/amd64)

#### 下载安装包nvidia-docker2_2.6.0-1_all.deb

[nvidia-docker/ubuntu18.04/amd64 at gh-pages · NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker/tree/gh-pages/ubuntu18.04/amd64)

### 安装deb包 顺序安装

```bash
 dpkg -i libnvidia-container1_1.9.0-1_amd64.deb 
 dpkg -i libnvidia-container-tools_1.9.0-1_amd64.deb
 dpkg -i nvidia-container-toolkit_1.5.1-1_amd64.deb
 dpkg -i nvidia-container-runtime_3.5.0-1_amd64.deb
 dpkg -i nvidia-docker2_2.6.0-1_all.deb

```

### 修改配置文件重启docker 服务

```bash
➜  ~ cat /etc/docker/daemon.json
{
 "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

#重启
systemctl restart docker
```

### 测试

```bash
➜  ~ docker run --rm nvidia/cuda:11.1-base-ubuntu20.04 nvidia-smi
Thu Mar 31 16:28:03 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.67       Driver Version: 460.67       CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro RTX 4000     Off  | 00000000:01:00.0 Off |                  N/A |
| 30%   33C    P8    15W / 125W |   7682MiB /  7973MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```