---
title: Jenkins下Join插件介绍
date: 2021-01-08 17:06:48
tags: [Jenkins,Join]
---

接上一篇的内容，我们今天将介绍Jenkins下Join插件的作用。


Join插件允许在直接下游作业完成后运行作业。这样，执行可以实现并行分支或执行其他步骤，然后在完成所有并行工作后最终聚合。

## 例子
我们的构建由四个作业组成--Test，TestDown1，TestDown2，TestJoin。这些作业都将按照既定的顺序执行。

我们需要去定义作业之间的顺序关系，在**Post-build Actions**里设置“Build other projects”和“Join Trigger”以设定你的任务的下一级job以及在下一级Job完成后接着要运行的job，内容如图所示。

## 基本作业的配置
![](jenkins_post_build_actions.png)

在这个过程中，testDown和testDown2会在初始job完成后并行，并在上述下游任务完成后运行testJoin。我们也可以选择下游Job的执行权限是依据上游任务的成功与否去决定。

![](build_other_projects.png)

