+++
category = 'Other'
title = '[Other] 在 Travis CI 上生成、测试并部署 VSCode 扩展'
date = '2019-04-14T00:00:00+08:00'
tags = ['vscode', 'travis']
draft = false
toc = false
backtotop = true
+++

本文主要说明如何使用 Travis CI 生成、测试 VSCode 扩展，并介绍如何部署到 GitHub Release 和 VS Marketplace
扩展的开发以及 Marketplace 注册不在文章介绍范围内。

## 0. 基本配置

我们在仓库的目录下创建 `.travis.yml` ，文件内容的开始部分示例如下：

```yaml
env:
  global:
    - PACKAGE_VERSION="$(node -p -e 'require("./package.json").version')"
notifications:
  email: false
sudo: false
os:
- linux
language: node_js
node_js:
- 8
addons:
  apt:
    packages:
    - libsecret-1-dev
```

这个部分指定了一些基本配置

- env: 这里通过 package.json 拿到当前的版本号并设置到全局环境变量
- notification: 这里因为不想收到通知，所以关闭了邮件通知
- os/language: 这里设置系统环境为 Linux 并使用 Node.JS 8 运行环境
- addons: 这里安装了 VSCode 扩展打包时需要的软件包

## 1. 添加生成及测试配置

接下来是写一下生成和测试的配置，这里的测试需要一个无界面运行 VSCode 的环境，所以我们用上 `xvfb`

```yaml
before_install:
- if [ $TRAVIS_OS_NAME == "linux" ]; then
    export CXX="g++-4.9" CC="gcc-4.9" DISPLAY=:99.0;
    sh -e /etc/init.d/xvfb start;
    sleep 3;
  fi
script:
- npm test --silent
- npm run vscode:prepublish
```

- 在 before_install 中，我们启动了一个 xvfb 环境，提供无界面的 X 环境用来运行 VSCode
- 接下来，我们在 script 中执行了 `npm test --silent` 这个过程，这里测试时会自动下载 package.json 中指定的 VSCode 版本到 `.vscode-test` ，并运行扩展内编写的测试脚本
- 最后一步就是运行打包命令输出扩展到 out 目录了 `npm run vscode:prepublish`

在这个测试过程中因为会下载 VSCode 的特定版本到 `.vscode-test` ，我们完全可以通过 Travis CI 提供的 Cache 来加速这个过程，配置参考如下

```yaml
cache:
  directories:
    - .vscode-test
```

## 2. 部署到 GitHub Release

然后是在收到 Tag 推送的时候自动部署到 GitHub Release 的配置了，在部署之前需要写一个 before_deploy

```yaml
before_deploy:
- npm install -g vsce
- vsce package
```

这里根据官方的打包流程，安装 vsce ，然后使用 vsce 把扩展打包成 ~~zip~~ vsix 包

```yaml
deploy:
  - provider: releases
    api_key:
      secure: [GitHub Token]
    file_glob: true
    file: "*.vsix"
    skip_cleanup: true
    name: "Version ${PACKAGE_VERSION}"
    on:
      repo: [Reposity]
      tags: true
```

然后就是部署 Release 的配置，可以直接使用 `travis setup releases` 就可以直接生成了，然后只需要加一个 tags: true 即可。

这里使用 file_glob: true 启用通配符文件名匹配，然后 file: *.vsix 找到打包好的扩展。

## 3. 部署到 Marketplace

```yaml
deploy:
  - provider: script
    script: vsce publish -p $VS_TOKEN
    skip_cleanup: true
    on:
      repo: [Reposity]
      tags: true

```

部署到 Marketplace 很简单，在 deploy 段中添加这样的提交脚本，`vsce publish -p $VS_TOKEN` 用于上传扩展包，这里的 VS_TOKEN 环境变量需要在 Marketplace 申请并添加到 Travis CI 的设置里。

## **The END**
