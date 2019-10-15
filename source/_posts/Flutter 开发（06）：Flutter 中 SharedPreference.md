---
title: Flutter 开发（06）：Flutter 中 SharedPreference
permalink: flutter-sharedpreference-by-shared-preference
comments: true
date: 2019-02-28 16:10:35
updated: 2019-02-28 16:10:35
tags:
  - Flutter SharedPreference
  - shared_preferences.dart
categories:
  - Flutter
---

本文主要介绍 `Flutter` 上 `SharedPreference` 的使用，其目的是介绍 `shared_preferences.dart` 库的使用。[shared_preferences.dart](https://github.com/flutter/plugins/tree/master/packages/shared_preferences) 官方开源的轻量级数据存储库，并且屏蔽了 `Android` 和 `iOS` 平台上使用 `K-V` 型数据的差异。

使用 `shared_preferences.dart` 前需要添加配置并更新依赖：

```dart
shared_preferences: ^0.5.1+1
```

然后，在需要使用该库的地方引入该库：

```dart
import 'package:shared_preferences/shared_preferences.dart';
```
<!--more-->

## 初始化 SharedPreferences

使用如下代码可以初始化 `SharedPreferences` 对象：

```dart
SharedPreferences sharedPreferences = await SharedPreferences.getInstance();
```

## 读写数据

获取到 `SharedPreferences` 对象之后，我们可以通过一系列的 `get` 和 `set` 方法读写数据。目前 `shared_preferences.dart` 库支持的数据类型有：

* `bool`：布尔型数据，对应 `getBool` 和 `setBool` 两个方法
* `int`：`int` 型数据，对应 `getInt` 和 `setInt` 两个方法
* `double`：`double` 型数据，对应 `getDouble` 和 `setDouble` 两个方法
* `String`：`String` 型数据，对应 `getString` 和 `setString` 两个方法
* `List<String>`：`List<String>` 型数据，对用 `getStringList` 和 `setStringList` 两个方法

下面是写入数据的示例：

```dart
var result = await sharedPreferences.setString("key", "value");
```

下面是读取数据的示例：

```dart
String result = sharedPreferences.getString("key");
```

需要注意的是，读取操作并不需要使用 `await`。

## 清空数据

`SharedPreferences` 中提供了 `remove` 方法用来删除特定 `key` 的数据，以及 `clear` 方法用于删除所有的数据:

```dart
await sharedPreferences.remove("key");
await sharedPreferences.clear();
```
