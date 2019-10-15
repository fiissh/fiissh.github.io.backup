---
title: CMake was unable to find a build program corresponding to Ninja 错误
permalink: cmake-error-cmake-was-unable-to-find-build-program-corresponding-to-ninja
comments: true
date: 2019-03-19 18:09:58
updated: 2019-03-19 18:09:58
tags:
  - CMake Error
  - Ninja
categories:
  - Android Exception
---

`Android Studio` 升级之后新建项目（包含 `NDK`）之后编译报错：

```shell
CMake Error: CMake was unable to find a build program corresponding to Ninja.  CMAKE_MAKE_PROGRAM is not set.  You probably need to select a different build tool.
```

造成该问题的原因是系统没有安装 `Ninja`。使用 `brew` 安装 `Ninja` 即可解决该问题：

```shell
brew install ninja
```

[StackOverflow-Ninja not found by CMake](https://stackoverflow.com/questions/38658014/ninja-not-found-by-cmake)对于该问题的解决方案是使用 `ln -s /usr/bin/ninja /usr/bin/ninja-build` 修改 `Ninja` 的引用。
