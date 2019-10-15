---
title: Android 字节码插桩技术
permalink: android-bytecode-pitching-pile
comments: true
date: 2019-03-26 10:49:35
updated: 2019-03-26 10:49:35
tags:
  - 字节码
  - 插桩
  - AOP
  - 面向切面编程
categories:
  - Android 技术文章
---

`Android` 的字节码插桩是指在 `Android` 源代码编译打包的过程中，将一段代码通过某种策略插入或者替换另外一部分代码的过程。下图显示了从 `Java` 源代码到 `.dex` 的过程：

![](https://user-gold-cdn.xitu.io/2019/3/13/16974eb2b8fdd073?imageView2/0/w/1280/h/960/ignore-error/1)

本文中所说的字节码插桩指的是在 `.class` 文件转换为 `.dex` 文件的过程中，通过修改 `.class` 文件从而达到插入或者替换代码的目的。

通常情况下，字节码插装技术应用在面向切面编程（`AOP`）的过程中。与面向对象编程思想（`OOP`）要求我们通过分散职责降低复杂的做法不同，面向切面编程思想要求我们将一些公共的非核心业务的支撑模块抽象为切面，在应用程序编译打包的过程中通过插装技术完成代码的植入。例如我们可以将数据统计的功能抽象为切面，在应用程序编译打包的过程中，自动的将统计代码插入到特定的位置。
<!--more-->
## Android 打包流程

下图显示了 `Android` 应用程序的打包流程：

![](https://user-gold-cdn.xitu.io/2018/3/8/16204d3ceedb2328?imageView2/0/w/1280/h/960/ignore-error/1)

从上图可以看出，`Android` 应用程序从源码到 `apk` 的过程中，需要经过如下七个步骤：

1. `aapt` 工具对 `res` 下的资源文件进行编译，把除图片之外的大部分 `xml` 文件编译为二进制文件，同事生成 `R.java` 和 `resources.arsc` 文件，这两个文件保存了资源的 `ID` 信息以及资源在 `apk` 文件中的目录信息（包括每一个资源名称、类型、值、`ID` 以及所配置的维度信息）
2. 如果源码中存在 `.aidl` 文件，那么把 `.aidl` 编译为对应的 `.java` 文件
3. 将所有的 `.java` 文件（包括 `R.java` 和 `.aidl` 生成的 `.java` 文件）通过 `javac` 工具生成 `.class` 文件
4. 将生成的 `.class` 文件和第三方的 `.class` 文件通过 `dx` 工具生成 `classes.dex` 文件（如果需要分包，则会生成多个该文件）
5. 将资源文件、`classes.dex` 文件和第三方的非 `Java` 资源文件（主要是指 `.so` 库），通过 `apkbuilder` 工具生成未签名的 `apk` 包
6. 使用 `jarsigner` 对上一步的 `apk` 文件进行签名
7. 使用 `zipalign` 工具对 `apk` 文件中的未压缩资源（图片、视频等）进行对齐操作，使资源按照4字节的边界进行对其，以提升资源访问速度

## Java 字节码

`Java` 字节码是 `Java VM` 执行的一种指令格式。通常来说，`.java` 文件经过 `javac` 编译之后的 `.class` 文件的内容就是 `Java` 字节码。`.class` 文件包含了 `Java VM` 指令集和符号表以及其他辅助信息。`.class` 文件是一组以8位字节为基础单位的二进制流，各项数据严格按照顺序紧凑的排列在 `.class` 文件中，中间没有分隔符。这也就意味着整个 `.class` 文件中存储的内容几乎全是程序运行的必要数据。下图展示了 `Java` 字节码的结构：

![](https://user-gold-cdn.xitu.io/2018/3/8/1620560511a4a0c6?imageView2/0/w/1280/h/960/ignore-error/1)

`Java` 字节码中有两个比较重要的概念，全限定名和描述符。

### 全限定名

`.class` 文件中使用全限定名来表示一个类的引用。全限定名的表示与类的全路径一样，只需要把 `.` 替换成 `/` 即可：

```Java
android.widget.TextView
```

的全限定名表示为：

```java
android/widget/TextView
```

### 描述符

描述符的作用是描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符的规则，基本数据类型以及 `void` 都用一个大写字母来表示，对象类型使用字母 `L` 加对象的全限定名表示。一般情况下，对象类型的末尾需要添加一个 `;` 用来表示全限定名的结束。描述符的对应关系如下：

| 描述符 | 数据类型 |
| :-: | :-:  |
| B | byte |
| C | char |
| D | double |
| F | float |
| I | int |
| J | long |
| S | short |
| Z | boolean |
| V | void |
| L | 对象类型 |

对于数据组类型，每一个维度使用 `[` 字符表示。例如我们定义一个 `String` 类型的二维数组：

```java
java.lang.String[][]
```

将会表示为：

```java
[[java/lang/String;
```

使用描述符描述方法时，按照先参数列表后返回值的顺序进行描述。参数列表按照参数的顺序放到一组 `()` 之内：

```java
void setText(String s)
```

将会表示为：

```java
(Ljava/lang/String)V;
```

## 插桩入口

上文中我们介绍了 `Android` 应用程序的打包流程和字节码的基础信息。那么，我们是如何在 `javac` 编译出 `.class` 文件之后，`dx` 工具打包 `.class` 文件之前完成我们的插桩工作呢？

### transform API

`Android Gradle Plugin` 版本从 `V_1.5` 开始，`Gradle` 插件提供了 `transform API`。该 `API` 允许第三方插件在 `.class` 文件打包为 `.dex` 文件之前对编译好的 `.class` 文件进行操作。

#### 创建 Gradle 插件项目

在 `Android Studio` 中，我们在一个完整的 `Android Project` 中创建一个 `Gradle` 插件项目：

`build.gradle` 部分：

```gradle
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    compile gradleApi()
    compile localGroovy()
}
repositories {
    mavenCentral()
}

//设置maven deployer
uploadArchives {
    repositories {
        mavenDeployer {
            pom.groupId = 'com.example.plugin.gradle'
            pom.artifactId = 'my-gradle-plugin'
            pom.version = '1.0.0'
            //文件发布到下面目录
            repository(url: uri('../outputs/gradle/repo'))
        }
    }
}
```

`src/main/groovy/[package]/MyPlugin.groovy` 部分：

```groovy
import org.gradle.api.Plugin
import org.gradle.api.Project

class MyPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        System.out.println("==================================")
        System.out.println(project.getName())
        System.out.println("==================================")
    }
}
```

其中，`void apply(Project project)` 是该 `Gradle` 插件的入口。

`src/main/resources/META_INF/gradle-plugins/my-gradle-plugin.properties` 部分：

```xml
implementation-class=com.example.plugin.gradle.MyPlugin
```

其中，`my-gradle-plugin.properties` 的文件名是插件的名称，即 `my-gradle-plugin`，其内容为声明插件的入口类。

#### 项目中使用 Gradle 插件

在 `Android Project` 的根 `build.gradle` 文件中声明插件的 `Maven` 地址（本示例使用本地地址）以及 `classpath`：

```xml
maven {
    url uri('outputs/gradle/repo')
}

classpath 'com.example.plugin.gradle:my-gradle-plugin:1.0.0'
```

在需要使用插件的 `Android Module` 的 `build.gradle` 文件中声明插件：

```gradle
apply plugin: 'my-gradle-plugin'
```

编译项目，我们就可以看到对应的输出信息：

```shell
==================================
app
==================================
```

#### 使用 transform API

在 `Gradle` 插件项目的 `build.gradle` 文件中引入 `Gradle` 的依赖包即可引入 `transform API`：

```gradle
implementation 'com.android.tools.build:gradle:3.3.2'
```

> 需要注意的是，如果使用的是较早版本的 `Gradle`（`v_2.0.0` 版本之前），那么需要单独引入 `transform API` 的依赖：`compile 'com.android.tools.build:transform-api:1.5.0'`

在引入了 `transform API` 之后，我们需要继承 `Transform` 类并在 `MyPlugin` 中将其注册：

```groovy
class MyClassTransform extends Transform {

    private Project project

    MyClassTransform(Project project) {
        this.project = project
    }

    @Override
    String getName() {
        return "MyClassTransform"
    }

    // 需要处理的数据类型，有两种枚举类型: CLASSES和RESOURCES，CLASSES 代表处理的 java 的 class 文件，RESOURCES 代表要处理资源文件
    @Override
    Set<QualifiedContent.ContentType> getInputTypes() {
        return TransformManager.CONTENT_CLASS
    }

    //    transform 要操作内容的范围，官方文档 Scope 有7种类型：
    //    EXTERNAL_LIBRARIES            只有外部库
    //    PROJECT                       只有项目内容
    //    PROJECT_LOCAL_DEPS            只有项目的本地依赖(本地jar)
    //    PROVIDED_ONLY                 只提供本地或远程依赖项
    //    SUB_PROJECTS                  只有子项目
    //    SUB_PROJECTS_LOCAL_DEPS       只有子项目的本地依赖项(本地jar)
    //    TESTED_CODE                   由当前变量(包括依赖项)测试的代码
    @Override
    Set<? super QualifiedContent.Scope> getScopes() {
        return TransformManager.SCOPE_FULL_PROJECT
    }

    // 指明当前 transform 是否支持增量编译
    @Override
    boolean isIncremental() {
        return false
    }

    //    transform 中的核心方法
    //    inputs 中是传过来的输入流，其中有两种格式，一种是 jar 包格式一种是目录格式
    //    outputProvider 获取到输出目录，最后将修改的文件复制到输出目录，这一步必须做不然编译会报错
    @Override
    void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {

        System.out.println("============transform start ================")
        System.out.println("============transform end ================")
        super.transform(transformInvocation)
    }
}
```

然后在 `MyPlugin` 中注册：

```groovy
class MyPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        System.out.println("==================================")
        System.out.println("module name: " + project.getName())
        System.out.println("==================================")

        def android = project.extensions.getByType(AppExtension)
        android.registerTransform(new MyClassTransform(project))
    }
}
```


至此，我们就已经成功创建了一个 `Gradle` 插件项目。接下来我们讨论一下 `transform API` 的原理。

### Transform API 原理

每一个 `transform` 都是一个 `Gradle Task`，`Android` 编译器中的 `TaskManager` 将每个 `transform` 串联起来。第一个 `transform` 接收来自 `javac` 编译的结果以及已经拉取到在本地的第三方依赖、`assets` 目录下的资源等。这些编译的中间产物在 `transform` 组成的链条上流动，每个 `transform` 节点可以对 `class` 进行处理，然后再传递给下一个 `transform`。我们自定义的 `transform` 则会插入到整个链条的最前面：

![](http://quinnchen.me/images/transformconsume_transform.png)

`transform` 通过 `ContentType` 和 `Scope` 对输入的数据进行过滤，对应的 `transform API` 是：

```groovy
Set<QualifiedContent.ContentType> getInputTypes() {
    return TransformManager.CONTENT_CLASS
}

Set<? super QualifiedContent.Scope> getScopes() {
    return TransformManager.SCOPE_FULL_PROJECT
}
```

在 `Gradle` 插件开发中，我们一般只能使用 `CLASSES` 和 `RESOURCES` 两种类型，其中 `CLASSES` 类型包含了 `class` 文件和 `jar` 文件：

![](http://quinnchen.me/images/transformContentType.png)

相较于 `ContentType`，`Scope` 则是另外一个维度的数据过滤机制，如果是要处理所有的 `class` 字节码，`Scope` 我们一般使用 `TransformManager.SCOPE_FULL_PROJECT`：

![](http://quinnchen.me/images/transformScope.png)

#### transform(TransformInvocation transformInvocation) 方法

`transform(TransformInvocation transformInvocation)` 方法是 `Transform API` 中最为核心的方法。我们在 `transform` 方法中将每个 `jar` 和 `class` 文件复制到 `dest` 路径（`dest` 路径就是下一个 `transform` 的输入数据），然后在复制之前完成我们的插桩工作：

```groovy
void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {
    super.transform(transformInvocation)
    // 是否增量编译
    boolean isIncremental = transformInvocation.isIncremental()

    //消费型输入，可以从中获取jar包和class文件夹路径。需要输出给下一个任务
    Collection<TransformInput> transformInputs = transformInvocation.getInputs()

    // 引用型输入，无需输出
    Collection<TransformInput> referenceInputs = transformInvocation.getReferencedInputs()

    // OutputProvider管理输出路径，如果消费型输入为空，你会发现OutputProvider == null
    TransformOutputProvider outputProvider = transformInvocation.getOutputProvider()

    for (TransformInput transformInput : transformInputs) {
        for (JarInput jarInput : transformInput.getJarInputs()) {
            File dest = outputProvider.getContentLocation(jarInput.getFile().getAbsolutePath(), jarInput.getContentTypes(), jarInput.getScopes(), Format.JAR)
            //将修改过的字节码 copy 到 dest，就可以实现编译期间干预字节码的目的了
            FileUtils.copyFile(jarInput.getFile(), dest)
        }

        for (DirectoryInput directoryInput : transformInput.getDirectoryInputs()) {
            File dest = outputProvider.getContentLocation(directoryInput.getName(), directoryInput.getContentTypes(), directoryInput.getScopes(), Format.DIRECTORY)
            // 将修改过的字节码 copy 到 dest，就可以实现编译期间干预字节码的目的了
            FileUtils.copyDirectory(directoryInput.getFile(), dest)
        }
    }
}
```

## ASM

[ASM](https://asm.ow2.io)是 `Java` 平台上较为常用的字节码工具。相较于 `Javasist` 和 `AspectJ`，`ASM` 在处理速度、社区活跃性和内存占用方面有着较为明显的优势。

### 引入 ASM

在 `build.gradle` 文件中添加如下依赖：

```gradle
implementation 'org.ow2.asm:asm:6.0'
implementation 'org.ow2.asm:asm-util:6.0'
implementation 'org.ow2.asm:asm-commons:6.0'
```

其他版本的 `ASM` 请参考 [https://search.maven.org/search?q=g:org.ow2.asm](https://search.maven.org/search?q=g:org.ow2.asm)。

#### 使用 ASM API

`ASM` 提供了两种类型的 `API`：

* `Tree API`：将 `class` 文件的结构读取到内存中，构建一个树形结构
* `Visitor API`：通过接口的形式，分类读 `class` 和 写 `class` 的逻辑
