---
layout: post
title: 无文件横向移动

---

## 0x01

Sysinternals包里有一个PsExec工具，多用于内网横向移动，PsExec工具向目标主机的smb做身份验证，会访问DCE/RPC接口，PsExec会创建服务然后执行。学习模拟PsExec的话需要先对目标主机进行身份验证，然后在执行我们需要的代码，首先需要对DCE/RPC接口和服务控制的身份验证。

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924174941.png)

可以选择OpenSCMangerW API

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924175026.png)

