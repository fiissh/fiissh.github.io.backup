---
title: Flutter 开发（01）：Dart 语言常用库索引
tags:
  - Dart 常用库索引
categories:
  - Flutter
permalink: dart-language-libraries
comments: true
date: 2019-02-25 18:55:33
updated: 2019-02-25 18:55:33
mathjax: true
---


本文主要介绍 `Dart` 语言中常用的库，并不做详细介绍，只作为索引的目的。

本文中介绍的 `Dart` 语言特性是基于 `Dart 2.1.0`。

`Dart` 中的库可以分为如下三种类型：

* 核心库
  * `dart:dart:async`：支持异步编程的库，包含 `Future` 和 `Stream` 等类
  * `dart:collection`：用于补充 `dart:core` 中各类集合框架的库
  * `dart:convert`：支持数据类型转换、编码格式转换的库
  * `dart:core`：`Dart` 内置库，包括常用的数据类型以及核心的定义
  * `dart:developer`：支持开发调试的库，包含调试程序和检测程序
  * `dart:math`：支持数学运算的库，包含数学常数、运算以及随机数生成器
  * `dart:typed_data`：用于处理固定大小的数据（例如无符号8字节整数）和 `SIND` 类型的数据
* `Web` 相关的库
  * `dart:html`：用于处理浏览器和 `DOM` 交互的库
  * `dart:indexed_db`：支持基于的索引的 `k-v` 的数据库
  * `dart:js`：用于支持 `JavaScript` 的库
  * `dart:js_util`：支持 `JavaScript` 工具库
  * `dart:svg`：支持 `SVG` 渲染的库
  * `dart:web_audio`：支持音频开发的库
  * `dart:web_gl`：支持 `Web GL` 渲染的库
  * `dart:web_sql`：支持用于在浏览器存储可以使用 `SQL` 查询的数据的 API
* 虚拟机相关的库
  * `dart:cli`：用于暂停堆栈、运行微服务、处理传入时间的库
  * `dart:io`：用于在非 `Web` 应用中支持文件、`Socket`、`Http` 以及其他 `I/O` 的库
  * `dart:isolate`：用于支持协程的库
  * `dart:mirrors`：用于支持反射的库

本人中只对核心库和虚拟机相关的库做简单介绍。
<!--more-->

## 核心库

### dart:async

![dart_async_library.png](/images/dart_async_library.png)


### dart:collection

![dart_collection_library.png](/images/dart_collection_library.png)

### dart:convert

![dart_convert_library.png](/images/dart_convert_library.png)

### dart:core

![dart_core_library.png](/images/dart_core_library.png)

### dart:developer

![dart_developer_library.png](/images/dart_developer_library.png)

### dart:math

![dart_math_library.png](/images/dart_math_library.png)

### dart:typed_data

![dart_typed_data_library.png](/images/dart_typed_data_library.png)

## 虚拟机相关的库

### dart:cli

![dart_cli_library.png](/images/dart_cli_library.png)

### dart:io

![dart_io_library.png](/images/dart_io_library.png)

### dart:isolate

![dart_isolate_library.png](/images/dart_isolate_library.png)

### dart:mirrors

![dart_mirrors_library.png](/images/dart_mirrors_library.png)
