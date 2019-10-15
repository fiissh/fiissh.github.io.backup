---
title: Flutter 开发（03）：Http 请求
permalink: flutter-http-request
comments: true
date: 2019-02-27 10:30:46
updated: 2019-02-27 10:30:46
tags:
  - Flutter Http
  - http.dart
  - dart:io
categories:
  - Flutter
---

本文主要介绍 `Flutter` 项目中的 `Http` 网络请求。本文介绍的重点是，`Flutter` 对 `dart:io` 库中的 `HttpClient` 的使用，以及开源 `Flutter` 网络请求库 [package:http/http.dart](https://github.com/dart-lang/http) 的使用。

本文介绍的所有特性是基于 `Flutter V1.0.0` 以及 `Dart V2.1.0`。

由于 `dart:io` 库对于 `POST` 请求支持不好，本文中只简单介绍了使用 `dart:io` 库发起 `GET` 请求的情况。`http/http.dart` 库我们也只介绍 `GET` 请求和 `POST` 请求的使用，对于其他类型的情况，可参考官方文档。

`dio/dio.dart` 库也是非常优秀的 `Http` 请求库，如果对其感兴趣，请移步项目地址：[dio/dio.dart](https://github.com/flutterchina/dio)。
<!--more-->
## dart:io

如果使用 `HttpClient`，需要我们导入 `dart:io` 库：

```dart
import 'dart:io';
```

导入 `dart:io` 库之后，我们可以使用 `new` 关键字创建 `HttpClient` 的实例对象：

```dart
var httpClient = new HttpClient();
```

> `HttpClient` 的构造函数支持一个可选的 `SecurityContext` 参数。


使用 `HttpClient` 发送请求之前，需要先使用 `Uri` 构建请求的 `Uri` 对象。以 `http://tj.nineton.cn/Heart/index/all?city=CHSH000000&language=zh-chs` 请求接口为例：

```dart
Uri uri = new Uri.http('tj.nineton.cn', '/Heart/index/all', {'city': 'CHSH000000', 'language': 'zh-chs'});
```

最终，使用 `HttpClient` 发起请求的完整示例如下：

```dart
httpRequestGet() async {
  HttpClient client = new HttpClient();
  var authority = 'tj.nineton.cn';
  var unencodedPath = '/Heart/index/all';
  var params = {'city': 'CHSH000000', 'language': 'zh-chs'};

  Uri uri = new Uri.http(authority, unencodedPath, params);
  var request = await client.getUrl(uri);
  var response = await request.close();
  if (response.statusCode == HttpStatus.ok) {
    var responseMessage = await response.transform(Utf8Decoder()).join();
    print(responseMessage);
  } else {
    print(response.toString());
  }
}
```

> 由于使用了 `response.transform(Utf8Decoder())` 方法，需要导入 `dart:convert` 库。

## http/http.dart

[package:http/http.dart](https://github.com/dart-lang/http) 是一个开源的基于 `Flutter` 的网络请求框架。

使用 `http/http.dart` 库之前，我们需要先在 `pubspec.yaml` 文件中添加改库的依赖：

```yaml
dependencies:
  http: ^0.12.0+1
```

> 目前最新版为 `0.12.0+1`。更多信息请参考项目主页。

然后在项目的根目录使用如下命令安装该库：

```shell
flutter packages get
```

> 如果使用 `pub` 工具，可以使用 `pub get` 命令。

最后，在项目中需要使用的地方引入该库：

```dart
import 'package:http/http.dart';
```

或者使用别名的引用方式：

```dart
import 'package:http/http.dart' as http;
```

个人建议使用第二种方式进行引入（该章节的示例程序都默认使用了第二种引入方式）。

### GET 请求

以 `http://tj.nineton.cn/Heart/index/all?city=CHSH000000&language=zh-chs` 为例，使用 `http/http.dart` 发送 `GET` 请求的实例代码如下：

```dart
httpRequestGet() async {
  var url = 'http://tj.nineton.cn/Heart/index/all?city=CHSH000000&language=zh-chs';
  var client = http.Client();
  http.Response response = await client.get(url);

  if (response.statusCode == 200) {
    var responseMessage = response.body;
    print(responseMessage);
  }
}
```

或者使用如下方式：

```dart
httpRequestGet() {
  var url = 'http://tj.nineton.cn/Heart/index/all?city=CHSH000000&language=zh-chs';
  http.Client().get(url).then((http.Response response) {
    if (response.statusCode == 200) {
      var responseMessage = response.body;
      print(responseMessage);
    }
  });
}
```

### POST 请求

与 `GET` 请求不同的是，`POST` 请求需要将数据写入到 `body` 里面：

```dart
httpRequestPost() async {
  var url = "request_url";
  var params = new Map<String, String>();
  params['param_1'] = "value_1";
  params['param_2'] = "value_2";
  params['param_3'] = "value_3";
  var client = http.Client();
  http.Response response = await client.post(url, body: params);
  if (response.statusCode == 200) {
    var responseMessage = response.body;
    print(responseMessage);
  }
}
```

同样的，上述请求也可以使用如下代码实现：

```dart
httpRequestPost() {
   var url = "request_url";
   var params = new Map<String, String>();
   params['param_1'] = "value_1";
   params['param_2'] = "value_2";
   params['param_3'] = "value_3";
   http.Client().post(url, body: params).then((http.Response response) {
     if (response.statusCode == 200) {
       var responseMessage = response.body;
       print(responseMessage);
     }
   });
 }
```
