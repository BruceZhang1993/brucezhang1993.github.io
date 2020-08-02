+++
title = '[Android] 在 Travis CI 上编译并部署 Flutter 应用'
date = '2018-10-26T00:00:00+08:00'
tags = ['flutter', 'android', 'travis']
cover = '/assets/img/20181026/travis.png'
draft = false
toc = false
backtotop = true
+++

这篇文章的内容是如何使用 Travis CI 自动编译并部署 Flutter 应用，关于如何使用 Flutter 开发请前往 [官方文档](https://flutter.io/docs/) 自行学习，本文只探讨自动编译 Android，关于 iOS 的自动编译请参考 [Building Flutter APKs and IPAs on Travis](https://medium.com/@yegorj/building-flutter-apks-and-ipas-on-travis-98d84d8e9b4) 一文的后半部分。

## 0. 准备工作

在自动编译之前，首先需要根据 [Preparing an Android App for Release](https://flutter.io/android-release/) 一文，确保本地可以编译出签名完成的 APK，这个步骤中的 key.jks 等 keystore 配置需要用在后面的步骤中。
在完成官方 release 教程后，你的 android/app/build.gradle 可能如下面代码：

```gradle
signingConfigs {
  release {
    keyAlias keystoreProperties['keyAlias']
    keyPassword keystoreProperties['keyPassword']
    storeFile file(keystoreProperties['storeFile'])
    storePassword keystoreProperties['storePassword']
  }
}
```

在 Travis CI 配置之前，先进入 [Travis CI](https://travis-ci.org)，登录并同步你要部署的仓库，给 Travis CI 足够的权限，而且要分清楚是 org 不是 com (。

## 1. Travis CI 配置

```yaml
os: linux
language: android
licenses:
- android-sdk-preview-license-.+
- android-sdk-license-.+
- google-gdk-license-.+
android:
  components:
  - tools
  - platform-tools
  - build-tools-27.0.2
  - android-27
  - sys-img-armeabi-v7a-google_apis-27
  - extra-android-m2repository
  - extra-google-m2repository
  - extra-google-android-support
jdk: oraclejdk8
sudo: false
addons:
  apt:
    sources:
    - ubuntu-toolchain-r-test
    packages:
    - libstdc++6
    - fonts-droid
before_script:
- wget https://services.gradle.org/distributions/gradle-4.4-bin.zip
- unzip -qq gradle-4.4-bin.zip
- export GRADLE_HOME=$PWD/gradle-4.4
- export PATH=$GRADLE_HOME/bin:$PATH
- git clone https://github.com/flutter/flutter.git -b dev --depth 1
- export PATH=./flutter/bin:$PATH
- cd ./flutter
- git fetch origin pull/23397/head:pull_23397
- git checkout pull_23397
- cd ..
script:
- "./flutter/bin/flutter -v build apk"
cache:
  directories:
  - "$HOME/.pub-cache"

```

首先参考上面的的配置，将上面的文件保存到 Flutter 项目根目录下的 .travis.yml 文件，这个配置写了一些基本的 Flutter 编译过程，这里使用的是 dev channel 的 Flutter，所以我们从一个 pr 拉了一个修复补丁来修复 `lint-gradle-api could not find` 的问题。

接下来，我们需要使用 travis 添加一些自动配置，在此之前需要先安装 travis 命令行工具，从你的包管理器中安装 gem，然后 `gem install travis` 即可安装完成，安装后 travis 可执行文件可能在 `/home/$USER/.gem/ruby/$RUBY_VERSION/bin/travis`，如有需要可添加此路径到 $PATH。

然后，我们进入项目根目录，把 0 步生成的 keystore file 放在根目录下，注意这里**先把 keystore file 添加到 .gitignore 文件里**，避免把 keystore 提交到公开库去。

接下来执行 `travis encrypt-file key.jks --add`，key.jks 为你的 keystore file 名（travis 这里可能要求你进行登录，执行 `travis login --org` 登录到 travis.org，输入你的 GitHub 用户信息即可登录），`--add` 选项会自动给 .travis.yml 添加类似下面的配置：

```yaml
before_install:
- openssl aes-256-cbc -K $encrypted_[secure]_key -iv $encrypted_[secure]_iv
  -in key.jks.enc -out key.jks -d
```

同时你的 Travis.org 后台的项目设置里会添加两个环境变量用于对称解密，保护了机密文件的安全性。

接下来用 `travis setup releases`，命令可以添加自动部署到 GitHub Releases 的配置，不需要自动部署可以直接跳过，添加后的配置类似下面：

```yaml
deploy:
  provider: releases
  api_key:
    secure: [GitHub API Token]
  file: app/build/outputs/apk/app-release.apk
  skip_cleanup: true
  on:
    tags: true
```

注意这里设置要部署的文件路径为 `app/build/outputs/apk/app-release.apk`，设置 `tags: true`，可以只在打 git tag 的时候自动部署，这样只要我们给 GitHub 仓库推送一个 Tag, Travis 就会自动部署到这个 Tag 对应的 Release。

接下来，因为我们使用的是 API 27，我们可能需要手动同意 Android SDK 插入同意配置后如下：

```yaml
before_install:
- yes | sdkmanager "platforms;android-27"
- yes | sdkmanager "build-tools;27.0.3"
- openssl aes-256-cbc -K $encrypted_[secure]_key -iv $encrypted_[secure]_iv
  -in key.jks.enc -out key.jks -d
```

## 2. Gradle 配置

在 Gradle 的配置过程中，我们需要配置 KeyStore File 的 alias, key password 和 store password，我们不能直接在项目里不安全的写上这些关键信息，Travis 官方的建议是使用 Travis CI 的环境变量，于是我们需要对 Gradle 之前配置的 Release 做出修改。

我们怎么判断当前的环境是本机还是 Travis CI 呢，Travis CI 默认添加了一些环境变量供我们判断，这里我们给 build.gradle 加上判断条件，加上以后如下：

```gradle
// ...

def isRunningOnTravis = System.getenv("CI") == "true"
```

这里的 isRunningOnTravis 从环境变量中取得 $CI，如果不为空则判断是 Travis CI 编译的，方便后面来判断。

接下来我们改造第 0 步的 signingConfigs。我们需要根据环境的不同设置不同的 signingConfigs，参考配置如下：

```gradle
if (isRunningOnTravis) {
  // 如果 Travis CI 编译则从环境变量中获取机密信息
  // key.jks 为 Travis CI 解密后的文件路径，不要加 .enc
  signingConfigs {
    release {
      keyAlias System.getenv("keyAlias")
      keyPassword System.getenv("keyPassword")
      storeFile file('../../key.jks')
      storePassword System.getenv("storePassword")
    }
  }
} else {
  // 如果是本地编译，直接加载 keystoreProperitiesFile 中的配置
  keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
  signingConfigs {
    release {
      keyAlias keystoreProperties['keyAlias']
      keyPassword keystoreProperties['keyPassword']
      storeFile file(keystoreProperties['storeFile'])
      storePassword keystoreProperties['storePassword']
    }
  }
}
```

当然，我们需要进入 travis.org，找到你的项目的 Settings 页面，在 Environment variable 下添加三个环境变量，分别是你的 alias, key password 和 store password，示例如下：

![Environment Variables](/assets/img/20181026/ScreenShot-20181026120850.png)

于此同时，我们添加下面的选项避免 lint 过程中断了编译：

```gradle
lintOptions {
  checkReleaseBuilds false
  abortOnError false
}
```

## 3. 使用自动编译和自动部署

使用过程就很简单了，每一次 git push 到 GitHub，都会触发一次编译，编译后 Travis 会检查这次 push 是不是 Tag 提交，如果是 Tag 提交，就会自动部署到这个项目的 Releases 页面。
