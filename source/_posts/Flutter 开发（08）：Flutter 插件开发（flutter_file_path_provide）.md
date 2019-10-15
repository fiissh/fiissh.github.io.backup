---
title: Flutter 开发（08）：Flutter 插件开发（flutter_file_path_provide）
permalink: flutter-plugin-file-path-provider
comments: true
date: 2019-03-05 13:01:06
updated: 2019-03-05 13:01:06
tags:
  - Flutter 开发
  - Flutter 插件开发
  - flutter_file_path_provide
categories:
  - Flutter
---

本文主要介绍 `Flutter` 插件开发的相关内容，并在 [path_provider/path_provider.dart](https://github.com/flutter/plugins/tree/master/packages/path_provider) 插件项目的基础上开发我们自己的 `flutter_file_path_provide` 插件。

本文中涉及到的主要技术是基于 [Flutter 开发（07）：编写平台相关的代码](https://fiissh.tech/2019/flutter-platform-specific.html)。当然，在我们的 `flutter_file_path_provide` 插件项目中只使用到了 `MethodChannel`。另外两种平台通道我们将在其他文章中介绍。

由于肥肥从事 `Android SDK` 开发，所以 `flutter_file_path_provide` 插件项目中的 `iOS` 部分在本文撰写时并没有实现。
<!--more-->
## 创建插件项目

第一步，打开 `Android Studio`，在首页面选择 `Start a new Flutter project`：

![android-studio-welcome-page.jpg](/images/android-studio-welcome-page.jpg)

第二步，在 `New Flutter Project` 页面选择 `Flutter Plugin`，然后点击 `Next`：

![android-studio-select-flutter-plugin.jpg](/images/android-studio-select-flutter-plugin.jpg)

第三步，在 `Configure the new Flutter plugin` 界面填写相关的项目信息（`flutter_file_path_provide`），然后点击 `Next`：

![configure-new-flutter-plugin.jpg](/images/configure-new-flutter-plugin.jpg)

第四步，在 `Set the package name` 页面填写包名等信息（`tech.fiissh.flutter.plugin.path`），然后点击 `Finish`：

![set-package-name.jpg](/images/set-package-name.jpg)

如果我们的 `Android` 和 `iOS` 的部分需要使用 `Kotlin` 和 `Swift` 进行开发，请分别勾选 `Include Kotlin support for Android code` 和 `Include Swift support for iOS code`。默认情况下 `Android` 将使用 `Java` 语言，`iOS` 将使用 `Object-C` 语言。

本文中 `Android` 部分将使用 `Java` 语言进行开发。

第五步，等待项目构建完成之后，我们将得到如下项目结构：

![flutter-project.jpg](/images/flutter-project.jpg)

其中，`android` 目录是我们插件的 `Android` 部分的项目，用于编写 `Android` 的源代码。`ios` 目录是插件的 `iOS` 部分的项目，用于编写 `iOS` 的源代码。`example` 目录是我们插件项目的示例项目，其中包含了 `android` 和 `iOS` 两个平台的 `Module`。`lib` 目录则是我们编写 `Dart` 代码的地方。

虽然我们在创建项目的过程中填写的包名为 `tech.fiissh.flutter.plugin.path`，但是项目创建完成之后，插件的 `Android` 部分的包名则为 `tech.fiissh.flutter.plugin.flutter_file_path_provide`，而 `example` 的 `Android` 部分的包名则为 `tech.fiissh.flutter.plugin.flutter_file_path_provide_example`。如果我们需要修改包名，请将对应项目的 `AndroidManifest.xml` 文件中的 `package` 属性、`build.gradle` 文件中的 `applicationId` 属性都进行修改。修改之后，插件部分我们使用 `tech.fiissh.flutter.plugin.path` 包名，`example` 部分我们使用 `tech.fiissh.flutter.sample.path` 包名。

需要注意的是，修改报名之后，我们需要将项目根目录的 `pubspec.yaml` 文件中的 `plugin` 属性修改为新的包名：

```yaml
flutter:
  # This section identifies this Flutter project as a plugin project.
  # The androidPackage and pluginClass identifiers should not ordinarily
  # be modified. They are used by the tooling to maintain consistency when
  # adding or updating assets for this project.
  plugin:
    androidPackage: tech.fiissh.flutter.plugin.path
    pluginClass: FlutterFilePathProvidePlugin
```

## 开发插件

### 需求分析

我们插件的主要功能是获取对应设备平台上的可用的存储目录，并将该目录的绝对地址实例化为 `Directory` 对象。根据 `Android` 和 `iOS` 上常用的存储目录，我们总结出几个不同的目录，对应的 `Flutter` 层的方法如下：

* `getExternalStorageDirectory`：`Android` 平台下读取外部存储的路径，即 `SD` 卡目录，例如 `/storage/sdcard0/`
* `getExternalStorageApplicationTemporaryDirectory`：`Android` 平台下外部存储的缓存目录，例如 `/storage/emulated/0/Android/data/tech.fiissh.sample.path/temp/`
* `getInternalStorageApplicationTemporaryDirectory`： `Android` 平台下内部存储的缓存目录，例如 `/data/user/0/tech.fiissh.flutter.sample.path/temp/`
* `getExternalStorageApplicationCacheDirectory`: `Android` 平台下的外部存储的应用程序缓存目录，例如 `/storage/emulated/0/Android/data/tech.fiissh.sample.path/cache/`
* `getInternalStorageApplicationCacheDirectory`：`Android ` 平台下的内部存储的应用程序缓存目录，例如 `/data/user/0/tech.fiissh.flutter.sample.path/cache/`
* `getExternalStorageApplicationDirectory`： `Android` 平台下外部存储的应用程序目录，例如 `/storage/emulated/0/Android/data/tech.fiissh.sample.path/files/`
* `getInternalStorageApplicationDirectory`： `Android` 平台下内部存储的应用程序目录，例如 `/data/user/0/tech.fiissh.flutter.sample.path/files/`

### 插件的开发

#### Flutter 层的开发

项目创建完成之后，`Android Studio` 已经为我们创建了一个简单的获取设备平台操作系统版本号的示例。

我们打开 `lib/flutter_file_path_provide.dart` 文件，删除 `platformVersion` 方法，得到如下代码：

```dart
class FlutterFilePathProvide {
  static const MethodChannel _channel = const MethodChannel('flutter_file_path_provide');
}
```

其中，`FlutterFilePathProvide` 类中已经默认创建了一个名称为 `flutter_file_path_provide` 的 `MethodChannel` 对象 `_channel`。

根据我们在需求分析中定义的方法，我们得到如下 `FlutterFilePathProvide` 类：

```dart
class FlutterFilePathProvide {
  const FlutterFilePathProvide();

  static const String _METHOD_CHANNEL_NAME = 'fiissh.flutter.plugin/flutter_file_path_provide';
  static const MethodChannel _channel = const MethodChannel(_METHOD_CHANNEL_NAME);

  static const String _EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME = "getExternalStorageDirectory";
  static const String _EXTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY = "getExternalStorageApplicationTemporaryDirectory";
  static const String _INTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY = "getInternalStorageApplicationTemporaryDirectory";
  static const String _EXTERNAL_STORAGE_APPLICATION_CACHE_DIRECTORY_METHOD_NAME = "getExternalStorageApplicationCacheDirectory";

  static const String _EXTERNAL_STORAGE_APPLICATION_DIRECTORY_METHOD_NAME = "getExternalStorageApplicationDirectory";
  static const String _INTERNAL_STORAGE_APPLICATION_CACHE_DIRECTOR_METHOD_NAME = "getInternalStorageApplicationCacheDirectory";
  static const String _INTERNAL_STORAGE_APPLICATION_DIRECTOR_METHOD_NAME = "getInternalStorageApplicationDirectory";

  Future<Result> getExternalStorageDirectory() async {
    return _getDirectory(_EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME);
  }

  Future<Result> getExternalStorageApplicationCacheDirectory() async {
    return _getDirectory(_EXTERNAL_STORAGE_APPLICATION_CACHE_DIRECTORY_METHOD_NAME);
  }

  Future<Result> getInternalStorageApplicationCacheDirectory() async {
    return _getDirectory(_INTERNAL_STORAGE_APPLICATION_CACHE_DIRECTOR_METHOD_NAME);
  }

  Future<Result> getExternalStorageApplicationDirectory() async {
    return _getDirectory(_EXTERNAL_STORAGE_APPLICATION_DIRECTORY_METHOD_NAME);
  }

  Future<Result> getInternalStorageApplicationDirectory() async {
    return _getDirectory(_INTERNAL_STORAGE_APPLICATION_DIRECTOR_METHOD_NAME);
  }

  Future<Result> getExternalStorageApplicationTemporaryDirectory() async {
    return _getDirectory(_EXTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY);
  }

  Future<Result> getInternalStorageApplicationTemporaryDirectory() async {
    return _getDirectory(_INTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY);
  }

  Future<Result> _getDirectory(String methodName, [String extra]) async {
    String path;
    try {
      path = await _channel.invokeMethod(methodName, extra);
      print(path);
      return Result(Result.ERROR_CODE_SUCCESS, 'success', Directory('$path/'));
    } on PlatformException catch (e) {
      return Result(num.parse(e.code), e.message);
    }
  }
}
```

其中，`_getDirectory(String methodName, [String extra])` 方法是所有方法调用的核心。其作用是 `_channel` 对象根据传递进来的 `methodName` 参数以及可选参数 `extra`，将调用传递到设备平台上与 `MethodChannel` 对应的 `MethodCallHandler`（`android/src/main/java/tech.fiissh.flutter.plugin.path/FlutterFilePathProvidePlugin`）对象。

`MethodChannel` 类除了 `name` 和 `codec` 属性之外，还有另外几个重要的方法：

* `Future<T> invokeMethod<T>(String method, [dynamic arguments])`
* `Future<List<T>> invokeListMethod<T>(String method, [dynamic arguments])`
* `Future<Map<K, V>> invokeMapMethod<K, V>(String method, [dynamic arguments])`

上述三个方法，是 `MethodChannel` 方法调用最常用的方法。通过泛型的定义方式，使得其返回值可以有多种类型。另外的两个方法 `setMethodCallHandler` 和 `setMockMethodCallHandler` 则是用于设备平台通过平台通道调用 `Flutter` 层方法时使用。

我们重点说一下 `_getDirectory` 方法。该方法是 `Flutter` 层发起调用并处理结果的最核心的方法。我们看到 `await _channel.invokeMethod(methodName, extra);` 调用之后会得到返回值 `path`，此时 `path` 就是我们需要的结果（`path` 为空的情况已经在 `Android` 端进行了判断）。此处通过异常捕获的方式捕获了 `PlatformException` 对象，并将该异常信息写入到 `Result` 中并返回。那么，`PlatformException` 是如何产生的呢？我们将会在 `Android` 端代码实现中做讲解。

`Result` 类用于对外提供结果，其定义如下：

```dart
class Result {
  /// 调用成功，此时 _directory 不为 null
  static const int ERROR_CODE_SUCCESS = 0;

  /// 调用失败，设备平台返回值为空
  static const int ERROR_CODE_UN_SUCCESS_NULL = 1;

  /// 调用失败，设备平台出现异常
  static const int ERROR_CODE_UN_SUCCESS_EXCEPTION = 2;

  ///  调用失败，没有对应的方法
  static const int ERROR_CODE_UN_SUCCESS_NOT_IMPLEMENT_METHOD = 3;
  Directory _directory;

  int _errorCode = 0;
  String _errorMessage = '';

  Result(this._errorCode, [this._errorMessage, this._directory]);

  String get errorMessage => _errorMessage;

  int get errorCode => _errorCode;

  Directory get directory => _directory;
}
```

#### Android 端的开发

`Flutter` 层的代码开发完成之后，我们调用 `Flutter` 的方法，该方法将最终在 `Android` 端产生响应。

`Android` 层的代码实现在类 `FlutterFilePathProvidePlugin.java` 中：

```Java
package tech.fiissh.flutter.plugin.path;

import android.os.Environment;
import android.text.TextUtils;

import io.flutter.plugin.common.MethodCall;
import io.flutter.plugin.common.MethodChannel;
import io.flutter.plugin.common.MethodChannel.MethodCallHandler;
import io.flutter.plugin.common.MethodChannel.Result;
import io.flutter.plugin.common.PluginRegistry.Registrar;

/**
 * FlutterFilePathProvidePlugin
 */
public class FlutterFilePathProvidePlugin implements MethodCallHandler {

    private static final String _METHOD_CHANNEL_NAME = "fiissh.flutter.plugin/flutter_file_path_provide";
    private static final String _EXTERNAL_STORAGE_DIRECTORY_METHOD_NAME = "getExternalStorageDirectory";
    private static final String _EXTERNAL_STORAGE_APPLICATION_CACHE_DIRECTORY_METHOD_NAME = "getExternalStorageApplicationCacheDirectory";
    private static final String _EXTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY = "getExternalStorageApplicationTemporaryDirectory";
    private static final String _INTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY = "getInternalStorageApplicationTemporaryDirectory";

    private static final String _EXTERNAL_STORAGE_APPLICATION_DIRECTORY_METHOD_NAME = "getExternalStorageApplicationDirectory";
    private static final String _INTERNAL_STORAGE_APPLICATION_CACHE_DIRECTOR_METHOD_NAME = "getInternalStorageApplicationCacheDirectory";
    private static final String _INTERNAL_STORAGE_APPLICATION_DIRECTOR_METHOD_NAME = "getInternalStorageApplicationDirectory";
    private static Registrar registrar;


    /**
     * Plugin registration.
     */
    public static void registerWith(Registrar registrar) {
        FlutterFilePathProvidePlugin.registrar = registrar;
        final MethodChannel channel = new MethodChannel(registrar.messenger(), _METHOD_CHANNEL_NAME);
        channel.setMethodCallHandler(new FlutterFilePathProvidePlugin());
    }

    @Override
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
        } else if (TextUtils.equals(call.method, _EXTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY)) {

            try {
                String path = getExternalStorageApplicationTemporaryDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }
        } else if (TextUtils.equals(call.method, _INTERNAL_STORAGE_APPLICATION_TEMPORARY_DIRECTORY)) {

            try {
                String path = getInternalStorageApplicationTemporaryDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }
        } else if (TextUtils.equals(call.method, _EXTERNAL_STORAGE_APPLICATION_CACHE_DIRECTORY_METHOD_NAME)) {

            try {
                String path = getExternalStorageApplicationCacheDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }
        } else if (TextUtils.equals(call.method, _EXTERNAL_STORAGE_APPLICATION_DIRECTORY_METHOD_NAME)) {

            try {
                String path = getExternalStorageApplicationDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }
        } else if (TextUtils.equals(call.method, _INTERNAL_STORAGE_APPLICATION_CACHE_DIRECTOR_METHOD_NAME)) {
            try {
                String path = getInternalStorageApplicationCacheDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }

        } else if (TextUtils.equals(call.method, _INTERNAL_STORAGE_APPLICATION_DIRECTOR_METHOD_NAME)) {
            try {
                String path = getInternalStorageApplicationDirectory();
                if (TextUtils.isEmpty(path)) {
                    result.error("1", "path is null", "");
                } else {
                    result.success(path);
                }
            } catch (Exception e) {
                result.error("2", e.getMessage(), "");
            }
        } else {
            result.error("3", "method notImplemented", "");
        }
    }

    private String getExternalStorageDirectory() {
        return Environment.getExternalStorageDirectory().getAbsolutePath();
    }

    private String getExternalStorageApplicationTemporaryDirectory() {
        return registrar.activeContext().getExternalCacheDir().getParent() + "/temp";
    }

    private String getInternalStorageApplicationTemporaryDirectory() {
        return registrar.activeContext().getCacheDir().getParent() + "/temp";
    }


    private String getExternalStorageApplicationCacheDirectory() {
        return registrar.activeContext().getExternalCacheDir().getAbsolutePath();
    }

    private String getInternalStorageApplicationCacheDirectory() {
        return registrar.activeContext().getCacheDir().getAbsolutePath();
    }

    private String getExternalStorageApplicationDirectory() {
        return registrar.activeContext().getExternalFilesDir(null).getAbsolutePath();
    }

    /// 内部存储
    private String getInternalStorageApplicationDirectory() {
        return registrar.activeContext().getFilesDir().getAbsolutePath();
    }
}
```

`FlutterFilePathProvidePlugin` 继承自 `MethodCallHandler` 并实现了 `onMethodCall(MethodCall call, Result result)`。该方法的作用是接收来自于 `Flutter` 层的方法调用，并将方法名称和参数等信息通过回调的方式传递到 `Android` 层。

上述代码中，我们通过 `call.method` 和方法名称的一一对应关系来最终完成 `Android` 层代码的调用。以其中一个 `if` 语句为例，我们可以看到，在 `Android` 层的方法调用成功之后，会通过调用 `result.success` 将结果回写到 `Flutter` 层。如果调用不成功，则需要调用 `result.error` 将错误信息回写到 `Flutter` 层。

`Flutter` 层通过捕获 `PlatformException` 的方式读取到回写的错误信息。也就是说，`Android` 层的 `result.error` 调用最终体现在 `Flutter` 层是一个 `PlatformException` 异常。

事实上，`result` 还有一个 `notImplemented` 方法用于表示找不到对应的方法。在上述示例中我们是以 `result.error` 调用的方法替代了 `notImplemented` 方法。

静态方法 `registerWith` 最终在 `ManiActivity` 中得到调用：

```Java
FlutterFilePathProvidePlugin.registerWith(registry.registrarFor("tech.fiissh.flutter.plugin.path.FlutterFilePathProvidePlugin"));
```

上述方法中，`registrarFor` 方法最终返回一个 `PluginRegistry.Registrar` 对象：

```Java
public interface Registrar {
    Activity activity();

    Context context();

    Context activeContext();

    BinaryMessenger messenger();

    TextureRegistry textures();

    PlatformViewRegistry platformViewRegistry();

    FlutterView view();

    String lookupKeyForAsset(String var1);

    String lookupKeyForAsset(String var1, String var2);

    PluginRegistry.Registrar publish(Object var1);

    PluginRegistry.Registrar addRequestPermissionsResultListener(PluginRegistry.RequestPermissionsResultListener var1);

    PluginRegistry.Registrar addActivityResultListener(PluginRegistry.ActivityResultListener var1);

    PluginRegistry.Registrar addNewIntentListener(PluginRegistry.NewIntentListener var1);

    PluginRegistry.Registrar addUserLeaveHintListener(PluginRegistry.UserLeaveHintListener var1);

    PluginRegistry.Registrar addViewDestroyListener(PluginRegistry.ViewDestroyListener var1);
}
```

`Registrar` 中持有当前 `Activity` 的 `Activity` 和 `Context` 实例，并定义了一些动态权限申请、打开其他 `Activity` 等相关的方法。

## 发布插件

在插件开发完成之后（当然，`flutter_file_path_provide` 插件还没有开发完成），可以将插件发布到 [Pub](https://pub.dartlang.org) 上。

插件发布之前我们需要做如下准备工作：

* 修改 `README.md`，撰写插件的使用说明
* 修改 `CHANGELOG.md`，记录插件的版本变更
* 修改 `LICENSE` 条款的文件
* 撰写所有公共 `API` 说明

上述准备工作完成之后，使用如下命令进行检查：

```shell
flutter packages pub publish --dry-run
```

最后，通过如下命令进行发布：

```shell
flutter packages pub publish
```

关于发布插件的详细信息，请参考 [Publishing a Package](https://www.dartlang.org/tools/pub/publishing)

## 关于 flutter_file_path_provide

虽然 `flutter_file_path_provide` 项目尚处于开发阶段，但是该项目已经托管到 `Github`：[fiissh/flutter_file_path_provider](https://github.com/fiissh/flutter_file_path_provider)。

如果您对该项目感兴趣或者有任何建议或意见，可以联系我（fiissh.zhao@gmail.com）。

肥肥对于 `flutter_file_path_provide` 项目的计划是，完成 `iOS` 端代码的开发，经过测试之后正式发布到[Pub](https://pub.dartlang.org)。
