---
title: hexo 双线部署及seo优化
date: 2017-10-27 10:16:39
tags: hexo
categories: 技术分享
---

### 前言

断断续续大概半个月，终于把个人博客搭建好，整个过程虽然简单，但是也有很多”坑“。在查询资料的过程中，阅读的博文质量参差不齐，因此想细致的写出我搭建的过程，供大家参考。文章重点在于双线部署及seo，最开始的部署和搭建将简要略过。

<!-- more -->

### 安装

官方文档永远是最清楚正确的

https://hexo.io/zh-cn/docs/index.html

### 主题

选择hexo的很大原因是因为其主题和插件很多，而且配置方便，这里选择的主题是大名鼎鼎的**next**，强烈推荐使用。next的插件配置非常便捷，而且设计符合大众审美。

配置过程详见官方文档

http://theme-next.iissnan.com/

### 双线部署

由于大部分人使用hexo都是将其部署在github上，省去了服务器的钱。由于“国情”原因，github的访问速度较慢，所以才有了双线部署得必要性。将国内访问流量导向coding（国内类似github的网站），国外流量导向github，从而提高访问质量。

#### 域名申请

在部署之前，我们必须申请一个域名，便宜的域名1元/年，我的域名是在阿里云买的。进入阿里云官网，点击图片中的域名注册，根据自己的需求选择想要的域名。我的域名为yinjianwen.site，在部署得时候我将博客部署在该域名的二级域名blog下，因此以后访问地址为**blog.yinjianwen.site**。当然你也可以直接部署在主站。

![](https://ws2.sinaimg.cn/large/005PPQ5Ily1g0ympyj8vyj311h0it0ux.jpg)

#### coding

进入官网[coding](https://coding.net/)，注册账号并登录。配置好SSH公钥，这里默认你会使用git，如果不会，可以查询相关资料。

新建一个项目，名字格式为yourname.coding.me，其中yourname即为你的用户名。

![](https://ws2.sinaimg.cn/large/005PPQ5Ily1g0ympz924yj31140isq4j.jpg)

选择该项目，进入代码->pages服务，绑定你之前注册的域名。

![](https://ws3.sinaimg.cn/large/005PPQ5Ily1g0ymq0auwpj311e0h8409.jpg)

#### github

与上述过程类似，登陆之后，点击页面右上角的加号，选择New repository，在Repository name下填写yourname.github.io，其中yourname即为你的用户名。

![](https://wx1.sinaimg.cn/large/005PPQ5Ily1g0ymq132h5j311f0irgmx.jpg)

#### 域名解析配置

进入阿里云官网，在控制台选择域名服务，进入解析页面。然后即可进行配置。

![](https://wx2.sinaimg.cn/large/005PPQ5Ily1g0ymq25iiej311b0iuq54.jpg)

配置的意思是，blog子域名下的世界流量（国外）访问yinxiaojian.github.io，而国内流量访问yinxiaojian.coding.me。

#### hexo配置

最后我们需要在本地的hexo项目中进行相关配置，打开主项目下的_config.yml文件，在其中任意位置添加如下代码

```c++
deploy:
  type: git
  repo: 
    github: https://github.com/yinxiaojian/yinxiaojian.github.io.git,master
    coding: https://coding.net/yinxiaojian/yinxiaojian.coding.me.git,master
```

将其中的yinxiaojian修改为你的coding和github用户名，这行代码的意思是在你通过本地```hexo deploy```指令时，会将本地hexo同步到这两个网站。

之后需要在项目->source中添加文件CNAME，注意文件名大写且没有后缀。文件中写入你申请的域名，比如我的文件中写入的是

```blog.yinjianwen.site```

实际上由于github不想coding一样可以自己绑定域名，因此我们需要自己上传CNAME文件。

#### 部署

在上述操作完成后，输入以上命令，将本地项目推送到github和coding，双线部署完成。

```C++
hexo clean
hexo generate
hexo deploy
```

