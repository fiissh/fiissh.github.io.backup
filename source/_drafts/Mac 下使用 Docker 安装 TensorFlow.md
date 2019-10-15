---
title: Mac 下使用 Docker 安装 TensorFlow
permalink: install-tensorflow-by-docker
comments: true
date: 2019-03-22 17:04:25
updated: 2019-03-22 17:04:25
tags:
  - 安装 TensorFlow
  - Docker
categories:
  - TensorFlow
---
本文是 `TensorFlow` 官方文档的随手记录，详细信息还请移步 [Docker](https://www.tensorflow.org/install/docker?hl=zh-cn)。

官方 `TensorFlow Docker` 映像位于 [tensorflow/tensorflow Docker Hub](https://hub.docker.com/r/tensorflow/tensorflow/) 代码库中。映像版本按照以下格式进行标记：

| 标记	| 说明 |
| :-: | :-: |
| latest |	TensorFlow CPU 二进制映像的最新版本 |
| nightly| TensorFlow 映像的每夜版 |
| version | 指定 TensorFlow 二进制映像的版本 |
| tag-devel	| 指定的标记版本和源代码。 |
| tag-gpu	| 支持 GPU 的指定标记版本 |
| tag-py3 | 支持 Python 3 的指定标记版本 |
| tag-gpu-py3 | 支持 GPU 和 Python 3 的指定标记版本 |
| tag-devel-py3	| 支持 Python 3 的指定标记版本以及源代码 |
| tag-devel-gpu	| 支持 GPU 的指定标记版本以及源代码 |
| tag-devel-gpu-py3	| 支持 GPU 和 Python 3 的指定标记版本以及源代码 |

使用如下命令安装：

```shell
docker pull tensorflow/tensorflow
```

或者指定版本安装：

```shell
docker pull tensorflow/tensorflow:nightly-devel-gpu
```

使用如下命令启动 `TensorFlow Docker` 容器：

```shell
docker run [-it] [--rm] [-p hostPort:containerPort] tensorflow/tensorflow[:tag] [command]
```

例如：

```shell
docker run --name my-tensorflow -it -p 8888:8888 -v ~/tensorflow:/test/data tensorflow/tensorflow
```

* `-name`：创建的容器名，即 `my-tensorflow`
* `-it`：保留命令行运行
* `-p 8888:8888`：将本地的8888端口和 `http://localhost:8888/` 映射
* `-v ~/tensorflow:/test/data`:将本地的 `~/tensorflow` 挂载到容器内的 `/test/data` 下
* `tensorflow/tensorflow`：默认是 `tensorflow/tensorflow:latest`，指定使用的镜像

输入以上命令后，默认容器就被启动了，命令行显示：

```shell
[I 15:08:31.949 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[W 15:08:31.970 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 15:08:31.975 NotebookApp] Serving notebooks from local directory: /notebooks
[I 15:08:31.975 NotebookApp] 0 active kernels
[I 15:08:31.975 NotebookApp] The Jupyter Notebook is running at:
[I 15:08:31.975 NotebookApp] http://[all ip addresses on your system]:8888/?token=649d7cab1734e01db75b6c2b476ea87aa0b24dde56662a27
[I 15:08:31.975 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 15:08:31.975 NotebookApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        ；
[I 15:09:08.581 NotebookApp] 302 GET /?token=649d7cab1734e01db75b6c2b476ea87aa0b24dde56662a27 (172.17.0.1) 0.42ms
```

拷贝带 `token` 的 `URL` 在浏览器打开:

```shell
http://[p addresses on your system]:8888/?token=649d7cab1734e01db75b6c2b476ea87aa0b24dde56662a27
```

你将会看到如下页面：

![](https://user-gold-cdn.xitu.io/2018/2/24/161c694e4193e2b7?imageView2/0/w/1280/h/960/ignore-error/1)

使用如下命令关闭容器：

```shell
docker stop my-tensortflow
```

再次打开时可以使用如下命令：

```shell
docker start my-tensortflow
```

或者使用如下命令创建基于命令行的容器：

```shell
docker run -it --name bash_tensorflow tensorflow/tensorflow /bin/bash
```

使用 `start` 命令启动容器：

```shell
docker start bash_tensorflow
```

再连接上容器：

```shell
docker attach bash_tensorflow
```

关于 `Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX AVX2` 错误的解决方案：

* 如果安装的是 `CPU` 版本，在代码中加入如下代码：

```Python
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
```

* 如果安装的是 `GPU` 版本，在代码中加入如下代码：

```Python
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
```
