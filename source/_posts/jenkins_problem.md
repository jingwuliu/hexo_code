---
title: Jenkins+Docker+Selenium使用过程中碰到的一些问题
date: 2021-01-08 16:43:50
tags: [Docker,Dockerfile,Docker-compose]
---

由于工作上的需求，需要使用Jenkins去检测每一次的git commit行为，检测到后触发测试代码，重启docker服务并使用Selenium对界面数据进行自动化获取并比对。接下来会分享一些我在工作过程中遇到的问题及解决方案。

## Jenkins构建找不到python相关依赖
我自己用的是python进行test case编写，但是在jenkins上进行测试时，发现总是提示找不到相应的import，后来发现我们在测试机上装的python相关依赖是在用户下的，而jenkins所在另外一个组，无法调用相关python依赖包，所以出现了相关错误。
## Jenkins无法monitor仓库内submodule下的行为
工作任务需要监视一个repository及其submodule下的行为并基于git行为触发相应动作。由于是初次使用Jenkins，所以在build Job时选择的是freestyle project，并在Source Code Management下选择的Git，并填入了相应的Repository URL和Credentials，由于任务中存在Submodule，所以也在Add中选择了**Recursively update submodules**和**Use cendentials from default remote of parent repository**，其中前者递归更新子模块会帮助递归检索所有子模块；后者保证子模块可以使用来自父仓库的默认远程凭据。

![Advanced sub-modules behaviours](advanced_submodules_behaviours.png)
为了实现监视仓库行为并自动触发job任务的过程，我们在Build Triggers下选择了**Build with Bitbucket Push and Pull Request Plugin**插件，该插件使得我们的主仓库在有Git行为时Job可以实时的build任务。

![Build with Bitbucket Push and Pull Request Plugin](Build_with_Bitbucket_Push_Pull_Request_Plugin.png)

并在Bitbucket上的repository下配置了Webhooks，webhook简单来说就是callback，针对Bitbucket上的行为会触发request并将request传递给相应地址。

![Webhooks](webhooks.png)

如图所示，我们可以自由选择Webhooks需要监控的Events

![](webhooks_generation.png)

但是在实际应用中发现当Submodule上有新的行为时Job无法检测到，所以这里我们考虑建立多个Job，每个Job都会连接相应的Repository且唯一任务就是监控Repository的行为，并在监测到后利用**Post-build Actions**将任务传递到主Job上再run任务。

![](post_build_actions.png)

对于**Join Plugin**，我们将在后面的小节进行介绍。由于我们共有两个submodule，所以最终的流程类似于下面的图形。
这样我们的整个过程就实现啦。