---
layout: post
title: DotNetToJScript学习
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

