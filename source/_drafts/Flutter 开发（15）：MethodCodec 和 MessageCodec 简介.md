---
title: Flutter 开发（15）：MethodCodec 和 MessageCodec 简介
permalink: flutter-methodcodec
comments: true
date: 2019-04-18 14:55:02
updated: 2019-04-18 14:55:02
tags:
  - MethodCodec
  - MessageCodec
categories:
  - Flutter
---

通过前文我们针对 `MethodChannel`、`EventChannel` 和 `BasicMessageChannel` 的介绍我们发现，`MethodChannel` 和 `EventChannel` 的编解码器是 `MethodCodec` ，而 `BasicMessageChannel` 则使用 `MessageCodec` 编解码器。

本文主要介绍 `MethodCodec` 和 `MessageCodec`。
<!--more-->

## MethodCodec

`MethodCodec` 用于二进制数据与方法调用和返回结果之间的编解码：

```dart
abstract class MethodCodec {
  /// Encodes the specified [methodCall] into binary.
  ByteData encodeMethodCall(MethodCall methodCall);

  /// Decodes the specified [methodCall] from binary.
  MethodCall decodeMethodCall(ByteData methodCall);

  /// Decodes the specified result [envelope] from binary.
  ///
  /// Throws [PlatformException], if [envelope] represents an error, otherwise
  /// returns the enveloped result.
  dynamic decodeEnvelope(ByteData envelope);

  /// Encodes a successful [result] into a binary envelope.
  ByteData encodeSuccessEnvelope(dynamic result);

  /// Encodes an error result into a binary envelope.
  ///
  /// The specified error [code], human-readable error [message], and error
  /// [details] correspond to the fields of [PlatformException].
  ByteData encodeErrorEnvelope({@required String code, String message, dynamic details});
}
```

其中：

* `ByteData encodeMethodCall`：将方法调用编码为二进制数据类型
* `MethodCall decodeMethodCall`：将 `encodeMethodCall` 中编码的二进制的方法调用解码为 `MethodCall`
* `dynamic decodeEnvelope`：将二进制的方法调用结果解码
* `ByteData encodeSuccessEnvelope`：方法调用成功时，将 `result` 编码为二进制数据
* `ByteData encodeErrorEnvelope`：方法调用失败时，将 `code`、`message` 和 `details` 编码为二进制数据

`Flutter` 提供了如下两个 `MethodCodec` 的默认实现：

* `JSONMethodCodec`
* `StandardMethodCodec`

### JSONMethodCodec

### StandardMethodCodec
