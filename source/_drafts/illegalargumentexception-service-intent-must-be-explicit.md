---
title: IllegalArgumentException Service Intent must be explicit
permalink: illegalargument-exception-service-intent-must-be-explicit
comments: true
date: 2019-03-21 17:21:54
updated: 2019-03-21 17:21:54
tags:
   - IllegalArgumentException
   - explicit Intent
categories:
  - Android Exception
---

在 `AndroidMenifest` 文件中注册一个 `Service`，并设置其 `action`属性：

```xml
<service
        android:name=".MathService"
        android:process=".math">
    <intent-filter>
        <action android:name="tech.fiissh.sample.math"/>
    </intent-filter>
</service>
```

然后使用 `Intent` 绑定该 `Service` 时抛出异常：

```Java
Intent intent = new Intent("tech.fiissh.sample.math")
bindService(intent, connection, Context.BIND_AUTO_CREATE);
```

异常信息：

```shell
Caused by: java.lang.IllegalArgumentException: Service Intent must be explicit: Intent
   at android.app.ContextImpl.validateServiceIntent(ContextImpl.java:1464)
   at android.app.ContextImpl.bindServiceCommon(ContextImpl.java:1609)
   at android.app.ContextImpl.bindService(ContextImpl.java:1557)
   at android.content.ContextWrapper.bindService(ContextWrapper.java:684)
```

造成该问题的主要原因是从 `Lollipop` 开始，启动 `Service` 必须使用显式 `Intent`。解决该问题的方案是为 `Intent` 添加 `package` 属性：

```java
Intent intent = new Intent("tech.fiissh.sample.math");
intent.setPackage(getPackageName());
bindService(intent, connection, Context.BIND_AUTO_CREATE);
```

更为详细的讨论请移步 [IllegalArgumentException: Service Intent must be explicit, after upgrading to Android L Dev Preview](https://stackoverflow.com/questions/24480069/google-in-app-billing-illegalargumentexception-service-intent-must-be-explicit/26318757#26318757)。
