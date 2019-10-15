---
title: Android Handler 消息机制
permalink: android-handler-message-looper
date: '2018-11-26 21:53'
updated: '2018-11-27 11:32'
comments: true
tags:
  - Android Handler
categories:
  - Android 技术文章
---

`消息机制`是 `Android` 系统下最为重要的组成部分：

* Handler 消息机制，适用于进程内线程间的消息传递
* Binder 消息机制，适用于进程间的消息传递

在 `Android` 系统的应用程序模型中，理想情况下，出于对性能的考量，`UI 线程（主线程）`只负责处理用户交互事件、执行组件的生命周期函数和执行渲染。`UI 线程`如果执行某一项任务的时间过长，则会产生 `ANR` 异常：

* Service 生命周期函数超过 20s 没有响应完毕
* 前台优先级的 `Broadcast Receiver` 超过 10s 没有响应完毕
* 后台优先级的 `Broadcast Receiver` 超过 20s 没有响应完毕
* 输入事件（`UI 线程` 中）超过 5s 没有响应完毕
* `Application` 启动（即进程启动）时超过 10s 没有响应完毕
* `Activity` 切换时超过 2s 没有响应完毕

`Handler` 消息机制的作用就是：

* 实现 `UI 线程`与子线程间的数据交互
* 实现 线程与线程间的数据交互
* 实现各组件的生命周期

`Handler` 消息机制主要由如下四个部分组成：

* Handler：负责 `Message` 的发送和处理。点击查看 [`Handler` 源代码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Handler.java)
* Message：消息的载体，用于保存消息的相关属性。点击查看 [`Message` 源代码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Message.java)
* MessageQueue：消息队列，用于存放 `Message`。点击查看 [`MessageQueue` 源代码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/MessageQueue.java)
* Looper：消息循环，负责从 `MessageQueue` 中取出需要响应的 `Message` 并交付给 `Handler` 进行处理。点击查看 [`Looper` 源代码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/os/Looper.java)

它们会见的关系如下图：

![](/images/handler组成部分之间的相互关系.png)


文中所有代码都基于 [`Android Pie 9.0.0_r3`](http://androidxref.com/9.0.0_r3/)。

<!--more-->

## Handler 消息机制在 Activity 生命周期中的作用

下图展示了 `Activity` 中各个生命周期的相互转换：

![](/images/activity-生命周期.png)

实际上，`Activity` 生命周期，都是其他线程通过 `Handler` 发送消息给 `UI 线程`，`UI 线程`对对应的生命周期方法产生调用实现的。

> 事实上，`Activity` 的生命周期控制是在 `ActivityManagerService` 中进行的。`ActivityManagerService` 是运行在 `system_server` 进程中的，而应用程序则是运行在 `Application 进程` 中。`ActivityManagerService` 通过 `Binder` 的方式跨进程调用 `Application` 进程空间中的 `ApplicationThread`，最后 `ApplicationThread` 通过 `Handler` 的方式发送消息通知 `UI 线程` 完成对应的生命周期调用。

[`ActivityThread` 类](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ActivityThread.java) 的内部类 `H` 中定义了大量的核心消息，最终通过 `H.handleMessage()` 来控制 `Activity` 的生命周期。`H` 中定义的各种类型的消息有：

```java
        public static final int BIND_APPLICATION        = 110;
        public static final int EXIT_APPLICATION        = 111;
        public static final int RECEIVER                = 113;
        public static final int CREATE_SERVICE          = 114;
        public static final int SERVICE_ARGS            = 115;
        public static final int STOP_SERVICE            = 116;

        public static final int CONFIGURATION_CHANGED   = 118;
        public static final int CLEAN_UP_CONTEXT        = 119;
        public static final int GC_WHEN_IDLE            = 120;
        public static final int BIND_SERVICE            = 121;
        public static final int UNBIND_SERVICE          = 122;
        public static final int DUMP_SERVICE            = 123;
        public static final int LOW_MEMORY              = 124;
        public static final int PROFILER_CONTROL        = 127;
        public static final int CREATE_BACKUP_AGENT     = 128;
        public static final int DESTROY_BACKUP_AGENT    = 129;
        public static final int SUICIDE                 = 130;
        public static final int REMOVE_PROVIDER         = 131;
        public static final int ENABLE_JIT              = 132;
        public static final int DISPATCH_PACKAGE_BROADCAST = 133;
        public static final int SCHEDULE_CRASH          = 134;
        public static final int DUMP_HEAP               = 135;
        public static final int DUMP_ACTIVITY           = 136;
        public static final int SLEEPING                = 137;
        public static final int SET_CORE_SETTINGS       = 138;
        public static final int UPDATE_PACKAGE_COMPATIBILITY_INFO = 139;
        public static final int DUMP_PROVIDER           = 141;
        public static final int UNSTABLE_PROVIDER_DIED  = 142;
        public static final int REQUEST_ASSIST_CONTEXT_EXTRAS = 143;
        public static final int TRANSLUCENT_CONVERSION_COMPLETE = 144;
        public static final int INSTALL_PROVIDER        = 145;
        public static final int ON_NEW_ACTIVITY_OPTIONS = 146;
        public static final int ENTER_ANIMATION_COMPLETE = 149;
        public static final int START_BINDER_TRACKING = 150;
        public static final int STOP_BINDER_TRACKING_AND_DUMP = 151;
        public static final int LOCAL_VOICE_INTERACTION_STARTED = 154;
        public static final int ATTACH_AGENT = 155;
        public static final int APPLICATION_INFO_CHANGED = 156;
        public static final int RUN_ISOLATED_ENTRY_POINT = 158;
        public static final int EXECUTE_TRANSACTION = 159;
        public static final int RELAUNCH_ACTIVITY = 160;
```

当 `Handler` 收到对应的消息后，最终在 `handleMessage(Message msg)` 方法中完成了对各生命周期的调用和响应：

```java
 public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    Looper.myLooper().quit();
                    break;
                case RECEIVER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastReceiveComp");
                    handleReceiver((ReceiverData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CREATE_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceCreate: " + String.valueOf(msg.obj)));
                    handleCreateService((CreateServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UNBIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceUnbind");
                    handleUnbindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SERVICE_ARGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceStart: " + String.valueOf(msg.obj)));
                    handleServiceArgs((ServiceArgsData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case STOP_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceStop");
                    handleStopService((IBinder)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CONFIGURATION_CHANGED:
                    handleConfigurationChanged((Configuration) msg.obj);
                    break;
                case CLEAN_UP_CONTEXT:
                    ContextCleanupInfo cci = (ContextCleanupInfo)msg.obj;
                    cci.context.performFinalCleanup(cci.who, cci.what);
                    break;
                case GC_WHEN_IDLE:
                    scheduleGcIdler();
                    break;
                case DUMP_SERVICE:
                    handleDumpService((DumpComponentInfo)msg.obj);
                    break;
                case LOW_MEMORY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "lowMemory");
                    handleLowMemory();
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case PROFILER_CONTROL:
                    handleProfilerControl(msg.arg1 != 0, (ProfilerInfo)msg.obj, msg.arg2);
                    break;
                case CREATE_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupCreateAgent");
                    handleCreateBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case DESTROY_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupDestroyAgent");
                    handleDestroyBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SUICIDE:
                    Process.killProcess(Process.myPid());
                    break;
                case REMOVE_PROVIDER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "providerRemove");
                    completeRemoveProvider((ProviderRefCount)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ENABLE_JIT:
                    ensureJitEnabled();
                    break;
                case DISPATCH_PACKAGE_BROADCAST:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastPackage");
                    handleDispatchPackageBroadcast(msg.arg1, (String[])msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SCHEDULE_CRASH:
                    throw new RemoteServiceException((String)msg.obj);
                case DUMP_HEAP:
                    handleDumpHeap((DumpHeapData) msg.obj);
                    break;
                case DUMP_ACTIVITY:
                    handleDumpActivity((DumpComponentInfo)msg.obj);
                    break;
                case DUMP_PROVIDER:
                    handleDumpProvider((DumpComponentInfo)msg.obj);
                    break;
                case SLEEPING:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "sleeping");
                    handleSleeping((IBinder)msg.obj, msg.arg1 != 0);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SET_CORE_SETTINGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "setCoreSettings");
                    handleSetCoreSettings((Bundle) msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UPDATE_PACKAGE_COMPATIBILITY_INFO:
                    handleUpdatePackageCompatibilityInfo((UpdateCompatibilityData)msg.obj);
                    break;
                case UNSTABLE_PROVIDER_DIED:
                    handleUnstableProviderDied((IBinder)msg.obj, false);
                    break;
                case REQUEST_ASSIST_CONTEXT_EXTRAS:
                    handleRequestAssistContextExtras((RequestAssistContextExtras)msg.obj);
                    break;
                case TRANSLUCENT_CONVERSION_COMPLETE:
                    handleTranslucentConversionComplete((IBinder)msg.obj, msg.arg1 == 1);
                    break;
                case INSTALL_PROVIDER:
                    handleInstallProvider((ProviderInfo) msg.obj);
                    break;
                case ON_NEW_ACTIVITY_OPTIONS:
                    Pair<IBinder, ActivityOptions> pair = (Pair<IBinder, ActivityOptions>) msg.obj;
                    onNewActivityOptions(pair.first, pair.second);
                    break;
                case ENTER_ANIMATION_COMPLETE:
                    handleEnterAnimationComplete((IBinder) msg.obj);
                    break;
                case START_BINDER_TRACKING:
                    handleStartBinderTracking();
                    break;
                case STOP_BINDER_TRACKING_AND_DUMP:
                    handleStopBinderTrackingAndDump((ParcelFileDescriptor) msg.obj);
                    break;
                case LOCAL_VOICE_INTERACTION_STARTED:
                    handleLocalVoiceInteractionStarted((IBinder) ((SomeArgs) msg.obj).arg1,
                            (IVoiceInteractor) ((SomeArgs) msg.obj).arg2);
                    break;
                case ATTACH_AGENT: {
                    Application app = getApplication();
                    handleAttachAgent((String) msg.obj, app != null ? app.mLoadedApk : null);
                    break;
                }
                case APPLICATION_INFO_CHANGED:
                    mUpdatingSystemConfig = true;
                    try {
                        handleApplicationInfoChanged((ApplicationInfo) msg.obj);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    break;
                case RUN_ISOLATED_ENTRY_POINT:
                    handleRunIsolatedEntryPoint((String) ((SomeArgs) msg.obj).arg1,
                            (String[]) ((SomeArgs) msg.obj).arg2);
                    break;
                case EXECUTE_TRANSACTION:
                    final ClientTransaction transaction = (ClientTransaction) msg.obj;
                    mTransactionExecutor.execute(transaction);
                    if (isSystem()) {
                        // Client transactions inside system process are recycled on the client side
                        // instead of ClientLifecycleManager to avoid being cleared before this
                        // message is handled.
                        transaction.recycle();
                    }
                    // TODO(lifecycler): Recycle locally scheduled transactions.
                    break;
                case RELAUNCH_ACTIVITY:
                    handleRelaunchActivityLocally((IBinder) msg.obj);
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }

```

在使用 `H` 发送各个消息之前，`ActivityThread.java` 做了哪些准备工作呢？我们知道，如果在子线程中使用 `Handler`，需要先通过 `Looper.prepare()` 将 `Looper` 进行初始化，然后在 `Handler` 初始化之后通过 `Looper.loop()` 将消息循环启动起来。那么，`UI 线程`中是如何操作的呢？我们看一下 `ActivityThread.main(String[] args)` 方法：

```java
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

是的，`UI 线程`中是通过 `Looper.prepareMainLooper()` 和 `Looper.loop()` 完成初始化和启动消息循环的。

消息从发送到最终的响应过程，便是本文要讨论的主要内容。

## Handler 消息机制实现线程间通信的原理

通过 `Handler` 消息机制实现线程间通信，大致需要如下七个步骤：

1. 消息循环：该过程首先会通过 `Looper.prepare()\Looper.prepareMainLooper()` 完成 `Looper`、`MessageQueue` 和 `ThreadLocal` 的初始化，然后通过 `Looper.loop()` 将消息循环启动起来。也就是，通过无限循环的方式，不停地从 `MessageQueue` 中取出满足条件的 `Message`（通过 `Message msg = queue.next();` 获取 `Message` 对象），然后分发给对应的 `Handler` 进行响应（通过 `msg.target.dispatchMessage(msg);`）；
2. 发送消息：该过程会通过 `Handler` 一系列的 `send` 或 `post` 方法，将 `Message` 对象或 `Runnable` 对象（最终都是 `Message` 对象）放入到 `MessageQueue` 中。当然，`Handler` 也提供了一系列方法用于将 `Message` 对象或 `Runnable` 对象从 `MessageQueue` 中移除；
3. 存储消息：`MessageQueue` 会根据 `Message` 的相关属性（比如响应时间）将 `Message` 放入链表的对应位置（通过 `MessageQueue.enqueueMessage(Message msg, long when)` 方法实现）；
4. 消息分发：`Looper` 的消息循环中会不停地遍历 `MessageQueue`，此时如果有 `Message` 进入且符合响应条件，则会通过 `MessageQueue.next()` 取出对应的 `Message`，并将该 `Message` 分发给对应的 `Handler` 进行响应（通过 `Looper.loop()` 中的 `msg.target.dispatchMessage(msg);` 实现）；
5. 消息响应：`Message` 的响应会有三种形式：

* 通过 `obtain` 方法获取 `Message` 对象时传入 `Runnable` 对象
* `Handler` 初始化时通过其构造函数传入 `Handler.Callback` 对象
* 重写 `Handler.handleMessage(Message msg)` 方法

1. 资源回收：通过 `Message.obtain` 方法获取 `Message` 对象时会比使用 `new Message()` 的方式更为高效，最本质的原因是使用了 `Message 对象池`。而 `Message 对象池`的高效则依赖于 `Message` 响应完毕之后的对象回收（通过 `Message.recycleUnchecked()`实现）；
2. 退出循环：当我们不再使用 `Handler` 消息机制的时候，需要通过 `Looper.quit()` 终止当前的消息循环。

### 消息循环

#### Looper 初始化

上文中我们有讲到 `UI 线程` 创建时，会通过 `Looper.prepareMainLooper()` 和 `Looper.loop()`完成消息循环的启动。我们先来看下 `Looper.prepareMainLooper()` 方法：

> `Looper.prepareMainLooper()` 和 `Looper.loop()` 都可以创建消息循环，其区别在于前者创建的是 `UI 线程`的消息循环，使用时并不需要显式的调用 `Looper.prepareMainLooper()`;而后者则创建的是子线程的消息循环，需要在使用时显式的调用。

```java
    public static void prepareMainLooper() {
        // 通过 prepare 方法创建一个不能退出的 Looper 对象
        prepare(false);
        synchronized (Looper.class) {
            // ActivityThread 中已经存在对 prepareMainLooper() 的调用
            // 如果代码中重复调用 prepareMainLooper()，则会在 prepare 方法中抛出 RuntimeException，此处是否有点多余呢？
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            // 将当前 Looper 对象（实际上是从 ThreadLocal 中取出来的）赋值给 SMainLooper
            sMainLooper = myLooper();
        }
    }
```

`prepareMainLooper()` 方法实际上是通过 `prepare()` 方法创建了一个不可以退出的 `Looper` 对象，并将当前 `Looper` 对象赋值给 `SMainLooper`。`prepare()` 方法的具体业务如下：

```java
    // 默认情况下，除主线程外，其他线程创建的 Looper 都是允许退出的
    private static void prepare(boolean quitAllowed) {
        // 每个线程只能调用一次该方法，否则会抛出 RuntimeException 异常
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        // 创建 Looper 对象并存放到 ThreadLocal 中
        // ThreadLocal： 线程本地存储区（Thread Local Storage，简称为TLS），每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的TLS区域。
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

上述过程中，先判断当前线程是否已经初始化过 `Looper` 对象，如果已经初始化过 `Looper`（即同一线程里重复调用 `Looper.prepare()`）则抛出 `RuntimeException`。然后创建一个 `Looper` 对象并将其存放于 `ThreadLocal` 对象 `sThreadLocal` 中。

> `ThreadLocal` 用于提供线程局部变量，在多线程环境可以保证各个线程里的变量独立于其它线程里的变量。也就是说 `ThreadLocal` 可以为每个线程创建一个`单独的变量副本`，相当于线程的 `private static` 类型变量。`ThreadLocal` 的作用和同步机制有些相反：同步机制是为了保证多线程环境下数据的一致性，而 `ThreadLocal` 是保证了多线程环境下数据的独立性。

`Looper` 的构造函数被声明为 `private`，也就是说，我们只能通过 `prepare()` 或 `prepareMainLooper()` 方法初始化 `Looper` 对象。我们看一下 `Looper` 的构造函数都有哪些业务：

```java
    private Looper(boolean quitAllowed) {
        // 初始化 MessageQueue，用于存放 Message 对象
        // 并将 MessageQueue 设置为：主线程不允许退出，子线程允许退出
        mQueue = new MessageQueue(quitAllowed);
        // 获得当前的 Thread 对象，即初始化 Looper 的 Thread 对象
        mThread = Thread.currentThread();
    }
```

`Looper` 的构造函数中对 `MessageQueue` 进行了初始化，并将 `Looper` 初始化所在的线程赋值给 `mThread` 对象。

> 每一个 `Looper` 对象都有一个与之对应的 `MessageQueue` 对象。

`Looper` 提供了 `getMainLooper()` 和  `myLooper()`  用于获取对应的 `Looper` 对象。由于 `UI 线程`的特殊性，我们可以方便的通过 `getMainLooper()` 获取到 `UI线程`的 `Looper` 对象，从而向 `UI 线程`的消息队列发送消息。

#### MessageQueue 初始化

`MessageQueue ` 会根据 `quitAllowed` 参数完成其初始化。`MessageQueue` 真正初始化的逻辑是通过 `native static long nativeInit()` 函数完成的：

```java
    MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }
```

##### android_os_MessageQueue_nativeInit 函数

`nativeInit()` 函数是 `JNI` 层方法，是通过 `android_os_MessageQueue_nativeInit`([android_os_MessageQueue.cpp](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)) 方法实现的 ：

```c++
static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) {
    NativeMessageQueue* nativeMessageQueue = new NativeMessageQueue();
    if (!nativeMessageQueue) {
        jniThrowRuntimeException(env, "Unable to allocate native queue");
        return 0;
    }

    nativeMessageQueue->incStrong(env);
    return reinterpret_cast<jlong>(nativeMessageQueue);
}
```

`android_os_MessageQueue_nativeInit` 首先会创建一个 `NativeMessageQueue` 对象，如果创建成功，则将对象 `nativeMessageQueue` 的指针地址返回给 `MessageQueue`，最终与 `mPtr` 进行绑定。

> `nativeMessageQueue->incStrong(env);` 是将 `NativeMessageQueue` 的引用计数增加一次。`

##### NativeMessageQueue 初始化

`NativeMessageQueue` 在初始化时，先通过 `Looper::getForThread();` 判断当前线程是否已经创建过 `Native` 层的  `Looper` 对象，如果没有，则创建一个新的 `Looper`（`Native` 层）对象：

```c++
NativeMessageQueue::NativeMessageQueue() : mPollEnv(NULL), mPollObj(NULL), mExceptionObj(NULL) {
    mLooper = Looper::getForThread();
    if (mLooper == NULL) {
        mLooper = new Looper(false);
        Looper::setForThread(mLooper);
    }
}
```

`Looper::setForThread(mLooper);` 用于获取当前线程的 `Looper` 对象，其功能与 `Java` 层的 `Looper.myLooper()` 类似。而 `Looper::setForThread(mLooper);` 则是将 `Native` 层的 `Looper` 对象保存起来，其功能与 `Java` 层的 `ThreadLocal` 类似。

##### Native 层的 Looper 初始化

`Native` 层的 `Looper`([`Android9.0.0_r3/system/core/libutils/Looper.cpp`](http://androidxref.com/9.0.0_r3/xref/system/core/libutils/Looper.cpp)) 的初始化是在 `NativeMessageQueue` 初始化时完成的。我们来看下其构造函数：

```c++
Looper::Looper(bool allowNonCallbacks) : mAllowNonCallbacks(allowNonCallbacks), mSendingMessage(false), mPolling(false), mEpollRebuildRequired(false), mNextRequestSeq(0), mResponseIndex(0), mNextMessageUptime(LLONG_MAX) {
    mWakeEventFd.reset(eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC));
    LOG_ALWAYS_FATAL_IF(mWakeEventFd.get() < 0, "Could not make wake event fd: %s", strerror(errno));

    AutoMutex _l(mLock);
    rebuildEpollLocked();
}
```

`mWakeEventFd` 是定义在 [`Android9.0.0_r3/system/core/libutils/include/utils/Looper.h`](http://androidxref.com/9.0.0_r3/xref/system/core/libutils/include/utils/Looper.h) 的 `int` 型对象，其作用是在 `epoll` 的监听中，通过向 `mWakeEventFd` 写入数据来控制唤醒和阻塞的状态。`rebuildEpollLocked();` 则是用于重建 `epoll` 事件：

```c++
void Looper::rebuildEpollLocked() {
    // Close old epoll instance if we have one.
    if (mEpollFd >= 0) {
        #if DEBUG_CALLBACKS
        ALOGD("%p ~ rebuildEpollLocked - rebuilding epoll set", this);
        #endif
        close(mEpollFd);
    }

    // Allocate the new epoll instance and register the wake pipe.
    mEpollFd = epoll_create(EPOLL_SIZE_HINT);
    LOG_ALWAYS_FATAL_IF(mEpollFd < 0, "Could not create epoll instance: %s", strerror(errno));

    struct epoll_event eventItem;
    memset(& eventItem, 0, sizeof(epoll_event)); // zero out unused members of data field union
    eventItem.events = EPOLLIN;
    eventItem.data.fd = mWakeEventFd;
    int result = epoll_ctl(mEpollFd, EPOLL_CTL_ADD, mWakeEventFd, & eventItem);
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not add wake event fd to epoll instance: %s", strerror(errno));

    for (size_t i = 0; i < mRequests.size(); i++) {
        const Request& request = mRequests.valueAt(i);
        struct epoll_event eventItem;
        request.initEventItem(&eventItem);

        int epollResult = epoll_ctl(mEpollFd, EPOLL_CTL_ADD, request.fd, & eventItem);
        if (epollResult < 0) {
            ALOGE("Error adding epoll events for fd %d while rebuilding epoll set: %s", request.fd, strerror(errno));
        }
    }
}

```

`rebuildEpollLocked()` 会先会通过 `close(mEpollFd);` 关闭已有的 `epoll` 实例，然后通过 `epoll_create(EPOLL_SIZE_HINT);` 创建一个新的 `epoll` 实例，并将文件描述符保存在 `mEpollFd` 中。紧接着，会将前面创建的管道的读一端文件描述符添加到 `epoll` 实例中。以便可以对它所描述的管道的写一端的操作进行监听。当一个线程没有新的消息需要处理时，它就会阻塞在这个管道的读一端的文件描述符上，直到有洗的消息需要处理为止。而当其他线程向这个线程的的消息队列发送了一个消息之后，其他线程就会通过这个管道的写一端的文件描述符向管道写入一个数据，从而将这个线程唤醒。

> `Linux` 内核的 `epoll` 机制是为了能够同时监听多个文件描述符的 `IO` 事件而设计的，是一个轻量但是强大的多路复用 `IO` 接口，类似于 `Linux` 内核的 `select` 机制增强版。如果一个 `epoll` 实例监听了大量的文件描述符的 `IO` 事件，但是只有少量的文件描述符是活跃的，也就是说，只有少量的文件描述符会产生 `IO` 事件，那么这个 `epoll` 实例可以显著地减少 `CPU` 的使用率，从而提高系统的并发处理能力。

#### 开启消息循环

`Looper` 初始化完成之后，需要通过 `Looper.loop()` 方法开启消息循环：

```java
    public static void loop() {
        // 获取当前线程的 Looper 对象
        final Looper me = myLooper();
        // 如果 Looper 为 null，则说明当前线程还没有调用 prepare()
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        // 获取当前 Looper 对应的 MessageQueue
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        // 清空远程调用端的 uid 和 pid，用当前本地进程的 uid 和 pid 替代
        Binder.clearCallingIdentity();
        // 确保在权限检查时时基于本地的进程，而不是基于做出调用时的进程。
        final long ident = Binder.clearCallingIdentity();

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride = SystemProperties.getInt("log.looper." + Process.myUid() + "."
                + Thread.currentThread().getName()
                + ".slow", 0);

        // Log 相关
        boolean slowDeliveryDetected = false;

        // loop 真正开启无限循环的地方
        for (; ; ) {
            // 从 MessageQueue 中取出一个 Message。需要注意的是，该过程可能会阻塞
            Message msg = queue.next();
            // 没有符合条件的 Message，则退出循环
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            // 默认情况下，Looper 的 mLogging 为 null，需要通过 setMessageLogging(Printer printer) 指定 Printer，一般用于 debug
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " + msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            try {
                // 将 Message 分发到对应的 Handler 进行响应
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery", msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            // 确保分发过程中identity不会损坏
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            // 将完成响应的 Message 对象回收放入 Message 对象池
            msg.recycleUnchecked();
        }
    }
```



上述过程会通过无限循环的方式，从 `MessageQueue`  中取出满足响应条件的 `Message` 对象，然后分发给对应的 `Handler` 对象进行响应：

* 从 `MessageQueue` 中读取一个 `Message`
* 把 `Message` 分发给 `target`（实际上是 `Handler` 对象）
* 被分发的 `Message` 响应完成之后，将 `Message` 对象放入对象池中进行回收

上述过程就是 `Looper` 的初始化以及消息循环的启动过程。

如果此时 `MessageQueue` 中没有 `Message`，那么 `queue.next()` 将会进入阻塞状态，等待 `Message` 的到来。

> 为什么 `Handler` 的初始化必须在 `Looper.prepare()` 和 `Looper.loop()` 之间呢？如果  `Handler` 在 `Looper.prepare()` 之前或 `Looper.loop()` 之后初始化会出现怎样的情况呢？
>
> `Handler` 在其构造函数中会通过 `mLooper = Looper.myLooper();` 获取当前 `Looper` 对象，如果没有调用 `Looper.prepare()`，那么，`ThreadLocal` 中存储的 `Looper` 对象则是空的，`Handler` 会在其初始化时抛出 `RuntimeException`。
>
> `Looper.loop()` 之后为什么不能初始化 `Handler` 呢？从上述 `Looper` 源码中我们知道，`MessageQueue` 为空时，其 `next()` 方法会阻塞，直到有新的消息进来。这也就意味着，`Looper.loop()` 会因为 `MessageQueue` 为空而阻塞在 `Message msg = queue.next()`，所以，在它后面的业务根本得不到执行。也就是说，`Handler` 根本没有得到初始化。

### 发送消息

上文中已经将 `Looper` 的消息循环启动起来，但是由于此时 `MessageQueue` 中还没有需要分发的 `Message`，所以会阻塞在 `Looper.loop()` 。接下来，我们具体分析 `Handler` 是如何将 `Message` 放入 `MessageQueue` 的。

`Handler` 提供如下方法用于将 `Message` 或 `Runnable` 放入 `MessageQueue` 中：

```java
  sendEmptyMessage(int what)
  sendEmptyMessageDelayed(int what, long delayMillis)
  sendEmptyMessageAtTime(int what, long uptimeMillis)
  sendMessageDelayed(Message msg, long delayMillis)
  sendMessageAtTime(Message msg, long uptimeMillis)
  sendMessageAtFrontOfQueue(Message msg)
  post(Runnable r)
  postAtTime(Runnable r, long uptimeMillis)
  postAtTime(Runnable r, Object token, long uptimeMillis)
  postDelayed(Runnable r, long delayMillis)
  postDelayed(Runnable r, Object token, long delayMillis)
  postAtFrontOfQueue(Runnable r)
  ```

`MessageQueue` 中能够存放的是 `Message` 类型的对象。那么，`Runnable` 类型的对象是如何放入 `MessageQueue` 的呢？这是因为，`Handler` 中通过 `getPostMessage(Runnable r)` 方法将 `Runnable` 对象封装为了 `Message` 对象：

```java
    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
```

> 实际上，是将 `Runnable` 对象赋值给了 `Message.callback`。这种方式等同于通过 `Message.obtain(Handler h, Runnable callback)` 获取对象时传入一个 `Runnable` 对象。

最终，通过 `Handler.enqueueMessage` 方法，最终将 `Message` 对象放入到 `MessageQueue` 中：

```java
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        // 将 Message 的target 设置为 Handler 对象自身
        msg.target = this;
        // 如果是 异步 Handler，则设置 Message 的 setAsynchronous(true)
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        // 将 Message 以及其响应时间放入 MessageQueue
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

`MessageQueue` 会根据我们传入的 `uptimeMillis` 将 `Message` 放入对应的位置。那么，`uptimeMillis` 是如何计算的呢？`uptimeMillis` 跟 `delayMillis` 有什么联系呢？

在 `Handler` 中，`delayMillis` 指的是 `Message` 的延时执行时间，是一个相对的时间，`uptimeMillis` 则指的是 `Message` 的具体执行时间，是一个绝对时间。`uptimeMillis` 和 `delayMillis` 的换算关系如下：

```java
SystemClock.uptimeMillis() + delayMillis
```

#### 删除消息

`Handler` 发送 `Message` 之后，如果想要从 `MessageQueue` 中将 `Message` 移除，可以通过如下方法：

```java
removeCallbacks(Runnable r)
removeCallbacks(Runnable r, Object token)
removeCallbacksAndMessages(Object token)
removeMessages(int what)
removeMessages(int what, Object object)
```

这一系列移除 `Message` 的方法最终也是通过 `MessageQueue` 来实现的。

#### 异步 Handler

`Handler` 提供了一系列方法用于创建一个`异步 Handler`：

```java
Handler createAsync(@NonNull Looper looper)
Handler createAsync(@NonNull Looper looper, @NonNull Callback callback)
```

上述方式实际上最终是通过调用 `Handler` 的构造函数实现的：

```java
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

字段 `mAsynchronous` 用来标志一个 `Handler` 是否是异步 `Handler`。最终，`Handler.enqueueMessage` 方法会根据 `mAsynchronous` 的值将一个 `Message` 标志为`异步 Message`。

### 存储消息

通过 `Handler` 发送 `Message` 以后，会计算该 `Message` 的具体执行时间，然后将其放入 `MessageQueue` 的对应位置等待分发：

```java
    boolean enqueueMessage(Message msg, long when) {
        // Message 需要与 Handler 绑定，不然没法分发
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        // 一个 Message 如果已经在使用中，那么就不能再次被使用了
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            // 如果 MessageQueue 正在退出的话，就不要再往里面放了
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                // 把 Message 回收一下
                msg.recycle();
                return false;
            }

            // 标记一下 Message 正在使用中
            msg.markInUse();
            // Message 的执行时间
            msg.when = when;
            // mMessages 永远指向队列中第一个 Message
            Message p = mMessages;
            // native 层是否要 wake
            boolean needWake;

            if (p == null || when == 0 || when < p.when) {
                // p 为空，说明 MessageQueue 中还没有消息
                // when == 0 需要立即执行
                // when < p.when 当前 Message 需要执行的时间小于目前队列中第一个 Message 的执行时间
                // New head, wake up the event queue if blocked.
                // 进来的第一个 Message或者是需要立即执行的 Message
                msg.next = p;
                // 将 传进来的 Message 放到队列的第一个位置
                mMessages = msg;
                // 如果阻塞的话，需要唤醒
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                // 将 Message 按照时间顺序插入到 MessageQueue 中
                // 一般情况下不需要唤醒队列，除非消息队列的头部存在 Barrier（p.target == null），并且 Message 是异步消息
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                // 队列中的上一个 Message
                Message prev;
                for (; ; ) {
                    // 将队列的第一个Message赋值给 preev
                    prev = p;
                    // 将队列当前的Message 的 next 赋值给 p
                    p = p.next;
                    // 如果p为空或者放入的 Message 执行时间小于 当前队列第一个Message执行时间
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                // 将 p 指向 传入 msg 的 next
                msg.next = p; // invariant: p == prev.next
                // 当前队列中第一个Message的 next 指向 msg
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

`MessageQueue` 是按照 `Message` 响应时间的先后顺序就行排列的。确切的说，`Message` 是以链表的方法进行存储的。当有新的 `Message` 进入到 `MessageQueue`是，会从链表第一个元素开始遍历，比较链表中的每一个 `Message` 与新进入的 `Message` 的响应时间，直到将新的 `Message` 放入合适的位置。

#### 删除消息

从上文中我们知道，`Handler` 移除 `Message` 的具体逻辑，实际上是 `MessageQueue` 从队列中将 `Message` 移除。`MessageQueue` 提供了如下方法用于删除 `Message`：

```java
removeMessages(Handler h, int what, Object object)
removeMessages(Handler h, Runnable r, Object object)
removeCallbacksAndMessages(Handler h, Object object)
```

`MessageQueue` 删除 `Message` 的逻辑，实际上就是根据具体条件将符合条件的 `Message` 对象从链表中删除的过程。

### 消息分发

经过上面一系列过程之后，`MessageQueue` 中终于有了 `Message`。`Looper` 的消息循环将会结束阻塞状态，将 `Message` 对象进行分发。

#### 取出消息

`Looper` 的无限循环从 `MessageQueue` 中取出消息的具体业务（`queue.next()`），是在 `MessageQueue.next()` 方法中完成的：

```java
    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        // 当前的消息循环已经退出了
        if (ptr == 0) {
            return null;
        }

        // 循环迭代的首次为 -1
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (; ; ) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            // 进入阻塞
            // 将会阻塞 nextPollTimeoutMillis 时间
            // 如果消息队列被唤醒（nativeWake(mPtr);），也将结束阻塞
            //            1.如果nextPollTimeoutMillis=-1，一直阻塞不会超时。
            //            2.如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
            //            3.如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)，如果期间有程序唤醒会立即返回。
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    //当消息的 Handler 为 null时,找出下一个异步 Message
                    do {
                        //
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }

                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        // 当异步消息的响应时间大于当前时间，设置下一次轮训的超时时间
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 获取一个 Message
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        // 将消息标志为使用状态
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    // 没有消息了，进入阻塞
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                // MessageQueue 正在退出
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                // 第一次进入
                // 消息队列为空或者是当前时间小于消息响应时间
                if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }

                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    // 没有需要运行的 IdleHandler
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            // 重置idle handler 个数为0，以保证不会再次重复运行
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            //当调用一个空闲handler时，一个新message能够被分发，因此无需等待可以直接查询pending message.
            nextPollTimeoutMillis = 0;
        }
    }
```

`pendingIdleHandlerCount` 用来保存注册在 `MessageQueue` 中的 `IdleHandler` 的数量，当线程处于空闲状态时，不会马上就如阻塞状态，而是先调用注册的 `IdleHandler` 的 `queueIdle` 方法，用于处理一些非紧急的任务。
`mMessages` 永远指向 `MessageQueue` 中的第一个 `Message`，`Message` 的 `next` 字段则指向 `Message` 单向链表中的下一个元素。

`nativePollOnce(ptr, nextPollTimeoutMillis);` 是 `Native` 层的阻塞函数，`nextPollTimeoutMillis` 用于描述当前消息队列中没有新的消息需要处理时，当前线程的阻塞时间：

* 如果 `nextPollTimeoutMillis = -1`，消息队列中没有消息需要处理时，当前线程进入阻塞状态，直到被其他线程唤醒
* 如果 `nextPollTimeoutMillis` = 0，消息队列中没有消息需要处理时，当前线程不要进入阻塞状态
* 如果 `nextPollTimeoutMillis > 0` ，消息队列中没有消息需要处理时，最长阻塞 `nextPollTimeoutMillis` 毫秒，如果中途有程序唤醒则会立即返回

如果 `msg.target == null`，那么找出第一个异步 `Message`。然后根据 `Message` 的响应时间判断是否有 `Message` 需要分发。如果没有的话，设置 `nextPollTimeoutMillis` 阻塞时间，进入下次循环时会根据 `nextPollTimeoutMillis` 时间进行阻塞。如果有消息需要分发，则将需要分发的消息返回，并设置 `mBlocked = false` 表示目前没有阻塞。

> 异步 `Message` 是一种特殊的消息类型，相较于其他 `Message`，其特点是 `target` 字段为 `null`。它的作用是向 `MessageQueue` 中插入一个消息屏障，在消息屏障之后的所有同步消息都不会执行。我们可以通过使用 `void removeSyncBarrier(int token)` 将屏障移除（`token` 是 `post` 方法的返回值）。

总的来说，`MessageQueue` 获取消息的步骤可以分为如下几步：

* 首次进入消息循环是，`nextPollTimeoutMillis= 0`，`nativePollOnce` 方法会立即返回，不会阻塞；
* 读取链表中的 `Message`，如果发现消息屏障，则跳过后面的同步消息。也就是说，会根据当前时间、消息响应时间和是否存在屏障来返回符合条件的消息；
* 如果没有符合条件的消息，则会执行 `IdleHandler`。

> 可以理解为  `IdleHandler` 是一种较为特殊的 `Message`，其主要作用是在消息循环空闲时执行一些非紧急的任务。

`MessageQueue` 中定义的 `MMessage` 用来表示当前线程需要出来的消息。当前线程从 `nativePollOnce` 唤醒之后，如果有新的消息需要处理，如果 `MessageQueue` 的 `mMessages` 不为 `null` 则进行处理，否则按照 `Message` 的响应时间进行判断，如果消息的响应时间小于等于系统当前时间，说明当前线程需要马上处理该 `Message`，则将 `mMessages` 指向下一个 `Message` 并返回需要处理的 `Message` 对象。否则，将会重新计算当前线程下一次调用前 `nativePollOnce` 的阻塞时间。

`nativePollOnce` 是定义在 `MessageQueue` 在 `JNI` 层的定义，其最终调用了 `NativeMessageQueue` 的 `pollOnce` 函数：

```c++
void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);
    mPollObj = NULL;
    mPollEnv = NULL;

    if (mExceptionObj) {
        env->Throw(mExceptionObj);
        env->DeleteLocalRef(mExceptionObj);
        mExceptionObj = NULL;
    }
}
```

该函数的主要作用是通过 `mLooper->pollOnce(timeoutMillis);` 检查当前线程是否有新的消息需要处理。

#### 分发消息

消息循环在取到需要分发的 `Message` 之后，会通过 `msg.target.dispatchMessage(msg);` 将 `Message` 分发到对应的 `Handler` 进行响应：

```java
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

首先会判断 `Message` 的 `callback` 是否为 `null`，如果不为 `null`，则通过 `handleCallback(msg)` 进行响应：

```java
    private static void handleCallback(Message message) {
        message.callback.run();
    }
```

而如果 `callback` 为 `null`，则判断 `Handler` 的 `mCallback` 是否为 `null`，如果不为 `null`，则通过 `Callback.handleMessage(Message msg)` 进行响应，否则通过 `Handler.handleMessage(Message msg)` 进行响应。

### 消息响应

通过上文 `消息分发`我们知道，消息的响应有三种不同的形式。首先，优先级最高的是 `Message` 自身的 `Runnable`，其次则是 `Callback` 接口，最后才是 Handler.handleMessage(Message msg)` 。

我们可以根据需求，使用不同的方式响应 `Message`。

### 资源回收

通过 `Message.obtain` 方法可以高效的获取 `Message` 对象。其实现原理是将响应完毕的 `Message` 放入对象池中，使用时从对象池中高效的获取 `Message` 对象：

```Java
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            sPoolSize--;
            return m;
        }
    }
    return new Message();
}
```


通过将响应完毕的 `Message` 放入对象池进行缓存，以达到高效的目的。最终，`Message` 是通过 `recycleUnchecked()` 方法放入对象池的：

```Java
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```



### 退出循环

`Looper` 在不再使用的情况下，需要退出消息循序以节省资源。`Looper` 退出消息循环的 `Looper.quitSafely()` 和 `Looper.quit()` 最终是通过 `MessageQueue.quit(boolean safe)` 实现的。

`safe` 参数决定了 `MessageQueue` 在退出时对 `Message` 的处理策略：

* 如果 `safe` 为 `true`，则会等待所有的 `Message` 响应完成之后再退出：

```java
    private void removeAllFutureMessagesLocked() {
        final long now = SystemClock.uptimeMillis();
        Message p = mMessages;
        if (p != null) {
            if (p.when > now) {
                removeAllMessagesLocked();
            } else {
                Message n;
                for (; ; ) {
                    n = p.next;
                    if (n == null) {
                        return;
                    }
                    if (n.when > now) {
                        break;
                    }
                    p = n;
                }
                p.next = null;
                do {
                    p = n;
                    n = p.next;
                    p.recycleUnchecked();
                } while (n != null);
            }
        }
    }
```

* 如果 `safe` 为 `false`，则会立即退出：

```java
    private void removeAllMessagesLocked() {
        Message p = mMessages;
        while (p != null) {
            Message n = p.next;
            p.recycleUnchecked();
            p = n;
        }
        mMessages = null;
    }

```

## IdleHandler

`IdleHandler` 定义在 `MessageQueue` 中的内部接口：

```java
    public static interface IdleHandler {
        /**
         * Called when the message queue has run out of messages and will now
         * wait for more.  Return true to keep your idle handler active, false
         * to have it removed.  This may be called if there are still messages
         * pending in the queue, but they are all scheduled to be dispatched
         * after the current time.
         */
        boolean queueIdle();
    }
```

我们可以通过 `MessageQueue.addIdleHandler(@NonNull IdleHandler handler)` 或 `MessageQueue.removeIdleHandler(@NonNull IdleHandler handler)` 向 `MessageQueue` 添加或移除一个 `IdleHandler`。

`IdleHandler` 会在消息循环空闲时或阻塞时得到执行，用于处理一些非紧急的任务。`Android` 系统中大量使用了 `IdleHandler` 执行一些非紧急任务，例如 [`ActivityThread`](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ActivityThread.java) 中使用 `IdleHandler` 在空闲时执行 `GC` 操作：

```java
    final class GcIdler implements MessageQueue.IdleHandler {
       @Override
       public final boolean queueIdle() {
           doGcIfNeeded();
           return false;
       }
   }
```

通过 `Looper.myQueue().addIdleHandler(mGcIdler);` 将 `IdleHandler` 添加到队列中。

## 消息屏障

`Handler` 中的 `Message` 可以分为两种：

* 同步 `Message`
* 异步 `Message`

可以通过 `Message` 的 `isAsynchronous()` 方法查询该 `Message` 是否是异步 `Message`：

```java
    public boolean isAsynchronous() {
        return (flags & FLAG_ASYNCHRONOUS) != 0;
    }
```

通常情况下，上述两种类型的 `Message` 的处理方式并没有差异，只有通过 `MessageQueue.postSyncBarrier(long when)` 向队列中插入一个消息屏障之后，才会出现差异：

```java
    private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        synchronized (this) {
            final int token = mNextBarrierToken++;
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;
        }
    }

```

上述方法实际上是创建了一个 `target` 为 `null` 的 `Message` 然后放入到了消息队列中。

消息屏障在 `Looper` 的消息循环中对过滤消息起着重要的作用：当设置了消息屏障之后，`MessageQueue.next()` 方法将会忽略所有的同步 `Message`，只返回异步 `Message`。也就是说，设置了消息屏障之后，`Handler` 只能处理异步 `Message` 而忽略同步 `Message`，直到使用 `MessageQueue.removeSyncBarrier(int token)` 将屏障从消息队列中移除。

可以通过如下方法创建异步 `Message`：

```java
Handler createAsync(@NonNull Looper looper, @NonNull Callback callback)
Handler createAsync(@NonNull Looper looper)
```

> 对于早期 `Android` 版本的 `Handler`，可能不存在该方法。

在 `Android` 应用程序框架中，为了能够更快的响应 `UI 渲染`事件，[`ViewRootImpl`](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/view/ViewRootImpl.java) 中就是用了消息屏障：

```java
   void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
```

## Handler 造成的内存泄漏

我们经常会因为某些便利性的原因，使用了如下代码对 `Handler` 进行初始化：

```java
    private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };
```

使用上述方式初始化的 `Handler`，由于 `Message` 会持有 `Handler` 对象，而 `Handler` 对象如果是非 `static` 的话，`Handler` 对象又会持有外部类的引用（比如 `Activity`）。如果我们通过 `Handler` 发送一个延时执行的 `Message`，那么该 `Message` 对象会长期存在于消息队列中，造成 `Handler` 长期持有外部对象，进而造成内存泄漏的问题。

对应的解决方案则是使用静态内部类初始化 `Handler`，并在 `Handler` 内部使用 `WeakReferenc` 持有对象：

```java
    private static final class DefaultHandler extends Handler {

        private final WeakReference<Activity> activityWeakReference;

        public DefaultHandler(Activity activity) {
            activityWeakReference = new WeakReference<Activity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            Activity activity = activityWeakReference.get();

            if (activity != null) {
                // do somethings
            }
        }
    }
```

上述代码中，先是通过 `static` 的方式确保 `Handler` 对象不会持有外部类对象，然后通过 `WeakReference` 的方式保证 `Handler` 持有的对象能够及时有效的被垃圾回收。

## Looper 与 ANR

通过 `ActivityThread` 的代码我们知道，应用程序启动时，主要工作就是创建 `UI 线程`的消息循环。`Android` 应用程序是基于事件驱动的，`Looper.loop()` 会不断地接收、处理事件。当 `UI 线程` 没有任何消息需要处理的时候，`Looper.loop()` 将会处于阻塞状态，直到有新的消息进入并唤醒 `UI 线程`。

造成 `ANR` 的主要原因，则是 `Activity` 各个生命周期等待时间过长（也就是说，每个任务的执行时间超过了阈值）。而 `Looper.loop()` 本身则不会导致应用程序出现 `ANR`。

## 总结

`Handler` 消息机制与 `Binder` 消息机制是 `Android` 系统中最为重要的消息传递机制。前者解决的是同一进程内跨线程的消息传递，而后者则解决的是同一系统内跨进程的消息传递。

`Handler` 在 `Android` 中的使用非常广泛。`Activity` 生命周期的控制、`View` 的渲染等方面都使用了 `Handler`。而 `HandlerThread` 和 `AyncTask` 等工具类则是基于 `Handler` 进行的封装。

## 版本

* 2018年11月26日，初稿撰写完毕。
* 2018年11月27日，增加 `Handler` 机制的脑图。`XMind` 格式脑图点击下载：[Handler 消息机制脑图](/download/Handler-消息机制脑图-2018-11-26.xmind)。

![](/images/Handler-消息机制脑图-2018-11-26.png)
