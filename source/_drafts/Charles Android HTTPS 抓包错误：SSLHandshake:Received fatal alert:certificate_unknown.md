---
title: Charles Android HTTPS 抓包错误：SSLHandshake:Received fatal alert:certificate_unknown
permalink: charles-android-https-error-ertificate_unknown
comments: true
date: 2018-12-11 21:46:03
updated: 2018-12-11 21:46:03
tags:
  - certificate_unknown
  - SSLHandshake
categories:
  - 随手记录
---

正确安装`Charles` 证书之后，使用 `Charles` 抓取 `Android` 应用程序的 `https` 请求时，无法正常解析抓取的数据，报文中提示错误：`SSLHandshake: Received fatal alert: certificate_unknown`：

![charles-error.png](/images/charles-error.png)

出现该问题的主要原因是由于 `Android 7.0` 调整了系统安全策略，针对 `Android 7.0` 以及更高版本，系统不再为 `https` 请求信任由用户安装的 `CA` 证书。也就是说，即便是在系统中设置为信任证书，系统也不会信任该证书。具体详情请参考 [Changes to Trusted Certificate Authorities in Android Nougat](https://android-developers.googleblog.com/2016/07/changes-to-trusted-certificate.html) 一文。

<!--more-->
解决该问题的方案为：

* 方案一：如果 `Android` 系统中已经正确安装 `https` 证书，可在应用程序的 `AndroidManifest.xml` 文件的 `<application>` 标签中添加如下属性：

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

然后，在 `res/xml` 目录下创建 `network_security_config.xml` 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <domain-config>
          <!--请将 {domain} 替换为你要抓取的域名（例如：baidu.com）-->
        <domain includeSubdomains="true">{domain}</domain>
        <trust-anchors>
        <!--信任用户自己安装的证-->
        <certificates src="user"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

* 方案二：如果 `Android` 系统中并没有安装 `https` 证书，可在应用程序的 `AndroidManifest.xml` 文件的 `<application>` 标签中添加如下属性：

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

然后，在 `res/xml` 目录下创建 `network_security_config.xml` 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <domain-config>
          <!--请将 {domain} 替换为你要抓取的域名（例如：baidu.com）-->
        <domain includeSubdomains="true">{domain}</domain>
        <trust-anchors>
        <!--请将 {certificates_name}替换为 res/raw 目录下证书的名称，注意不需要添加扩展名 -->
        <certificates src="@raw/{certificates_name}"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

最后，在 `res/raw` 目录下放入所需要的 `https` 证书，文件名为 `{certificates_name}.pem(或者 ca 等证书扩展名)`。

`Android` 下更多网络安全性配置请移步 [Network Security Configuration](https://developer.android.com/training/articles/security-config)。

`Charles` 证书下载请移步 [Charles SSL CA Certificate installation](http://charlesproxy.com/getssl)。
