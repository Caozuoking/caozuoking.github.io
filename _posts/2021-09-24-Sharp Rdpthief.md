---
layout: post
title: Sharp Rdpthief

---



# Sharp Rdpthief

## 0x01

键盘记录程序通常用于捕获明文凭证。但是，很难用通用的键盘记录程序隔离密码，而且冗长的会话可能导致非常详细的输出，很难解析。

然后就有了rdp thief，通过API Hooking 从 mstsc.exe 中提取明文密码。

https://github.com/0x09AL/RdpThief

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924110524.png)

## 0x02

利用原理利用 API  HOOK相关 API 处理完 mstsc 中输入 的明文凭据后，将其窃取。 MDSec 发现负责处理用户名，密码和域 的 API 分 别 为 CredIsMarshaledCredentialW ， CryptProtectMemory 和 SspiPrepareForCredRead。

RDPthief要在用户输入账号密码前先把dll注入到mstsc进程中，找一个sharp dll注入模板
