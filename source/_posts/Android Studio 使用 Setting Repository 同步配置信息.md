---
title: Android Studio 使用 Setting Repository 同步配置信息
permalink: android-studio-setting-repository
comments: true
date: 2019-11-06 10:56:51
updated: 2019-11-06 10:56:51
tags:
  - Setting Repository
categories:
  - Android Studio
---

`Setting Repository` 是 `IntelliJ IDEA（Android Studio）` 的一项配置托管服务，我们可以通过在 `Github` 或者其他 `Git` 服务上创建一个 `Git Repository` 的方式统一托管、同步配置信息：

* 在 `Github` 创建一个代码仓库（例如： `android-studio-setting-repository`）
* 在 `Github` 上创建一个 `Personal access tokens`（保留 `read` 和 `write` 相关权限即可）
* 将版本库地址填入 `Android Studio` 的 `File/Setting Repository/Upstream URL`（尽量使用 `https` 而不是 `ssh` 的方式）
* 点击 `Overwrite Remote` 完成配置信息的提交

> `Overwrite Remote` 是将本地配置信息写入到远程仓库，`Overwrite Load` 是将远程仓库的信息写入到本地，`Merge` 则是合并两者之间的差异。

我的 `Android Studio` 配置仓库为 [fiissh/android-studio-setting-repository](https://github.com/fiissh/android-studio-setting-repository)。
