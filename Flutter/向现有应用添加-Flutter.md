**Add-to-app**

有些时候，一次性把现有的应用重写至 Flutter 不实际。在这种情况下，您可以把 Flutter 当作一个单独的库或模块集成到现有的应用中。随后，这个模块可被导入到 Android 或 iOS (目前所支持的平台) 应用中，并利用 Flutter 来渲染应用的部分 UI，或者直接运行共享的 Dart 逻辑。只需简单几步，您便可以在自己的应用中展现 Flutter 出色的开发效率和表达效果。Flutter 1.12 已为一些基础场景提供 add-to-app 支持，允许开发者使用 add-to-app 每次为一个应用添加一个全屏 Flutter 页面。以下为该功能目前所知的局限:

*   同时运行多个 Flutter 实例或在部分视图中运行实例可能会导致未定义行为。
*   在后台模式下使用 Flutter 尚处于开发阶段。
*   不支持将一个 Flutter 库打包到另一个可分享的库内，或者将多个 Flutter 库打包到同一个应用中。

**支持特性**

**添加至 Android 应用**

![](https://upload-images.jianshu.io/upload_images/19956127-9b2bca52fbcda99e?imageMogr2/auto-orient/strip)

*   可通过在 Gradle 脚本中添加一个 Flutter SDK hook 来自动构建和导入 Flutter 模块。
*   请将您的 Flutter 模块构建至通用 Android Archive (AAR) 文件中，从而实现与构建系统的集成并提高与 Jetifier 和 AndroidX 的兼容性。
*   请通过 FlutterEngine API 启用并保留您的 Flutter 环境，以便让 Flutter 运行环境模块与 FlutterActivity 或 FlutterFragment 显示模块托偶。
*   Android Studio 里 Android/Flutter 协同编辑以及模块创建/导入向导。
*   支持 Java 和 Kotlin 宿主应用。
*   Flutter 模块可通过 Flutter 插件与平台进行互动。如需取得最佳的 add-to-app 效果，请将 Android 插件迁移至 V2 插件 API。在 Flutter 1.12 中，由 Flutter 团队或 FlutterFire 维护的大部分插件已完成迁移。
*   支持通过 IDE 中的 flutter attach 或命令行连接到包含 Flutter 的应用，以便调试 Flutter 或启用有状态的热重启。

*   Android Archive (AAR) 文件

    https://developer.android.google.cn/studio/projects/android-library

*   FlutterEngine

    https://api.flutter.dev/javadoc/io/flutter/embedding/engine/FlutterEngine.html

*   FlutterActivity

    https://api.flutter.dev/javadoc/io/flutter/embedding/android/FlutterActivity.html

*   FlutterFragmenthttps://api.flutter.dev/javadoc/io/flutter/embedding/android/FlutterFragment.html
*   Flutter 插件https://pub.dev/flutter/packages
*   迁移至 V2 插件 API

    https://flutter.dev/docs/development/packages-and-plugins/plugin-api-migration

*   Flutter 团队

    https://github.com/flutter/plugins/tree/master/packages

*   FlutterFire

    https://github.com/FirebaseExtended/flutterfire/tree/master/packages  

**添加至 iOS 应用**

![](https://upload-images.jianshu.io/upload_images/19956127-83ebc5756f9f118c?imageMogr2/auto-orient/strip)

*   可通过 CocoaPods 在 Xcode 构建阶段内添加一个 Flutter SDK hook 来自动构建和导入 Flutter 模块。
*   请将您的 Flutter 模块构建至一个通用 iOS 框架中，从而实现与构建系统的集成。
*   请通过 FlutterEngine API 启用并保留您的 Flutter 环境，以便单独添加一个 FlutterViewController。
*   支持 Objective-C 和 Swift 宿主应用。
*   Flutter 模块可通过 Flutter 插件与平台进行互动。
*   支持通过 IDE 中的 flutter attach 或命令行连接到包含 Flutter 的应用，以便调试 Flutter 或启用有状态的热重启。

请前往 GitHub 网站探索 add-to-app 示例 repo，学习如何在 Android 和 iOS 项目中导入 Flutter 模块，打造精美的应用 UI。

*   iOS 框架

    https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html

*   FlutterEngine

    https://api.flutter.dev/objcdoc/Classes/FlutterEngine.html

*   FlutterViewController

    https://api.flutter.dev/objcdoc/Classes/FlutterViewController.html

*   Flutter 插件

    https://pub.dev/flutter/packages

*   add-to-app 示例 repo

    https://github.com/flutter/samples/tree/master/experimental/add_to_app

**现在开始**

如果您想开始向现有应用添加 Flutter，请参阅以下项目集成指南:

*   Android

    https://flutter.dev/docs/development/add-to-app/android/project-setup

*   iOS

    https://flutter.dev/docs/development/add-to-app/ios/project-setup

**API 使用方法**

当您把 Flutter 集成到项目中后，请参阅以下开发者文档:

*   Android

    https://flutter.dev/docs/development/add-to-app/android/add-flutter-screen

*   iOS

    https://flutter.dev/docs/development/add-to-app/ios/add-flutter-screen
原文链接：https://mp.weixin.qq.com/s/DckZviEm6P1cNC1oZBvXKw
来源： 谷歌开发者微信公众号
