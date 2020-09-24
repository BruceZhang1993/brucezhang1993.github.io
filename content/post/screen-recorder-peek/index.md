+++
category = 'Linux'
title = '[Linux] 可能是最好用的屏幕录制软件 -- Peek'
date = '2017-11-29T00:00:00+08:00'
tags = ['linux', 'recorder', '录屏']
image = '0855.png'
draft = false
toc = false
backtotop = true
+++

本博客的第一篇博文，给大家介绍一款开源屏幕录制软件 -- [Peek](https://github.com/phw/peek){:target="_blank"}，支持直接录制内容并渲染为 GIF 图片，非常适合录制桌面短视频和演示桌面操作，无需录制视频和手动转码操作，方便快捷。

| 项目名称 | Peek |
| --- | --- |
| 项目描述 | an animated GIF recorder |
| 维护者 | [phw](https://github.com/phw) |
| 授权协议 | [GPL-3](https://github.com/phw/peek/blob/master/LICENSE) |
| 下载地址 | <https://github.com/phw/peek/releases> |
| 官方文档 | [README](https://github.com/phw/peek/blob/master/README.md) |

---

## 运行依赖

  - GTK+ >= 3.14
  - GLib >= 2.38
  - libkeybinder3
  - FFmpeg
  - gifski (Improved GIF)

---

## 安装

### Gentoo

  `emerge media-video/peek`

### OpenSUSE Tumbleweed

  `zypper install peek`

### Arch Linux (AUR)

  `yaourt -S peek`

### Ubuntu

  ```shell
  sudo add-apt-repository ppa:peek-developers/stable
  sudo apt update
  sudo apt install peek
  ```

### Debian

  ```shell
  sudo apt install cmake valac libgtk-3-dev libkeybinder-3.0-dev libxml2-utils gettext txt2man
  git clone https://github.com/phw/peek.git
  mkdir peek/build
  cd peek/build
  cmake -DCMAKE_INSTALL_PREFIX=/usr -DGSETTINGS_COMPILE=OFF ..
  make package
  sudo dpkg -i peek-*-Linux.deb
  ```

### Fedora

  **启用** [RPM Fusion](https://rpmfusion.org/Configuration){:target="_blank"} reposity 并安装 FFmpeg.

  ```shell
  sudo dnf config-manager --add-repo http://download.opensuse.org/repositories/home:/Bajoja/Fedora_25/home:Bajoja.repo
  sudo dnf install peek
  sudo dnf install gstreamer1-plugins-ugly
  ```

### Other (From Source)

  使用软件管理器安装运行依赖

  ```shell
  git clone https://github.com/phw/peek.git
  mkdir peek/build
  cd peek/build
  cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
  make

  # Run directly from source
  ./peek

  # Install system wide
  sudo make install
  ```
