---
title: 记录-DockerCE安装
date: 2017-09-06 09:52:58
tags: Docker
---
<img src="https://i.imgur.com/EN8O4bt.png" width="680"/>

配置如下:
Ubuntu 16.04LTS(64位版本)

本文的大部分步骤来自于docker的官方文档

> https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/

<!-- more -->

**Docker是一种容器化的一种技术**

由于docker目前分为了两个版本一个是商业版，一个是社区版，因此这里我们选择的是社区版也就是CE版本。

##### Docker是一种类似虚拟机的一种技术，但是比虚拟机要轻量的多。
#### 1.docker ce安装的运行环境
DockerCE的安装版本的要求
* Zesty 17.04
* Xenial 16.04 (LTS)
* Trusty 14.04 (LTS)

如果你的系统是14.04版本则需要安装linux-image-extra-*，因为docker需要这两个包。

    sudo apt-get update

    sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual

#### 2.删除之前版本的docker
之前版本的docker叫做docker或者docker-engine，如果之前安装过，卸载它们

    sudo apt-get remove docker docker-engine docker.io

现在我们安装的docker的包名叫做**docker-ce**

#### 3.安装docker-ce
1.更新包的索引

    sudo apt-get update

2.安装允许让apt允许的ssl仓库的包(ps:\代表这句脚本没有写完，进行换行)

    sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

3.添加Docker GPG Key

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

4.设置仓库地址,这里设置的x86的仓库地址

	sudo add-apt-repository \
	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	   $(lsb_release -cs) \
	   stable"


5.安装
ubuntu14.04以上的版本都是自带docker安装包的,所以可以直接安装,但是这个一般不是最新版本,所以一般需要进行更新源

	sudo apt-get update
	sudo apt-get install docker.io

2.安装完成了之后可以通过以下命令进行查看安装版本
	docker -v

3.下载docker的镜像

	docker run ubuntu:15.10 /bin/echo "Hello world"


##### 重点来了！！镜像地址默认是国外的dockerhub，国内速度较慢，因此需要一个国内镜像去加速，这里使用的是的DaoCloud

如果本地没有ubuntu 15.10的镜像，那么docker会去从仓库中下载，但是下载的速度很慢。因此这里使用了Docker镜像的加速器

https://www.daocloud.io/mirror#accelerator-doc

通过下面的命令即可完成配置加速器。

	curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://d606b909.m.daocloud.io