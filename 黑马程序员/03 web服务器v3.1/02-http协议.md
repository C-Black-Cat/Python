# 02-http协议

## 1.HTTP协议简介

### 使用谷歌/火狐浏览器分析

在web应用中，服务器把网页传给浏览器，实际上就是把把网页的HTML代码发送给浏览器，让浏览器显示出来，而浏览器和服务器之间的传输协议是HTTP，所以说：

- HTTP是一种用来定义网页的文本，会HTML，就可以编写网页；
- HTTP是在网络上传输HTML的协议，用于浏览器和服务器的通信。

Chrome浏览器提供了一套完整的调试工具，非常适合web开发。

安装好浏览器后，打开 Chrome，在菜单中选择 “视图”，“开发者”，“开发者工具”，就可以显示开发者工具了。

![image-20220729112533017](D:\Typora\my_file\图片\image-20220729112533017.png)

> HTTP：超文本传输协议。
>
> HTTP是通过TCP来传输数据的，因此可以使用之前用到的网络调试助手来充当web服务器。

### 浏览器的请求和响应

**请求头（request header）** 

```
GET /favicon.ico HTTP/1.1
Host: www.baidu.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:103.0) Gecko/20100101 Firefox/103.0
Accept: image/avif,image/webp,*/*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Referer: https://www.baidu.com/
Connection: keep-alive
Cookie: BAIDUID=89E87D8E13399A3AB625C2AB817448DE:FG=1; BIDUPSID=89E87D8E13399A3AFF23EE86FE9051D9; PSTM=1659324239; BDRCVFR[Fc9oatPmwxn]=mk3SLVN4HKm; delPer=0; BD_CK_SAM=1; PSINO=5; H_PS_PSSID=36561_36752_36726_36981_36413_36843_36954_36165_36917_36569_36944_36746_26350_36862; BD_UPN=13314752; H_PS_645EC=ac47My4x8H8H0GOwqc9NnyNbt4lbX6ZrHZF5HVIUmIWQcZb1B1xSul3ySaJCHHcqJ6iS; BA_HECTOR=8120212h0l850k8h25002eft1heehqh17; BDORZ=FFFB88E999055A3F8A630C64834BD6D0; ZFY=JBwhd98ymt2niHHFZHDM7blKJmj7kRsiiXuxRb53dm8:C; baikeVisitId=3e25f65d-2e8f-44ac-b779-9042d70cc537; BD_HOME=1
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
```

>`Host`：所访问网址的主机。
>
>`User-Agent`：浏览器的标识信息。在写爬虫时一般会用到，用来模拟浏览器。

**响应头（response header）**

```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Encoding: gzip
Content-Length: 1966
Content-Type: image/x-icon
Date: Mon, 01 Aug 2022 03:24:00 GMT
Etag: "423e-5bd257db4e500"
Last-Modified: Wed, 10 Mar 2021 02:33:24 GMT
Server: Apache
Vary: Accept-Encoding,User-Agent
```







