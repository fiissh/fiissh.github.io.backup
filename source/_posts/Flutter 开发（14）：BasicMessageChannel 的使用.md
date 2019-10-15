---
title: Flutter 开发（14）：BasicMessageChannel 的使用
permalink: flutter-basic-message-channel
comments: true
date: 2019-04-16 15:04:32
updated: 2019-04-16 15:04:32
tags:
  - BasicMessageChannel
categories:
  - Flutter
---

与 `MethodChannel` 和 `EventChannel` 接收一个 `MethodCodec` 对象所不同的是，`BasicMessageChannel` 接收一个 `MessageCodec<T>` 作为其编解码器的实现。`BasicMessageChannel` 允许我们使用自定义的消息编解码器进行异步消息传递。

本文中所使用的代码示例出自 `Flutter` 官方示例，感兴趣的小伙伴请移步 [flutter/examples/flutter_view](https://github.com/flutter/flutter/tree/master/examples/flutter_view)

<!--more-->
## BasicMessageChannel 的初始化

`BasicMessageChannel` 不提供默认的 `MessageCodec` 的实现，我们需要在初始化 `BasicMessageChannel` 时指定具体的 `MessageCodec`。我们以使用 `StringCodec` 初始化 `BasicMessageChannel` 并传递 `String` 类型的数据为例：

```dart
static const BasicMessageChannel<String> platform = BasicMessageChannel<String>(_channel, StringCodec());
```

对象初始化之后，我们通过 `setMessageHandler` 方法设置一个 `MessageHandler` 回调函数：

```dart
platform.setMessageHandler(_handlePlatformIncrement);
Future<String> _handlePlatformIncrement(String message) async {
  setState(() {
    _counter++;
  });
  return _emptyMessage;
}
```

初始化完成之后，我们可以通过 `platform.send('pong');` 向设备平台端发送一个 `String` 类型的数据。

## 设备平台 Android 端的初始化

相应的，我们需要在设备平台端（`Android`） 初始化 `BasicMessageChannel`：

```java
messageChannel = new BasicMessageChannel<>(flutterView, CHANNEL, StringCodec.INSTANCE);
messageChannel.
    setMessageHandler(new MessageHandler<String>() {
        @Override
        public void onMessage(String s, Reply<String> reply) {
            onFlutterIncrement();
            reply.reply(EMPTY_MESSAGE);
        }
    });
```

设备平台端收到数据之后，`onMessage` 方法会被触发。

同样的，在设备平台端可以使用 `messageChannel.send(PING);` 向 `Flutter` 端发送 `String` 类型的数据，`Flutter` 端收到数据之后，`_handlePlatformIncrement` 方法会被触发。

## BasicMessageChannel 的原理

与 `MethodChannel` 一样，`BasicMessageChannel` 在发送数据时需要先通过 `codec` 将消息编码，然后通过 `BinaryMessages` 将编码之后的消息发送出去，最后通过 `codec` 再将返回值进行解码：

```dart
Future<T> send(T message) async {
  return codec.decodeMessage(await BinaryMessages.send(name, codec.encodeMessage(message)));
}
```

## 总结

`BasicMessageChannel` 提供了一种机制允许我们自己定义消息的编解码器。相应的，`Flutter` 中已经提供了 `BinaryCodec`、`StringCodec` 和 `JSONMessageCodec` 作为常用数据类型的编解码器实现。如果还不能满足需求，我们可以通过扩展 `MessageCodec<T>` 达到目的。
