---
title: ubuntu 环境搭建
date: 2018-08-01 23:25:31
tags: ubuntu
---

以下是我每次安装 ubuntu 都会搭建的开发/娱乐环境，请读者参考。下面的内容比较精简，我主要是点出有这个东西，并且是我自己必备的东西，肯定也是蛮好用的，具体是什么，怎么用，网上已经有很多比我写的很好的轮子了，我就不重造了。另外如果你有好的推荐，也请告知我，我是工具/效率控，也想继续提升。

![ubuntu环境搭建](http://oi435vw1u.bkt.clouddn.com/ubuntu.png)

## 1. 编辑器

- neovim（更快的vim ?）

  ```
  sudo apt-get install ninja-build gettext libtool libtool-bin autoconf automake cmake g++ pkg-config unzip
  git clone https://github.com/neovim/neovim.git
  make -j8
  make install
  
  ## 我最后使用的是 apt 安装，现在住的地方（颠沛流离啊）网络实在不行，会卡在某些下载处。
  sudo apt-get install neovim
  sudo apt-get install xclip
  sudo apt-get install xclip
  pip install neovim
  pip3 install neovim
  ```

- SpaceVim（neovim / vim 强大配置）

  ```
  curl -sLf https://spacevim.org/install.sh | bash
  ```
  关于 SpaceVim 要说的很多，后续专门成文说明。

- tmux（分屏工具）

  ```
  sudo apt-get -y install libevent-dev libncurses-dev
  git clone https://github.com/tmux/tmux.git
  cd tmux
  sh autogen.sh
  ./configure && make
  
  # 去除<200b>[我从《tmux 2 Productive Mouse-Free Development》这本书拷贝配置的时候遇到这个问题]
  sed -i 's/\xe2\x80\x8b//g' inputfile
  ```

- typora

  markdown写作神器，跨各种平台，推荐！

  ```
  # optional, but recommended
  # sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv BA300B7755AFCFAE
  # add Typora's repository
  sudo add-apt-repository 'deb https://typora.io/linux ./'
  sudo apt-get update
  # install typora
  sudo apt-get install typora
  ```

## 2. 命令行

- zsh + oh_my_zsh（zsh 及配置）

  ```
  sudo apt-get -y install zsh
  sudo apt -y install curl
  sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
  ```
- fzf（模糊搜索历史命令、文件等）

  ```
  git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
  ~/.fzf/install
  ```
- z（快速跳转目录）

  ```
  git clone https://github.com/rupa/z.git
  echo ". ~/tools/z/z.sh" >> ~/.zshrc
  source ~/.zshrc
  ```

- ag（快速搜索）

  ```
  sudo apt-get install -y automake pkg-config libpcre3-dev zlib1g-dev liblzma-dev
  cd the_silver_searcher && ./build.sh
  make -j8 && sudo make install
  ```

## **3. 工具**

- 搜狗输入法

- VNote（笔记软件）

  本来有道云笔记很好用，可是就是没有 linux 版本…

  VNote 跨三种平台，并且支持 vi 模式。

  结合下面的坚果云，可以实现同步。

- 坚果云（同步文件）

  ```
  wget https://www.jianguoyun.com/static/exe/installer/ubuntu/nautilus_nutstore_amd64.deb
  sudo apt-get install gdebi
  sudo gdebi nautilus_nutstore_amd64.deb
  
  # SSL 连接错误解决方案
  sudo apt purge 'openjdk-*'
  [ -d /usr/lib/jvm ] && sudo mv /usr/lib/jvm /usr/lib/jvm-backup
  sudo apt install -y openjdk-8-jre
  ```

- Xmind-ZEN

  思维导图，免费版就够用了，不过导出图片就有水印，土豪请支持正版。

  效果请看本文第一图。

## 4. 科学上网

- vps

  科学上网可以选择购买市面上的 vpn，简单方便，不过随时面临倒闭的风险，毕竟是国家重点照顾的领域。我相信在看这篇文章的朋友大部分是程序员，至少应该是 IT 从业者，所以我觉得应该自己购买 vps 搭梯子，多折腾。弄好了

- 安装Shadowsocks-qt5

  ```
  wget https://github.com/shadowsocks/shadowsocks-qt5/releases/download/v3.0.1/Shadowsocks-Qt5-3.0.1-x86_64.AppImage
  vpsIP
  ss端口
  ss密码
  aes-256-cfb（ss加密方式）
  1080
  ```

- 代理

  ```
  sudo apt-get install python-pip
  sudo pip install genpac
  genpac --pac-proxy "127.0.0.1:1080" --output="autoproxy.pac"
  Network proxy 设置: file:///home/longhai/autoproxy.pac
  ```

## 5. 博客

- hexo

  简单，可定制，需折腾

  ```
  sudo apt-get install npm
  sudo npm install hexo-cli -g
  sudo npm install hexo -g
  
  # github 上创建名为 username.github.io 的空库（username为github账户）
  git clone https://github.com/longhaiqwe/longhaiqwe.github.io.git
  
  mkdir -p ~/blog/hexo
  cd hexo
  hexo init
  npm install hexo-generator-index –save 
  npm install hexo-generator-index --save 
  npm install hexo-generator-archive --save
  npm install hexo-generator-category --save
  npm install hexo-generator-tag  --save
  npm install hexo-server --save
  npm install hexo-deployer-git --save
  npm install hexo-deployer-heroku --save
  npm install hexo-deployer-rsync --save
  npm install hexo-deployer-openshift --save
  npm install hexo-deployer-rsync --save
  npm install hexo-renderer-marked --save
  npm install hexo-renderer-stylus
  npm install hexo-renderer-stylus --save
  npm install hexo-generator-feed 
  npm install hexo-generator-feed  --save
  npm install hexo-generator-sitemap  --save
  
  ## 或者可以使用如下一条命令解决：
  npm install
  
  ## 并且好像上面那一条语句也可以不用执行～
  
  ## Quick Start
  hexo new "My New Post"
  hexo server
  hexo generate
  hexo deploy
  ```
