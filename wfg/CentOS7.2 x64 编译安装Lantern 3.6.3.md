# CentOS7.2 x64 编译安装Lantern 3.6.3

[TOC]

CentOS7 x64 编译安装Lantern 3.6.3
操作系统：CentOS-7-x86_64-DVD-1511
软件：https://github.com/getlantern/lantern

```
Building Lantern Prerequisites
Custom fork of Go is currently required. We'll eventually switch to Go 1.7 which supports what we need due to this.
An OSX or Linux host. Building on Windows is only partially supported with the help of Cygwin.
Git - brew install git, apt-get install git, etc
GNU Make
Nodejs & NPM
GNU C Library (linux only) - apt-get install libc6-dev-i386, etc
Gulp - npm i gulp-cli -g
```

## 防火墙和selinux

```shell
禁用selinux，firewalld无所谓
[root@localhost ~]# getenforce
Disabled
[root@localhost ~]# systemctl status firewalld|grep Active
   Active: active (running) since Sat 2017-09-30 13:30:17 CST; 9min ago
```

## 准备编译环境

```shell
[root@localhost ~]# yum install -y epel-release
Installed:
  epel-release.noarch 0:7-9
[root@localhost ~]# yum install -y git  
Installed:
  git.x86_64 0:1.8.3.1-12.el7_4
[root@localhost ~]# yum groupinstall -y "Development Tools"

```

## 安装二进制go1.7.5

```shell
[root@localhost ~]# cd /usr/local/src
[root@localhost src]# wget https://storage.googleapis.com/golang/go1.7.5.linux-amd64.tar.gz
[root@localhost src]# tar zxvf go1.7.5.linux-amd64.tar.gz
[root@localhost src]# mv go ~/go1.4
[root@localhost src]# cd ~
[root@localhost ~]# export GOROOT=$HOME/go1.4
[root@localhost ~]# export PATH=$PATH:$GOROOT/bin
[root@localhost ~]# go version
go version go1.7.5 linux/amd64
[root@localhost ~]# which go
/root/go1.4/bin/go
```

## 安装Custom fork of Go

```shell
[root@localhost ~]# pwd
/root
[root@localhost ~]# yum install -y glibc-static
Installed:
  glibc-static.x86_64 0:2.17-196.el7
[root@localhost ~]# git clone  https://github.com/getlantern/go.git
[root@localhost ~]# cd go/src  
[root@localhost src]# ./all.bash 
#编译成功会显示 all tests passed
##### API check
Go version is "go1.6.2_lantern_20160503", ignoring -next /root/go/api/next.txt
ALL TESTS PASSED
---
Installed Go for linux/amd64 in /root/go
Installed commands in /root/go/bin
*** You need to add /root/go/bin to your PATH.
[root@localhost src]# cd ~
[root@localhost ~]# ls
anaconda-ks.cfg  go  go1.4
退出当前shell，重新登录
[root@localhost ~]# exit

备注：使用export设定的环境变量，以下的操作都要在该shell环境中执行
如果出错检查go的版本，确定使用的是go1.6.2_lantern_20160503
[root@localhost ~]# export GOROOT=$HOME/go
[root@localhost ~]# export PATH=$PATH:$GOROOT/bin
[root@localhost ~]# go version
go version go1.6.2_lantern_20160503 linux/amd64
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/root/go/bin

```

## Nodejs & NPM

```shell
[root@localhost ~]# yum install -y nodejs npm --enablerepo=epel
Installed:
  nodejs.x86_64 1:6.11.3-1.el7
[root@localhost ~]# yum install -y openssl
Updated:
  openssl.x86_64 1:1.0.2k-8.el7
[root@localhost ~]# yum provides \*/libgcc_s.so  
[root@localhost ~]# npm i gulp-cli -g
[root@localhost ~]# yum install -y libappindicator-devel.x86_64 libappindicator-gtk3-devel.x86_64
Installed:
  libappindicator-devel.x86_64 0:12.10.0-11.el7
[root@localhost ~]# yum install -y glibc.i686 libgcc.i686 glibc-devel.i686
Installed:
  glibc.i686 0:2.17-196.el7                                                              glibc-devel.i686 0:2.17-196.el7                                                              libgcc.i686 0:4.8.5-16.el7
```

## 编译Lantern

```shell
[root@localhost src]# git clone https://github.com/getlantern/lantern.git
[root@localhost src]# cd lantern/
[root@localhost lantern]# git checkout -b my-3.6.3 3.6.3     //使用当前最新的版本
Switched to a new branch 'my-3.6.3'
[root@localhost lantern]# npm i gulp-cli -g
[root@localhost lantern]# npm update minimatch@3.0.2 -g
[root@localhost lantern]# npm i gulp-clean-css -g
[root@localhost lantern]# npm i minimatch@3.0.2 -g


[root@localhost lantern]# HEADLESS=true make linux
编译能成功，但有如下警告
npm WARN deprecated gulp-minify-css@1.2.4: Please use gulp-clean-css
npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated minimatch@0.2.14: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated graceful-fs@1.2.3: graceful-fs v3.0.0 and before will fail on node releases >= v7.0. Please update to graceful-fs@^4.0.0 as soon as possible. Use 'npm ls graceful-fs' to find it in the tree.
npm WARN deprecated graceful-fs@2.0.3: graceful-fs v3.0.0 and before will fail on node releases >= v7.0. Please update to graceful-fs@^4.0.0 as soon as possible. Use 'npm ls graceful-fs' to find it in the tree.
npm WARN deprecated node-uuid@1.4.8: Use uuid module instead

重编译前的准备
[root@localhost lantern]# npm i gulp-clean-css -g
/usr/lib
└── gulp-clean-css@3.9.0
[root@localhost lantern]# npm i minimatch -g
/usr/lib
└── minimatch@3.0.4
[root@localhost lantern]# npm ls graceful-fs
/usr/local/src/lantern
└── (empty)

npm ERR! code 1
[root@localhost lantern]# npm i graceful-fs -g
/usr/lib
└── graceful-fs@4.1.11
重新编译
[root@localhost lantern]# make clean
[root@localhost lantern]# HEADLESS=true make linux
[root@localhost lantern]# ls lantern*
lantern_linux_386  lantern_linux_amd64
[root@localhost lantern]# cp ./{lantern_linux_386,lantern_linux_amd64} /usr/local/bin/
使用方法
[root@localhost ~]# lantern_linux_amd64 -addr 0.0.0.0:8787 -startup=false -proxyall
or
[root@localhost ~]# lantern_linux_amd64 -addr 0.0.0.0:8787
测试
[root@localhost ~]# curl -ivx 127.0.0.1:8787 https://www.google.com
git使用代理
[http]
    proxy = http://mydomain\\myusername:mypassword@myproxyserver:8080
# vim .gitconfig
[http]
    proxy = http://127.0.0.1:8787
```