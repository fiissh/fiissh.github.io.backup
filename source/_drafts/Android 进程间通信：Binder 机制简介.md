---
title: Android 进程间通信：Binder 机制简介
permalink: android-inter-processer-communication-binder
comments: true
date: 2019-03-20 15:07:30
updated: 2019-03-20 15:07:30
tags:
  - 进程间通信
categories:
  - Binder 机制
---

`Binder` 是 `Android` 平台常用的进程间通信的方案。`AIDL`、`Messenger` 和 `ContentProvider` 这三种常用的进程间通信的方案，本质上都是 `Binder` 的不同表现形式而已。`Android` 平台使用 `Binder` 机制作为进程间通信的实现，主要基于如下几个原因：

* 从性能的角度来说，`Binder` 只进行了一次数据拷贝，而管道、消息队列和 `UDS` 等方案都需要两次数据拷贝。相较于共享内存不需要数据拷贝的情况，`Binder` 机制在性能方面仅次于共享内存的方式
* 共享内存的方式很难解决数据并发访问的同步性问题，而 `Binder` 机制基于 `Client-Server` 架构的设计很好的解决了数据同步的问题

`Binder` 采用了 `Client-Server` 的架构设计。从 `Binder` 组件的角度来说，其包含了 `Client` 端、`Server` 端、`ServiceManager` 以及 `Binder` 驱动：

![](http://gityuan.com/images/binder/prepare/IPC-Binder.jpg)

上述过程中，无论是注册服务还是获取服务，都是通过 `ServiceManager`（`Native` 层的 `ServiceManager` 而非 `Framework` 层） 进行的。`ServiceManager` 是整个 `Binder` 机制的总指挥官，是 `Android` 进程间通信机制 `Binder` 的守护进程。

当 `ServiceManager` 启动之后，`Client` 端和 `Server` 端之间的通信都需要先获取到 `ServiceManager` 对象，然后才能进行通信。但事实上，`Client` 端和 `Server` 端获取 `ServiceManager` 时都是通过 `Binder` 驱动完成的。与 `Client` 端、`Server` 端以及 `ServiceManager` 位于用户空间所不同的是，`Binder` 驱动是位于内核空间的。
<!--more-->

## 进程间通信的经典实现

### 共享内存

共享内存是一种常用的进程间通信的方式。由于两个进程可以直接共享访问同一块内存区域，减少了数据的复制操作，在一定程度上提升了效率。通常情况下，使用共享内存的方式实现进程间通信需要如下步骤：

* 创建内存共享区

在需要共享内存的进程上通过使用 `shmget`（`Linux` 内核） 函数在内核空间创建一块共享的内存区域，该共享内存区域将与某个特定的 `key`（`shmget` 函数的 `key` 参数）进行绑定。

```c++
int shmget(key_t key,size_t size, int shmflg);
```

其中，`key` 为 `IPC_PRIVATE`（或者不为 `IPC_PRIVATE` 但是 `shmflg` 参数为 `IPC_CREAT`） 时，调用该方法将会创建新的共享内存区域。`size` 为申请的共享内存的大小（单位为字节）。`shmflg` 标志位可选的参数如下：

* `IPC_CREAT`：申请新建内存区域
* `IPC_EXCL`：和 `IPC_CREAT` 共同使用，如果指定的区域已经存在，则返回错误码
* `SHM_HUGETLB`：使用 `huge page` 机制申请内存
* `SHM_NORESERVER`：此区域不保留 `swap` 空间

该函数的返回值为共享内存区域的 `ID`。

* 映射内存共享区

成功创建了内存共享区域之后，在原来的进程上使用 `shmat`（`Linux` 内核） 函数将上述过程申请的位于内核空间的共享内存区域映射到进程所在的用户空间。

```c++
char *shmat(int shmid, void *shmaddr, int shmflg);
```

其中，`shmid` 是 `shmget` 执行成功之后返回的共享内存区域的 `ID`，`shmaddr` 表示将共享内存映射到指定的地址，如果为0，系统将自动分配地址，`shmflg` 标志位同 `shmget` 函数。

该函数的返回值为该内存区域的起始地址。

* 访问共享内存

在某一条进程中已经成功的创建了一个共享内存区域。另外的进程通过使用 `shmget` 并传入第一步中的 `key`，然后使用 `shmat` 将共享内存映射到该进程所在的用户空间。

* 进程间通信

各个进程都实现了内存映射之后，就可以使用该共享内存区域完成进程间的数据交互了。由于共享内存的方式本身没有同步机制，所以需要参与通信的各个进程自行协商数据同步的问题。

* 撤销内存映射

完成进程间的数据交互之后，各个进程都需要使用 `shmdt`（`Linux` 内核） 函数来解除与共享内存的映射关系。

```c++
int shmdt(char *shmaddr);
```

其中，`shmaddr` 参数介绍同 `shmat`。该函数的返回值使用0表示成功，其他数字表示失败。

* 删除共享内存区域

在解除映射关系之后，需要使用 `shmctl`（`Linux` 内核） 函数删除共享的内存区域。

```c++
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

其中，`shmid` 同 `shmget` 返回值，`cmd` 是控制命令，可选的值如下：

* `IPC_STAT`：状态查询，结果存入 `buf` 中
* `IPC_SET`：在权限允许的情况下，将共享内存中的数据更新为 `buf` 中的数据
* `IPC_RMID`：删除共享内存区域

该函数返回值使用0表示成功，其他数字表示失败。

### 管道

管道是基于文件描述符的半双工的进程间通信的方式。通常情况下，我们所说的管道是指匿名管道。通过使用 `pipe` 函数（`Linux` 内核）打开一个管道：

```c++
int pipe(int pipefd[2], int flags);
```

其中，`pipefd[2]` 表示管道打开之后的通信双方（即管道的两端）。

> 管道的文件描述符是一种只存在于内存中的特殊的文件系统。

需要注意的是，数据在管道中只能单向传输。这也就意味着数据只能从管道的一端写入，从另一端读出。一个进程如果既需要写入数据，也需要读取数据，那么就需要建立两根管道用于通信。

写入管道的数据没有特定的格式，需要管道的读取方和写入方预先约定数据格式。写入方按照约定的格式写入数据，读取方按照数据的写入顺序读取数据。这也就是说，管道中的数据遵循先入先出的原则进行传输。

默认情况下，如果管道中没有数据，那么读取方读取数据时将会阻塞，直到写入方写入数据。

由于管道没有名字，所以其只适用于具有亲缘关系的进程间的数据交互（即父子进程或者兄弟进程之间）。对于那些没有任何关系的进程间的数据交互，需要使用有名管道的方式。

有名管道将一个文件路径与之关系，以 `FIFO` 的文件形式存储于文件系统中，这使得它可以在没有任何关系的进程中使用。与匿名管道随着进程的结束而销毁的特点不同的是，有名管道需要在不再使用该管道时必须手动销毁。

### 消息队列

消息队列是一种进程间发送二进制数据块的通信机制，每个数据块都有特定的类型，接收者进程可以独立的接收不同类型的数据块。我们可以通过发送消息来避免管道和共享文件的方式所带来的同步和阻塞的问题。

使用消息队列的方式实现进程间数据交互需要如下步骤：

* 创建消息队列

使用 `msgget` 函数可以创建一个消息队列：

```c++
int msgget(key_t key, int msgflg);
```

其中，`key` 用来标识一个全局唯一的消息队列，`msgflg` 可选的值如下：

* `IPC_CREAT`：如果消息队列不存在，则创建一个消息队列，如果存在，则打开该消息队列
* `IPC_EXCL`：只有在共享内存不存在的时候，新的共享内存才建立，否则就产生错误

该函数返回一个正整数（消息队列的标识符）表示成功，否则会返回-1。

* 向消息队列中写入数据

使用 `msgrcv` 向消息队列中写入数据：

```C++
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
```

其中，`msqid` 是消息队列的标识符，`*msgp` 表示指向消息缓冲区的指针，此位置被用来缓存接收和发送的消息，`msgsz` 用来标识消息的大小，`msgflg` 用来标识消息队列为空时的动作。

* 从消息队列中读取数据

使用 `msgrcv` 从消息队列中读取数据：

```C++
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

其中，`msgtyp` 为0时表示读取消息队列中的第一个消息，大于0时表示读取该消息队列中第一个类型为 `msgtyp` 的消息，小于0时则表示读取消息队列中第一个类型值比 `msgtyp` 的绝对值小的消息。

### UNIX Domain Socket

相对于广泛意义上的 `Socket` 通信，`UNIX Domain Socket`（`UDS`）具有如下优势：

* 使用 `UDS` 的方式传输数据不需要经过网络协议栈，不需要对数据进行重新打包拆包等操作，`UDS` 只有对数据的拷贝过程
* `UDS` 支持两种数据类型 `SOXKET_STREAM` 和 `SOCKET_DGRAM`。由于是在本机环境中通过内核进行通信，不会出现丢包以及包次序不一致的问题

使用 `UDS` 的方式进程间数据交互需要如下步骤：

![](http://images.cnblogs.com/cnblogs_com/goodcandle/socket3.jpg)

* 创建 `Socket`

使用 `socket` 函数可以创建一个 `Socket`：

```C++
int socket(int domain, int type, int protocol);
```

其中，`domain` 表示协议族，通常使用 `AF_UNIX` 表示 `Unix Domain Socket`，`type` 用于指定 `Socket` 的数据类型，一般使用 `SOCK_STREAM` 表示有序可靠的双相连接的字节流，无论数据多大都不会截断，`SOCK_DGRAM` 表示固定最大长度的无序连接，数据超过最大长度之后会被截断。`protocol` 表示协议，通常使用0表示自动选择 `type` 对应的默认协议。

* 绑定 Socket 地址

获取到 `Socket` 的文件描述符之后，通过使用 `bind` 函数将其绑定到一个文件上：

```C++
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

其中，`sockfd` 表示 `Socket` 的唯一标示（由 `socket` 函数创建），`*addr` 表示指定给 `sockfd` 的协议地址，`addrlen` 则是对应的地址长度。

需要注意的是，只有通信双方的进程中，作为服务端角色的进程需要调用 `bind` 函数，客户端只需要通过 `connect` 函数连接到 `Socket` 服务即可。

* 创建服务端监听并接受连接

服务端进程在 `bind` 之后，需要使用 `listen` 函数创建一个监听用于接收数据，并使用 `accept` 接受来自客户端进程的连接：

```C++
int listen(int sockfd, int backlog);

int accept(int sockfd, struct sockaddr *restrict address, socklen_t *restrict address_len);
```

其中，`backlog` 表示连接队列的长度，`address` 和 `address_len` 设置为 `NULL` 即可（`UDS` 中不存在客户端调用地址的问题）。

* 客户端进程连接

客户端进程通过使用 `connect` 函数发送连接请求到服务端进程：

```C++
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

* 读取、写入数据

由于 `UDS` 是双向通信的，所以一旦建立了连接，无论是服务端进程还是客户端进程，都可以读取和写入数据：

```C++
ssize_t send(int sockfd, const void *buf, size_t len, int flags);

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

* 关闭连接

完成数据交互的操作之后，需要使用 `close` 函数手动关闭 `Socket` 连接：

```C++
int close(int sockfd);
```

## Binder 机制涉及到的源码

`Binder` 机制涉及到 `Java` 层、`Native` 层以及 `Linux` 内核层的源代码，主要涉及到如下代码：

* `/framework/base/core/java/android/os/`：
    * `IInterface.java`
    * `IBinder.java`
    * `Parcel.java`
    * `IServiceManager.java`
    * `ServiceManager.java`
    * `ServiceManagerNative.java`
    * `Binder.java`
* `/framework/base/core/jni/`:
  * `android_os_Parcel.cpp`
  * `AndroidRuntime.cpp`
  * `android_util_Binder.cpp`
* `/framework/native/libs/binder/`:
  * `IServiceManager.cpp`
  * `BpBinder.cpp`
  * `IPCThreadState.cpp`
  * `ProcessState.cpp`
* `/framework/native/include/binder/`:
  * `IServiceManager.h`
  * `IInterface.h`
* `/framework/native/cmds/servicemanager/`:
  * `service_manager.c`
  * `binder.c`
* `/kernel/drivers/staging/android/`：
  * `binder.c`
  * `uapi/binder.h`

## Parcel

`Parcel` 是 `Binder` 机制的数据载体。也就是说，通过 `Binder` 机制传递的数据都是以 `Parcel` 作为载体的。其工作原理是把序列化之后的数据按照类型和顺序写入到一块共享内存中，在其他进程中通过 `Parcel` 将数据从共享内存中读取出来，并反序列化为原来的类型。`Parcel` 的工作原理如下图所示：

![](http://upload-images.jianshu.io/upload_images/5889165-0bc444ac5c6f505e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Parcel 属性信息

`Parcel` 提供了大量的接口用于查询或设置属性信息：

* `int dataSize()`：获取当前已经存储的数据大小
* `void setDataCapacity(int size)`： 设置 `Parcel` 的最大存储空间
* `setDataPosition(int pos)`：改变 `Parcel` 中数据的读写位置，必须介于 0 和 `dataSize()` 之间
* `int dataAvail()`：当前 `Parcel` 中可读数据的大小
* `int dataCapacity()`：当前 `Parcel` 的存储能力
* `int dataPosition()`：当前数据的位置
* `int dataSize()`：当前 `Parcel` 所包含的数据大小

### Parcel 中写入和读取数据

`Parcel` 提供了下列方法用于写入数据：

* `writeByte(byte val)`、`writeByteArray(byte[] b)`：写入 `byte` 类型的数据
* `writeString(String val)`、`writeStringArray(String[] val)`：写入 `String` 类型的数据
* `writeStringList(List<String> val)`：写入泛型类型为 `String` 的 `List` 对象
* `writeBoolean(boolean val)`、`writeBooleanArray(boolean[] val)`：写入 `boolean` 类型的数据
* `writeSparseBooleanArray(SparseBooleanArray val)`：写入 `SparseBooleanArray` 类型的数据
* `writeDouble(double val)`、`writeDoubleArray(double[] val)`：写入 `double` 类型的数据
* `writeFloat(float val)`、`writeFloatArray(float[] val)`：写入 `float` 类型的数据
* `writeLong(long val)`、`writeLongArray(long[] val)`：写入 `long` 类型的数据
* `writeInt(int val)`、`writeIntArray(int[] val)`：写入 `int` 类型的数据
* `writeCharSequence`：写入 `CharSequence` 类型的数据
* `writeCharArray(char[] val)`：写入 `char` 类型的数组
* `writeMap(Map val)`：写入 `Map` 对象
* `writeBundle(Bundle val)`：写入 `Bundle` 对象
* `writePersistableBundle(PersistableBundle val)`：写入 `PersistableBundle` 对象
* `writeSize(Size val)`：写入 `Size` 对象
* `writeSizeF(SizeF val)`：写入 `SizeF` 对象
* `writeList(List val)`：写入 `List` 对象
* `writeParcelable(Parcelable p, int parcelableFlags)`、`writeParcelableArray(T[] value, int parcelableFlags)`：写入 `Parcelable` 对象
* `writeSparseArray(SparseArray<Object> val)`：写入 `SparseArray` 对象
* `writeSerializable(Serializable s)`：写入 `Serializable` 数据
* `writeValue(Object v)`、`writeArray(Object[] val)`：写入 `Object` 对象
* `writeException(Exception e)`：写入 `Exception` 对象
* `writeStrongBinder(IBinder val)`、`writeBinderArray(IBinder[] val)`：写入 `IBinder` 数组
* `writeBinderList(List<IBinder> val)`：写入泛型为 `IBinder` 的 `List` 对象

在写入数据时，如果数据超过了 `Parcel` 的存储能力，他会自动扩展内存空间并更新 `dataCapacity()`。与 `write` 方法相对应的，`Parcel` 提供了 `read` 方法用于读取数据。

## Binder 机制的实现原理

`Binder` 机制中有四个重要的组成部分：
* `Client` 进程：最终消费服务的进程
* `Server` 进程：提供服务的进程
* `ServiceManager`：管理各种 `Service` 的注册和查询，将字符串形式的 `Binder` 名称转换成 `Client` 进程中对该 `Binder` 的引用
* `Binder` 驱动：一种虚拟的 `Linux` 设备驱动，持有每个 `Server` 进程在内核空间中的 `Binder` 实体，并为 `Client` 进程提供 `Binder`实体的引用，是连接 `Client` 进程、`Server` 进程和 `ServiceManager` 的桥梁，主要有如下两个功能：
  * 使用内存映射的方式传递进程间的数据
  * 管理 `Binder` 线程池

根据各部分的分工不同，`Binder` 机制大致需要经过如下三个步骤：

* 注册服务
* 获取服务
* 使用服务

### 注册服务

注册服务是指 `Server` 进程向 `ServiceManager` 注册的过程：
* `Server` 进程向 `Binder` 驱动发起注册服务的请求
* `Binder` 驱动将注册服务的请求转发给 `ServiceManager`
* `ServiceManager` 添加该 `Server` 进程到列表
