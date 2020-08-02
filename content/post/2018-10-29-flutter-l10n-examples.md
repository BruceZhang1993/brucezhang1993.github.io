+++
category = 'Android'
title = '[Android/iOS] 如何给你的 Flutter 应用添加多语种支持'
date = '2018-10-29T00:00:00+08:00'
tags = ['flutter', 'android', 'l10n']
draft = false
toc = false
backtotop = true
+++

这边文章简单讲讲如何给你的 Flutter 应用添加多语种本地化支持，示例中，我们尝试为应用添加简体中文与英文的支持。

## 0. 准备

首先，我们需要添加一项依赖以实现多语言支持。在项目根目录找到 **pubspec.yaml**，在里面添加 **flutter_localizations** 内容如下：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  # Other dependencies
```

添加以后在项目根目录执行 `flutter packages get` 更新依赖关系。

## 1. 添加多语言

准备完成以后，我们要给 **lib/main.dart** 中的 MaterialApp 稍作修改，示例如下所示：

```dart
import 'package:flutter_localizations/flutter_localizations.dart';

class Main extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      localizationsDelegates: [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
      ],
      supportedLocales: [
        const Locale('en', 'US'), // English
        const Locale('zh', 'CN'), // Simplified Chinese
      ],
      debugShowCheckedModeBanner: false,
      title: 'ngXDA',
      theme: new ThemeData(
        primarySwatch: Colors.deepOrange,
      ),
      home: new HomePage(),
    );
  }
}
```

这里的关键是添加 **localizationsDelegates** 和 **supportedLocales**，然后添加支持的语种到 **supportedLocales**，Flutter 目前默认支持超过 [27 种语言](https://github.com/flutter/flutter/tree/master/packages/flutter_localizations/lib/src/l10n)，添加的两个 Delegate 是 Flutter 默认提供的，加入以后会为比如右键菜单，组件提示等 Flutter 自带组件添加多语言支持。

加入 Flutter 自带组件的本地化以后，我们继续为应用内容添加多语言支持。

## 2. 应用多语言

进入项目，创建文件名称类似 **lib/localization.dart**：

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class AppLocalizations {
  AppLocalizations(this.locale);
  final Locale locale;

  static AppLocalizations of(BuildContext context) {
    return Localizations.of<AppLocalizations>(context, AppLocalizations);
  }

  static Map<String, Map<String, String>> _localizedValues = {
    'en': {
      'settings': 'Settings',
      'yes': 'Yes',
      'no': 'No',
    },
    'zh': {
      'settings': '设置',
      'yes': '是',
      'no': '否',
    },
  };

  Map<String, String> get translate {
    return _localizedValues[locale.languageCode];
  }
}

class AppLocalizationsDelegate extends LocalizationsDelegate<AppLocalizations> {
  const AppLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) => ['en', 'zh'].contains(locale.languageCode);

  @override
  Future<AppLocalizations> load(Locale locale) {
    // Returning a SynchronousFuture here because an async "load" operation
    // isn't needed to produce an instance of DemoLocalizations.
    return SynchronousFuture<AppLocalizations>(AppLocalizations(locale));
  }

  @override
  bool shouldReload(AppLocalizationsDelegate old) => false;
}
```

上面的文件内容定义了一个 Delegate 用于翻译本地化字符串  **AppLocalizationsDelegate**

定义好以后，我们需要在 MaterialApp 中引入，我们对 **lib/main.dart** 再次修改：

```dart
// imports
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:ngxda/localization.dart';

void main() => runApp(new Main());

class Main extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      localizationsDelegates: [
        AppLocalizationsDelegate(),
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
      ],
      supportedLocales: [
        const Locale('en', 'US'), // English
        const Locale('zh', 'CN'), // Simplified Chinese
      ],
      debugShowCheckedModeBanner: false,
      title: 'ngXDA',
      theme: new ThemeData(
        primarySwatch: Colors.deepOrange,
      ),
      home: new HomePage(),
    );
  }
}
```

这里添加了 import 引入刚刚定义的 **AppLocalizationsDelegate**，然后在 **localizationsDelegates** 中引入了它，这里就完成了应用内的多语种支持。

## 3. 应用使用多语种

在应用内的组件内使用自定义多语种也是非常简单，在需要翻译的页面如下的方式引入：

```dart
import 'package:ngxda/localization.dart';
```

然后，你就可以这样使用 **AppLocalizationsDelegate**

```dart
String translated = AppLocalizations.of(context).translate['settings'];
print(translated); // 英文环境显示 Settings，中文环境显示 设置
```

**The End**
