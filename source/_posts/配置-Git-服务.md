---
title: 配置 Git 服务
date: 2015-12-31 16:41:55
tags: git
---

>配置 Git 服务，创建 Git 仓库

## 客户端密钥

>每个需要使用 Git 的客户端都需要生成一份密钥, 首先去 ~/.ssh 查看自己是否已经有密钥

```bash
$ cd ~/.ssh/
$ ls
authorized_keys   id_dsa       known_hosts
config            id_dsa.pub
```
>默认命名为 id_dsa / id_rsa 个人文件夹目录 ~/.ssh/

>其中一个带有 .pub 扩展名， .pub 文件是你的公钥，另一个则是私钥。 如果找不到这样的文件（或者根本没有 .ssh 目录），你可以通过运行 ssh-keygen 程序来创建它们。


### 生成密钥

<!-- more -->
```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):  //你可以选择保存文件的位置或者直接回车使用默认位置
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):  // 如果不想使用时输入密码 直接按回车
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
08:5e:fe:16:c8:e9:4f:65:4d:dd:2d:01:a1:1f:7b:f9
```

>生成了客户端自己的一对密钥于 ~/.ssh/id_rsa

## 服务端设置

### 添加新用户

>添加为了 git 的用户来仅创建或修改 Git 仓库相关的内容，禁止该用户登录和其他操作

#### 首先登录 root 用户，创建该用户
```bash
sudo adduser git
```

#### 切换至 git 用户
```bash
su git
```

#### 创建 authorized_keys 并加入需要使用 Git 仓库客户端的 公钥
```bash
cd 
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

>将我们信任的开发者密钥(即第一步客户端生成的密钥)加入 authorized_keys 中，多个用户可多次添加至 authorized_keys 文件中
```bash
cat /tmp/zacklee.pub >> ~/.ssh/authorized_keys
```

#### 保证 git 用户安全 (仅可使用 Git 服务)

>切换回 root 用户，为 git 用户创建密码并禁止 git 用户的活动范围

```bash
su root
```

> 为 git 用户创建密码 (客户端使用 Git 服务时需要使用该密码，或将其他客户端公钥加入服务端实现免密码)

```bash
passwd git
```

>为了保证 git 用户安全（仅使用 git 服务）借助一个名为 git-shell 的受限 shell 工具，你可以方便地将用户 git 的活动限制在与 Git 相关的范围内。

首先查看 是否有 git-shell，若没有则继续添加
```bash
$ cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/bin/zsh
```

查询 git-shell 所在目录
```bash
$ which git-shell
/usr/bin/git-shell


```
添加 git-shell 目录行余文件尾
```bash
sudo vim /etc/shells
```

>加入 git-shell 所在目录 /usr/bin/git-shell 至 /etc/shells 文件尾

修改用户的 shell

```bash
sudo chsh git /usr/bin/git-shell # git-shell 路径
```

>这样，git 用户就只能利用 SSH 连接对 Git 仓库进行推送和拉取操作，而不能登录机器并取得普通 shell。 如果试图登录，你会发现尝试被拒绝。


### 创建新仓库

>使用 root (或非 git 用户) 登录并创建新仓库，这里我们将所有 Git 仓库放置于 /opt/git/ 文件夹下，创建名为 project.git 的 Git 仓库并初始化一个空仓库

```bash
$ cd /opt/git
$ mkdir project.git
$ cd project.git
git init --bare
Initialized empty Git repository in /opt/git/project.git/
```

>为 git 用户创建修改本仓库的相关权限

```bash
sudo chown -R git:git /opt/git/project.git
```

## 客户端初始化 Git 仓库

>所有信任的客户端都应先将公钥添加至服务端，即使我们已经为 Git 服务单独创建名为 git 的用户，我们也应该保证尽量不暴露 git 用户的密码

>创建项目文件夹 myproject，初始化并提交至 project.git 
```bash
cd myproject
git init
git add .
git commit -m 'initial commit'
git remote add origin git@服务端地址:/opt/git/project.git
git push origin master #推送代码至 master
```

## Clone 并参与项目

>其他客户端参与项目，Clone project.git 并提交新代码

```bash
git clone git@服务端地址:/opt/git/project.git
cd project
cat "readme" > README
git add README
git commit -m 'fix for the README file'
git push origin master
```