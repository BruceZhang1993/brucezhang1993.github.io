+++
category = 'Linux'
title = '[Linux] 给 Arch Linux 配置启动画面 (UEFI)'
date = '2019-01-09T00:00:00+08:00'
tags = ['Arch', 'Linux', 'Plymouth']
cover = '/assets/img/20181207/mojave.png'
draft = false
toc = false
backtotop = true
+++

Plymouth 是来自 Fedora 社区的项目，目标是提供美化内核启动过程的功能，配置前请先前往[官网](http://fedoraproject.org/wiki/Releases/FeatureBetterStartup)了解此项目，本文参考了 [Arch Wiki](https://wiki.archlinux.org/index.php/Plymouth) 和 Reddit 及 StackOverflow 上的一些帖子上提供的解决方案。


## 0. 准备工作

Plymouth 使用 KMS 启动图形界面，在 UEFI 启动环境下，Plymouth 会使用 EFI Framebuffer，如果你无法使用 EFI Framebuffer 你可以使用 Uvesafb 方案替代，本文首先介绍 UEFI 启动环境下的配置。

## 1. 安装

首先，你需要在 Arch Linux 下安装 `plymouth` 软件包，该软件包目前在 [plymouth](https://aur.archlinux.org/packages/plymouth/)<sup>AUR</sup> 中提供，你也可以使用 [Arch Linux CN 源](https://www.archlinuxcn.org/)，并使用 `pacman -S plymouth` 直接安装它。

    GDM 用户可以直接安装 `gdm-plymouth`，它包含了 plymouth 支持的 GDM

## 2. 配置钩子

Plymouth 在 KMS 阶段就启用图形界面，所以你需要配置钩子才能使用它。

首先打开 `/etc/mkinitcpio.conf` ，找到 HOOKS 一行，检查是否包含 **systemd** 钩子。如果你正在使用 systemd 钩子，添加 `sd-plymouth` 钩子如下

    HOOKS=(base systemd sd-plymouth ...)

如果你没有使用 systemd 钩子，请使用 `plymouth` 钩子来替代 sd-plymouth，并添加在 base udev 之后。

如果你还使用了 encrypt 磁盘加密钩子，你可能还需要使用 `plymouth-encrypt` 或 `sd-encrypt` 带替代它。

## 3. 配置内核参数

接下来，你需要在你的引导加载器 (eg. Grub2,Systemd-boot,EFISTUB) 中添加下面的内核参数

    quiet splash loglevel=3 rd.udev.log-priority=3 vt.global_cursor_default=0 rd.systemd.show_status=false vga=current

如果你使用 Intel Graphics 你可能还需要添加

    i915.fastboot=1

在启动过程中，你的 UEFI 环境可能会提供品牌启动 LOGO 覆盖，如果看不惯的话，你可以添加下面的参数来禁用它

    fbcon=nodefer

配置全部完成以后，执行 `mkinitcpio -p linux` 来重新生成你的 initrd (如果使用其他内核请对应替换)

*如果你使用 Grub2 启动，修改 `/etc/default/grub` 后请别忘记 grub-mkconfig
*如果你使用 systemd-boot 请修改 `/boot/loader/entries` 下的启动项
*如果你使用 EFISTUB 你应该是个~~大佬~~吧

## 4. 配置

如果你使用 KDE 桌面环境，你可以安装 **plymouth-kcm** 它提供了简单的启动画面配置界面。

如果要实现 plymouth 到图形界面的平滑过渡，你需要先禁用你的 DM 例如

```bash
systemctl disable sddm.service # SDDM
systemctl disable gdm.service # GDM
# ...
```

然后你需要启用 plymouth 提供的对应 DM Unit

```bash
systemctl enable sddm-plymouth.service # SDDM
systemctl enable gdm-plymouth.service # GDM
# ...
```

然后你需要打开你的 **/etc/plymouth/plymouthd.conf** 修改配置如下

    [Daemon]
    Theme=spinner
    ShowDelay=0
    DeviceTimeout=10

## 5. 主题和美化

AUR 中提供了一些 plymouth 主题，我们可以直接搜索安装并配置它们。

例如，我们选择 plymouth-theme-arch-breeze-git 首先从 AUR 安装它，然后我们重新修改 **/etc/plymouth/plymouthd.conf**

    [Daemon]
    Theme=arch-breeze
    ...

你也可以在 KDE 系统设置 -- 开机和关机 -- 启动画面 里面直接设置它。
