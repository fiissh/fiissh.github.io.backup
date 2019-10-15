---
title: Flutter 开发（11）：Flutter 层异常捕获
permalink: flutter-exception-handler
comments: true
date: 2019-03-25 14:23:29
updated: 2019-04-16 11:23:29
tags:
  - Flutter 异常捕获
categories:
  - Flutter
---

`Flutter` 中的异常可以分为同步异常和异步异常。通常情况下我们可以使用 `try-catch` 捕获同步异常，而异步异常则需要使用 `Flutter` 提供的其他 `API`。
<!--more-->
## 使用 try-catch 捕获异常

与 `Java` 语言类似的，`Dart` 中提供了 `try-catch` 代码块用于对同步异常的捕获：

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

### Flutter 框架层的异常捕获

`Flutter` 框架在很多关键位置自动进行了异常处理。例如，如果我们的 `Flutter` 页面出现错误，会自动弹出一个错误页面：

![](https://user-gold-cdn.xitu.io/2019/1/16/1685691a02afb4b6?imageView2/0/w/1280/h/960/ignore-error/1)

这是由于系统在调用 `build()` 方法构建页面时进行了 `try-catch` 的处理，并提供一个 `ErrorWidget` 用于出现异常信息时调用的页面：

```dart
@override
void performRebuild() {
 ...
  try {
    //执行build方法  
    built = build();
  } catch (e, stack) {
    // 有异常时则弹出错误提示  
    built = ErrorWidget.builder(_debugReportException('building $this', e, stack));
  }
  ...
}
```

`_debugReportException` 方法中最终通过 `FlutterError.reportError` 将错误信息进行上报：

```dart
FlutterErrorDetails _debugReportException(
  String context,
  dynamic exception,
  StackTrace stack, {
  InformationCollector informationCollector
}) {
  //构建错误详情对象  
  final FlutterErrorDetails details = FlutterErrorDetails(
    exception: exception,
    stack: stack,
    library: 'widgets library',
    context: context,
    informationCollector: informationCollector,
  );
  //报告错误
  FlutterError.reportError(details);
  return details;
}
```

`reportError` 方法中最终通过 `onError` 属性处理错误信息（实际上 `onError` 有默认实现 `dumpErrorToConsole`）。我们可以通过重写 `onError` 属性拦截上报信息（`reportError` 方法）：

```dart
void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    reportError(details);
  };
  runApp(FilePathProviderApplication());
}
```

如果我们需要自定义错误页面，则需要重写 `ErrorWidget.builder`：

```dart
void main() {
  ErrorWidget.builder = (FlutterErrorDetails flutterErrorDetails) {
    print(flutterErrorDetails.toString());
    return Center(
      child: Text("Flutter 崩溃了"),
    );
  };
  runApp(FilePathProviderApplication());
}
```

## 使用 Future API 捕获异常

`Future API` 提供了 `catchError` 方法用于捕获使用 `Future` 时抛出的异步异常：

```dart
filePathProvide.getExternalStorageApplicationDirectory().then((value) {
  doSomething(value);
  throw ("exception message");
}).catchError(_handlerError);

_handlerError() {

}
```

## 使用 Zone 捕获未处理的异常

`Flutter` 提供了 `Zone.runZoned` 方法(`dart:async` 包)用于捕获所有未处理的异常信息：

```dart
R runZoned<R>(R body(),{Map zoneValues, ZoneSpecification zoneSpecification, Function onError})
```

其中，`zoneValues` 是 `Zone` 的私有属性，`zoneSpecification` 是 `Zone` 的一些配置信息，可以自定义一些代码行为，`onError` 是 `Zone` 中未捕获异常的回调方法。

如果开发者提供了 `onError` 回调方法或者通过 `ZoneSpecification.handleUncaughtError` 指定了错误处理的回调方法，那么这个 `Zone` 将会变为一个 `error zone`，无论是同步环境还是异步环境，该 `error zone` 都会在出现未捕获的异常信息时调用：

```dart
void main() {
  runZoned(() => runApp(FilePathProviderApplication()), onError: (Object obj, StackTrace stack) {
    uploadException("$obj\n$stack");
  });
}
```

使用 `runZoned` 可以捕获 `Flutter` 层全局的未处理异常信息。我们可以结合 `FlutterError`，在非生产环境下将 `Flutter UI` 层的异常信息使用 `FlutterError` 进行处理，而生产环境下使用 `runZoned`进行处理：

```dart
const bool inProduction = const bool.fromEnvironment("dart.vm.product");

FlutterError.onError = (FlutterErrorDetails details) async {
  if (inProduction) {
    Zone.current.handleUncaughtError(details.exception, details.stack);
  } else {
    FlutterError.dumpErrorToConsole(details);
  }
};
runApp(FilePathProviderApplication());
```

## Flutter 端异常捕获最佳实践

对于 `Flutter` 端的异常信息，除了能够使用 `try-catch` 捕获的同步异常之外，大多是情况下异常信息集中在 `Flutter` 端的页面渲染方面，所以我们需要重写 `FlutterError.onError` 用于处理框架层自动添加的异常捕获：

```dart
const bool inProduction = const bool.fromEnvironment("dart.vm.product");

FlutterError.onError = (FlutterErrorDetails details) async {
  if (inProduction) {
    Zone.current.handleUncaughtError(details.exception, details.stack);
  } else {
    FlutterError.dumpErrorToConsole(details);
  }
};
```

在非生产环境下，异常信息通过 `FlutterError.dumpErrorToConsole(details)` 交由控制台进行输出，生产环境则通过 `Zone.current.handleUncaughtError(details.exception, details.stack)` 将异常信息交付给 `Flutter` 层的全局异常捕获进行处理。

然后，我们再为整个应用程序的 `Flutter` 层添加全局异常捕获：

```dart
runZoned<Future<Null>>(() async {
    runApp(new HomeApp());
  }, onError: (error, stackTrace) async {
    await _reportError(error, stackTrace);
  });
```
