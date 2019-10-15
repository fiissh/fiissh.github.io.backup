---
title: Flutter 开发（02）：Flutter 开发环境的搭建（macOS）
permalink: fluuter-macos-environemnt-setup
comments: true
date: 2019-02-26 15:31:21
updated: 2019-02-26 15:31:21
tags:
  - Flutter 环境搭建
categories:
  - Flutter
---

本文主要介绍 `macOS` 下 `Flutter` 开发环境的搭建。由于肥肥主要从事 `Android SDK` 开发，故本文中一些内容仅围绕 `Android` 展开。

`Android` 和 `iOS` 开发环境的搭建不在本文的讨论范围。

本文基于 `Flutter V1.0.0` 展开讨论。

<!--more-->
## 系统要求

如果需要安装运行 `Flutter`，那么您的设备应该满足如下最低要求：

* `64-bit` 的 `macOS`
* 不少于 `700M` 的磁盘可用空间
* 已经正常安装 `AndroidStudio` 或 `IntelliJ IDEA` 和 `XCode`
* 已经正常安装如下命令行工具：
  * `bash`
  * `mkdir`
  * `rm`
  * `git`
  * `curl`
  * `unzip`
  * `which`
  * 可能会需要使用 `brew`

需要注意的是，畅通的网络环境是 `Flutter` 开发必不可少的。

## 获取并安装 Flutter SDK

### 下载 Flutter SDK

我们可以在 `Flutter` 官网的 [Flutter SDK archive](https://flutter.dev/docs/development/tools/sdk/archive?tab=macos#macos) 获取最新的 `Flutter SDK`。

目前最新的 `Flutter SDK` 为 `V1.0.0` 版本。您可以直接点击 [下载 Flutter SDK V1.0.0 Stable for macOS](https://storage.googleapis.com/flutter_infra/releases/stable/macos/flutter_macos_v1.0.0-stable.zip) 来下载官方的 `Flutter SDK` 版本，也可以通过点击 [下载 Flutter SDK V1.0.0 Stable for macOS](https://pan.baidu.com/s/1f7es65Kyg2RMb2aVEHErFw) 从百度网盘进行下载。

`Github` 上也托管了 `Flutter SDK`，我们也可以在 [flutter/flutter release](https://github.com/flutter/flutter/releases) 上下载对应的安装包。

如果上述官方下载地址均无法访问，请使用 `Google` 为国内用户部署的镜像服务下载。[点击使用国内镜像下载 Flutter SDK V1.0.0 Stable for macOS](https://https://flutter-io.cn/flutter_infra/releases/stable/macos/flutter_macos_v1.0.0-stable.zip)。

`Flutter` 国内镜像的地址为 [https://flutter-io.cn](https://flutter-io.cn)。

如果在后续使用 `Flutter` 的过程中由于网络而造成的问题，比如执行 `flutter doctor` 时无法访问网络，或者创建 `Flutter` 项目时一直处于 `create flutter project` 过程中，那么请尝试使用代理或者在 `.bash_profile` 文件中做如下配置：

```shell
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

### 解压缩 Flutter SDK

在下载好 `Flutter SDK` 之后，我们需要将 `Flutter SDK` 解压缩：

```shell
unzip flutter_macos_v1.0.0-stable.zip
```

### 配置环境变量

第一步，打开 `.bash_profile` 文件（路径通常是 `~/.bash_profile`）：

```shell
vim ~/.bash_profile
```

第二步，在 `.bash_profile` 文件的末尾添加如下字段：

```shell
export PATH=PATH_TO_FLUTTER_GIT_DIRECTORY/flutter/bin:$PATH
````

其中，`PATH_TO_FLUTTER_GIT_DIRECTORY` 为你的 `Flutter SDK` 安装路径。

保存并退出编辑器。

第三步，执行如下命令刷新环境变量：

```shell
source ~/.bash_profile
```

> 如果使用 `zsh` 终端，请在 `~/.zshrc` 文件的末尾添加 `source ~/.bash_profile`。

### 运行 flutter doctor

运行如下命令查看是否需要安装其他依赖项：

```shell
flutter doctor
```

该命令会对设备的环境进行检测并最终显示一份检测结果。请仔细阅读错误提示并按照要求进行安装。下图显示了检测通过之后的最终输出：

![flutter-doctor-result.jpg](/images/flutter-doctor-result.jpg)

> 由于肥肥使用 `IntelliJ IDEA` 进行 `Android` 开发，故而忽略 `Android Studio` 未安装的提示。

### 安装插件（Android 环境）

无论使用 `Android Studio` 还是 `IntelliJ IDEA`，都需要安装 `Dart` 插件和 `Flutter` 插件。

`Dart` 插件的作用是提供对 `Dart` 语言的基础支持，`Flutter` 插件则提供对 `Flutter` 开发的支持。

安装插件的具体方案请参考 `Android Studio` 或 `IntelliJ IDEA` 对安装插件的介绍。

## 创建项目

选择 `File > New Flutter Project > Flutter application `，输入项目名称（比如 `flutter_sample`），最终可以创建一个 `Flutter` 项目:

![flutter-sample-project.jpg](/images/flutter-sample-project.jpg)

其中，`android` 目录是是配到 `Android` 设备的项目，`ios` 目录是是配到 `iOS` 设备的项目，`lib` 目录是我们编写的 `Dart` 代码的目录。

> 与纯粹的 `Android` 开发不同的是，在 `Android Studio` 中也可以将项目部署到 `iOS`。

此时，点击 `Run` 运行项目，编译部署之后，我们可以看到默认的 `Flutter` 项目的界面。

### Flutter 项目的热重载

`Flutter` 可以通过热重载实现快速的开发周期，热重载的目的是无需重启应用程序就能实时加载修改后的代码，并且不会丢失状态：

* 在应用运行之后，不要点击 `Stop` 结束运行
* 修改相应的代码
* 调用 `Sava All`（保存）或者点击热重载按钮（⚡️）

之后，应用程序会立刻更新为我们修改完之后的状态。

热重载技术是通过将最新的源代码文件注入到正在运行的 `Dart VM` 中来实现的。这个过程不仅仅支持修改字符串内容，还支持添加新的类、新的方法和字段以及更新现有的方法。但是需要注意的是，如下几种类型的更改无法通过热更新的方式实现：

* 全局变量的初始化
* 静态字段的初始化
* `main()` 应用程序的方法
