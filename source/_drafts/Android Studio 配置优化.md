---
title: Android Studio 配置优化
permalink: android-studio-config-index
comments: true
date: 2018-11-29 11:07:14
updated: 2018-11-29 11:07:14
tags:
  - Android Studio
  - Android Studio 配置
categories:
  - 随手记录
---

`Android Studio` 的 `studio.vmoptions` 配置文件：

```shell
-server
-Xms4096m
-Xmx4096m
-XX:ReservedCodeCacheSize=1024m
-XX:NewRatio=3
-Xmn2048m
-XX:+UseParNewGC
-XX:+CMSScavengeBeforeRemark
-XX:CMSInitiatingOccupancyFraction=126
-XX:+UseCMSInitiatingOccupancyOnly
-XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=3
-Xss32m
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:ConcGCThreads=8
-XX:+AlwaysPreTouch
-XX:+TieredCompilation
-XX:+UseCompressedOops
-Djsse.enableSNIExtension=false
-Dfile.encoding=UTF-8
-XX:SoftRefLRUPolicyMSPerMB=126
-da
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true

-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Xverify:none
-Djna.nosys=true
-Djna.boot.library.path=
-Xbootclasspath/a:../lib/boot.jar
```

`Android Studio` 依赖的 `Gradle` 配置文件 `gradle.properties`：

```shell
org.gradle.daemon=true
android.enableD8 = true
-Xmx1024m
-XX:MaxPermSize=512m
org.gradle.jvmargs=-Xmx2048m
-XX:MaxPermSize=512m
-XX:+HeapDumpOnOutOfMemoryError
-Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.configureondemand=true
```

上述配置基于 `16G` 内存。
