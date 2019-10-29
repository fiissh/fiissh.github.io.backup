---
title: Git Commit Message 模板
permalink: git-commit-template
comments: true
date: 2019-10-29 12:15:17
updated: 2019-10-29 12:15:17
tags:
  - Git Message 模板
categories:
  - 项目管理
---
## Commit Message 规范

每一个 `Commit Message` 应该包含一个 `header`、`body` 和 `footer`。其中，`header`、`body` 和 `footer` 之间以空行作为间隔。`header` 是必须要填写的：

* `header` 通常包含此次提交的类型：
  * `feat` 新特性
  * `fix` `bug` 修复
  * `docs` 文档改动
  * `style` 格式化
  * `refactor` 重构代码
  * `test` 添加缺失的测试, 重构测试, 不包括生产代码变动
  * `chore` 更新grunt任务等; 不包括生产代码变动
* `body` 通常包含此次改动的影响范围、改动的详细信息
* `footer` 通常包含一些需要关闭的 `issues` 信息等


## 模板文件

建议将如下内容保存到本机的某个目录，例如 `MAC` 的 `~/.git-commit-template.txt`：

```shell
# <类型>: (类型的值见下面描述) <主题> (最多50个字)

# 解释为什么要做这些改动
# |<----  请限制每行最多72个字   ---->|

# 提供相关文章和其它资源的链接和关键字
# 例如: Github issue #23

# --- 提交 结束 ---
# 类型值包含
#    feat (新特性)
#    fix (bug修复)
#    docs (文档改动)
#    style (格式化, 缺失分号等; 不包括生产代码变动)
#    refactor (重构代码)
#    test (添加缺失的测试, 重构测试, 不包括生产代码变动)
#    chore (更新grunt任务等; 不包括生产代码变动)
# --------------------
# 注意
#    主题和内容以一个空行分隔
#    主题限制为最大50个字
#    主题行大写
#    主题行结束不用标点
#    主题行使用祈使名
#    内容每行72个字
#    内容用于解释为什么和是什么,而不是怎么做
#    内容多行时以'-'分隔
# --------------------
```

## 使用模板

使用如下命令将模板文件指定为 `Git` 的 `commit.template`：

```shell
git config --global commit.template <.git-commit-template.txt file path>
```

例如：

```shell
git config --global commit.template ~/.git-commit-template.txt
```

## 使用插件

对于 `Android Studio` 或者 `IDEA` 来说，可以使用 [Git Commit Template](https://plugins.jetbrains.com/plugin/9861-git-commit-template/) 插件，在图形化 `Commit` 界面使用该插件生成标准 `Commit Message`。
