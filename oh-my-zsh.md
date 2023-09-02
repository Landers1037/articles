---
title: oh-my-zsh配置使用
name: oh-my-zsh
date: 2021-07-14 01:41:14
id: 0
tags: [zsh,shell,linux]
categories: [zsh]
abstract: ""
---

在Mac上配置使用`oh-my-zsh`

![](/images/zsh1.png)

<!--more-->

## 前置条件

- zsh
- git
- brew
- item2可选

## 安装oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## 安装主题

默认的主题比较单调 可以使用自定义的主题

安装spaceship主题

```bash
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
```

将克隆下来的主题软连接到 zsh 的主题文件夹中

```bash
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

修改`~/.zshrc`中的主题名

```bash
ZSH_THEME="spaceship"
```

在命令行输入`source ~/.zshrc` 让`zshrc`生效。

或者使用`agnoster-fcamblor`主题

```bash
git clone https://github.com/fcamblor/oh-my-zsh-agnoster-fcamblor.git
cd oh-my-zsh-agnoster-fcamblor/
./install
```

## 安装插件

### AutoJump

根据历史的访问记录快速访问目标目录

命令行安装：

```bash
brew install autojump
```

将 autojump 放入`~/.zshrc`中的插件设置中：

```bash
plugins=(git autojump)
```

在`~/.zshrc`的结尾添加：

```bash
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
```

### zsh-syntax-highlighting

命令语法高亮插件，错的命令显示红色，对的命令显示绿色

克隆项目到 Oh My Zsh 的插件文件夹里面：

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

将 zsh-syntax-highlighting 放入`~/.zshrc`中的插件设置中：

```bash
plugins=(git autojump zsh-syntax-highlighting)
```

### zsh-autosuggestions

命令补全插件，会显示出你用过的命令

克隆项目到 Oh My Zsh 的插件文件夹里面：

```bash
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

将 zsh-autosuggestions 放入`~/.zshrc`中的插件设置中：

```bash
plugins=(git autojump zsh-syntax-highlighting zsh-autosuggestions)
```

在命令行输入`source ~/.zshrc` 让`zshrc`生效。

## 安装字体

默认的一些图标不能显示全 所以安装powerline的字体

[powerline fonts](https://github.com/powerline/fonts)

```bash
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

在iterm2中将字体改为powerline字体即可

## 自定义主题spaceship

官方配置文档

[spaceship-prompt/options.md at master · spaceship-prompt/spaceship-prompt (github.com)](https://github.com/spaceship-prompt/spaceship-prompt/blob/master/docs/options.md)

常用的配置

设置显示时间

`SPACESHIP_TIME_SHOW="true"`

设置显示用户名

`SPACESHIP_USER_SHOW="always"`

用户名颜色

`SPACESHIP_USER_COLOR="212"`

## 警告信息

source配置文件的时候可能出现一些警告

```bash
Insecure completion-dependent directories detected:
[oh-my-zsh] For safety, we will not load completions from these directories until

[oh-my-zsh] you fix their permissions and ownership and restart zsh.
```

给予足够的权限

```bash
chmod 755 /usr/local/share/zsh

chmod 755 /usr/local/share/zsh/site-functions
```

添加`ZSH_DISABLE_COMPFIX=true`

## 效果

![](/images/zsh2.png)

## 参考

[知乎]([打造一个更好用的 Mac —— Terminal 篇 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/162863654))