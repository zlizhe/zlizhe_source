---
title: VIM 配色方案 Molokai
date: 2015-07-09 22:01:17
tags: [vim, molokai]
---

修改 VIM 配置文件来支持本本色方案

```bash
sudo vim ~/.vimrc
```

添加以下配置

<!-- more -->
```bash
syntax enable

syntax on

set nu

set t_Co=256

set background=dark

colorscheme molokai
```

下载配色方案 https://github.com/tomasr/molokai/blob/master/colors/molokai.vim
