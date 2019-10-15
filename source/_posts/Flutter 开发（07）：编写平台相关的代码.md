---
title: Flutter 开发（07）：编写平台相关的代码
permalink: flutter-platform-specific
comments: true
date: 2019-03-01 10:33:14
updated: 2019-03-01 10:33:14
tags:
   - Flutter 平台相关
   - BasicMessageChannel
   - MethodChannel
   - EventChannel
   - Platform
   - Handler
   - codec
categories:
   - Flutter
---

前面文章介绍的 `path_provider.dart` 库和 `shared_preferences.dart` 库的实际上是依赖于设备的具体环境的。比如说在 `shared_preferences.dart` 库在存储数据时，在 `Android` 设备上使用的是 `SharedPreference` 相关的功能，而在 `iOS` 上则是使用了 `NSUserDefaults` 的相关功能。换句话说，这些代码都是平台相关的。

本文主要介绍 `Flutter` 中平台相关的代码的开发，其目的是快速了解 `Android` 和 `iOS` 原生代码与 `Flutter` 之间的通信。

本文将以 `Flutter` 的官方 [path_provider/path_provider.dart](https://github.com/flutter/plugins/tree/master/packages/path_provider) 库作为示例。

平台通道的作用是在 `Flutter` 和设备之间传递消息，如下所示：

![flutter-platform-channel](/images/flutter-platform-channel.png)

`Flutter` 中平台特定的 `API` 支持并不依赖于代码生成，而是依赖于灵活的消息传递方式：

* 应用程序的 `Flutter` 部分通过平台通道（`Platform Channel`）将消息发送到该应用程序的设备平台（`Android` 或 `iOS`）
* 设备平台通过对平台通道的监听接收到消息之后，调用设备平台上对应的 `API`（使用设备平台原生开发语言），然后将响应数据发送回应用程序的 `Flutter` 部分

本文中有大量的内容参考自阿里咸鱼技术团队的 [深入理解 Flutter Platform Channel](https://zhuanlan.zhihu.com/p/43163159) 一文。该文章对 `Flutter` 的平台通道有非常详细的讲解。

再次表达对咸鱼团队的感谢以及对肥肥作为伸手党的不齿。
<!--more-->
## 平台通道类型

`Flutter` 中定义了三种不同类型的平台通道，他们的代码实现在 `platform_channel.dart` 文件中：

* `BasicMessageChannel`：用于传递字符串和半结构化的消息
* `MethodChannel` 和 `OptionalMethodChannel`：用于传递方法调用
* `EventChannel`：用于传递数据流

上述三种类型的平台通道相互独立，都有着各自的用途。从源码的角度来说，每个平台通道都有三个非常重要的成员变量：

* `name`： `String` 类型，表示平台通道的名称，也是通道的唯一标识符
* `BinaryMessenger`： `BinaryMessenger` 类型，用于表示消息，是接收和发送消息的具体实现
* `codec`： `MessageCodec` 或者 `MethodCodec` 类型，是消息的编解码器

### name

`name` 属性是平台通道的唯一标识符。一个 `Flutter` 应用中可能会存在多个平台通道，每一个平台通道在创建时必须指定一个位移的 `name` 属性（`Flutter` 使用该属性来区分每一个通道）。

当有消息从 `Flutter` 端发送到设备平台时，会根据其传递过来的 `name` 属性找到该通道对应的 `Handler`（消息处理器）。

### BinaryMessenger

上述三种类型的平台通道都是通过 `BinaryMessenger` 达到与 `Flutter` 通信的目的的：

![binary-messenger](/images/binary-messenger.jpg)

`BinaryMessenger` 使用的消息数据格式为二进制格式的数据。当我们初始化一个通道并向钙通道注册消息处理器（`Handler`）时，实际上会生成一个与之对应的 `BinaryMessageHandler`，并以通道的唯一标示符（`name`）作为 `key` 存储到 `Map<String, _MessageHandler> _handlers` （定义在 `platform_messages.dart/BinaryMessages` 中）对象中。当我们需要使用 `BinaryMessenger` 发送消息时，会根据 `name` 属性查找对应的 `BinaryMessageHandler`，然后完成消息的发送。

> `BinaryMessages` 在 `Android` 端是一个接口，其具体实现是 `FlutterNativeView`。在 `iOS` 端是一个协议，其名称为 `FlutterBinaryMessenger`，`FlutterViewController` 遵循了它。

在 `Flutter` 中，`BinaryMessenger` 并不知道平台通道的存在。事实上，`BinaryMessenger` 只和 `BinaryMessengerHandler` 直接通信（`name` 属性与 `BinaryMessengerHandler` 一一对象）。

由于平台通道从 `BinaryMessengerHandler` 接收到的消息是二进制格式的，需要使用 `codec` 将消息解码为可识别的数据并传递给 `Handler` 进行处理。

当 `Handler` 处理完消息之后，会通过回调方法返回 `result`，并将 `result` 通过 `codec` 编码为二进制数据，最终通过 `BinaryMessenger` 发送回 `Flutter`。

### codec

`Flutter` 中定义了两种类型的 `codec`：

* `MessageCodec`：用于二进制格式的数据与基础数据之间的编解码
* `MethodCodec`：用于二进制格式的数据与方法调用以及返回结果之间的编解码

![flutter-codec](/images/flutter-codec.jpg)

#### MessageCodec

`MessageCodec` 用于二进制格式的数据与基础数据之间的编解码，其主要目的是将消息处理为消息处理器（`Handler`）能够识别的数据。通道 `BasicMessageChannel` 使用的编解码器类型就是 `MessageCodec`。`MessageCodec` 的代码定义如下（`message_codec.dart/MessageCodec`）：

```dart
abstract class MessageCodec<T> {
  /// Encodes the specified [message] in binary.
  ///
  /// Returns null if the message is null.
  ByteData encodeMessage(T message);

  /// Decodes the specified [message] from binary.
  ///
  /// Returns null if the message is null.
  T decodeMessage(ByteData message);
}
```

其中，`ByteData encodeMessage` 接口用于编码数据，`T decodeMessage` 用于解码数据。

> 在 `Android` 中，`MessageCodec` 是一个接口，其中定义了两个方法 `encodeMessage` 和 `decodeMessage`。其中，`encodeMessage` 接收一个特定的数据类型，并将其编码为二进制数据 `ByteBuffer`。`decodeMessage` 则是接收二进制数据 `ByteBuffer`，并将其解码为特定的数据类型。在 `iOS` 中的名称是 `FlutterMessageCodec`，是一个协议类型，定义了两个方法 `encode` 和 `decode`。其中，`encode` 接收一个类型为 `id` 的消息，并将其编码为 `NSData` 类型。`decode` 则是接收 `NSData` 类型的信息，并将其解码为 `id` 类型的数据。

在 `Flutter` 中，`MessageCodec` 有如下四种类型的定义（`message_codecs.dart`）：

* `BinaryCodec`： 二进制数据的编解码
* `StringCodec`： `String` 类型和二进制数据之间的编解码（`UTF-8`）
* `JSONMessageCodec`： `JSON` 格式的基础数据类型与二进制数据之间的编解码
* `StandardMessageCodec`： `BasicMessageChannel` 的默认编解码器


`Flutter` 的 `BasicMessageChannel` 通道使用 `StandardMessageCodec`编解码器，支持简单的类似于 `JSON` 的高效二进制序列数据，例如 `bool`、`num`、`String`、`List`、`Map` 以及 `byte buffer` 等（详细信息请移步 [StandardMessageCodec class](https://docs.flutter.io/flutter/services/StandardMessageCodec-class.html)）。当我们发送和接收消息时，这些值在消息中的序列化和反序列化操作都将自动进行。

##### BinaryCodec

`BinaryCodec` 是二进制类型数据的编解码器：

```dart
class BinaryCodec implements MessageCodec<ByteData> {
  /// Creates a [MessageCodec] with unencoded binary messages represented using
  /// [ByteData].
  const BinaryCodec();

  @override
  ByteData decodeMessage(ByteData message) => message;

  @override
  ByteData encodeMessage(ByteData message) => message;
}
```

从代码中可以看出，无论是编码还是解码，`BinaryCodec` 对 `message` 没有做任何的处理（实际上，无论编解码操作，`BinaryCodec` 接收的都是 `ByteData` 类型的数据）。在 `Android` 上，`ByteData` 类型的数据表示为 `java.nio.ByteBuffer`，而在 `iOS` 上则表示为 `NSData`。

##### StringCodec

`StringCodec` 用于 `UTF-8` 类型的字符串数据和二进制数据之间的编解码：

```dart
class StringCodec implements MessageCodec<String> {
  /// Creates a [MessageCodec] with UTF-8 encoded String messages.
  const StringCodec();

  @override
  String decodeMessage(ByteData message) {
    if (message == null)
      return null;
    return utf8.decoder.convert(message.buffer.asUint8List(message.offsetInBytes, message.lengthInBytes));
  }

  @override
  ByteData encodeMessage(String message) {
    if (message == null)
      return null;
    final Uint8List encoded = utf8.encoder.convert(message);
    return encoded.buffer.asByteData();
  }
}
```

`String` 类型的数据 `message` 在 `Android` 上的表示为 `java.util.String`，而在 `iOS` 上则表示为 `NSString`。

数据类型的转换使用了 `dart.convert` 库中的 `Utf8Codec`。


##### JSONMessageCodec

`JSONMessageCodec` 用于 `UTF-8` 的 `JSON` 型消息与二进制数据之间的编解码：

```dart
class JSONMessageCodec implements MessageCodec<dynamic> {
  // The codec serializes messages as defined by the JSON codec of the
  // dart:convert package. The format used must match the Android and
  // iOS counterparts.

  /// Creates a [MessageCodec] with UTF-8 encoded JSON messages.
  const JSONMessageCodec();

  @override
  ByteData encodeMessage(dynamic message) {
    if (message == null)
      return null;
    return const StringCodec().encodeMessage(json.encode(message));
  }

  @override
  dynamic decodeMessage(ByteData message) {
    if (message == null)
      return message;
    return json.decode(const StringCodec().decodeMessage(message));
  }
}
```

该解码器所支持的基础数据类型有如下几种：

* `null`
* `bool`
* `num`
* `String`
* `List`
* `Map`

在 `Android` 上，`dynamic message` 的数据表示为 `JSON` 对象（`org.json` 包），而在 `iOS` 上则表示为 `NSJSONSerialization`。

`JSON` 类型的数据转换为二进制数据使用了 `StringCodec().encodeMessage(json.encode(message))`，该过程实际上是先将 `json` 对象转换为 `String`，然后再通过 `StringCodec().encodeMessage` 方法将 `String` 数据转换为 `ByteData`。相反的，首先，通过使用 `StringCodec().decodeMessage(message)` 方法将二进制数据转换为 `String`，然后可以通过使用 `json.decode` 方法将 `String` 数据转换为 `JSON` 对象。

##### StandardMessageCodec

`StandardMessageCodec` 用于基本数据类型与二进制数据的编解码。

下表显示了 `StandardMessageCodec` 中需要转换的基础数据类型在 `Android` 和 `iOS` 中对应的关系：

| Flutter | Android | iOS |
| --- | --- | --- |
| null | null | nil (NSNull when nested) |
| bool | ava.lang.Boolean | NSNumber numberWithBool: |
| int |  java.lang.Integer | NSNumber numberWithInt: |
| int, if 32 bits not enough | java.lang.Long |	 NSNumber numberWithLong: |
| int, if 64 bits not enough | java.math.BigInteger | FlutterStandardBigInteger |
| double | java.lang.Double | NSNumber numberWithDouble: |
| String | java.lang.String | NSString |
| Uint8List | byte[] | FlutterStandardTypedData typedDataWithBytes: |
| Int32List | int[] |	FlutterStandardTypedData typedDataWithInt32: |
| Int64List |	long[] |	FlutterStandardTypedData typedDataWithInt64: |
| Float64List |	double[] |	FlutterStandardTypedData typedDataWithFloat64: |
| List | java.util.ArrayList | NSArray |
| Map |	java.util.HashMap |	NSDictionary |

`StandardMessageCodec` 中对基础数据类型的编码操作是在 `writeValue` 方法中完成的，而解码操作则是在 `readValue` 方法中完成。对基础数据类型和二进制数据相互转换感兴趣的小伙伴可以深入学习这两个方法。本文对此不再做介绍。

#### MethodCodec

`MethodCodec` 用于二进制数据与方法调用和返回结果之间的编解码。`MethodChannel` 和 `EventChannel` 两个通道类型所使用的编解码器就是 `MethodCodec`。`MethodCodec` 的代码定义如下（`message_codec/MethodCodec`）：

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

`MethodCodec` 中各个方法的说明如下：

* `ByteData encodeMethodCall`：编码方法调用
* `MethodCall decodeMethodCall`：解码方法调用
* `dynamic decodeEnvelope`：将方法调用的二进制数据解码
* `ByteData encodeSuccessEnvelope`：方法调用成功时，将 `result` 编码为二进制数据
* `ByteData encodeErrorEnvelope`：方法调用失败时，将 `code`、`message` 和 `details` 编码为二进制数据

与 `MessageCodec` 所不同的是，`MethodCodec` 用于方法调用（方法对象）的编解码。一个方法对象代表一次从 `Flutter` 发起的一次方法调用。每个方法调用有两个成员变量（以 `MethodChannel` 的 `invokeMethod` 方法为例）：

```dart
Future<dynamic> invokeMethod(String method, [dynamic arguments]) async {
  assert(method != null);
  final dynamic result = await BinaryMessages.send(
    name,
    codec.encodeMethodCall(MethodCall(method, arguments)),
  );
  if (result == null)
    throw MissingPluginException('No implementation found for method $method on channel $name');
  return codec.decodeEnvelope(result);
}
```

其中，`method` 表示需要调用的方法名称，通用类型（`Android` 中的 `Object` 和 `iOS` 中的 `id`） `arguments` 表示需要调用的方法参数。

相对于 `MessageCodec`，`MethodCodec` 编解码器多了针对于调用结果的处理接口。当方法调用成功是，使用 `encodeSuccessEnvelope` 将 `result` 编码为二进制数据，而当方法调用失败时，则使用 `encodeErrorEnvelope` 将 `code`、`message` 和 `details` 编码为二进制数据。

在 `Flutter` 中，`MethodCodec` 有如下两种类型的定义（`message_codecs.dart`）：

* `JSONMethodCodec`：编解码 `JSON` 类型的方法调用，实际上依赖于 `JSONMessageCodec`
* `StandardMethodCodec`：`MethodCodec` 的默认实现，二进制类型的方法调用编解码器，实际上依赖于 `StandardMessageCodec`

##### JSONMethodCodec

`JSONMethodCodec` 用于 `UTF-8` 的 `JSON` 类型的方法调用与二进制数据之间的转换：

```dart
class JSONMethodCodec implements MethodCodec {
  // The codec serializes method calls, and result envelopes as outlined below.
  // This format must match the Android and iOS counterparts.
  //
  // * Individual values are serialized as defined by the JSON codec of the
  //   dart:convert package.
  // * Method calls are serialized as two-element maps, with the method name
  //   keyed by 'method' and the arguments keyed by 'args'.
  // * Reply envelopes are serialized as either:
  //   * one-element lists containing the successful result as its single
  //     element, or
  //   * three-element lists containing, in order, an error code String, an
  //     error message String, and an error details value.

  /// Creates a [MethodCodec] with UTF-8 encoded JSON method calls and result
  /// envelopes.
  const JSONMethodCodec();

  @override
  ByteData encodeMethodCall(MethodCall call) {
    return const JSONMessageCodec().encodeMessage(<String, dynamic>{
      'method': call.method,
      'args': call.arguments,
    });
  }

  @override
  MethodCall decodeMethodCall(ByteData methodCall) {
    final dynamic decoded = const JSONMessageCodec().decodeMessage(methodCall);
    if (decoded is! Map)
      throw FormatException('Expected method call Map, got $decoded');
    final dynamic method = decoded['method'];
    final dynamic arguments = decoded['args'];
    if (method is String)
      return MethodCall(method, arguments);
    throw FormatException('Invalid method call: $decoded');
  }

  @override
  dynamic decodeEnvelope(ByteData envelope) {
    final dynamic decoded = const JSONMessageCodec().decodeMessage(envelope);
    if (decoded is! List)
      throw FormatException('Expected envelope List, got $decoded');
    if (decoded.length == 1)
      return decoded[0];
    if (decoded.length == 3
        && decoded[0] is String
        && (decoded[1] == null || decoded[1] is String))
      throw PlatformException(
        code: decoded[0],
        message: decoded[1],
        details: decoded[2],
      );
    throw FormatException('Invalid envelope: $decoded');
  }

  @override
  ByteData encodeSuccessEnvelope(dynamic result) {
    return const JSONMessageCodec().encodeMessage(<dynamic>[result]);
  }

  @override
  ByteData encodeErrorEnvelope({@required String code, String message, dynamic details}) {
    assert(code != null);
    return const JSONMessageCodec().encodeMessage(<dynamic>[code, message, details]);
  }
}
```

从源码中可以看出，`JSONMethodCodec` 实际上是依赖 `JSONMessageCodec` 实现的。当使用 `JSONMethodCodec` 编码方法调用时，会先将方法调用转化为 `{"method":method,"args":args}`，然后通过 `JSONMessageCodec().encodeMessage` 完成调用。而 `JSONMethodCodec` 在编码调用结果 `result` 时，会先通过 `JSONMessageCodec().decodeMessage(envelope)` 将其转换为一个数组，如果调用成功，则通过 `JSONMessageCodec().encodeMessage(<dynamic>[result])` 将 `result` 编码为二进制数据，如果失败，则通过 `JSONMessageCodec().encodeMessage(<dynamic>[code, message, details])` 将 `code`、`message` 和 `details` 编码为二进制数据。该过程是在 `MethodChannel` 的 `_handleAsMethodCall` 完成的：

```dart
Future<ByteData> _handleAsMethodCall(ByteData message, Future<dynamic> handler(MethodCall call)) async {
  final MethodCall call = codec.decodeMethodCall(message);
  try {
    return codec.encodeSuccessEnvelope(await handler(call));
  } on PlatformException catch (e) {
    return codec.encodeErrorEnvelope(
      code: e.code,
      message: e.message,
      details: e.details,
    );
  } on MissingPluginException {
    return null;
  } catch (e) {
    return codec.encodeErrorEnvelope(code: 'error', message: e.toString(), details: null);
  }
}
```

##### StandardMethodCodec

`StandardMethodCodec` 是 `MethodCodec` 的默认实现，主要用于基础数据类型与二进制数据之间的编解码。`StandardMethodCodec` 的实现依赖于 `StandardMessageCodec`：

```dart
class StandardMethodCodec implements MethodCodec {
  // The codec method calls, and result envelopes as outlined below. This format
  // must match the Android and iOS counterparts.
  //
  // * Individual values are encoded using [StandardMessageCodec].
  // * Method calls are encoded using the concatenation of the encoding
  //   of the method name String and the arguments value.
  // * Reply envelopes are encoded using first a single byte to distinguish the
  //   success case (0) from the error case (1). Then follows:
  //   * In the success case, the encoding of the result value.
  //   * In the error case, the concatenation of the encoding of the error code
  //     string, the error message string, and the error details value.

  /// Creates a [MethodCodec] using the Flutter standard binary encoding.
  const StandardMethodCodec([this.messageCodec = const StandardMessageCodec()]);

  /// The message codec that this method codec uses for encoding values.
  final StandardMessageCodec messageCodec;

  @override
  ByteData encodeMethodCall(MethodCall call) {
    final WriteBuffer buffer = WriteBuffer();
    messageCodec.writeValue(buffer, call.method);
    messageCodec.writeValue(buffer, call.arguments);
    return buffer.done();
  }

  @override
  MethodCall decodeMethodCall(ByteData methodCall) {
    final ReadBuffer buffer = ReadBuffer(methodCall);
    final dynamic method = messageCodec.readValue(buffer);
    final dynamic arguments = messageCodec.readValue(buffer);
    if (method is String && !buffer.hasRemaining)
      return MethodCall(method, arguments);
    else
      throw const FormatException('Invalid method call');
  }

  @override
  ByteData encodeSuccessEnvelope(dynamic result) {
    final WriteBuffer buffer = WriteBuffer();
    buffer.putUint8(0);
    messageCodec.writeValue(buffer, result);
    return buffer.done();
  }

  @override
  ByteData encodeErrorEnvelope({@required String code, String message, dynamic details}) {
    final WriteBuffer buffer = WriteBuffer();
    buffer.putUint8(1);
    messageCodec.writeValue(buffer, code);
    messageCodec.writeValue(buffer, message);
    messageCodec.writeValue(buffer, details);
    return buffer.done();
  }

  @override
  dynamic decodeEnvelope(ByteData envelope) {
    // First byte is zero in success case, and non-zero otherwise.
    if (envelope.lengthInBytes == 0)
      throw const FormatException('Expected envelope, got nothing');
    final ReadBuffer buffer = ReadBuffer(envelope);
    if (buffer.getUint8() == 0)
      return messageCodec.readValue(buffer);
    final dynamic errorCode = messageCodec.readValue(buffer);
    final dynamic errorMessage = messageCodec.readValue(buffer);
    final dynamic errorDetails = messageCodec.readValue(buffer);
    if (errorCode is String && (errorMessage == null || errorMessage is String) && !buffer.hasRemaining)
      throw PlatformException(code: errorCode, message: errorMessage, details: errorDetails);
    else
      throw const FormatException('Invalid envelope');
  }
}
```

当使用 `StandardMethodCodec` 编码方法调用时，会先通过 `StandardMessageCodec` 的 `writeValue` 将 `call.method` 和 `call.arguments` 写入 `buffer`。在解码方法调用时，会通过 `readValue` 将写入的二进制数据取出。

在编码调用结果时，如果调用成功，则会先向 `buffer` 中写入数据 0（表示调用成功），然后再写入编码后的 `result`。如果调用失败，则先向 `buffer` 中写入 1（表示调用失败），然后再写入 编码后的 `code`、`message` 和 `details`。该过程也是在 `MethodChannel` 的 `_handleAsMethodCall` 方法中实现的。

### 消息处理器

当我们接收到二进制消息，并通过 `codec` 对象将消息处理为消息处理器能够是别的数据之后，消息处理器才开始工作：

```dart
static Future<ByteData> send(String channel, ByteData message) {
  final _MessageHandler handler = _mockHandlers[channel];
  if (handler != null)
    return handler(message);
  return _sendPlatformMessage(channel, message);
}
```

以通道类型一一对应的，`Flutter` 中定义了三种消息处理器：

* `MessageHandler`： 用于处理字符串或者半结构化的消息类型，对应 `BasicMessageChannel`
* `MethodHandler`： 用于处理方法的调用，对应 `MethodChannel`
* `StreamHandler`： 用于处理事件流，对应 `EventChannel`

## path_provider

为了学习方便，肥肥基于官方 [path_provider/path_provider.dart](https://github.com/flutter/plugins/tree/master/packages/path_provider) 项目重新在 `Github` 进行了部署： [fiissh/path_provider](https://github.com/fiissh/path_provider)。需要的小伙伴可以自行 `clone`。

在 `Android Studio`（请确保环境已经正确配置） 中打开项目，我们首先来看一下 `lib/path_provider.dart` 文件。该文件中定义了三个方法：

* `Future<Directory> getTemporaryDirectory()`
* `Future<Directory> getApplicationDocumentsDirectory()`
* `Future<Directory> getExternalStorageDirectory()`

以及在文件开头的位置定义了一个编译时常量 `_channel`：

```dart
const MethodChannel _channel = MethodChannel('plugins.flutter.io/path_provider');
```

> `path_provider` 主要使用了三种平台通道类型中的 `MethodChannel` 通道。`plugins.flutter.io/path_provider` 作为 `MethodChannel` 构造参数，在其初始化时将会作为该通道的唯一标识。

### getTemporaryDirectory() 和 getApplicationDocumentsDirectory() 方法

实际上两个方法的实现机制是一样的。我们只分析 `getApplicationDocumentsDirectory()` 方法：

```dart
Future<Directory> getApplicationDocumentsDirectory() async {
  // TODO(amirh): remove this on when the invokeMethod update makes it to stable Flutter.
  // https://github.com/flutter/flutter/issues/26431
  // ignore: strong_mode_implicit_dynamic_method
  final String path = await _channel.invokeMethod('getApplicationDocumentsDirectory');
  if (path == null) {
    return null;
  }
  return Directory(path);
}
```

该方法的定义非常简单，最关键的代码只有一行：

```dart
final String path = await _channel.invokeMethod('getApplicationDocumentsDirectory');
```

其作用是通过 `_channel` 通道的 `invokeMethod` 方法调用 `Android` 和 `iOS` 平台上的 `getApplicationDocumentsDirectory` 方法，并在调用之后返回 `String` 类型的返回值 `path`。

现在我们比较明确的一点是，`getApplicationDocumentsDirectory()` 方法调用之后会通过 `MethodChannel` 平台通道调用 `Android` 或 `iOS` 上的 `getApplicationDocumentsDirectory` 方法。

`Android` 或者 `iOS` 设备是如何知道 `Flutter` 的具体调用意图的呢？

我们进入到 `path_provider` 项目的 `Android` 部分（项目右键-`Flutter`-`Open Android module in Android Studio`）可以看到，在 `io.flutter.plugins.pathprovider` 包下有一个 `PathProviderPlugin.java` 类。该类继承自 `MethodCallHandler` 并实现了 `onMethodCall` 方法：

```Java
@Override
public void onMethodCall(MethodCall call, Result result) {
  switch (call.method) {
    case "getTemporaryDirectory":
      result.success(getPathProviderTemporaryDirectory());
      break;
    case "getApplicationDocumentsDirectory":
      result.success(getPathProviderApplicationDocumentsDirectory());
      break;
    case "getStorageDirectory":
      result.success(getPathProviderStorageDirectory());
      break;
    default:
      result.notImplemented();
  }
}
```

> 在 `IntelliJ IDEA` 时，使用项目右键-`Flutter`-`Open Android module in Android Studio` 的方式可能会出现无法打开的情况。此时建议使用 `Android Studio`。

`onMethodCall` 方法接收来自 `Flutter` 端传递过来的 `call` 和 `result` 两个参数。其中，`call` 对象存储了我们需要调用的方法名称（比如，`Flutter` 层需要调用的方法为 `getApplicationDocumentsDirectory`，通过 `call.method` 属性实现与 `Android` 端方法的对应）。`result` 对象则负责将 `Android` 端方法调用的结果传递回 `Flutter` 层。

静态方法 `registerWith` 则用于创建 `MethodChannel` 对象以及初始化 `PathProviderPlugin` 对象：

```Java
public static void registerWith(Registrar registrar) {
  MethodChannel channel = new MethodChannel(registrar.messenger(), "plugins.flutter.io/path_provider");
  PathProviderPlugin instance = new PathProviderPlugin(registrar);
  channel.setMethodCallHandler(instance);
}
```

通过在 `Android` 端初始化一个 `MethodChannel` 对象，并设置与 `Flutter` 端一样的平台通道唯一标识（`plugins.flutter.io/path_provider`），可以实现 `Flutter` 端和设备端的平台通道的映射。`PathProviderPlugin` 作为消息处理器的具体实现（`MethodCallHandler`），通过 `channel.setMethodCallHandler(instance)` 实现了平台通道与消息处理器的对应关系。

在实现了平台通道与消息处理器之间的对应关系之后，来自 `Flutter` 层的调用又是如何准确的知道消息处理器的位置呢？`path_provider` 的示例项目的 `Android` 部分给了我们答案（`io.flutter.plugins` 包）：

```Java
public final class GeneratedPluginRegistrant {
  public static void registerWith(PluginRegistry registry) {
    if (alreadyRegisteredWith(registry)) {
      return;
    }
    PathProviderPlugin.registerWith(registry.registrarFor("io.flutter.plugins.pathprovider.PathProviderPlugin"));
  }

  private static boolean alreadyRegisteredWith(PluginRegistry registry) {
    final String key = GeneratedPluginRegistrant.class.getCanonicalName();
    if (registry.hasPlugin(key)) {
      return true;
    }
    registry.registrarFor(key);
    return false;
  }
}
```

`io.flutter.plugins.pathprovider.PathProviderPlugin` 是我们的消息处理器的绝对路径。该过程将消息处理器的对决路径注册到 `Flutter` 在设备端的引擎中，然后通过 `Flutter` 端和设备端设置的通道唯一标识将平台通道、消息处理器实现一一对应的关系。

静态方法 `registerWith` 在 `Flutter` 应用项目的唯一一个 `Activity`（`Android` 端）中被调用：

```Java
public class MainActivity extends FlutterActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    GeneratedPluginRegistrant.registerWith(this);
  }
}
```

### getExternalStorageDirectory()

在实现原理上，`getExternalStorageDirectory()` 与其他另外两个方法并没有实际的区别。`getExternalStorageDirectory()` 比较特殊的地方在于，通过在 `Flutter` 端使用 `Platform.isIOS` 方法判断设备是否是 `iOS`，以此达到判断平台的目的。这也就意味着，`iOS` 设备上使用 `Flutter` 调用 `getExternalStorageDirectory()` 方法将会抛出一个异常：

```dart
Future<Directory> getExternalStorageDirectory() async {
  if (Platform.isIOS)
    throw UnsupportedError("Functionality not available on iOS");
  // TODO(amirh): remove this on when the invokeMethod update makes it to stable Flutter.
  // https://github.com/flutter/flutter/issues/26431
  // ignore: strong_mode_implicit_dynamic_method
  final String path = await _channel.invokeMethod('getStorageDirectory');
  if (path == null) {
    return null;
  }
  return Directory(path);
}
```

`Platform` 类（`platform.dart` 文件，`dart.io` 库）定义了大量的当前设备环境相关的信息，其中包括判断当前设备类型、获取设备操作系统相关信息等。关于 `Platform` 类的详细信息，请移步 `Flutter` 官方文档 [Platform class](https://docs.flutter.io/flutter/dart-io/Platform-class.html)。
