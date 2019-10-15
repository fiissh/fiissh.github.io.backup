---
title: Flutter 开发（13）：EventChannel 的使用
permalink: flutter-event-channel
comments: true
date: 2019-04-16 11:22:38
updated: 2019-04-16 11:22:38
tags:
  - EventChannel
categories:
  - Flutter
---

`EventChannel` 适用于 `Flutter` 端和设备平台端数据事件的通信。比较常见的使用 `EventChannel` 的场景是通过广播监听（`Android` 平台上的 `BroadcastReceiver`）动态的监听设备事件。

本文中的示例代码是 `Flutter` 官方示例，感兴趣的小伙伴可以移步 [flutter/examples/platform_channel/](https://github.com/flutter/flutter/tree/master/examples/platform_channel)。
<!--more-->

## EventChannel 的初始化

与 `MethodChannel` 相类似的，`EventChannel` 也需要一个全局唯一的 `name` 属性作为其标识符：

```dart
static const EventChannel eventChannel = EventChannel('samples.flutter.io/charging');
```

创建 `EventChannel` 的对象之后，我们需要向 `EventChannel` 注册一个监听回调方法用于监听事件：

```dart
eventChannel.receiveBroadcastStream().listen(_onEvent, onError: _onError);
void _onEvent(Object event) {
  setState(() {
    _chargingStatus =
        "Battery status: ${event == 'charging' ? '' : 'dis'}charging.";
  });
}

void _onError(Object error) {
  setState(() {
    _chargingStatus = 'Battery status: unknown.';
  });
}
```

## 设备平台 Android 端的初始化

在设备平台（`Android`）端，我们需要配置 `EventChannel`：

```java
new EventChannel(getFlutterView(), CHARGING_CHANNEL).setStreamHandler(
    new StreamHandler() {
      private BroadcastReceiver chargingStateChangeReceiver;
      @Override
      public void onListen(Object arguments, EventSink events) {
        chargingStateChangeReceiver = createChargingStateChangeReceiver(events);
        registerReceiver(
            chargingStateChangeReceiver, new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
      }

      @Override
      public void onCancel(Object arguments) {
        unregisterReceiver(chargingStateChangeReceiver);
        chargingStateChangeReceiver = null;
      }
    }
);
```

其中，`onListen` 是 `Flutter` 端设置监听回调方法之后调用，我们在该方法中触发时向系统注册监听电量变化的广播：

```java
private BroadcastReceiver createChargingStateChangeReceiver(final EventSink events) {
    return new BroadcastReceiver() {
      @Override
      public void onReceive(Context context, Intent intent) {
        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);

        if (status == BatteryManager.BATTERY_STATUS_UNKNOWN) {
          events.error("UNAVAILABLE", "Charging status unavailable", null);
        } else {
          boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                               status == BatteryManager.BATTERY_STATUS_FULL;
          events.success(isCharging ? "charging" : "discharging");
        }
      }
    };
  }
```

`onCancel` 是事件流断开时触发，我们在其中将广播取消。由此一来，设备平台端的事件变化可以通过 `EventChannel` 从设备平台端传递到 `Flutter` 端。

当设备平台端的事件发生变化时，通过 `EventSink` 对象 `events` 将对应的状态回传给 `Flutter` 端。

## EventChannel 的原理

事实上，`EventChannel` 是使用 `MethodChannel` 和 `Stream` 以达到监听事件变化的目的的：

```dart
Stream<dynamic> receiveBroadcastStream([dynamic arguments]) {
  final MethodChannel methodChannel = MethodChannel(name, codec);
  StreamController<dynamic> controller;
  controller = StreamController<dynamic>.broadcast(onListen: () async {
    BinaryMessages.setMessageHandler(name, (ByteData reply) async {
      if (reply == null) {
        controller.close();
      } else {
        try {
          controller.add(codec.decodeEnvelope(reply));
        } on PlatformException catch (e) {
          controller.addError(e);
        }
      }
      return null;
    });
    try {
      await methodChannel.invokeMethod<void>('listen', arguments);
    } catch (exception, stack) {
      FlutterError.reportError(FlutterErrorDetails(
        exception: exception,
        stack: stack,
        library: 'services library',
        context: 'while activating platform stream on channel $name',
      ));
    }
  }, onCancel: () async {
    BinaryMessages.setMessageHandler(name, null);
    try {
      await methodChannel.invokeMethod<void>('cancel', arguments);
    } catch (exception, stack) {
      FlutterError.reportError(FlutterErrorDetails(
        exception: exception,
        stack: stack,
        library: 'services library',
        context: 'while de-activating platform stream on channel $name',
      ));
    }
  });
  return controller.stream;
}
```

我们在 `Flutter` 端调用 `receiveBroadcastStream` 方法之后，会返回一个 `Stream` 对象用于接收数据流，其中 `listen` 方法则用于监听数据的变化。

而在 `receiveBroadcastStream` 方法调用时，其中通过 `MethodChannel` 调用了对应设备平台上的 `onListen` 和 `onCancel` 两个方法。

## 总结

与 `MethodChannel` 一样，`EventChannel` 最为核心的实现也是 `BinaryMessages` 和 `MethodCodec`。默认的，`EventChannel` 使用 `StandardMethodCodec` 作为 `codec` 属性的默认实现。

与 `MethodChannel` 一样，`EventChannel` 也可以使用 `JSONMethodCodec` 作为其编解码器的实现。
