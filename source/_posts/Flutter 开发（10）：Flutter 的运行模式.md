---
title: Flutter 开发（10）：Flutter 的运行模式
permalink: flutter-running-mode
comments: true
date: 2019-03-14 11:26:40
updated: 2019-03-14 11:26:40
tags:
  - Flutter 运行模式
categories:
  - Flutter
---

本文主要是对 `Flutter` 的基本框架进行介绍。本文主要参考了 [美团技术团队-Flutter 原理与实践](https://tech.meituan.com/2018/08/09/waimai-flutter-practice.html) 一文。[美团技术团队-Flutter 原理与实践](https://tech.meituan.com/2018/08/09/waimai-flutter-practice.html) 是一篇非常优秀的介绍 `Flutter` 原理的文章。

`Flutter` 的设计目标是高性能的跨平台的（`Android` 和 `iOS`）`UI` 框架。与传统的基于 `WebView` 的（例如 `Cordova` 或者 `AppCan` 等框架）或者基于原生控件系统的（例如 `React Native`）跨平台开发框架所不同的是，`Flutter` 是基于跨平台的渲染引擎 `Skia` 来实现的全新的跨平台的开发框架。这也就是说，`Flutter` 对于设备平台的依赖只有图形渲染相关的部分，这在一定程度上保证了不同平台、不同设备上的一致的用户体验。从执行效率的角度来说，`Flutter` 使用的 `Dart` 语言相较于其他大多数跨平台开发框架使用的 `JavaScript` 语言在执行效率上高效很多。
<!--more-->
## Flutter 架构

`Flutter` 的架构主要由三部分组成：

* `Framework` 层
* `Engine` 层
* `Embedder` 层

![Flutter 架构](http://www.techug.com/wordpress/wp-content/uploads/2018/07/66372-20180706175503147-184515056.jpg)

### Framework 层

`Flutter` 的 `Framework` 层使用 `Dart` 语言实现，包括了 `Material Design` （针对于 `Android` 平台）和 `Cupertino` （对于 `iOS` 平台）两种风格的文本、图片、按钮等基础 `Widget`、以及动画、手势处理和渲染引擎等模块。这一部分的源代码对应的是 `Flutter` 仓库下的 [flutter/packages/](https://github.com/flutter/flutter/tree/master/packages) `Flutter` 包以及 [engine/sky/packages/sky_engine/](https://github.com/flutter/engine/tree/master/sky/packages/sky_engine) 仓库下的 `io`、`async` 和 `ui` 等 `package`。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/9924fbed.png)


`Flutter` 提供了非常丰富的控件，通过对不同类型的控件的嵌套和组合，我们可以方便的构建出任意功能、任意复杂度的界面。与其他将控件、控制器、布局以及其他属性分离开的框架不同的是，`Flutter` 提供了统一的对象模型：`Widget`。

`Widget` 本身由许多更小的、功能单一的 `Widget` 组成。

### Engine 层

`Flutter` 的 `Engine` 层主要包含 `Skia`、`Dart` 和 `Text` 三部分。`Engine` 主要使用 `C/C++` 开发。

`Skia` 是开源的二维图形渲染库，提供了适用于多种平台的通用 `API`。

`Dart` 主要包括 `Dart Runtime`、`Garbage Collection`，如果是 `Debug` 模式的话，该部分还提供了对 `JIT`（`Just In Time`）的支持。`Release` 和 `Profile` 模式下，是通过 `AOT`（`Ahead of Time`）编译成了原生代码，并不包含 `JIT` 部分。

`Text` 是文本渲染库。`Android` 上使用 `FreeType` 渲染，`iOS`上使用 `CoreGraphics` 进行渲染。

### Embedder 层

`Embedder` 层主要包含 `Surface`、线程以及插件等，是 `Flutter` 的嵌入层，主要负责把 `Flutter` 嵌入到各个平台上。

## Flutter 的运行模式

`Flutter` 提供了四种运行模式：

* `Debug` 模式
* `Release` 模式
* `Profile` 模式
* `Test` 模式

上述四种模式在 `Flutter` 编译时是完全独立的。

### Debug 模式

`Debug` 模式可以在真机和模拟器上运行。该模式会打开所有的断言（包括 `debugging` 信息、`debugger aida` 和服务扩展），优化了快速 `develop/run` 循环，但是没有优化执行速度、二进制文件和部署。`Android Studio` 的 `run` 命令或者 `flutter run` 命名都是以该模式运行应用程序的。对应的编译命令为 `sky/tools/gn --android` 或者  `sky/tools/gn --ios`。

### Release 模式

`Release` 模式只能在真机中运行，不能在模拟器中运行。该模式会关闭所有的断言（包括 `debugging` 信息、`debugger aida` 和服务扩展），优化启动速度并减少包体积。该模式是为了部署应用程序给最终的用户使用的。使用 `flutter run --release` 命令可以让应用程序在该模式下运行。对应的编译命令为 `sky/tools/gn --android --runtime-mode=release` 或者 `sky/tools/gn --ios --runtime-mode=release`。

### Profile 模式

`Profile` 模式只能在真机上运行，不能在模拟器中运行。该模式除了启用了服务扩展和 `tracing`（包含一些最低限度支持 `tracing` 运行的功能），其他方面和 `Release` 模式是一样的。使用 `flutter run --profile` 命令可以应用程序在该模式下运行。对应的编译命令为 `sky/tools/gn --android --runtime-mode=profile` 或者 `sky/tools/gn --ios --runtime-mode=profile`。

### Test 模式

`Test` 模式只能在桌面上运行，与 `Debug` 模式基本上一样。

## Flutter 应用程序运行机制

在 `Android` 系统上，`Flutter` 构建出的 `APK` 在运行时会将所有 `assets` 目录下的资源文件解压到应用程序私有文件目录中的 `flutter` 目录下，主要包括处理字符编码的 `icudtl.dat`，以及 `Debug` 模式的 `kernel_blob.bin`、`platform.dill` 和 `Release` 模式下的4个 `snapshot` 文件。

默认情况下，`Flutter` 在 `Application` 的 `onCreate` 时调用 `FlutterMain` 的 `startInitialization` 方法来启动解压任务，然后在 `FlutterActivityDelegate` 的 `onCreate` 中调用 `FlutterMain` 的 `ensureInitializationComplete` 等待解压任务结束。

`Flutter` 在 `Debug` 模式下使用 `JIT` 编译方式，主要是为了支持热加载功能：

![](https://p1.meituan.net/travelcube/89aa7ce412124d5618877589b07f115426247.png)

触发热加载时 `Flutter` 会检测发生改变的 `Dart` 文件，将其同步到应用程序私有缓存目录下，`Dart VM` 加载并且修改对应的类或者方法，重建控件树后立即就可以在设备上看到效果。

在 `Release` 模式下，`Flutter` 会直接将 `snapshot` 文件映射到内存中执行其中的指令：

![](https://p0.meituan.net/travelcube/ef7a282f280957a1a67020a2aa7922f524600.png)

在 `Release` 模式下，`FlutterActivityDelegate` 的 `onCreate` 中调用 `FlutterMain` 的 `ensureInitializationComplete` 方法时会将 `AndroidManifest` 中设置的 `snapshot` 文件名等运行参数设置到对应的 `C/C++` 同名类对象中，构造 `FlutterNativeView` 实例时调用 `nativeAttach` 来初始化 `Dart VM`，最后运行编译好的 `Dart` 代码。
