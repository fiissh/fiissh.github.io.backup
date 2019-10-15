---
title: Android 进程间通信的经典实现
permalink: inter-process-communication-classic-implement
comments: true
date: 2019-03-18 15:32:50
updated: 2019-03-18 15:32:50
tags:
  - 进程间通信
  - 深入理解 Android 内核设计思想
categories:
  - Android Binder
---

通常情况下，操作系统中的每一个进程都拥有各自独立的用户空间，任何一个进程的全局变量对于其他的进程来说都是不可见的。内核通过在其内核空间中开辟缓冲区用以支持进程间的数据交互。内核提供的这种机制被称为进程间通信。

常用的进程间通信的方式有如下几种：

* 共享内存
* 管道
  * 匿名管道
  * 有名管道
* 消息队列
* `UNIX Domain Socket`

<!--more-->
## 进程间通信的经典实现


## Android 进程间通信

`Android` 系统为我们提供了如下几种进程间通信机制的实现：

* 文件
* `Binder`
* `AIDL`
* `Messenger`
* `ContentProvider`
* `Socket`

### 文件

共享文件的方式实现进程间数据交互适用于对数据同步不敏感的情况。共享文件的方式主要是通过 `ObjectOutputStream` 和 `ObjectInputStream` 实现的。

首先，我们在进程 `A` 中使用 `ObjectOutputStream` 向文件中写入一个 `User` 对象：

```java
File file = new File(context.getDataDir().getPath() + "/shared_file.file");
OutputStream outputStream = new FileOutputStream(file);
ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
objectOutputStream.writeObject(new User("fiissh.zhao", 18));
```

`ObjectOutputStream` 最终接收一个 `File` 对象用于存放数据。我们通过 `writeObject` 方法将需要共享的对象（也可以通过其他方法写入其他类型的数据）写入到文件中去。

然后，我们在进程 `B` 中使用 `ObjectInputStream` 将共享的数据从文件中读取出来：

```Java
File file = new File(context.getDataDir().getPath() + "/shared_file.file");
InputStream inputStream = new FileInputStream(file);
ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
User user = (User) objectInputStream.readObject();
```

`ObjectInputStream` 与 `ObjectOutputStream` 一样，也是接收一个 `File` 对象，然后通过 `readObject` 将对象从文件中读取出来。

需要注意的是，这种方式读取数据时，需要严格按照数据的写入顺序进行。而且这种方式除了能够共享基本数据类型的数据之外，还可以共享 `Serializable` 序列的数据。那么，我们的 `User` 对象的定义如下：

```Java
public class User implements Serializable {

    public String name;
    public int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

共享文件的方式可以共享基本数据类型和 `Serializable` 序列化的数据。但是这种方式需要使用额外的存储空间，并且数据在序列化和反序列化的过程中会产生一些临时文件，在性能方面有些欠缺。而且由于数据的写入和读取的时机的不确定性，我们很难保证数据的同步性。

### Binder

`Binder` 是 `Android` 平台最为重要的进程间通信的方案。我们后文中提到的 `AIDL`、`Messenger` 和 `ContentProvider` 三种进程间通信的方式，都是 `Binder` 的不同表现形式而已。`Android` 平台使用 `Binder` 机制作为进程间通信的实现，主要基于如下几个原因：

* 从性能的角度来说，`Binder` 只进行了一次数据拷贝，而管道、消息队列和 `UDS` 等方案都需要两次数据拷贝。相较于共享内存不需要数据拷贝的情况，`Binder` 机制在性能方面仅次于共享内存的方式
* 共享内存的方式很难解决数据并发访问的同步性问题，而 `Binder` 机制基于 `Server-Client` 架构的设计很好的解决了数据同步的问题

`Binder` 机制采用分层架构，每一层都有明确的功能分工，并向更高层架构提供服务：

![](http://gityuan.com/images/binder/binder_start_service/binder_ipc_arch.jpg)

### AIDL

`AIDL` 是 `Android` 接口定义语言，是一种基于 `Binder` 的进程间通信的实现方式。通常情况下，我们
