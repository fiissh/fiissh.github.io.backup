---
title: OutOfMemoryError:pthread_create (1040KB stack) failed 异常分析
permalink: oom-exception-caused-by-pthread-create-failed
comments: true
date: 2020-03-03 11:17:57
updated: 2020-03-03 11:17:57
tags:
  - OutOfMemoryError
categories:
  - Exception
---

线上出现一些 `OutOfMemoryError`，经过分析崩溃数据，发现出现 `OOM` 时进程的可用空间还是非常大的。

从崩溃的堆栈信息中可以分析出引发该 `OOM` 的主要与 `OkHttp` 有关系，而业务场景中都是普通的数据请求（`JSON`），并没有使用 `BitMap` 这一类的较大内存占用的资源。更何况，进程的可用内存空间还是很充足的。

虽然堆栈信息不尽相同，但是最终从大量堆栈 `LOG` 中分析出有价值的信息：

```java
java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Try again
at java.lang.Thread.nativeCreate(Native Method)
at java.lang.Thread.start(Thread.java:733)
at tech.fiissh.base.utils.c.u(SourceFile:7)
at tech.fiissh.base.common.e.c.a(SourceFile:40)
at tech.fiissh.base.common.e.c.a(SourceFile:156)
at tech.fiissh.base.common.e.b.a(SourceFile:30)
at tech.fiissh.base.common.e.b.a$1.handleMessage(SourceFile:6)
at android.os.Handler.dispatchMessage(Handler.java:105)
at android.os.Looper.loop(Looper.java:164)
at android.app.ActivityThread.main(ActivityThread.java:6942)
at java.lang.reflect.Method.invoke(Native Method)
at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:327)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1374)

java.lang.OutOfMemoryErrorat java.lang.String.<init>(String.java:255)
at libcore.io.IoUtils$FileReader.toString(IoUtils.java:272)
at libcore.io.IoUtils.readFileAsString(IoUtils.java:114)
at com.android.org.conscrypt.CertPinManager.readPinFile(CertPinManager.java:111)
at com.android.org.conscrypt.CertPinManager.rebuild(CertPinManager.java:85)
at com.android.org.conscrypt.CertPinManager.<init>(CertPinManager.java:49)
at com.android.org.conscrypt.TrustManagerImpl.<init>(TrustManagerImpl.java:137)
at com.android.org.conscrypt.TrustManagerImpl.<init>(TrustManagerImpl.java:97)
at com.android.org.conscrypt.TrustManagerFactoryImpl.engineGetTrustManagers(TrustManagerFactoryImpl.java:80)
at javax.net.ssl.TrustManagerFactory.getTrustManagers(TrustManagerFactory.java:219)
at tech.fiissh.thrid.okhttp.internal.Util.platformTrustManager(Util.java:670)
at tech.fiissh.thrid.okhttp.OkHttpClient.<init>(OkHttpClient.java:256)
at tech.fiissh.thrid.okhttp.OkHttpClient$Builder.build(OkHttpClient.java:1035)
at tech.fiissh.base.common.net.e.b.<init>(OkHttpStack.java:73)
at tech.fiissh.base.common.net.g.<init>(NetworkDispatcher.java:51)
at tech.fiissh.base.common.net.i$1.run(RequestQueue.java:110)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
at java.lang.Thread.run(Thread.java:841)
```

经过与源码文件进行比较，`OkHttpStack` 第73行的方法实际上是通过 `OkHttpClient.Builder` 来实例化一个 `OkHttpClient` 对象。

通常情况下我们所理解的 `OOM` 是由于进程的可用堆内存不够（即 `Runtime.getRuntime().maxMemory()` 小于需要申请的内存大小）的情况下系统会抛出 `OutOfMemoryError`。其报错信息中详细的记录了我们的内存分配需求和可用堆内存的大小信息：

```java
java.lang.OutOfMemoryError: Failed to allocate a XXX byte allocation with XXX free bytes and XXXKB until OOM
```

通过对 `pthread_create` 关键字的搜索，最终在 [thread.cc](https://cs.android.com/android/platform/superproject/+/master:art/runtime/thread.cc) 中找到了抛出该异常的位置，并且通过对 `Thread::CreateNativeThread` 函数的分析发现，在创建线程时，系统会先判断当前线程数是否超过了系统对线程数的限制，如果超过该限制则抛出 `java.lang.OutOfMemoryError:pthread_create (XXXXKB stack) failed` 异常。

那么，问题的根源在哪里？通过对代码的分析，发现每次发起一个网络请求的时候都会创建一个 `OkHttpClient`，而每个 `OkHttpClient` 对象都会初始化一个线程池（线程的生命周期又较长），如果在短时间内发起多次请求，那么线程池会被创建多个，而线程数也会随之增加。解决的方案则是将 `OkHttpClient` 通过单例的方式对外提供。

针对上述线程创建的问题，可以参考 [不可思议的OOM](https://juejin.im/entry/59f7ea06f265da43143ffee4)，作者针对出现类似问题的场景做了深入分析。

关于 `HTTPClient` 的问题，可以参考 [OkHttp竟然玩出OOM？](https://www.jianshu.com/p/3b232d9f38c2)，作者针对为什么创建多个线程池的场景进行了深入的分析。
