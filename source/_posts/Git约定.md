---
layout: post
title: Git约定
tags: 约定
category: git
description: 定义Git的使用约定，统一开发人员规范
---


# Git约定

[TOC]

## 版本号

1. 版本号分3段，比如 A.B.C。（主版本号.次版本号.修订号）

   A 表示大版本号

   B 表示功能更新

   C 表示小修改

   修复问题但不影响API 时，递增修订号；API 保持向下兼容的新增及修改时，递增次版本号；进行不向下兼容的修改时，递增主版本号。

   ​

2. 版本号修饰词

   * 版本号与修饰词之间，用**中划线**"-"连接。

   * 测试或开发中的版本号，可以用日期做修饰

   * 当开发产品或项目时，使用该版本号约定

     > alpha: 内部版本
     > beta: 测试版
     > release：正式版发布
     > lts: 长期维护
     >
     > 示例：
     >
     > 1.0.0-alpha
     >
     > 1.0.0-alpha-20180101

   * 当开发工具模块时，使用下面的版本号约定

     > snapshot
     >
     > release
     >
     > 示例：
     >
     > 1.0.0-snapshot
     >
     > 1.0.0-release-20180101
     >

   * 版本号默认起点：0.0.1

   * 版本号发布后的软件，禁止修改，如需修改，必须以新版本发布

   * 每当主版本号递增时，次版本号和修订号“必须 MUST ”归零

   * MAVEN项目的版本号命名和Git标签的版本号尽量保持一致。对alpha版本号和beta版本号允许maven和git不同。

3. 文档参考：https://semver.org/lang/zh-CN/

## 分支

![image](https://tva1.sinaimg.cn/large/007S2YVMgy1gdzj05dq4vj30vy16cwhl.jpg)

1. 必须存在的分支：master，develop

2. 约定的分支命名：master，develop，feature，bugfix（取代hotfix命名），release。

3. 分支名修饰名，按照任务名称或者任务编号命名，用**中划线**分割

   示例：feature-user，bugfix-1101

4. release分支使用

   * 用途：更新版本号，修复bug，发布测试版本
   * release分支完成后，要合并到master上。
   * 当项目较简单时，可以没有release分支，直接将develop合并至master

5. bugfix如果是需要在master分支修复，则需要将bugfix分支同时合并到develop和master分支，并升级master版本号。

6. 当根据该Git项目生产出多个子项目时，需要将master分支检出为多个受保护的项目分支，作为子项目的主分支

7. 子项目分支可以从master分支合并稳定的新的特性功能，把master分支作为一个通用功能开发的主分支。

8. 如果master分支的历史版本需要修复，那么需要将重新检出该历史版本做一个新的保护分支，作为历史版本的主分支，在该版本上修复bug。

   历史版本主分支命名：主版本号.次版本号.x，示例：v1.0.x

9. 文档参考：http://blog.csdn.net/hherima/article/details/50386011

## 标签

1. 这里的Git标签，一般就是约定的Git版本号，尽量参考版本号的说明

2. 标签必须打在master分支或release分支

3. 标签约定用"v" + 版本号，例如：v1.0.0

   ​