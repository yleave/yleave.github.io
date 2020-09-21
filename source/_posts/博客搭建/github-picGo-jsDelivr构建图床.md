---
title: github+picGo+jsDelivr构建图床
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/jsDelivr.jpg
banner_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/11.png
date: 2020-09-19 01:12:35
categories:
    - 博客搭建
tags:
    - 图床
    - CDN
    - PicGo
    - jsDelivr
---

# 前言

&emsp;&emsp;不论是写博客还是记笔记，图床的选择很重要。且对于一个博客平台来说，网站中的图片、视频以及音频等资源的下载速度直接影响到了整个博客网站的加载和体验，这点我深有体会。


&emsp;&emsp;就我个人来说，最开始，我在博客中有加入音乐播放器，音频和部分图片是直接存储在博客项目里的，而文章中的一些图片则使用 **SM.MS 图床**存储，然后每次打开网站时都要等很久网站的资源才能加载出来，开始我还认为是因为博客是部署在 githubPages 上导致的网站访问缓慢，也试了一些方法如 [使用Netlify部署博客](https://yleave.top/2020/09/12/%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E4%BD%BF%E7%94%A8Netlify%E9%83%A8%E7%BD%B2%E5%8D%9A%E5%AE%A2/) 来尝试加速网站的访问，然并卵...

&emsp;&emsp;直到有一天我受不了图片的上传和加载速度的缓慢，就尝试了下经常在博客搭建教程中看到的 **七牛云图床**。结果确实很香~，我不仅将图片存储在了七牛云上，并且将音频也一并存储了。重新部署后网站的访问速度有了肉眼可见的提升。直到这时，我才直到原来之前**访问速度慢的原因不止是部署服务器的问题，和网站中的静态资源也有很大的关系！** 

&emsp;&emsp;使用 七牛云图床 前后对比：

<img src="https://i.loli.net/2020/09/18/IcYpjFWJnC6Kmft.png" alt="img" style="zoom: 40%;" /> <img src="https://i.loli.net/2020/09/18/1GJKUjIVdBLrTno.png" alt="image-20200918152333510" style="zoom:40%;" />

&emsp;&emsp;效果确实好了很多。



## 七牛云图床吐槽


&emsp;&emsp;<font color="skyblye" style="font-size:150%;font-weight:bold"> 不过!!! </font>

&emsp;&emsp;虽然七牛云有提供免费的 10G 空间，但是如果想要真正使用七牛云图床的完整服务的话，你的网站**需要备案**，也就是说需要自己租一个服务器，在上面部署网站（掏了下口袋，空空的，想想还是算了吧）。

<img src="https://i.loli.net/2020/09/18/Ei4Smw3g5lD8M2I.jpg" alt="穷" style="zoom:50%;" />

&emsp;&emsp;否则，你只能得到一个**临时域名**，也就是你你资源路径的前面部分，如 `http://xxxxx.hd-bkt.clouddn.com/1.jpg` 。然后这个临时域名会在 30 天之后回收！！这意味着 30 天后你网站中所有使用七牛云存储的静态资源都不能正常访问了！！

<img src="https://i.loli.net/2020/09/18/meyl9OXUJAZu2xr.png" alt="image-20200918153307205" style="zoom:80%;" />

&emsp;&emsp;其次，七牛云上的临时域名还是 `HTTP` 的，就算使用了备案的域名绑定，想使用 `HTTPS` 的域名也需要交费（又掏了掏口袋）。

&emsp;&emsp;你的网站就会变成这样（看着是不是很不爽）

<img src="https://i.loli.net/2020/09/18/9jygoJbrIdp4q7l.png" alt="image-20200918154227852" style="zoom:80%;" />

&emsp;&emsp;因此，果断抛弃七牛云，尝试使用 github 图床 + jsDelivr CDN加速访问！



&emsp;&emsp;好了，说了这么多废话，下面是配置过程。

# 配置 github 图床

&emsp;&emsp;首先，新建一个 github 仓库：

<img src="https://i.loli.net/2020/09/18/uk7X3qAhBJTIzNQ.png" alt="image-20200918154812090" style="zoom: 50%;" />

&emsp;&emsp;依次点击 Settings -> DeveloperSettings -> Personal access tokens ->  Generate new token



<img src="https://i.loli.net/2020/09/18/ZzsiVL1nSIbkKlH.png" alt="image-20200918155050224" style="zoom: 67%;" /> <img src="https://i.loli.net/2020/09/18/GKp6XhkMWPgnaoT.png" alt="image-20200918155149215" style="zoom:50%;" /> <img src="https://i.loli.net/2020/09/18/A6iyYX3ObQLZraI.png" alt="image-20200918155324860" style="zoom:50%;" />

&emsp;&emsp;填写描述，再勾选 repo，最后拉到底点击 Generate token。

<img src="https://i.loli.net/2020/09/18/w2dSAbWyuz6QXF8.png" alt="image-20200918155537060" style="zoom:67%;" /> 

<img src="https://i.loli.net/2020/09/18/raUQvJ4lgGb7R1y.png" alt="image-20200918155736272" style="zoom:50%;" /> 



# PicGo 配置

&emsp;&emsp;然后打开 [PicGo](https://github.com/Molunerfinn/picgo/releases) 进行配置 （PicGo 是图床管理工具，能配合种图床使用，typora 中也支持使用 PicGo 自动上传图片，使用起来不要太方便了~

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200918174221758.png" alt="image-20200918174221758" style="zoom: 67%;" />

&emsp;&emsp;配置如上。

- 仓库名：格式是 ：github用户名/图床仓库名
- 分支：master
- token：前面生成的 token
- 存储路径：自己定义，如 `img/` 的话就会在仓库下生成一个 `img` 文件夹来存储图片
- 自定义域名：在图片上传后，PicGo会按照`自定义域名+储存路径+上传的图片名`的方式生成访问链接，放到粘贴板上，因为我们要使用 jsDelivr 加速访问，所以可以设置为`https://cdn.jsdelivr.net/gh/用户名/图床仓库名` ，上传完毕后，我们就可以通过`https://cdn.jsdelivr.net/gh/用户名/图床仓库名/图片路径`加速访问我们的图片了。

# jsDelivr

&emsp;&emsp;关于 jsDelivr 是不需要配置的，对于 github ，只要符合 jsDelivr 规定的路径，且**仓库大小小于 50M ，文件不超过 20M** 的都能直接引用。

&emsp;&emsp;规定的路径格式就是上面提到的：`https://cdn.jsdelivr.net/gh/用户名/图床仓库名/图片路径`。

&emsp;&emsp;如我在 `imagehost` 仓库的 `img` 文件夹下有一张图片 `default.png` ，这样就能通过：https://cdn.jsdelivr.net/gh/yleave/imagehost/img/default.png 来访问它了。



&emsp;&emsp;当然，jsDelivr 不止能加速图片的访问，对于视频文件和音频文件也能使用同样的方法访问，不过美中不足的是仓库大小不能超过 50M。



# 其他

## Imagine

&emsp;&emsp;jsDelivr 限制了仓库大小，因此图片的压缩很重要，这里再推荐一个图片压缩工具 [Imagine](https://github.com/meowtec/Imagine) ，非常好用，压缩效果很好，1M多的图片压缩成不到 100k 的图片也基本上看不出什么差别：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200918175722716.png" alt="image-20200918175722716" style="zoom: 67%;" />

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200918175745739.png" alt="image-20200918175745739" style="zoom:67%;" />



## staticaly 与 githack

&emsp;&emsp;除了 jsDelivr 外，还有几个免费的且不限流量的CDN， `staticaly` `githack` ，它们都是全球通用的。

### staticaly

&emsp;&emsp;官网地址：[https://www.staticaly.com](https://statically.io/)

&emsp;&emsp;轻松地从GitHub / GitLab / Bitbucket等加载您的项目 没有流量限制或限制。

&emsp;&emsp;文件通过超快速全球CDN提供。 在URL（不是分支）中使用特定标记或提交哈希。

&emsp;&emsp;根据URL永久缓存文件。 除master分支外，文件在浏览器中缓存1年。 具体用法：

&emsp;&emsp;`staticaly` 的用法：进入网站，将你的资源链接贴上去就会有了可用的链接：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200919005010468.png" alt="image-20200919005010468" style="zoom: 50%;" />

### githack

**该 CDN 亲测需要接外网**

&emsp;&emsp;直接从GitHub，Bitbucket或GitLab提供原始文件

&emsp;&emsp;官网地址：[http://raw.githack.com/](https://raw.githack.com/) 具体用法和上面的`staticaly`很类似

&emsp;&emsp;同样，在网站中输入资源链接，如：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200919005149597.png" alt="image-20200919005149597" style="zoom: 80%;" />

&emsp;&emsp;有两个 URL 版本：生产模式和开发模式，亲测生产模式速度会更快些，开发模式的 URL 会更“工整” 些，以便测试。不过别看生产模式的 URL 是有一长串字符，这些只要你前面的路径没变，这些字符都是一样的，因此同样比较方便统一路径。

&emsp;&emsp;当然，最最具有吸引力的就是它没有限制仓库容量，并且是永久免费的！



---

[参考1](https://blog.csdn.net/qq_36759224/article/details/98058240)

[参考2](https://www.itrhx.com/2019/02/10/A18-free-cdn/)

[参考3](https://www.cnblogs.com/lfri/p/12212878.html)


