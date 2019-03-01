---
layout: post
title: nvidia-docker安装
date: 2019-03-01
categories: blog
---

转载自七星

参考资料：https://www.cnblogs.com/zzcit/p/5845717.html

## 安装说明
### 本文档说明下列系统下安装nvidia-docker   
Ubuntu Trusty 14.04 (LTS)   
Ubuntu Xenial 16.04 (LTS)

## 安装docker
### 更新apt源
#### 更新安装包信息
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
```
#### 添加新的GPGkey
```
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```
#### 修改apt sources
```
sudo vim /etc/apt/sources.list.d/docker.list
```
#### 根据系统版本添加如下内容
##### Ubuntu Trusty 14.04 (LTS)
```
deb https://apt.dockerproject.org/repo ubuntu-trusty main
```
##### Ubuntu Xenial 16.04 (LTS)
```
deb https://apt.dockerproject.org/repo ubuntu-xenial main
```
#### 更新apt软件索引
```
sudo apt-get update
```
#### 清除旧的repo
```
sudo apt-get purge lxc-docker
```
#### 确保apt是从正确的代码库拉取下来
```
apt-cache policy docker-engine
```

### 安装docker
```
sudo apt-get install docker-engine
```
#### 开启docker
```
sudo service docker start
```
#### 确认docker被正确安装
```
sudo docker run hello-world
如果正确安装，会有下载镜像并运行的输出
```

### 创建docker group
#### 创建docker用户组
为了不使用sudo也能正确使用docker,需要创建docker用户组，并将当前用户加进去
```
sudo groupadd docker
sudo usermod -aG docker $USER
```
#### 确认不使用sudo可以运行docker
```
docker run hello-world
```

### 安装nvidia-docker
为了在docker环境中能够使用GPU资源，需要安装nvidia-docker   
#### 安装包下载地址   
```
https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
```
#### 安装
```
sudo dpkg -i   nvidia-docker*
```
#### 建立用户组
```
sudo groupadd nvidia-docker
sudo usermod -aG nvidia-docker $USER
```
