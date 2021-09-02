---
layout: post
title: DotNetToJScript modify ShellcodeLoader
---

## 0#01

This file is part of DotNetToJScript - A tool to generate a JScript which bootstraps an arbitrary .NET Assembly and class.
Copyright (C) James Forshaw 2017

DotNetToJScript is free software: you can redistribute it and/or modifyit under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

项目地址[https://github.com/tyranid/DotNetToJScript](https://github.com/tyranid/DotNetToJScript)

```
Usage Notes:

This only works from full trust JScript(obviously), so should work in
scriptlets etc. By default it will only works if v2/v3/v3.5 is installed.
However if you specify the '-ver auto' switch when building the output it
will also work on v4+ only, however that will introduce a dependency on
WScript.Shell which you might not want.

To use this you'll need to create an assembly which targets .NET 2 (though
in most cases you can also use 3.5 as you don't tend to see .NET 2 installed
in isolation. In the assembly implement a class called TestClass which does
something you want to do in the public, parameterless constructor.
```

通过DoNetToJscript的方法把之前开发过的shellcode loader添加进去，复用前面文章提到过的shellcode xor加密的方式，试图再shellcode编码上绕过一定的AV

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210901153110.png)

然后我们编译生成ExampleAssembly.dll，再用DotNetToJscript项目编译成dll文件

command: DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4,v2 -o loader.js

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210901154546.png)

360的静态查杀过了

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210901155146.png)

尝试运行wscript loader.js ，成功上线

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210901160448.png)

查看下js的源代码，大致看下代码含义

再做一次加密混淆，替换文件把文件中的.全部替换为/

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20210901163528.png)

