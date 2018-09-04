---
title: Oh My Zsh
date: 2015-11-02 22:52:15
tags: [zsh,bash,终端]
---

>MAC 下的默认命令行工具是 bash 但同时也有 zsh 等，通过扩展 zsh 的  Oh My Zsh 来让我们的命令行更加便捷，提供诸如主题、插件等

>zsh是兼容bash的，所以在bash上的绝大部分命令在zsh也是可用的 Oh My Zsh 只是 zsh的扩展，可以在任意终端上使用只需要有zsh支持即可安装该扩展

## 查看 Zsh 版本
```bash
zsh -version
```

输出类似版本号即已安装 zsh
```bash
zsh 5.0.8 (x86_64-apple-darwin15.0)
```

<!-- more -->

若没有安装 zsh 可通过 apt-get 进行安装
```bash
sudo apt-get install zsh
```

## 安装扩展 Oh My Zsh

```bash
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

>没有安装 wget 的可以使用 curl

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

>看到以下画面即为安装成功

```bash
         __                                     __   
  ____  / /_     ____ ___  __  __   ____  _____/ /_  
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \ 
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / / 
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/  
                        /____/                       ....is now installed!


Please look over the ~/.zshrc file to select plugins, themes, and options.

p.s. Follow us at https://twitter.com/ohmyzsh.

p.p.s. Get stickers and t-shirts at http://shop.planetargon.com.
```

## 修改 Oh My Zsh 主题本色

编辑配置文件于本用户目录下

```bash
vim ~/.zshrc
```

编辑主题 找到行 ZSH_THEME

```bash
ZSH_THEME="robbyrussel"
```

>修改 ZSH_THEME 值来修改主题，主题样式参考 https://github.com/robbyrussell/oh-my-zsh/wiki/Themes 编辑完成后新建一个窗口即可看到效果 

## 修改默认终端为 zsh

```bash
chsh -s $(which zsh)
```

重新登录 

```bash
echo $SHELL
```

>若显示为 zsh 即修改成功, 新版本 Oh My Zsh 只需要提供 root 密码即可自动修改

```bash
/bin/zsh
```

