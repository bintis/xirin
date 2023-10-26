---
notetype: feed
title: 玩转域名系列之完全Serverless邮件服务器
date: 2022-04-05
---

#Domain #Mailserver

买了域名又不知道除了写博客还能干什么？ 今天写一下如何利用免费的服务_并且使用Gmail_实现用自己的域名自定义的邮箱来收发邮件。而且不需要懂太多的技术，点点点即可。

**为什么不自建邮件服务器？**

不想写废话，网上已经说的很清楚了。

    https://poolp.org/posts/2019-08-30/you-should-not-run-your-mail-server-because-mail-is-hard/

除非公司真的很大要么项目必须要用，不然真的不建议自己搭建邮件服务器。

**那这样做有什么有好处？**

优点很多，比如可以用gmail完善的网页，有谷歌帮我们隔离垃圾邮件，还不需要很多的知识，小白也可以完成。

**前提条件**

1.  一个自己的域名。    
2.  一个cloudflare账号（免费即可）。
3.  一个Gmail账号。

**实验环境**

1.  域名： example.com
2.  自定义邮件：admin@example.com
4.  Gmail：gmail@gmail.com
    

Step1 实现收取自定义邮箱邮件的设定： 按照以下步骤来操作，有各种语言版本的翻译，就不重复造轮子了。

```
https://blog.cloudflare.com/introducing-email-routing/
```


完成这一步后，向example.com发邮件就会被转发到gmail@gmail.com。但是此时回复的话发件人依旧是gmail@gmail.com。

Step2 实现用自定义邮箱发件的设定：

  在DNS记录里添加如下 SPF记录

    v=spf1 include:spf.hanami.run include:_spf.google.com ~all


2. 获取谷歌程序认证密码

```
https://myaccount.google.com/apppasswords
```

_这一步需要账户已经启用了MFA二段验证，如果没有启用，需要先开启MFA。_

```
https://support.google.com/accounts/answer/185839?hl=zh-Hans&co=GENIE.Platform%3DDesktop
```



3. 打开Gmail添加自定义邮件账号

定位到 `设置 → 账号和导入 → 用这个地址发送邮件 → 添加其他电子邮件地址`

![](/uploads/2022-04-05-230335.png)

这里名称任意，电子邮件地址为自定义域名的邮件地址。

![](/uploads/2022-04-05-230609.png)

这里的用户名是你的gmail的用户名，密码是Step2 .2中获取到的应用密码。

4. 打开gmail 点击确认邮件或输入邮件验证码。

至此全部设定完毕，在使用Gmail发送邮件的时候，可以选择使用自定义邮箱发件。

我们实现了不启动任何邮件服务端，不需要公网ip，不需要各种奇奇怪怪端口，自定义邮箱的效果。用喜欢的id送给自己一个邮箱，岂不美哉。


参考链接：
```
https://hanami.run/docs/send_mail_with_gmail
```



***

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="クリエイティブ・コモンズ・ライセンス" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />この 作品 は <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">クリエイティブ・コモンズ 表示 - 非営利 - 改変禁止 4.0 国際 ライセンス</a>の下に提供されています。

@Bintis 著作权，不许抄。
