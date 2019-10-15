---
title: Mac 系统下载 Android 源代码以及编译
permalink: mac-os-download-aosp-and-buid
comments: true
date: 2018-11-27 13:42:09
updated: 2018-11-27 13:42:09
tags:
  - AOSP 编译
categories:
  - Android 技术文章
---

`Google` 官方推荐使用 `Ubuntu` 和 `Mac` 进行 `AOSP` 源码的编译。使用 `Mac` 下载并编译 `AOSP` 源码需要满足如下硬件和软件条件：

* 如果需要编译 `Gingerbread`（`Android 2.3.x`）及以上版本（包含 `master`） 分支，需要使用64位系统环境。如果是较低的版本，则可以使用32位系统环境；
* 如果只是查看源代码，则至少需要100G的可用磁盘空间，如果需要进行编译，则还需要额外的150G可用磁盘空间。如果要进行多次编译或使用 `ccache`，那么需要更多地磁盘空间，建议预留300G以上磁盘空间，并将磁盘影响设置为`大小写敏感`；
* 使用 `MacOSSierra v10.10`（`Yosemite`）及以上系统版本，并安装 `Xcode` 和 `命令行工具`及以上版本。`Xcode` 建议使用 `v4.5.2` 及以上版本。如果需要编译早期版本，则需要的系统环境以及 `Xcode` 版本如下：
  * `Android 6.0`（`Marshmallow`）到 `AOSP master` 需要使用 `Mac OS v10.10` （`Yosemite`）及更高版本和 `Xcode 4.5.2` 和`命令行工具`；
  * `Android 5.x` （`Lollipop`）需要使用 `Mac OS v10.8`（`Mountain Lion`）和 `Xcode 4.5.2` 和`命令行工具`；
  * `Android 4.1.x-4.3.x`（`Jelly Bean`）到 `Android 4.4.x`（`KitKat`）需要使用 `Mac OS v10.6` （`Snow Leopard`） 或 `Mac OS X v10.7`（`Lion`）和 `Xcode 4.2`；
  * `Android 1.5`（`Cupcake`）到 `Android 4.0.x` （`Ice Cream Sandwich`）需要使用 `Mac OS v10.5`（`Leopard`） 或 `Mac OS X v10.6`（`Snow Leopard`）和 `Mac OS X v10.5 SDK`；
* 目前，`AOSP` 的 `master` 分支带有预先编译的 `OpenJDK`，因此无需安装。如果需要编译早期版本，则需要单独安装：
  * `Android 7.0`（`Nougat`）到 `Android 8.0`（`O`）需要安装 `JDK 8u45` 版本；
  * `Android 5.x` （`Lollipop`）到 `Android 6.0` （`Marshmallow`）需要安装 `JDK 7u71` 版本；
  * `Android 2.3.x` （`Gingerbread`）到 `Android 4.4.x` （`KitKat`）需要安装 `JDK 6` 版本；
  * `Android 1.5` （`Cupcake`）到 `Android 2.2.x` （`Froyo`）需要安装 `JDK 5` 版本；
* 系统环境中，`Git` 版本不低于 `v1.7`；
* 系统环境中，`Python` 版本为 `v2.6` 或 `v2.7`；
* 系统环境中，`GNU make` 版本为 `v3.82`；
<!--more-->

 如果需要编译 `Android 4.0.x`（`Ice Cream Sandwich`）以及更低的版本，需要将 `make v3.82` 工具降低到较低版本（`make v3.81`）：
 * 第一步：修改 `/opt/local/etc/macports/sources.conf`，在 `rsync` 行上方添加如下内容：
 ```
 file:///Users/Shared/dports
 ```
然后使用 `mkdir /Users/Shared/dports` 创建目录。

 * 第二步：在新的 dports 目录下，运行以下命令：
 ```shell
 svn co --revision 50980 http://svn.macports.org/repository/macports/trunk/dports/devel/gmake/ devel/gmake/
 ```
 * 第三步：为新的本地存储库创建一个端口索引：
 ```shell
 portindex /Users/Shared/dports
 ```
 * 第四步：使用以下命令安装旧版 `gmake`：
 ```shell
 sudo port install gmake @3.81
 ```

## 下载 `AOSP` 源代码

`Android` 源代码树位于由 `Google` 托管的 `Git` 代码库中。`Git` 代码库中包含 `Android` 源代码的元数据，其中包括与对源代码进行的更改以及更改日期相关的元数据。

### 安装 Repo

要处理 `Android` 源代码，你需要同时使用 `Git` 和 `Repo`。在大多数情况下，你可以只使用 `Git`（不必使用 `Repo`），或结合使用 `Repo` 和 `Git` 命令以组成复杂的命令。不过，使用 `Repo` 执行基本的跨网络操作可大大简化你的工作。如果需要安装 `Repo` 工具，请在准备存放 `Android` 的根目录下执行如下操作：

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > repo

chmod a+x ~/bin/repo
```

由于网络问题，可以使用[`清华大学开源软件镜像站`](https://mirrors.tuna.tsinghua.edu.cn)提供的 `Repo` 镜像：


```shell
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > repo

chmod a+x ~/bin/repo
```

> 如果由于网络问题无法下载 `AOSP` 源代码，可以使用[`清华大学开源软件镜像站`](https://mirrors.tuna.tsinghua.edu.cn)提供的 `AOSP` 源码镜像。使用方式是将下载的 `repo` 文件中的 `REPO_URL` 地址修改为 `https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/`。

### 初始化 `Repo`

请在准备存放 `Android` 的根目录（工作目录）下执行如下操作用来获取最新版本的 `Repo`以及其最近的所有错误更正内容：

```shell
./repo init -u https://android.googlesource.com/platform/manifest
```

`https://android.googlesource.com/platform/manifest` 用于指定 `Android` 源码中包含的各个代码库将位于工作目录的具体位置。

如果要对 `master` 以外的分支进行校验，则需要使用 `-b` 来指定相应的分支。分支列表请参考：[`Android 源代码标记和版本`](#Android 源代码标记和版本)。

```shell
./repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```

`Repo` 初始化成功之后，系统将会显示成功的消息，并在工作目录下生成一个 `.repo` 的目录，用于保存清单文件等。

> 如果由于网络问题不能初始化 `Repo`，请使用 [`清华大学开源软件镜像站`](https://mirrors.tuna.tsinghua.edu.cn)。具体方法是，如果没有替换 `repo` 文件中的 `REPO_URL` 地址，请先替换该地址。然后，将上述 `https://android.googlesource.com` 替换为 `https://aosp.tuna.tsinghua.edu.cn`。

### 下载 Android 源代码树

要将 `Android` 源代码树从默认清单中指定的代码库下载到工作目录，请运行以下命令：

```shell
./repo sync -jn
```

`-jn` 表示当前使用 `n` 条线程进行下载，请视网络环境适当更改 `n` 的值，建议使用 `-j4`。如果 `n` 值过大，可能会出现服务端拒绝服务的情况，在使用[`清华大学开源软件镜像站`](https://mirrors.tuna.tsinghua.edu.cn)是尤为明显。

### 使用身份验证

默认情况下，访问 `Android` 源代码均为匿名操作。为了防止服务器被过度使用，每个 `IP` 地址都有一个相关联的配额。

当与其他用户共用一个 `IP` 地址时（例如，在越过 `NAT` 防火墙访问源代码代码库时），系统甚至会针对常规使用模式（例如，许多用户在短时间内从同一个 `IP` 地址同步）触发配额。

在这种情况下，可以使用进行身份验证的访问方式，此类访问方式会对每位用户使用单独的配额，而不考虑 `IP` 地址：

* 第一步：使用[密码生成器](https://android.googlesource.com/new-password)生成密码，然后按照密码生成器页面中的说明进行操作；
* 第二步：强制使用进行身份验证的访问方式初始化 `Repo`：

```shell
./repo init -u https://android.googlesource.com/a/platform/manifest
```

### 验证 Git 标记

将以下公钥加载到您的 GnuPG 密钥数据库中。该密钥用于签署代表各版本的带注释标记：

```shell
gpg --import
```

复制并粘贴以下密钥，然后输入 EOF (Ctrl-D) 以结束输入并处理密钥：

```shell
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.2.2 (GNU/Linux)

mQGiBEnnWD4RBACt9/h4v9xnnGDou13y3dvOx6/t43LPPIxeJ8eX9WB+8LLuROSV
lFhpHawsVAcFlmi7f7jdSRF+OvtZL9ShPKdLfwBJMNkU66/TZmPewS4m782ndtw7
8tR1cXb197Ob8kOfQB3A9yk2XZ4ei4ZC3i6wVdqHLRxABdncwu5hOF9KXwCgkxMD
u4PVgChaAJzTYJ1EG+UYBIUEAJmfearb0qRAN7dEoff0FeXsEaUA6U90sEoVks0Z
wNj96SA8BL+a1OoEUUfpMhiHyLuQSftxisJxTh+2QclzDviDyaTrkANjdYY7p2cq
/HMdOY7LJlHaqtXmZxXjjtw5Uc2QG8UY8aziU3IE9nTjSwCXeJnuyvoizl9/I1S5
jU5SA/9WwIps4SC84ielIXiGWEqq6i6/sk4I9q1YemZF2XVVKnmI1F4iCMtNKsR4
MGSa1gA8s4iQbsKNWPgp7M3a51JCVCu6l/8zTpA+uUGapw4tWCp4o0dpIvDPBEa9
b/aF/ygcR8mh5hgUfpF9IpXdknOsbKCvM9lSSfRciETykZc4wrRCVGhlIEFuZHJv
aWQgT3BlbiBTb3VyY2UgUHJvamVjdCA8aW5pdGlhbC1jb250cmlidXRpb25AYW5k
cm9pZC5jb20+iGAEExECACAFAknnWD4CGwMGCwkIBwMCBBUCCAMEFgIDAQIeAQIX
gAAKCRDorT+BmrEOeNr+AJ42Xy6tEW7r3KzrJxnRX8mij9z8tgCdFfQYiHpYngkI
2t09Ed+9Bm4gmEO5Ag0ESedYRBAIAKVW1JcMBWvV/0Bo9WiByJ9WJ5swMN36/vAl
QN4mWRhfzDOk/Rosdb0csAO/l8Kz0gKQPOfObtyYjvI8JMC3rmi+LIvSUT9806Up
hisyEmmHv6U8gUb/xHLIanXGxwhYzjgeuAXVCsv+EvoPIHbY4L/KvP5x+oCJIDbk
C2b1TvVk9PryzmE4BPIQL/NtgR1oLWm/uWR9zRUFtBnE411aMAN3qnAHBBMZzKMX
LWBGWE0znfRrnczI5p49i2YZJAjyX1P2WzmScK49CV82dzLo71MnrF6fj+Udtb5+
OgTg7Cow+8PRaTkJEW5Y2JIZpnRUq0CYxAmHYX79EMKHDSThf/8AAwUIAJPWsB/M
pK+KMs/s3r6nJrnYLTfdZhtmQXimpoDMJg1zxmL8UfNUKiQZ6esoAWtDgpqt7Y7s
KZ8laHRARonte394hidZzM5nb6hQvpPjt2OlPRsyqVxw4c/KsjADtAuKW9/d8phb
N8bTyOJo856qg4oOEzKG9eeF7oaZTYBy33BTL0408sEBxiMior6b8LrZrAhkqDjA
vUXRwm/fFKgpsOysxC6xi553CxBUCH2omNV6Ka1LNMwzSp9ILz8jEGqmUtkBszwo
G1S8fXgE0Lq3cdDM/GJ4QXP/p6LiwNF99faDMTV3+2SAOGvytOX6KjKVzKOSsfJQ
hN0DlsIw8hqJc0WISQQYEQIACQUCSedYRAIbDAAKCRDorT+BmrEOeCUOAJ9qmR0l
EXzeoxcdoafxqf6gZlJZlACgkWF7wi2YLW3Oa+jv2QSTlrx4KLM=
=Wi5D
-----END PGP PUBLIC KEY BLOCK-----
```

导入密钥后，您可以通过以下命令验证任何标记：

```shell
git tag -v TAG_NAME
```

### 优化编译环境

我们可以根据需要指示编译过程使用 `ccache` 编译工具，`ccache` 是适用于 `C/C++` 的编译器缓存，有助于提高编译速度。这对于编译服务器和其他高容量生产环境来说尤其有用。`ccache` 可用作用于加快重新编译速度的编译器缓存。如果您经常使用 `make clean`，或者经常在不同的编译产品之间切换，则非常适合使用 `ccache`。

> 需要注意的是，如果我们执行的是增量编译，`ccache` 可能会由于没有命中缓存而带来额外的问题，从而降低了编译速度。

要使用 `ccache`，请在源代码树的根目录下执行以下命令：

```shell
export USE_CCACHE=1
export CCACHE_DIR=/<path_of_your_choice>/.ccache
prebuilts/misc/darwin-x86/ccache/ccache -M 50G
```

建议的缓存大小为 50G 到 100G。

请将以下内容添加到 ``.bashrc`（或等同文件）中：

```shell
export USE_CCACHE=1
```

默认情况下，缓存将存储在 `~/.ccache` 下。如果主目录位于 `NFS` 或一些其他的非本地文件系统中，还需要在 `.bashrc` 文件中指定目录。

在编译 `Ice Cream Sandwich` (`4.0.x`) 或更低版本时，`ccache` 位于其他位置：

```shell
prebuilt/linux-x86/ccache/ccache -M 50G
```

该设置会存储在 `CCACHE_DIR` 中，并且为永久设置。

## 编译 AOSP 源代码

以下关于编译 `Android` 源代码树的说明适用于所有分支，`master` 除外。编译命令的基本顺序如下：

* 下载专有二进制文件
* 设置环境
* 选择目标
* 编译代码
* 开始运行

### 下载专有二进制文件

我们不能只通过纯源代码来使用 `AOSP`，还需要运行与硬件相关的其他专有库（例如用于硬件图形加速的专有库）。如需其他资源的下载链接和[设备二进制文件](https://source.android.com/setup/build/requirements.html#binaries)，请参阅以下各部分。

#### 下载专有二进制文件

对于运行带标记的 `AOSP` 版本分支的受支持设备，我们可以从 [`Google` 的驱动程序](https://developers.google.com/android/drivers)下载相关的官方二进制文件。有了这些二进制文件，我们将有权使用采用非开源代码的其他硬件功能。在针对某种设备编译 `master` 分支时，请使用位于 `Android` 源代码树的 `vendor/` 层次结构中的二进制文件。

#### 解压专有二进制文件

每组二进制文件都是压缩包中的一个自解压脚本。解压每个压缩包，从源代码树的根目录运行附带的自解压脚本，然后确认我们同意附带的许可协议的条款。二进制文件及其对应的 `Makefile` 将会安装在源代码树的 `vendor/` 层次结构中。

#### 清理

为了确保新安装的二进制文件在解压后会被适当考虑在内，请使用以下命令删除所有以前编译操作的已有输出：

```shell
make clobber
```

### 设置环境

使用如下命令初始化编译环境：

```shell
source build/envsetup.sh
```

或者

```shell
. build/envsetup.sh
```

### 选择目标

使用 `lunch` 选择要编译的目标。确切的配置可作为参数进行传递。例如，以下命令表示针对模拟器进行完整编译，并且所有调试功能均处于启用状态：

```shell
lunch aosp_arm-eng
```

如果我们没有提供任何参数就运行命令，lunch 将提示从菜单中选择一个目标。

所有编译目标都采用 `BUILD-BUILDTYPE` 形式，其中 `BUILD` 是表示特定功能组合的代号。`BUILDTYPE` 是以下类型之一：

* user：权限受限；适用于生产环境，是旨在用作最终版本配置步骤的编译类型：
  * 安装带有 `user` 标记的模块。
  * 除了带有标记的模块之外，还会根据产品定义文件安装相应模块
  * `ro.secure=1`
  * `ro.debuggable=0`
  * `adb` 默认处于停用状态
* userdebug：与 `user` 类似，除了以下几点之外，其余均与 user 相同：
  * 安装带有 `debug` 标记的模块
  * `ro.debuggable=1`
  * `adb` 默认处于启用状态
* eng：具有额外调试工具的开发配置，是默认的编译类型：
  * 安装带有 `eng` 和 `/` 或 `debug` 标记的模块
  * 除了带有标记的模块之外，还会根据产品定义文件安装相应模块
  * `ro.secure=0`
  * `ro.debuggable=1`
  * `ro.kernel.android.checkjni=1`
  * `adb` 默认处于启用状态

`userdebug` 版本应和 `user` 版本以相同的方式运行，且能够启用通常会破坏平台安全模型的额外调试。这可让 `userdebug` 版本具有更强大的诊断功能，从而成为进行 `user` 测试的最佳选择。使用 `userdebug` 版本进行开发时，我们应遵循 `userdebug` 准则:

* `userdebug` 定义为已启用 `root` 权限的 `user` 版本，以下情况除外：
  * 仅由用户视需要运行且仅用于 `userdebug` 版本的应用
  * 仅在空闲维护状态（连接充电器/充满电）下执行的操作，例如，使用 `dex2oatd` 与 `dex2oat` 比较后台编译情况
* 请勿使用任何依靠编译类型以在默认状态下启用/不启用的功能。建议开发者不要使用任何影响电池续航时间的日志记录形式（例如调试日志记录或堆转储）
* 在 `userdebug` 版本中默认启用的任何调试功能都应明确定义，并告知负责该项目的所有开发者。请仅在限定时间里启用这些调试功能，直到问题得以解决

`eng` 版本会优先考虑平台负责工程师的工程生产力。`eng` 版本会关闭用于提供良好用户体验的各种优化。否则，`eng` 版本的运行方式将类似于 `user` 和 `userdebug` 版本，以便设备开发者看到代码在这些环境下的运行方式。

要详细了解如何针对实际硬件进行编译以及如何在实际硬件上运行编译系统，请参阅[运行编译系统](#运行编译系统)。

### 编译代码

可以使用 `make` 编译 `AOSP` 源代码。`GNU make` 可以借助 `-jn` 参数处理并行任务，通常使用的任务数 `n` 介于编译时所用计算机上硬件线程数的 1-2 倍之间：

```shell
make -jn
```

编译完成之后，将会在工作目录生成一个 `out` 文件夹，该文件夹主要包含：

* `out/host/`：该目录下包含了针对主机的 `Android` 开发工具的产物。即 `SDK` 中的各种工具，例如：`emulator`，`adb`，`aapt` 等
* `out/target/common/`：该目录下包含了针对设备的共同的编译产物，主要是 `Java` 应用代码和 `Java` 库
* `out/target/product/[product_name]`：包含了针对特定设备的编译结果以及平台相关的 `C/C++` 库和二进制文件
* `out/dist/`：包含了为多种分发而准备的包，通过 `make disttarget` 将文件拷贝到该目录，默认的编译目标不会产生该目录

编译完成之后，最重要的三个镜像文件都位于 `/out/target/product/[product_name]` 目录下：

* `system.img`：包含了 `Android` 的系统文件、库、可执行文件以及预置的应用程序，将被挂载为根分区
* `ramdisk.img`：在启动时将被 `Linux 内核`挂载为只读分区，它包含了 `/init` 文件和一些配置文件。它用来挂载其他系统镜像并启动 `init` 进程
* `userdata.img`：将被挂载为 `/data`，包含了应用程序相关的数据以及和用户相关的数据



`out` 目录详情可参考如下表：

```shell
|-- host/                           # 构建源码需要的工具和库文件
|-- target/product/generic/         # 生成最后产品的目录
    |-- data                        # 这个目录是用来生成<数据文件系统镜像>（data file system image）userdata.img
    |-- obj                         # 生成的中间文件,最后都要拷贝到root或system文件夹中，最后生成镜像img文件
    |   |-- APPS                    # android应用
    |   |-- ETC
    |   |-- EXECUTABLES             # 所有本地运行工具 ping toolbox
    |   |-- include
    |   |-- JAVA_LIBRARIES
    |   |-- lib                     # 从SHARED_LIBRARIES拷贝，各种.so共享库
    |   |                         
    |   |-- PACKAGING
    |   |-- SHARED_LIBRARIES        # 共享库
    |   |   |-- {LOCAL_MODULE_NAME}_intermediates    # 各种共享库 {LOCAL_MODULE_NAME}模块名称
    |   |       |                             
    |   |        -- LINKED          # 链接到二进制文件, e.g, .so文件
    |    -- STATIC_LIBRARIES        # 静态库
    |-- root                        # 这个目录用来创建<root文件系统>（root file system）,  生成的ramdisk.img是用这个文件夹生成的镜像
    |   |-- data
    |   |-- dev
    |   |-- proc
    |   |-- sbin
    |   |-- sys
    |    -- system
    |-- symbols                     # 带调试信息的
    |   |-- data
    |   |-- sbin
    |    -- system
     -- system                      # 用来创建system.img, 大部分的应用程序和库都在system中
        |-- app
        |-- bin
        |-- etc
        |-- fonts
        |-- framework
        |-- lib
        |-- media
        |-- tts
        |-- usr
         -- xbin
```

在执行过一次完整编译之后，我们可以针对系统的某些模块进行单独编译。

> 在源码没有完全编译成功或执行了 `make clean` 或 `make clobber` 之后，也是不能进行单模块编译的。如果在完全编译成功之后，我们退出了终端，需要再次执行`设置环境`和 `选择目标`两个步骤。

以编译 `Calculator` 为例：

* 进入 `Calculator` 所在的源码目录（packages/apps/Calculator/）
* 在 `Calculator` 源码目录下执行 `mmm` 命令进行编译，编译成功之后会输出 `apk` 的存放路径

常用的编译命令：


| 命令 | 备注 |
|:---:|:---:|
| m | 在源码树的根目录执行编译 |
| mm | 编译当前路径下所有模块，但不包含依赖 |
| mmm [module_path] | 编译指定路径下所有模块，但不包含依赖 |
| mma | 编译当前路径下所有模块，且包含依赖 |
| mmma [module_path] | 编译指定路径下所有模块，且包含依赖 |
| make [module_name] | 无参数，则表示编译整个Android代码 |

部分模块的编译命令如下：

| 模块 | make 命令 | mmm 命令 |
|:---:|:---:|:---:|
| init | make init | mmm system/core/init |
| zygote | make app_process | mmm frameworks/base/cmds/app_process |
| system_server | make services | mmm frameworks/base/services |
| java framework | make framework | mmm frameworks/base |
| framework 资源 | make framework-res | mmm frameworks/base/core/res |
| jni framework | make libandroid_runtime | mmm frameworks/base/core/jni |
| binder | make libbinder |  mmm frameworks/native/libs/binder |

## 编译内核

### 选择内核

下表列出了内核源代码和二进制文件的名称及所在位置：

![](/images/选择内核.png)

确定要使用的设备项目之后，请查看内核二进制文件的 `Git` 日志。设备项目采用 `device/VENDOR/NAME` 形式:

```shell
git clone https://android.googlesource.com/kernel/hikey-linaro
cd hikey-linaro
git log --max-count=1 kernel
```

内核二进制文件的提交消息中包含用于编译二进制文件的内核源代码的部分 `Git` 日志。该日志中的第一个条目是最新内容（也即用于编译内核的条目）。请记下提交消息，因为我们在后续步骤中会用到该消息。

### 确定内核版本

要确定系统映像中使用的内核版本，请对内核文件运行以下命令：

```shell
dd if=kernel bs=1 skip=$(LC_ALL=C grep -a -b -o $'\x1f\x8b\x08\x00\x00\x00\x00\x00' kernel | cut -d ':' -f 1) | zgrep -a 'Linux version'
```

对于 `Nexus 5` (`hammerhead`)，请运行以下命令：

```shell
dd if=zImage-dtb bs=1 skip=$(LC_ALL=C od -Ad -x -w2 zImage-dtb | grep 8b1f | cut -d ' ' -f1 | head -1) | zgrep -a 'Linux version'
```

### 下载源代码

使用适当的 `git clone` 命令，下载您要编译的内核的源代码。例如，以下命令会克隆 `common` 内核（一种可自定义的通用内核）：

```shell
git clone https://android.googlesource.com/kernel/common
```

内核项目的完整列表在 [Kernel](https://android.googlesource.com/kernel) 目录下。以下是一些常用内核及其各自的 `git clone` 命令:

* `exynos` 项目包含适用于 `Nexus 10` 的内核源代码，可用作在 `Samsung Exynos` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/exynos
```

* `goldfish` 项目包含适用于所模拟的平台的内核源代码
```shell
git clone https://android.googlesource.com/kernel/goldfish
```

* `hikey-linaro` 项目用于 `HiKey` 参考板，可用作在 `HiSilicon 620` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/hikey-linaro
```

* `msm` 项目包含适用于 `ADP1`、`ADP2`、`Nexus One`、`Nexus 4`、`Nexus 5`、`Nexus 6`、`Nexus 5X`、`Nexus 6P`、`Nexus 7` (2013)、`Pixel` 和 `Pixel XL` 的源代码，可用作在 `Qualcomm MSM` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/msm
```

* `omap` 项目用于 `PandaBoard` 和 `Galaxy Nexus`，可用作在 `TI OMAP` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/omap
```

* `samsung` 项目用于 `Nexus S`，可用作在 `Samsung Hummingbird` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/samsung
```

* `tegra` 项目用于 `Xoom`、`Nexus 7` (2012)、`Nexus 9`，可用作在 `NVIDIA Tegra` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/tegra
```

* `x86_64` 项目包含适用于 `Nexus Player` 的内核源代码，可用作在 `Intel x86_64` 芯片组上开展相关工作的着手点
```shell
git clone https://android.googlesource.com/kernel/x86_64
```

### 编译内核

当我们了解了内核的最后一条提交消息并已成功下载内核源代码和预编译的 `gcc` 后，就可以编译内核了。以下编译命令使用了 `hikey` 内核：

```shell
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
cd hikey-linaro
git checkout -b android-hikey-linaro-4.1 origin/android-hikey-linaro-4.1
make hikey_defconfig
make
```

要编译不同的内核，只需将 `hikey-linaro` 替换为我们要编译的内核的名称即可。

映像会输出到 `arch/arm64/boot/Image` 目录；内核二进制文件会输出到 `arch/arm64/boot/dts/hisilicon/hi6220-hikey.dtb` 文件。请将 `Image` 目录和 `hi6220-hikey.dtb` 文件复制到 `hikey-kernel` 目录。

或者，您可以在使用 `make bootimage`（或编译启动映像的任何其他 `make` 命令行）时添加 `TARGET_PREBUILT_KERNEL` 变量。所有设备均支持该变量，因为它是通过 `device/common/populate-new-device.sh` 进行设置的。例如：

```shell
export TARGET_PREBUILT_KERNEL=$your_kernel_path/arch/arm/boot/zImage-dtb
```

### 开始运行

如果我们使用模拟器运行，则使用如下命令：

```shell
emulator
```

## 常用命令

| 命令 | 备注 |
|:---:|:---:|
| cgrep | 所有 `C/C++` 文件执行搜索操作 |
| jgrep | 所有 `Java` 文件执行搜索操作 |
| ggrep | 所有 `Gradle` 文件执行搜索操作 |
| mangrep [keyword] | 所有 `AndroidManifest.xml` 文件执行搜索操作 |
| mgrep [keyword] | 所有 `Android.mk` 文件执行搜索操作 |
| sepgrep [keyword] | 所有 `sepolicy` 文件执行搜索操作 |
| resgrep [keyword] | 所有本地 `res/*.xml` 文件执行搜索操作 |
| sgrep [keyword] | 所有资源文件执行搜索操作 |
| croot | 切换至 `Android` 根目录 |
| cproj | 切换至工程的根目录 |
| godir [filename] | 跳转到包含某个文件的目录 |
| hmm | 查询所有的指令 `help` 信息 |
| findmakefile | 查询当前目录所在工程的 `Android.mk` 文件路径 |
| print_lunch_menu | 查询 `lunch` 可选的 `product` |
| printconfig | 查询各项编译变量值 |
| gettop | 查询 `Android` 源码的根目录 |
| gettargetarch | 获取 `TARGET_ARCH` 值 |
| make clean | 执行清理操作，等价于 `rm -rf out/` |
| make update-api |  更新 `API`，在 `framework API` 改动后需执行该指令，`api` 记录在目录 `frameworks/base/api` |

## Android 源代码标记和版本

简要来说，`Android` 的开发是围绕着版本系列进行的，这些版本使用美味的点心名字（按字母顺序）作为代号。

### 平台代号、版本、API 级别和 NDK 版本

为方便起见，代号与以下版本号、`API` 级别和 `NDK` 版本相对应。

![](/images/平台代号-版本-API级别和 NDK版本.png)

从 `Oreo` 开始，每个细分版本均采用新的细分版本号格式，即 `PVBB.YYMMDD.bbb`：
* `P` 部分表示平台版本代号的第一个字母，例如 `O` 表示 `Oreo`
* `V` 部分表示支持的行业。按照惯例，`P` 表示主要平台分支
* `BB` 部分表示由字母和数字组成的代码，`Google` 可通过该代码识别相应细分版本所属的确切代码分支
* `YYMMDD` 部分表示相应版本从开发分支细分出来或与开发分支同步的日期。它并不一定是细分版本的确切构建日期，`Google` 常常会在现有细分版本中增加细微的更改，并在新细分版本中重复使用与现有细分版本相同的日期代码
* `bbb` 部分表示具有相同日期代码的不同版本，从 `001` 开始

从 `Cupcake` 到 `Nougat` 的较早 `Android` 版本所用的细分版本号格式有所不同。这些 `Android` 细分版本均有一个简短的细分版本代码，以作区分，例如 `FRF85B`：
* 第一个字母代表相应版本系列的代号，例如 `F` 表示 `Froyo`
* 第二个字母是分支代码，`Google` 用它来表示细分版本所属的确切代号分支。按照惯例，`R` 表示主要版本分支
* 接下来的字母和两个数字是日期代码。字母表示季度，其中 `A` 表示 2009 年第 1 季度。因此，`F` 表示 2010 年第 2 季度。两个数字表示相应季度内的第某天，因此 `F85` 表示 2010 年 6 月 24 日
* 最后，末尾字母表示具有相同日期代码的不同版本，从 `A` 开始；但 `A` 实际上并不会显示，通常会为了简洁而省略

日期代码并不一定是某个细分版本的确切构建日期，`Google` 常常会在现有细分版本中增加细微的更改，并在新细分版本中重复使用与现有细分版本相同的日期代码。

### 源代码标记和细分版本

下表完整列出了从 `Donut` 开始的细分版本和标记。您可以从 `Android` 开发者网站下载 `Nexus` 和 `Pixel` 设备的出厂映像、二进制文件以及完整的 `OTA` 映像：

* [映像](https://developers.google.com/android/images)
* [驱动程序](https://developers.google.com/android/drivers)
* [`OTA`](https://developers.google.com/android/ota)

![](/images/源代码标记和细分版本.png)

要区分各个版本，可以使用以下命令并指定两个分支标记，以获取与每个项目相关联的更改列表：

```shell
repo forall -pc 'git log --no-merges --oneline branch-1..branch-2'
```

例如：

```shell
repo forall -pc 'git log --no-merges --oneline android-4.4.2_r2..android-4.4.2_r1'
```

要输出到文本文件，请使用以下命令：

```shell
repo forall -pc 'git log --no-merges --oneline android-4.4.2_r2..android-4.4.2_r1' > /tmp/android-4.4.2_r2-android-4.4.2_r1-diff.txt
```

## 运行编译系统

### 编译 fastboot 和 adb

如果您还没有 `fastboot` 和 `adb`，则可以使用常规编译系统来编译。请按照[编译准备工作](#编译 AOSP 源代码)中的说明操作，并将主 `make` 命令替换为以下命令：

```shell
make fastboot adb
```

### 启动进入 fastboot 模式

`fastboot` 是一种引导加载程序模式，我们可以在该模式下刷写设备。在设备冷启动过程中，可使用以下组合键进入 `fastboot` 模式：

![](/images/fastboot-按键.png)

我们还可以使用命令 `adb reboot bootloader` 直接在 `Android` 系统中重新启动进入引导加载程序，而无需使用任何组合键。

### 解锁引导加载程序

只有在引导加载程序允许的情况下，我们才可以刷写定制系统，而引导加载程序默认处于锁定状态。我们可以解锁引导加载程序，但这样做会导致系统出于保护隐私方面的考虑而删除用户数据。解锁之后，系统会清空设备上的所有数据，即应用中的个人数据以及可通过 `USB` 访问的共享数据（包括照片和影片）。请务必先备份设备上的所有重要文件，然后再尝试解锁引导加载程序。

我们只需解锁引导加载程序一次即可，并可视需要将其重新锁定。

#### 解锁新款设备

自 2014 年以来发布的所有 `Nexus` 和 `Pixel` 设备（从 `Nexus 6` 和 `Nexus 9` 开始）都内置有恢复出厂设置保护功能，需要通过多个步骤才能解锁引导加载程序。

1. 要在设备上启用 `OEM` 解锁功能，请执行以下操作：
  * 在`设置`中，点按关于手机，然后点按版本号七次
  * 当看到 `您已处于开发者模式` 这条消息后，点按返回按钮
  * 点按开发者选项，然后启用 `OEM` 解锁和 `USB` 调试（如果 `OEM` 解锁处于停用状态，请连接到互联网，以便设备可以至少签到一次。如果 `OEM` 解锁仍处于停用状态，则说明您的设备可能已被运营商锁定 `SIM` 卡，系统无法解锁引导加载程序）
2. 重新启动进入引导加载程序，然后使用 `fastboot` 解锁：
  * 对于新款设备（2015 年及之后发布的设备）：
  ```sehll
  fastboot flashing unlock
  ```
* 对于老款设备（2014 年及之前发布的设备）：
  ```shell
  fastboot oem unlock
  ```
3. 在屏幕上确认解锁


#### 重新锁定引导加载程序

要重新锁定引导加载程序，请执行以下命令：

* 对于新款设备（2015 年及之后发布的设备）：
```shell
fastboot flashing lock
```

* 对于老款设备（2014 年及之前发布的设备）：
```shell
fastboot oem lock
```

### 使用刷写解锁

`getFlashLockState()` 系统 `API` 会传输引导加载程序状态，`PersistentDataBlockManager.getFlashLockState()` 系统 `API` 会返回兼容设备上引导加载程序的锁定状态:

![](/images/使用刷写解锁.png)

制造商应测试由已锁定/解锁引导加载程序的设备返回的值。例如，`Android` 开源项目 (`AOSP`) 包含根据 `ro.boot.flash.locked` 启动属性返回值的参考实现。示例代码位于以下目录中：

* `frameworks/base/services/core/java/com/android/server/PersistentDataBlockService.java`
* `frameworks/base/core/java/android/service/persistentdata/PersistentDataBlockManager.java`

### 选择设备编译系统

`lunch` 菜单中提供了建议的设备编译系统，在不使用任何参数的情况下运行 `lunch` 命令即可查看。 我们可以从 [developers.google.com](developers.google.com) 下载 `Nexus` 设备的出厂映像和二进制文件。请参阅[设备二进制文件](https://source.android.com/setup/build/requirements#binaries)进行下载。有关详情以及其他资源，请参阅[下载专有二进制文件](https://source.android.com/setup/build/building.html#obtaining-proprietary-binaries)。

![](/images/选择设备编译系统.png)

### 刷写设备

我们可以通过运行一个命令来刷写整个 `Android` 系统；这样做可验证正在刷写的系统与已安装的引导加载程序和无线通信模块的驱动程序是否兼容，还可以将启动、恢复和系统分区一起写入，然后重新启动系统。与 `fastboot oem unlock` 类似，刷写设备也会清空所有用户数据。
要刷写设备，请执行以下操作：

1. 在启动时按住相应的组合键或使用以下命令使设备进入 `fastboot` 模式：
```shell
adb reboot bootloader
```

2. 在设备处于 `fastboot` 模式后，运行以下命令：
```shell
fastboot flashall -w
```
`-w` 选项会清除设备上的 `/data` 分区；该选项在您第一次刷写特定设备时非常有用，但在其他情况下则没必要使用。

## Repo 和 Git 常用命令

在 `Android` 代码库中使用 `Git` 和 `Repo` 会涉及到执行以下常见任务：

![](/images/git-repo.png)
