+++
category = 'Linux'
title = '[Linux] 关于 Skylake 的 VP9 解码支持'
date = '2018-11-26T00:00:00+08:00'
tags = ['linux', 'vp9', 'skylake']
draft = false
toc = false
backtotop = true
+++

Intel Skylake 在 Linux 平台一直没有 VP9 解码的支持，于是在 Chromium VAAPI 上也就没法硬件 Youtube 视频，真的很难愉快玩耍啊。然后这篇文章讲讲如何在 Arch Linux 环境下安装 Skylake VP9 解码驱动。

## 0. 测试环境和配置

- Arch Linux
- Intel i5-6200U
- Intel HD Graphics 520

## 1. 安装前

在开始安装之前，如果安装过 intel-media-driver(iHD) 驱动，建议先禁用或卸载，即还原默认的环境变量。

## 2. 安装带 Hybrid 支持的 libva

这个包是用来替换默认的 libva 驱动的，和默认驱动的区别就是默认开启了 hybrid 支持，嘛用上 VP9 解码的第一步～

Arch Linux 可以从 AUR 中安装 [libva-intel-driver-hybrid](https://aur.archlinux.org/packages/libva-intel-driver-hybrid) 这个包。

## 3. 安装 intel-hybrid-codec-driver

然后，我们接着从 AUR 安装 [intel-hybrid-codec-driver](https://aur.archlinux.org/packages/intel-hybrid-codec-driver)，来给我们的~~辣鸡~~ Skylake 插上翅膀！

## 4. 各种测试

注销或重启后，我们在终端输入 `vainfo` 测试，输出结果如下：

```
vainfo: VA-API version: 1.3 (libva 2.3.0)
vainfo: Driver version: Intel i965 driver for Intel(R) Skylake - 2.2.0
vainfo: Supported profile and entrypoints
      ...
      ...
      VAProfileVP9Profile0            : VAEntrypointVLD
```

如果上面的结果中使用的驱动是 i965 并且最后出现 VP9 字样，基本可以确定已经启用 VP9 解码支持。

然后我们可以使用 chrome-vaapi 打开一个 Youtube 视频开始播放，在新的标签页打开 `chrome://media-internals/` 确认视频播放是否使用 GpuDecoder

## 5. 视频性能测试

使用的测试视频为 [https://www.youtube.com/watch?v=1La4QzGeaaQ](https://www.youtube.com/watch?v=1La4QzGeaaQ)，测试使用 mpv 直接播放时长 30s，给定 mpv 参数分别使用软解和混合硬解模式。

| 测试项目 | 性能指标  | 默认参数/软解 | -hwdec=vappi/VP9混合硬解 |
| 1080P   | CPU     |  20-35%     |   <10%          |
|         | 流畅度   |  流畅 (0 Drop) | 流畅 (0 Drop)  |
| 1080P60 | CPU     |  40-60%     |   10-15%        |
|         | 流畅度   |  基本流畅 (35 Drop) | 流畅 (8 Drop) |
| 1440P   | CPU     |  45-55%     |   10-20%        |
|         | 流畅度   |  稍有卡顿 (60 Drop) | 稍有卡顿 (147 Drop) |

**The End**
