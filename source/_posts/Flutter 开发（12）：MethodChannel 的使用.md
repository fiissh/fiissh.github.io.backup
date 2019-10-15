---
title: Flutter 开发（12）：MethodChannel 的使用
permalink: flutter-method-channel
comments: true
date: 2019-04-15 14:15:12
updated: 2019-04-15 14:15:12
tags:
  - MethodChannel
categories:
  - Flutter
---

在 `Flutter` 中，`MethodChannel` 允许我们发送与方法调用相关的消息。`MethodChannel` 适用于 `Flutter` 端和设备平台端进行直接的方法调用，这种方法调用是双向的，即既可以通过 `MethodChannel` 从 `Flutter` 调用设备平台上的方法，也可以从设备平台上调用 `Flutter` 层的代码。
<!--more-->

## MethodChannel 实现方法调用的步骤

在 `Flutter` 层，我们需要通过 `MethodChannel` 的构造方法创建其对象，然后通过 `invokeMethod`、`invokeListMethod` 或者 `invokeMapMethod` 方法将需要在设备平台响应的方法名称和参数，通过 `BinaryMessages` 传递给设备平台对应的响应对象（`Android` 平台为 `MethodChannel`，`iOS` 平台为 `FlutterMethodChannel`），设备平台响应完成之后将结果传递回 `Flutter`。相应的，从设备平台调用 `Flutter` 层的方法也是一样的：

![](https://upload-images.jianshu.io/upload_images/4313776-f4f8d3493b076946.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/580)

* 首先，`MethodChannel` 收到方法调用的请求之后，先通过 `MethodCodec` 的 `encodeMethodCall` 方法将需要调用的方法名称和参数信息进行编码
* 然后，通过 `BinaryMessages.send` 方法将编码之后的数据传递给对应的设备平台，并接收其返回值
* 最后，根据返回结果的值进行处理：
  * 如果返回值为 `null`，抛出 `MissingPluginException`
  * 如果返回值不为 `null`，通过 `MethodCodec` 的 `decodeEnvelope` 将返回值解码为具体的类型

## Flutter 层的 MethodChannel

### 构造方法

`MethodChannel` 的构造方法接收一个 `name` 和一个可选的 `codec` 变量：

```dart
const MethodChannel(this.name, [this.codec = const StandardMethodCodec()]);
```

其中，`name` 是 `String` 类型，作用是为每一个 `MethodChannel` 设置一个唯一的 `ID`，`codec` 是 `MethodCodec` 类型，默认值是 `StandardMethodCodec`，作用是对方法调用和返回值进行编解码：

```dart
static const String _METHOD_CHANNEL_NAME = 'fiissh.flutter.plugin/flutter_file_path_provide';
static const MethodChannel _channel = const MethodChannel(_METHOD_CHANNEL_NAME);
```

通常情况下，我们使用默认的 `MethodCodec` 对象即可。自定义 `MethodCodec` 的方式我们会在后文中介绍。

### 方法调用

`MethodChannel` 提供了下列三种方法用于方法调用：

* `Future<T> invokeMethod<T>(String method, [dynamic arguments])`：普通方法调用
* `Future<List<T>> invokeListMethod<T>(String method, [dynamic arguments])`：返回值是数组的方法调用
* `Future<Map<K, V>> invokeMapMethod<K, V>(String method, [dynamic arguments])`：返回值是 `Map` 的方法调用

我们以 `invokeMethod` 方法为例：

```dart
Future<T> invokeMethod<T>(String method, [dynamic arguments]) async {
  assert(method != null);
  final ByteData result = await BinaryMessages.send(
    name,
    codec.encodeMethodCall(MethodCall(method, arguments)),
  );
  if (result == null) {
    throw MissingPluginException('No implementation found for method $method on channel $name');
  }
  final T typedResult = codec.decodeEnvelope(result);
  return typedResult;
}
```

在接收到方法调用的参数之后，首先会通过 `codec.encodeMethodCall(MethodCall(method, arguments))` 将方法名称和参数进行编码，然后再通过 `BinaryMessages.send` 将 `MethodChannel` 的 `name` 和编码之后的数据发送到对应的设备平台，并接收 `ByteData` 类型的返回值。

在接收到 `ByteData` 类型的返回值之后，如果返回值为 `null`，则抛出 `MissingPluginException`，视为设备平台上没有对应的方法。否则通过 `codec.decodeEnvelope(result)` 将返回值解码为具体的数据类型。我们以 [path_provider/path_provider.dart](https://github.com/flutter/plugins/tree/master/packages/path_provider) 项目代码为例：

```dart
Future<Result> _getDirectory(String methodName, [String extra]) async {
  String path;
  try {
    path = await _channel.invokeMethod(methodName, extra);
    return Result(Result.ERROR_CODE_SUCCESS, 'success', Directory('$path/'));
  } on PlatformException catch (e) {
    return Result(num.parse(e.code), e.message);
  }
}
```

最终，我们通过如下代码完成一次 `Flutter` 层到设备平台的方法调用：

```dart
static const String _EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME = "getExternalStorageDirectory";
Future<Result> getExternalStorageDirectory() async
  return _getDirectory(_EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME);
}
```

### 响应来自设备平台的方法调用 setMethodCallHandler

`setMethodCallHandler` 方法通过一个方法回调用来接收设备平台通过 `MethodChannel` 调用 `Flutter` 代码时的响应信息：

```dart
void setMethodCallHandler(Future<dynamic> handler(MethodCall call)) {
  BinaryMessages.setMessageHandler(
    name,
    handler == null ? null : (ByteData message) => _handleAsMethodCall(message, handler),
  );
}
```

该方法最终是通过 `BinaryMessages.setMessageHandler` 将 `MethodChannel` 的 `name` 属性和 `Future<ByteData> handler(ByteData message)` 回调方法赋值给了 `BinaryMessages` 对象中的 `_handlers`。

`_handleAsMethodCall` 方法则是对返回的结果进行处理：

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
}
```

如果方法调用成功，则通过 `MethodCodec` 的 `encodeSuccessEnvelope` 方法将结果编码为 `ByteData`，如果调用失败，则通过 `MethodCodec` 的 `encodeErrorEnvelope` 将错误码、错误信息和详情进行编码。我们以响应来自设备平台的一个方法调用（名称为 `a_method`）为例：

```dart
_channel.setMethodCallHandler(_handler);

Future<dynamic> _handler(MethodCall methodCall) {
  if ("a_method" == methodCall.method) {
    print(methodCall.method);
  }
  return Future.value(true);
}
```

通过 `setMethodCallHandler` 设置方法调用的监听之后，我们可以根据 `methodCall.method` 判断具体的方法调用。

## Android 层的 MethodChannel

`Android` 平台上响应 `Flutter` 层的方法调用是通过 `MethodCallHandler` 实现的：

```Java
public interface MethodCallHandler {
    void onMethodCall(MethodCall methodCall, MethodChannel.Result result);
}
```

设备平台端（`Android` 为例）响应 `Flutter` 层发送的方法调用的请求，需要实现 `MethodCallHandler` 接口用于接收方法调用的详细信息：

```Java
public class FlutterFilePathProvidePlugin implements MethodCallHandler {

    @Override
    public void onMethodCall(MethodCall call, Result result) {

    }
}
```

`onMethodCall` 方法回调时会传入 `methodCall` 和 `result` 两个对象。其中，`methodCall` 包含了方法调用的基本信息（方法名称和参数）以及用于处理参数的几个接口。`result` 参数则用于在方法响应完毕之后，将返回值或者错误信息传递到 `Flutter` 层。上文中我们通过 `MethodChannel` 向设备平台发送了一个名称为 `getExternalStorageDirectory` 的方法调用，那么在 `Android` 平台上的具体响应代码为：

```java
private static final String _EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME = "getExternalStorageDirectory";
public void onMethodCall(MethodCall call, Result result) {
    if (TextUtils.equals(call.method, _EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME)) {
        try {
            String path = getExternalStorageDirectory();
            if (TextUtils.isEmpty(path)) {
                result.error("1", "path is null", "");
            } else {
                result.success(path);
            }
        } catch (Exception e) {
            result.error("2", e.getMessage(), "");
        }
    }
}

private String getExternalStorageDirectory() {
    return Environment.getExternalStorageDirectory().getAbsolutePath();
}
```

上述代码中，根据 `Flutter` 端传递过来的 `name` 属性，完成对应方法的调用，并通过 `result` 对象将结果回写回 `Flutter` 层：

```Java
public interface Result {
    void success(@Nullable Object var1);

    void error(String var1, @Nullable String var2, @Nullable Object var3);

    void notImplemented();
}
```

与 `Flutter` 层相对应的，`Android` 端也需要初始化一个 `MethodChannel` 对象：

```Java
public static void registerWith(Registrar registrar) {
    FlutterFilePathProvidePlugin.registrar = registrar;
    final MethodChannel channel = new MethodChannel(registrar.messenger(), _METHOD_CHANNEL_NAME);
    channel.setMethodCallHandler(new FlutterFilePathProvidePlugin());
}
```

### 构造方法

`Android` 端的 `MethodChannel` 提供了两个构造方法：

```Java
public MethodChannel(BinaryMessenger messenger, String name) {
    this(messenger, name, StandardMethodCodec.INSTANCE);
}

public MethodChannel(BinaryMessenger messenger, String name, MethodCodec codec) {
    assert messenger != null;

    assert name != null;

    assert codec != null;

    this.messenger = messenger;
    this.name = name;
    this.codec = codec;
}
```

与 `Flutter` 端相对应的，`Android` 端的 `MethodChannel` 也需要 `BinaryMessenger`、`name` 和 `MethodCodec` 三个对象。其中，`name` 属性需要与 `Flutter` 端初始化 `MethodChannel` 时传入的 `name` 保持一致。

### 方法调用

`Android` 端的 `MethodChannel` 提供了下列两个方法用于调用 `Flutter` 层的方法：

* `invokeMethod(@NonNull String method, @Nullable Object arguments)`
* `invokeMethod(String method, @Nullable Object arguments, MethodChannel.Result callback)`

与 `Flutter` 层调用设备平台代码一样，`Android` 端调用 `Flutter` 层代码也需要先通过 `MethodCodec` 的 `encodeMethodCall` 方法将方法调用进行编码，然后通过 `BinaryMessenger.send` 将对应的信息发送到 `Flutter` 层。

### 响应 Flutter 层的方法调用

`setMethodCallHandler(@Nullable MethodChannel.MethodCallHandler handler)` 方法通过接收一个 `handler` 对象来响应来自于 `Flutter` 层的方法调用。

与 `Flutter` 层调用设备平台的方法一样的，设备平台调用 `Flutter` 层的方法也是通过 `MethodChannel` 实现的：

```Java
channel.invokeMethod("a_method", null);
```

## OptionalMethodChannel

`OptionalMethodChannel` 是 `MethodChannel` 的子类。与 `MethodChannel` 所不同的是，如果调用 `invokeMethod` 找不到与之对应的设备平台的方法，`MethodChannel` 以抛出异常的形式返回，而 `OptionalMethodChannel` 则是返回 `null`。

## 总结

`MethodChannel` 的主要作用是提供一种 `Flutter` 端方法和设备平台端的方法相互调用的机制。默认情况下，`MethodChannel` 使用 `StandardMethodCodec` 作为数据的编解码器，并最终通过 `BinaryMessages` 完成与设备平台端数据的接收和传递。

在 `MethodChannel` 中除了默认的 `StandardMethodCodec` 作为 `codec` 的默认实现，还可以选择 `JSONMethodCodec` 作为 `codec` 编解码器的实现，用于传递 `JSON` 类型的数据。
