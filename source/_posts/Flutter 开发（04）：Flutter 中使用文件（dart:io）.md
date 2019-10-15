---
title: Flutter 开发（04）：Flutter 中使用文件（dart:io）
permalink: flutter-use-file-by-dart-io
comments: true
date: 2019-02-28 09:13:29
updated: 2019-02-28 09:13:29
tags:
  - Flutter 文件
  - path_provider/path_provider.dart
categories:
  - Flutter
---

本文主要介绍 `Flutter` 中针对文件的操作，主要目的是掌握使用 `dart:io` 库相关方法操作文件。

本文是基于 `Dart V2.1.0` 和 `Flutter V1.0.0` 版本进行的介绍。

`Flutter` 操作文件的类主要涉及到：

* `Directory`：用于操作文件目录
* `File`： 用于操作文件
* `Stream`：用于接收数据流（`dart:async` 库）

如果要使用 `dart:io` 库，我们需要在使用的位置导入：

```dart
import 'dart:io';
```

如果需要使用到 `Stream`，那么还需要导入：

```dart
import 'dart:async';
```
<!--more-->
## 目录

`Flutter` 中使用 `Directory` 类表示文件目录。

我们可以通过如下代码创建一个 `Directory` 对象：

```dart
var directory = new Directory("test_directory");
```

需要注意的是，我们必须指定一个确切的路径参数给 `Directory`。

我们可以通过使用 `directory.exists()` 来判断该目录是否存在。`directory.exists()` 是一个异步方法，需要使用 `await` 修饰。

### 创建和删除目录

如果目录不存在，我们可以使用如下方法创建目录：

```dart
await directory.create();
```

`create` 方法有一个可选的 `recursive` 参数。其作用是，如果 `recursive` 为 `false`，那么只创建目录中最后一个文件夹，如果为 `true`，则会创建整个目录中所有不存在的文件夹。

我们也可以使用如下方式创建目录，从而省略 `await` 关键字：

```dart
directory.createSync();
```

如果我们需要删除某个目录，可以使用 `delete` 方法：

```dart
await directory.delete();
```

或者

```dart
directory.deleteSync();
```

与 `create` 方法一样，`delete` 方法也有一个可选的 `recursive`（默认 `false`） 参数。其作用是，如果 `recursive` 为 `true`，即使 `FileSystemEntity` 不匹配也会删除。

### 重命名目录

可以使用 `rename` 方法对目录进行重命名：

```dart
await directory.rename("new_directory_name");
```

或者

```dart
directory.renameSync("new_directory_name");
```

### 获取目录文件列表

使用 `list` 可以列出该目录下的所有文件：

```Dart
await directory.list();
```

或者

```dart
directory.listSync();
```

## 文件

`Flutter` 中使用 `File` 类表示文件。我们可以使用如下代码创建一个 `File` 对象：

```dart
var file = new File("file_name");
```

同样的，我们可以通过 `exists` 方法判断文件是否存在：

```dart
await file.exists();
```

或者

```dart
file.existsSync();
```

### 创建和删除文件

如果文件不存在，我们可以使用如下方法创建文件：

```dart
await file.create();
```

或者

```dart
file.createSync();
```

与 `Directory` 相同的，`File` 的 `create` 方法也支持一个名为 `recursive` 可选参数。该参数如果为 `false`，则只创建文件，如果为 `true`，则会把不存在的文件夹也一起创建。

我们可以使用 `delete` 方法删除文件：

```dart
await file.delete();
```

或者

```dart
file.deleteSync();
```

### 读取和写入文件

相较于 `Java`，`Flutter` 中读写文件相对容易。

通过 `writeAsString` 方法实现 `String` 对象的写入：

```dart
await file.writeAsString("hello word!");
```

同样的，可以通过 `readAsStringSync` 方法将文件内容读取为 `String` 对象：

```dart
var fileMessage = await file.readAsString();
```

### 文件的复制

`File` 类中提供了 `copy` 方法用于文件的复制操作：

```Dart
await file.copy("new_file_path");
```
