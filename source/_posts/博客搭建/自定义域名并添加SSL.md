---
title: 自定义域名并填加SSL
date: 2020-09-11 17:33:34
categories: 
    - 博客搭建
tags:
    - 自定义域名
    - SSL
---

&emsp;&emsp;本博客是部署在 Github Pages 上的，因此访问时域名为： `yleave.github.io` ，不过还是感觉太长了，于是乎捣鼓了两天，给博客加了个域名并添加了SSL证书，下面记录了一些捣鼓经验。

# 自定义域名

&emsp;&emsp;前置条件：一个已部署在 github 上的博客



&emsp;&emsp;首先，需要注册一个域名，我是在[阿里云](https://wanwang.aliyun.com/domain/?spm=5176.100251.111252.21.72014f15uvzIEz)上注册的，首年 9 元，续费 27 元，价格还可以接受，步骤也不麻烦。

&emsp;&emsp;第一次注册的需要实名认证，而名认证审核时间大概半天就能完成。

## 域名解析

&emsp;&emsp;域名注册完成后，进入解析页面，点击添加记录：

<img src="https://i.loli.net/2020/09/11/izd5jS7e2TVDgM4.png" alt="image-20200911163555583" style="zoom:80%;" />

&emsp;&emsp;先获取你的博客 IP 地址，可以使用命令行窗口 `ping` 一下：`ping name.github.io`  :

<img src="https://i.loli.net/2020/09/11/dUDfhNnM5B6xTtL.png" alt="image-20200911163939089" style="zoom:80%;" />

&emsp;&emsp;或是去[网站](http://tool.chinaz.com/dns)查一下：

<img src="https://i.loli.net/2020/09/11/rp169ZtWCVNwMfD.png" alt="image-20200911164127950" style="zoom:80%;" />



&emsp;&emsp;这几个其实都是 github 的 IP 地址，应该所有在 github 上部署的博客都是这几个地址。



&emsp;&emsp;可以只选择一个 ip 进行解析，不过我这边将这四个地址都解析了：

<img src="https://i.loli.net/2020/09/11/iTPvBhr7J8nEmV4.png" alt="image-20200911164331299" style="zoom: 67%;" />

<img src="https://i.loli.net/2020/09/11/PGYWM2OEjw7ycRk.png" alt="image-20200911164419343" style="zoom:80%;" />

&emsp;&emsp;然后解析一个 `CNAME` 类型的记录：

<img src="https://i.loli.net/2020/09/11/zbDgrJtNh7d2uKL.png" alt="image-20200911164528656" style="zoom: 67%;" />



&emsp;&emsp;最后，在博客所在项目的 `source` 文件夹下新建一个 `CNAME` 文件，写入申请的域名：

<img src="https://i.loli.net/2020/09/11/buEdOwZFxce8jQz.png" alt="image-20200911164738664" style="zoom:80%;" />

&emsp;&emsp;不过这一步不知道它的作用是什么，但看其他教程都有这步，就加上去吧。



&emsp;&emsp;到这里，域名解析步骤就完成了。

## github 绑定

&emsp;&emsp;这个步骤中将解析的域名与 github 上的博客项目绑定起来。



&emsp;&emsp;先进入博客项目的 `Settings` 中： 


<img src="https://i.loli.net/2020/09/11/rEj2z1pJh76ctsg.png" alt="image-20200911165059055" style="zoom:80%;" />


&emsp;&emsp;拉到底部，在 `Custom domain` 栏中填入申请的域名：

<img src="https://i.loli.net/2020/09/11/3c5nN9RMHSADZbX.png" alt="image-20200911165140522" style="zoom:67%;" />

&emsp;&emsp;到这一步，自定义域名的步骤就完成了，但由于我是第一次注册，还需等待实名认证完成，审核完成后就能通过 `www.yleave.top` 和 `yleave.top` 访问了。

<img src="https://i.loli.net/2020/09/11/tSsPnkwdz59qO4F.png" alt="image-20200911165328548" style="zoom:80%;" />



# 给博客网站添加 SSL

<img src="https://i.loli.net/2020/09/11/WX46dDuBK8OvMig.png" alt="image-20200911165528080" style="zoom:80%;" />


&emsp;&emsp;关于 SSL 和 HTTPS 的介绍这里就不说明了，这篇[文章](https://yq.aliyun.com/articles/721195)有一些简介可以看看。



## 申请免费SSL证书

&emsp;&emsp;证书这边我是在 [腾讯云](https://console.qcloud.com/ssl)申请免费的 SSL 证书，不过只有一年免费时间。

<img src="https://i.loli.net/2020/09/11/qwMHnYxtACdPuiW.png" alt="image-20200911170245686" style="zoom:80%;" />

&emsp;&emsp;阿里云应该也能申请，不过我没找到，因此去腾讯云申请了，但是证书申请和域名申请不在同一个网站的话，后面上传证书时会稍微麻烦一些，不过也还好。

&emsp;&emsp;申请步骤很简单，只需要填写一个表格并实名认证就行。

<img src="https://i.loli.net/2020/09/11/QWJShizDbOuTClr.png" alt="image-20200911170359563" style="zoom:80%;" />

&emsp;&emsp;申请完成后需要进行 DNS 验证：

<img src="https://i.loli.net/2020/09/11/IQqkwEmhx62lcav.png" alt="image-20200911171152620" style="zoom:80%;" />

&emsp;&emsp;因此再回到阿里云的 DNS 解析页面，将上表中出现的字段填入解析记录中：

<img src="https://i.loli.net/2020/09/11/QGcaB6q4i9dOPVL.png" alt="image-20200911171242463" style="zoom:80%;" />

然后，等待[腾讯云](https://console.cloud.tencent.com/ssl/detail/gLRDXe3L)那边的 DNS 验证

<img src="https://i.loli.net/2020/09/11/YrRZfFmPxEBCqze.png" alt="image-20200911171319884" style="zoom:80%;" />

&emsp;&emsp;过了大概五分钟，腾讯云发来短信，提示 DNS 解析成功。

<img src="https://i.loli.net/2020/09/11/7MBHXatQUG5Z4A6.png" alt="image-20200911171410380" style="zoom:67%;" />

&emsp;&emsp;证书下载下来是一个压缩包，里面是这样子的，我估摸着是不同代理的证书：

<img src="https://i.loli.net/2020/09/11/X6adrQ8wRF79fU2.png" alt="image-20200911171553060" style="zoom:80%;" />

## 上传SSL证书

&emsp;&emsp;在腾讯云上申请完 SSL 证书后，需要将其上传到域名申请的地方，也就是阿里云的[证书管理中心](https://yundunnext.console.aliyun.com/?p=casnext#/overview/cn-hangzhou)：

<img src="https://i.loli.net/2020/09/11/e8iPSIDBodCVhlq.png" alt="image-20200911171741501" style="zoom:80%;" />

&emsp;&emsp;证书名称可以随便填，证书文件就是 `.crt` 的文件中的内容，一个证书文件对应了一个私钥，如 Apache 目录下， `_www.yleave.top.crt` 就是证书文件，`.key` 就是私钥，填入这两个就好了：

<img src="https://i.loli.net/2020/09/11/8Ns2I6nyMZFhrAK.png" alt="image-20200911171826131" style="zoom:80%;" />

<img src="https://i.loli.net/2020/09/11/DvlAJ6pV9KyXGdf.png" alt="image-20200911171859459" style="zoom:80%;" />

&emsp;&emsp;我这边先是上传了两个证书：

![image-20200911171934657](https://i.loli.net/2020/09/11/KFQq7to6xyhvfXC.png)

&emsp;&emsp;上传完后点击部署按钮：

<img src="https://i.loli.net/2020/09/11/eJ1HLE5PRY3mhqO.png" alt="image-20200911172010043" style="zoom:80%;" />

&emsp;&emsp;全选，点击部署：

<img src="https://i.loli.net/2020/09/11/rgBiSMDbJIOoNtp.png" alt="image-20200911172049141" style="zoom: 80%;" />

&emsp;&emsp;不过刚部署完后访问网站还是 `http` 类型的，当时感觉是不是自己哪错了（后来发现应该是要等一会儿才能生效），于是看[一个博文](https://www.cnblogs.com/sslwork/p/5984167.html) 里说还需要部署中间证书，于是按照上面的方法又上传部署了一个带中间证书内容的证书，但刚部署完仍未生效。



&emsp;&emsp;不过过了一会儿，正当我沮丧找其他博文参考时，又访问了一下发现证书好像生效了：

<img src="https://i.loli.net/2020/09/11/LlnEkSad2G4eMVP.png" alt="image-20200911172535154" style="zoom:80%;" />

&emsp;&emsp;最后，在浏览其他博文时，发现 github 上还有一个 `Enforce HTTPS` 选项，暂时不清楚作用，总之先勾选上：

<img src="https://i.loli.net/2020/09/11/MWEApjaytkvcVCT.png" alt="image-20200911172829156" style="zoom:67%;" />



&emsp;&emsp;捣鼓了两天，虽然过程中对一些步骤的作用还不太清楚，但最后成功完成，时间还是没白费的