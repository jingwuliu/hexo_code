---
title: 持续集成之Jenkins插件 Multiple SCMs Plugin搭配使用
date: 2021-01-11 09:45:34
tags: [Jenkins,Multiple SCMs]
---

## 背景
由于项目需要，我们的代码被分为了多个版本库进行管理，但我们希望使用一个Job去同时监控多个版本库，最终我们找到了它：[Multiple SCMs Plugin](https://plugins.jenkins.io/multiple-scms/)，该插件能实现以下目标：

- 同时监测多个版本库，其中有一个或多个版本库有新的提交，就能自动触发新的构建。

- 同时监测的多个版本库可以来自不同的代码管理器，实现混搭，如Github，Bitbucket，Mercurial等。

## 插件使用
插件可以在Manage Jenkins--Manage Plugins里搜索到并进行下载安装。进入Jenkins Job后选择Multiple SCMs，会出现Add SCMs，我们在下拉框中可以选择想要添加的版本库类型。
![Source_Code_Management](source_code_management.png)

在**Build Triggers**中选择上“Build when a change is pushed to BitBucket”和“Build with BitBucket Push and Pull Request Plugin”，并在对应的位置填入需要监测的版本库位置和分支。在Triggers中同样可以加入多个版本库地址进行行为监控。

![Build_Triggers](build_triggers.png)

最后在**Build**中选择Execute Shell加入需要执行的命令行内容，这样完整的监控多个版本库并实时CI的过程就基本完成了。

![Build](build.png)