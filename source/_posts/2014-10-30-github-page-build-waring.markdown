---
layout: post
title: "Github Page build waring"
date: 2014-10-30 10:24
comments: true
categories: [Github]
---

最近更新网站的时候都会收到一封Github的Warning邮件。意思是年初Github更新了自己的Page服务，更新了CDN，以防DoS攻击什么的，需要重新设置一下DNS。由于不影响网站的一般访问，之前一直懒得管,今天花了点时间重新弄了下，记录一下。<!--more--> 

##Step1.
首先检查一下source/CNAME文件中的内容是否是自己的域名：

``` bash
$ cat source/CNAME
bravematrix.org

$
```

一般没有问题，不对的话就修改过来，之后记得commit。我这里没问题，check下一步。

##Step2.
我的域名服务用的是GoDaddy(不要吐槽！)。进入My Account，点击DOMAINS的Launch：

![Step2](/images/postimgs/DNS_step2.png "Domains Launch")

进入到域名管理页面Domain Details，点击DNS Zone File，首先修改A(Host)记录的Points To，
Github提供了两个IP地址：

``` bash
>192.30.252.153
>192.30.252.154
```

随便选择一个填上。

另外，还要将CName(Alias)中的www Host重新Points To`username.github.io`就行了。

![Step3](/images/postimgs/DNS_step3.png "Domains Details")

##Issue: The zone file is unavailable because the domain's set nameservers do not belong to this registrar.
我一开始就遇到这个问题。出现这个问题是因为你用的名字服务不是GoDaddy默认的，比如我之前用的就是DNSPod的两个地址：f1g1ns1.dnspod.net、f1g1ns2.dnspod.net。
要添加记录就需要重新把Nameservers改回默认的：

![Issue](/images/postimgs/DNS_issue.png "zone file unavailable")



