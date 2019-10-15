---
title: Android Studio 编译 Resolved versions for app (26.1.0) and test app (27.1.1) differ 错误
permalink: resolved-version-for-app-and-test-app-differ-error
comments: true
date: 2019-03-08 12:04:59
updated: 2019-03-08 12:04:59
tags:
  - Resolved versions differ
categories:
  - Android Studio Exception
---
`AndroidStudio` 新建项目，编译时报错：

```shell
Conflict with dependency 'com.android.support:support-annotations' in project ':app'. Resolved versions for app (26.1.0) and test app (27.1.1) differ. See https://d.android.com/r/tools/test-apk-dependency-conflicts.html for details.
```

从报错信息来看，是 `app` 和 `test` 的依赖版本不一致造成的。项目 `build.gradle` 中的 `dependencies` 如下：

```shell
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support:support-v4:26.1.0'
    implementation 'com.android.support:design:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

目前来看，应该是 `support` 包与 `test` 包的版本不一致造成的。解决方案如下：

方案一：升级 `compileSdkVersion` 到 27（目前为 26），并将 `support` 包版本升级到 27

```shell
implementation 'com.android.support:appcompat-v7:27.1.1'
implementation 'com.android.support:support-v4:27.1.1'
implementation 'com.android.support:design:27.1.1'
```

方案二：将 `test` 包版本降低为 26 对应的版本：

```shell
androidTestImplementation 'com.android.support.test:runner:1.0.1'
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'

```

方案三：在 `build.gradle` 文件的 `android{ }` 属性前增加如下代码：

```shell
configurations.all { resolutionStrategy.force 'com.android.support:support-annotations:26.1.0' }
```
