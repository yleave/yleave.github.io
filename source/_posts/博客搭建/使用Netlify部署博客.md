---
title: 使用Netlify部署博客
date: 2020-09-12 11:54:47
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/Netlify.jpg
banner_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/15.png
categories:
    - 博客搭建
tags:
    - Netlify
    - 自动化部署
---

&emsp;&emsp;原先博客是部署在 githubPages 上的，稍微设置一下就能实现自动化部署和启用 https，还是蛮方便的，但是使用国内网络访问 githubPages 上部署的网站速度太慢了，体验很差，因此，搜了下解决方案，发现了 `Netlify ` 这个一站式自动化部署网站的平台，部署网站的过程很简单， githubPages 上的支持和不支持的功能 Netlify 上都有，最重要的是听说使用 Netlify 部署的网站，国内访问深度会比 githubPages 快多了。因此也捣鼓了一番，使用 Netlify 重新部署了网站。


&emsp;&emsp;但是。。。部署完后使用国内网络访问网站，感觉访问速度没什么变化。。。去站长之家测试了下速度，唉


**我理想中的变化：**

<img src="https://i.loli.net/2020/09/12/QFlGUA7px2aDmMN.png" alt="image-20200912114349985" style="zoom:80%;" />

**残酷的现实：**

<img src="https://i.loli.net/2020/09/12/cUhBsSGPVHZkApW.png" alt="image-20200912114441879" style="zoom:40%;" /> <img src="https://i.loli.net/2020/09/12/aWpBJ7IDlcXvFCx.png" alt="image-20200912114518298" style="zoom:40%;" />



&emsp;&emsp;总之记录一下部署的过程，不过可能还是会选用 githubPages 了，关于网站加速，后面再看了。



## github 项目部署

&emsp;&emsp;首先，进入 [Netlify](https://app.netlify.com/teams/yleave/sites) 官网，选择 github 账号登陆。

&emsp;&emsp;然后点击创建新网站：

<img src="https://i.loli.net/2020/09/12/UNgWzbrBpMFin7E.png" alt="image-20200912024233068" style="zoom:80%;" />

&emsp;&emsp;选择 github 

<img src="https://i.loli.net/2020/09/12/KgbkJuPSoFNQaB8.png" alt="image-20200912024310535" style="zoom:80%;" />

&emsp;&emsp;然后进入第二步，选择网站的项目仓库：

<img src="https://i.loli.net/2020/09/12/PGQSKXUyLz2clhF.png" alt="image-20200912024635089" style="zoom:80%;" />



&emsp;&emsp;然后选择部署的分支及打包命令与发布目录，我是使用了 hexo ，因此打包命令是 `hexo generate`：

<img src="https://i.loli.net/2020/09/12/dvts5UECVeaTRgQ.png" alt="image-20200912024803284" style="zoom: 67%;" />

&emsp;&emsp;点击 `Deploy` 按钮后，会自动进行部署，部署完成后，就能通过它给你的域名访问你的网站了：

<img src="https://i.loli.net/2020/09/12/oQVNMXCEUmgnSey.png" alt="image-20200912025256424" style="zoom: 67%;" />

&emsp;&emsp;网站名称默认会是一连串字符，可以在 `Site Setting` 里更改：

<img src="https://i.loli.net/2020/09/12/gLuZcst8EAmrB5D.png" alt="image-20200912025420094" style="zoom:67%;" />



## 自定义域名

&emsp;&emsp;Netify 的初始域名会是 `xxx.netify.app`

&emsp;&emsp;需要更改域名的话需要去申请域名，解析 DNS，有了之前的[经验](https://yleave.netlify.app/2020/09/11/%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%B9%B6%E6%B7%BB%E5%8A%A0ssl/) ，这步就很快了：

&emsp;&emsp;首先使用 [dns查询工具](http://tool.chinaz.com/dns) 查询网站的 ip：



<img src="https://i.loli.net/2020/09/12/cD7wf495nxWISud.png" alt="image-20200912025833859" style="zoom:67%;" />

&emsp;&emsp;然后去阿里云的 dns 解析平台解析这几条IP，并添加 `CNAME` 记录：

<img src="https://i.loli.net/2020/09/12/9mdcMTJOi61vRGK.png" alt="image-20200912030148298" style="zoom:80%;" />

&emsp;&emsp;操作完成后，回到 Neltify，点击 `Add custom domain` 添加你的域名：

<img src="https://i.loli.net/2020/09/12/ftZqQe9b6z5Juxm.png" alt="image-20200912030601733" style="zoom:80%;" />

&emsp;&emsp;然后是为网站添加 SSL，我[之前](https://www.yleave.top/2020/09/11/%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%B9%B6%E6%B7%BB%E5%8A%A0ssl/#%E7%BB%99%E5%8D%9A%E5%AE%A2%E7%BD%91%E7%AB%99%E6%B7%BB%E5%8A%A0-SSL)已经申请过 SSL 证书了，因此点击添加证书按钮，将证书的内容上传：

<img src="https://i.loli.net/2020/09/12/q1xW8fHOP6VXr2t.png" alt="image-20200912030811357" style="zoom:67%;" />

<img src="https://i.loli.net/2020/09/12/5sAOkrjQXTwGNPZ.png" alt="image-20200912030928206" style="zoom: 67%;" />

&emsp;&emsp;若没有申请过证书的话，Netlify 也提供了免费证书的发放服务：

&emsp;&emsp;点击 `Let's Encrypt`，完成后如下：

<img src="https://i.loli.net/2020/09/12/di6oRcH5FhTbsEQ.png" alt="image-20200912031035996" style="zoom: 67%;" />



&emsp;&emsp;至此，使用 Netlify 自动部署博客步骤就全部完成了，之后添加新内容提交后， Netlify 会自动打包发布，非常方便:

<img src="https://i.loli.net/2020/09/12/aVEbmrYw2MJtFQR.png" alt="image-20200912032343535" style="zoom: 80%;" />



---

**Ref:**

https://www.cnblogs.com/37Y37/p/12551839.html

https://zhuanlan.zhihu.com/p/77651304