---
title: Flutter 开发（09）：Flutter 异步编程
permalink: flutter-async-program
comments: true
date: 2019-03-11 10:16:59
updated: 2019-03-11 10:16:59
tags:
  - Flutter 异步编程
  - await
  - sync
  - Future
categories:
  - Flutter
---

本文主要介绍 `Flutter` 异步编程相关的内容。关键字 `async`、`async`<sup>*</sup>、`sync`、`sync`<sup>*</sup>、`await`、`yield` 以及 `Future`、`Stream` 和 `Timer` 对象构成了 `Flutter` 异步编程的全部内容。

`Future` 是一个泛型对象，表示程序在将来的某个时刻能够获取到一个值，泛型类型表示了该返回值的类型。当返回值是 `Future` 的方法被调用之后，`Flutter`（`Dart`）将会做两件事情：

* 将需要执行的的任务添加到异步队列中，并返回一个未完成的 `Future` 对象
* 如果任务的运行结果有效（包括异常信息），`Future` 以该值完成任务的执行。如果需要使用 `Future` 中的值，可以使用如下两种方式：
  * 使用 `async` 和 `await` 关键字
  * 使用 `Future API`

`Stream` 也是一个泛型对象，表示异步数据的流通管道。它只允许从一端插入数据并通过管道从另外一端流出数据。为了控制 `Stream`，我们通常使用 `StreamController` 对 `Stream` 进行管理：

* `StreamController` 提供了类型为 `StreamSink` 的 `sink` 属性作为数据的入口
* `StreamController` 提供了 `stream` 属性作为数据的出口

在`Dart 1.9` 中引入了函数生成器的概念，利用惰性方法计算结果序列，以提升性能。生成器有两种类型：

* 同步生成器：在需要的时候才生成值，然后使用者从生成器中拉取。使用 `sync`<sup>*</sup> 表示
* 异步生成器：会以它自身的速度生成值，然后推送到使用者可以使用的地方。使用 `async`<sup>*</sup> 表示

`Timer` 提供了一种使用计时器的异步任务的执行方式，
<!--more-->
## async 和 await

从 `Dart 1.9` 开始，`Dart` 添加了 `async` 和 `await` 关键字用来实现异步的功能。它们允许我们编写看起来像是同步代码而且不需要使用 `Future` 的异步代码。在方法体前面使用 `async` 关键字修饰即表示这是一个异步方法。对调用者来说，调用 `async` 方法和其他方法没有什么区别，而且对于 `return` 返回值来说也没有什么影响。

非异步调用的方法定义如下：

```Dart
String getVersionName() {
  return '1.0.0';
}
```

调用方式：

```dart
String versionName = getVersionName();
```

`async` 修饰的异步方法最终将返回值封装为 `Future` 对象。即：使用 `async` 修饰的异步方法，其返回值类型需要封装为 `Future` 对象。`getVersionName()` 方法修改为异步方法之后定义如下：

```dart
Future<String> getVersionName() async {
  return '1.0.0';
}
```

调用方式：

```dart
String versionName = await getVersionName();
```

如果同步方法的返回值类型为 `T`，那么该方法异步方式的返回值类型应该是 `Future<T>`。如果同步方法中返回值类型为 `Future<T>`，那么对应的异步方法的返回值类型同样是 `Future<T>` 而不是 `Future<Future<T>>`。

如果异步方法没有显式的使用 `return` 返回一个值，那么在 `Flutter` 中将自动封装一个 `Future<Null>` 作为返回值。

需要注意的是，`await` 只能在 `async` 修饰的方法中使用，而且可以使用多次。使用 `await` 修饰的方法会自动进入阻塞状态，一直到任务执行完成并返回对应的值。


## 函数生成器

在 `Dart 1.9` 中引入了函数生成器的概念，利用惰性函数计算结果序列，以提升性能。

函数生成器有两种类型：同步生成器和异步生成器。同步生成器在需要的时候才生成值，然后使用者从生成器中拉取。异步生成器会以它自身的速度生成值，然后推送到使用者可以使用的地方。

### 同步生成器：sync<sup>*</sup>

通过在方法主体前添加 `sync`<sup>*</sup> 可以标记该方法为同步生成器。同步生成器返回一个 `Iterable<T>` 泛型对象，并使用 `yield` 语句来传递值。以获取第一个自然数 `n` 为例：

```dart
Iterable<int> naturalsTo(int n) sync* {
  print("Begin");
  int k = 0;
  while (k < n) yield k++;

  print("End");
}
```

当调用方法 `naturalsTo` 的时候，会立刻返回 `Iterable<int>` 对象。在我们调用 `iterator.moveNext()` 之前，函数的主体并不会执行。当运行该方法时，代码会执行到 `yield` 关键字修饰的位置，并暂停运行，此时 `moveNext()` 方法会返回 `true`。方法会在下一次调用 `moveNext()` 的时候恢复执行。当循环结束的时候，上述方法会隐式的执行 `return` 并终止迭代，此时 `moveNext()` 方法会返回 `false`。

我们通过如下代码晚上上述方法的调用：

```dart
var iterator = naturalsTo(3).iterator;

while (iterator.moveNext()) {
  print(iterator.current);
}
```

输出结果：

```shell
Begin
0
1
2
End
```

### 异步生成器：async<sup>*</sup>

通过在方法主体前添加 `async`<sup>*</sup> 可以标记该方法为异步生成器。异步生成器返回一个 `Stream` 对象，并使用 `yield` 语句来传递值。以自然数生成器为例：

```Dart
Stream<int> asynchronousNaturalsTo(int n) async* {
  print("Begin");
  int k = 0;

  while (k < n) yield k++;
  print("End");
}
```

通过如下代码完成调用：

```dart
asynchronousNaturalsTo(3).listen((v) {
  print(v);
});
/// outputs
/// I/flutter (19802): Begin
/// I/flutter (19802): 0
/// I/flutter (19802): 1
/// I/flutter (19802): 2
/// I/flutter (19802): End
```

当调用 `asynchronousNaturalsTo` 方法的时候，程序会立刻返回 `Stream<int>` 对象，但是函数体并没有执行。一旦开始 `listen` 监听数据流，方法体就开始执行。当执行到 `yield` 修饰的位置的时候，会将 `yield` 修饰的表达式的运算结果添加到 `Stream` 数据流中。异步生成器没有必要暂停，因为数据流可以通过 `StreamSubscription` 进行控制：

```
StreamSubscription<int> subscription = asynchronousNaturalsTo(3).listen(null);
subscription.onData((value) {
  print(value);
  if (value == 1) {
    subscription.pause();
  }
```

我们可以通过 `StreamSubscription` 来控制 `Stream` 的暂停(`subscription.pause()`)和取消(`subscription.cancel()`)。如果被暂停，方法会执行到 `yield` 修饰的位置，然后暂停，直到我们通过 `subscription.resume()` 恢复 `Stream`。

### yield 和 yield<sup>*</sup>

在上述的示例中，可以使用 `yield` 方便的传递值。如果我们的生成器函数中使用到了递归调用，那么使用 `yield` 将会带来一些性能问题。以从大到小获取自然数的的代码为例：

```dart
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    for (int i in naturalsDownFrom(n - 1)) {
      yield i;
    }
  }
}
```

上述代码通过递归调用的方式获取自然数。需要注意的是，我们每次调用 `naturalsDownFrom` 方法都会构建一个新的序列，并通过遍历新的序列然后使用 `yield` 将元素插入到当前序列中。我们在代码的一些位置插入输出来进行分析：

```dart
int level = 0;
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    print("Begin");
    level = level + 1;
    print('当前 n 为：$n  level 执行次数：$level');
    yield n;
    for (int i in naturalsDownFrom(n - 1)) {
      print('当前 i 为：$i  level 执行次数：$level');
      yield i;
    }
    print("End");
  }
}
```

输出如下：

```shell
I/flutter (31880): Begin
I/flutter (31880): 当前 n 为：3  level 执行次数：1
I/flutter (31880): Begin
I/flutter (31880): 当前 n 为：2  level 执行次数：2
I/flutter (31880): 当前 i 为：2  level 执行次数：2
I/flutter (31880): Begin
I/flutter (31880): 当前 n 为：1  level 执行次数：3
I/flutter (31880): 当前 i 为：1  level 执行次数：3
I/flutter (31880): 当前 i 为：1  level 执行次数：3
I/flutter (31880): End
I/flutter (31880): End
```

在上述代码中：

* 当 `n` 为3时，只执行了一次 `yield n`
* 当 `n` 为2时，先执行了一次 `yield n`，然后执行了一次 `yield i`(实际上是执行了一次 `n = 3` 时的逻辑)
* 当 `n` 为1时，先执行了一次 `yield n`，然后执行了两次 `yield i`（实际上是分别执行了 `n = 3` 和 `n = 2` 时的逻辑）

也就是说，这种方式下，使用 `yield` 传递值的次数总共执行了 $x(x-1)/2$ 次，时间复杂度为 $O(n^2)$，存在明显的性能问题。

我们使用 `yield`<sup>*</sup> 对原来的递归调用进行修改：

```dart
int level = 0;

Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    print("Begin");
    level = level + 1;
    print('当前 n 为：$n  level 执行次数：$level');
    yield n;
    yield* naturalsDownFrom(n - 1);
    print("End");
  }
}
```

输出如下：

```shell
I/flutter (32505): Begin
I/flutter (32505): 当前 n 为：3  level 执行次数：1
I/flutter (32505): Begin
I/flutter (32505): 当前 n 为：2  level 执行次数：2
I/flutter (32505): Begin
I/flutter (32505): 当前 n 为：1  level 执行次数：3
I/flutter (32505): End
I/flutter (32505): End
```

使用 `yield`<sup>*</sup> 调整之后，通过 `yield` 传递值的过程只有三次。`yield`<sup>*</sup> 的作用是：将 `yield`<sup>*</sup> 后面的子序列（在 `sync`<sup>*</sup> 方法中，子序列必须是一个 `Iterable` 可迭代对象；在 `async`<sup>*</sup> 方法中，子序列必须是一个 `Stream` 数据流）的所有元素插入到当前创建的序列中，而不是创建新的序列。

综上，`yield` 在递归调用中造成性能开销过大的本质原因是，生成器函数在每次调用的时候都会生成一个新的序列。在递归调用中，需要额外的开销将子序列的元素插入到当前序列中。

## Future

如果要使用 `Future`，需要在代码中导入相应的库：

```dart
import 'dart:async';
```

`Future` 表示一个异步任务在将来执行时产生的结果。`Flutter` 使用的 `Dart` 语言是单线程模型的语言，在 `Flutter` 中使用 `isolate`（协程）来表示一个异步任务。在 `Flutter` 线程的消息机制中，有一个消息循环（`event loop`）和两个队列（`event queue` 和 `microtask queue`）：

* `event queue` 中包含所有的的外来事件，包括 `I/O`、`mouse events`、`drawing events`、`timers` 以及 `isolate` 之间的消息等。在任意的 `isolate` 中新增的事件都会放入到 `even loop` 中等待执行
* `microtask queue` 只在当前 `isolate` 的任务队列中排队，优先级高于 `event queue`，主要是通过 `scheduleMicrotask` 来调度

![](https://upload-images.jianshu.io/upload_images/1975877-7f5e775270e16dad?imageMogr2/auto-orient/strip%7CimageView2/2/w/471)

`Flutter` 中的事件处理遵循如下规则：

* 首先处理 `microtask queue` 中的任务（事件）
* 处理完 `microtask queue` 中的所有任务之后，从 `event queue` 中取出一个任务进行处理
* 回到 `microtask queue`，处理其中的任务，然后重复上述过程

那么 `Flutter` 中是如何实现异步任务的呢？我们只需要将任务放入到 `microtask queue` 或者 `event queue` 中即可。通过 `Future.microtask` 接口（实际上使用了 `scheduleMicrotask`）可以将任务放置在 `microtask queue`。而通过 `Future` 的构造方法 `Future(FutureOr<T> computation())` 创建的任务则放置到了 `event queue` 中（实际上是通过 `Timer.run` 实现的）。

### 创建异步（同步）任务

`Future` 中提供了多个接口用于创建异步或同步任务：

* 使用 `factory Future(FutureOr<T> computation())`：该方法可以在 `event loop` 中创建一个异步任务

```dart
Future(() => print('在 event queue 中运行的 Future'));
```

* 使用 `factory Future.delayed(Duration duration, [FutureOr<T> computation()])`：该方法可以在 `event loop` 中创建一个延时执行的异步任务

```dart
Future.delayed(const Duration(seconds:1), () => print('1秒后在 event queue 中运行的 Future'));
```

* 使用 `factory Future.microtask(FutureOr<T> computation())`：该方法可以在 `microtask queue` 中创建一个异步任务

```dart
Future.microtask(() => print('在 microtask queue 里运行的 Future'));
```

* 使用 `factory Future.sync(FutureOr<T> computation())`：该方法可以创建一个同步任务

```dart
Future.sync(() => print('同步运行的Future'));
```

需要注意的是，`Future.sync` 创建的同步任务指的是构造 `Future` 是传入的 `computation()` 对象是同步执行的。但是通过 `then` 注册的回调方法是在 `microtask queue` 中执行的。

#### Completer

`future.dart` 中有一个很特殊的类：`Completer`。我们通过 `Future` 的接口向队列添加任务之后，我们只能被动的通过注册的回调方法接收结果或者处理错误。而 `Completer` 提供了一系列接口允许我们对 `Future` 的执行过程进行控制。一个比较好的示例是 [http/http.dart 库](https://github.com/dart-lang/http/blob/21e02c10c042cbd396d39b825dc7538ea6cccab8/lib/src/utils.dart) 中的 `store` 方法：

```dart
Future store(Stream stream, EventSink sink) {
  var completer = new Completer();
  stream.listen(sink.add, onError: sink.addError, onDone: () {
    sink.close();
    completer.complete();
  });
  return completer.future;
}
```

### then

`Future`通过向 `then()` 方法注册回调方法来接收返回的结果：

```dart
Future<R> then<R>(FutureOr<R> onValue(T value), {Function onError});
```

如果任务正确执行并返回了正确的值（非异常值），`onValue` 回调方法会被调用。此时通过 `value` 获取对应的返回值。另外，`then` 方法中还有一个可选的 `onError` 回调方法用于处理 `onValue` 回调方法执行过程中出现的异常信息。如果不注册该回调方法，那么将默认通过 `catchError` 返回异常信息。

需要注意的是，`onError` 的回调方法需要提供一个或两个参数，其中第二个参数为 可选的 `StackTrace` 对象。

需要注意的是，在肥肥测试的过程中 `onError` 并没有被执行，而是将异常信息交给了 `catchError` 进行处理。目前不确定是何种原因造成的问题。测试使用的代码如下：

```dart
getVersionName().then((value) {
  print(value);
  String result = null;
  result.substring(0, 4);
}, onError: (Object e, StackTrace st) {
  print('出现异常 onError： {e.toString()}');
}).catchError((Object e) {
  print('出现异常 catchError：{e.toString()}');
});
```

### catchError

`Future` 通过向 `catchError` 接口注册回调方法的方式来处理异常：

```dart
Future<T> catchError(Function onError, {bool test(Object error)});
```

异步任务执行的过程中出现的异常信息将会由 `catchError` 进行处理。

我们也可以通过 `Future.error` 返回错误信息，该错误信息也会由 `catchError` 进行处理。

另外，`catchError` 方法还有一个可选的 `test` 参数用于接收一个回调。如果该方法返回 `true`，那么 `catchError` 捕获的异常信息将会先由 `test` 参数对应的回调方法进行响应，然后再由 `onError` 参数对应的回调方法进行响应。如果返回 `false`，那么 `test` 参数对应的回调执行之后，`onError` 参数对应的回调则不再执行，应用程序将会抛出 `Unhandled Exception`。

### whenComplete

`Future` 通过向 `whenComplete` 接口注册回调方法的方式来保证特定的代码逻辑一定能够得到执行。

```dart
Future<T> whenComplete(FutureOr action());
```

与使用 `try-catch-finally` 捕获异常信息类似的，`Future` 可以通过 `whenComplete` 的回调方法保证无论何种情况都会保证代码的执行。

### Future.wait

如果需要调用多个异步方法并且一并将返回值返回，可以使用 `Future.wait` 接口。当所有 `Future` 完成后，返回 `List` 类型的值列表：

```dart
static Future<List<T>> wait<T>(Iterable<Future<T>> futures, {bool eagerError: false, void cleanUp(T successValue)})
```

### Future.doWhile

如果需要执行循环的任务，可以使用 `Future.doWhile`。只有当执行结果返回 `false` 的时候才会停止：

```dart
static Future doWhile(FutureOr<bool> action())
```
