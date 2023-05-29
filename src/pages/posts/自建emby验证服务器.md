---
layout: '../../layouts/MarkdownPost.astro'
title: '自建Emby验证服务器'
pubDate: 2023-05-27
description: '自建Emby验证服务器'
author: 'ZhJy'
tags: ["分享","教程"] 
theme: 'light'
featured: true
cover:
    url: ''
    square: ''
    alt: 'cover'
meta:
 - name: author
   content: ZhJy
 - name: keywords
   content: key3, key4

keywords: 教程, Emby!
---

# 自建Emby验证服务器（实现白嫖）

群晖默认自带nginx服务，通过增加nginx反代配置，实现将Emby的验证劫持到本地，骗过emby服务器，从而解锁硬解功能，实现视频播放硬件解码，降低服务器CPU压力。

注：我的Emby服务器是DOCKER部署。

### 1.配置nginx

新建一个nginx配置文件，命名随意（例如emby.conf），在配置文件中写入下面的配置信息：

```
server {
 listen 443 ssl;
 server_name mb3admin.com;
 ssl_certificate /volume6/web/mb3admin.com/mb3admin.com.cert.pem;
 ssl_certificate_key /volume6/web/mb3admin.com/mb3admin.com.key.pem;
 ssl_session_timeout 5m;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
 ssl_prefer_server_ciphers on;
 location = /webdefault/images/logo.jpg {
 alias /usr/syno/share/nginx/logo.jpg;
 }

 location @error_page {
 root /usr/syno/share/nginx;
 rewrite (.*) /error.html break;
 }

 location ^~ /.well-known/acme-challenge {
 root /var/lib/letsencrypt;
 default_type text/plain;
 }

 location / {
 rewrite ^ / redirect;
 }

 location ~ ^/$ {
 rewrite / https://$host:5001/ redirect;
 }

 add_header Access-Control-Allow-Origin *;
 add_header Access-Control-Allow-Headers *;
 add_header Access-Control-Allow-Method *;
 add_header Access-Control-Allow-Credentials true;
 location /admin/service/registration/validateDevice {
 default_type application/json;
 return 200 '{"cacheExpirationDays": 7,"message": "Device Valid","resultCode": "GOOD"}';
 }

 location /admin/service/registration/validate {
 default_type application/json;
 return 200 '{"featId":"","registered":true,"expDate":"2099-01-01","key":""}';
 }

 location /admin/service/registration/getStatus {
 default_type application/json;
 return 200 '{"deviceStatus":"","planType":"","subscriptions":{}}';
 }
 }
```

将文件保存到指定位置，通过软链接到/etc/nginx/conf.d/目录下

```shell
ln -s /volume/XX/XX/emby.conf /etc/nginx/conf.d/emby.conf
```

### 2.申请证书

到[https://www.gmcert.org/subForm](https://www.gmcert.org/subForm)这个网站申请申请签发证书，参考下面填写：

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292103557.webp)

一个是加密算法选 RSA, 密钥长度至少选 2048, 然后除主题名称为`mb3admin.com`之外其他的按照规则随意填写。点开高级选项：

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292105504.webp)

然后勾选 `自动包含CA证书链` ，最后是证书有效天数，写 `824` 天即可。下载生成的证书，同时也将CA证书下载起来：

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292126065.webp)

将 `mb3admin.com.key.pem` 和 `mb3admin.com.cert.pem`文件上传到第一步配置文件中目录位置，我这里是/volume6/web/mb3admin.com/。

### 3.重启nginx服务

测试nginx配置文件是否正常

```shell
nginx -t
```

重载nginx配置文件

```shell
nginx -s reload
```

### 4.将证书文件追加到Emby服务器

```shell
cat xx/mb3admin.com.cert.pem >> /etc/ssl/certs/ca-certificates.crt
```

### 5.修改DNS或者HOSTS文件

可以在本地电脑中修改HOSTS文件，将mb3admin.com指向群晖IP地址，也可以直接在路由器中直接进行域名劫持，将mb3admin.com指向群晖IP

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292124432.webp)

### 6.浏览器添加CA证书

将CA证书添加到本地电脑：

将GMCert_RSACA01.cert.pem 文件名改为GMCert_RSACA01.cer，双击安装

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292131273.webp)

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292132905.webp)

然后在浏览器中打开下面两个浏览器，验证是否成功：

[https://mb3admin.com/admin/service/registration/validateDevice](https://mb3admin.com/admin/service/registration/validateDevice)

[https://mb3admin.com/admin/service/registration/validateDevice/666](https://mb3admin.com/admin/service/registration/validateDevice/666)

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292133083.webp)

### 7.在Emby服务器中输入秘钥

管理Emby Server-Emby Premiere中随便输入一个秘钥，见证奇迹：

![|inline](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305292136890.webp)



