layout: post
title: Options

# Shellcode Bypass Av

## Write a ShellCode loader

C#和C代码有些区别，调用Win32Api的时候需要 DllImport 语句将任何 Win32 API 导入并链接到 C＃中，然后通过 P / Invoke 技术再次将 C 代码式参数数据类型转换为 C＃。www.pinvoke.net 

以下是使用Message的一个代码案例

```
using System; 
using System.Runtime.InteropServices; 
class MainApp 
{ 
  //通过DllImport引用user32.dll类。MessageBox来自于user32.dll类 
  DllImport"user32.dll", EntryPoint="MessageBox" 
  public static extern int MessageBoxint hWnd, String strMessage, String strCaption, uint uiType; 
  public static void Main 
  { 
    MessageBox 0, "您好，这是 PInvoke！", ".net", 0 ; 
  } 
} 
```

调用Win32 api加载shellcode时候，导入3个API

