---
title: Flutter 开发（05）：Flutter 中使用文件（path_provider.dart）
permalink: flutter-use-file-by-pathprovider
comments: true
date: 2019-02-28 10:13:29
updated: 2019-02-28 10:13:29
tags:
  - Flutter 文件
  - path_provider
categories:
  - Flutter
---

本文主要介绍 `Flutter` 中 `Android` 和 `iOS` 两个平台下文件存储位置，主要目的是掌握 `Flutter` 官方开源项目 [path_provider/path_provider.dart](https://github.com/flutter/plugins/tree/master/packages/path_provider) 的使用。

使用 `path_provider.dart` 库之前，需要先在 `pubspec.yaml` 文件的 `dependencies:` 中添加如下依赖：

```dart
path_provider: ^0.5.0+1
```

然后在项目的根目录中使用如下命令更新依赖：

```dart
flutter packages get
```

最后，在项目中需要使用的地方引入该库：

```dart
import 'package:path_provider/path_provider.dart';
```

## 使用 path_provider.dart

`path_provider.dart` 库屏蔽了 `Android` 和 `iOS` 两个平台上文件存储路径的差异。`path_provider.dart` 库主要提供了如下几个方法用于获取存储路径：

* `getTemporaryDirectory()`：获取临时文件夹，针对于 `Android` 设备 `getCacheDir()` 和 `iOS` 设备 `NSTemporaryDirectory()` 返回的值
* `getApplicationDocumentsDirectory()`：获取 `Document` 文件夹，针对 `Android` 设备的 `AppDate` 目录，`iOS` 设备的 `NSDocumentDirectory` 目录
* `getExternalStorageDirectory()`： 获取存储卡目录，只有 `Android` 设备可用
