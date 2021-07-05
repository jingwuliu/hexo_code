---
title: Jenkins集成钉钉机器人实现在钉钉上发送任务信息
date: 2021-01-28 15:20:25
tags: [Jenkins, DingTalk]
---
 
由于工作需求，考虑使用钉钉机器人代替邮件的功能发送任务信息，所以自己也是摸索了一番。首先钉钉机器人自然是需要依托于钉钉应用的，且钉钉机器人只能在群组里以群主的身份进行创建。

## 创建钉钉机器人
在桌面钉钉上，点击左上角的头像后，点击机器人管理，选择添加自定义机器人(通过Webhook接入自定义服务)。

![](create1.png) &emsp; &emsp;![](create2.png)

接下来需要设置机器人名字，并选择添加到的群组。机器人会自动生成Webhook地址，这个我们后续会在Jenkins上使用。安全设置过程目前支持三种方式，分别是自定义关键词、加签、IP地址(段)。分别是填入自定义关键词、生成密钥、填入IP地址，并在Jenkins上进行映射的关系进行安全的保障。

![](create3.png)

创建完成后，在群里里就可以看到可爱的钉钉机器人的消息啦。

![](create4.png)

## Jenkins配置
- 钉钉插件安装。想要在Jenkins上使用钉钉机器人的功能，首先必须要安装钉钉插件。需要依次在Jenkins的首页点击**Manage Jenkins--->Manage Plugins--->Available**，搜索*DingTalk*，并进行安装，目前我是用的版本是2.4.3，我们也可以看到插件的[中文文档](https://jenkinsci.github.io/dingtalk-plugin/)。

![](config1.png)

- 安装完成后，我们可以回到首页点击**Manage Jenkins--->Configure System**下找到钉钉，填写机器人的名称，Webhook，关键字/加密的信息(上述部分信息可在生成机器人处查看)，并点击测试。显示测试成功的话,并在钉钉群组内可以看到反馈信息，就说明Jenkins和钉钉机器人间的桥梁就搭建成功啦！

![](config2.png)  &emsp;  ![](config3.png)

- 具体的针对钉钉机器人反馈的信息也可以进行自定义的设置。其中对于Freestyle Project和Pipeline都有不同的设置方法。具体也可见[钉钉机器人插件中文文档](https://jenkinsci.github.io/dingtalk-plugin/guide/getting-started.html)。