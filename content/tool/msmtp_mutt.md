---
title: "Msmtp + Mutt搭建邮件客户端"
date: 2013-08-17 07:36
---


# Msmtp + Mutt搭建邮件客户端 #

# 需求 #

最近在写一个脚本，需要定时给邮件发送一些备份的东西


# msmtp #

[主页](http://msmtp.sourceforge.net)

	msmtp is an SMTP client.
	In the default mode, it transmits a mail to an SMTP server which takes care of further delivery.
	To use this program with your mail user agent (MUA), create a configuration file with your mail account(s),
	and tell your MUA to call msmtp instead of /usr/sbin/sendmail.



# mutt #

[主页](http://www.mutt.org)

	Mutt is a small but very powerful text-based mail client for Unix operating systems.

# msmtp和mutt搭建邮件客户端 #

首先得了解MUA，MTA的概念（可参考鸟哥私房菜服务器篇）

* MUA(Mail User Agent):主要用户接收邮件和撰写邮件
* MTA(Mail Transfer Agent):主要用于邮件的转发，如Postfix，Sendmail, msmtp

因为Sendmail的配置很麻烦( 听说,还未验证:-( )

在网上找相关的方案，就搜到了mutt+msmtp组合


# msmtp配置 #

在用户home目录新建一个.msmtprc配置文件

	account default
	host smtp.163.com	#邮件服务器，我用的是163的
	from xxx@163.com
	auth login
	user xxx@163.com	#账号
	password *******	#密码
	logfile /var/log/msmtp.log

因为这里涉及到了密码，所以建议把配置文件的权限改为600

测试:

	>> msmtp xxx@xx.com(收件人邮箱)

然后随便输入一些字符，按Ctrl+D退出，查看收件人邮箱是否有邮件


# mutt配置 #

在用户home目录新建一个.muttrc配置文件

	set sendmail="/usr/bin/msmtp"	#这里设置调用的MTA
	set use_from=yes
	set realname="XXX"				#发件人名称
	set editor="vim"				#调用的编辑器

测试:

	>> echo 'HelloWorld' | mutt -s 'Hi, Just a test!' xxx@xx.com

# 备注 #

以上是通过网上查的资料总结，官方具体文档还未细看

待补充完善!

# Read More #

* [Gentoo文档:Mutt电子邮件快速入门指南](http://www.gentoo.org/doc/zh_cn/guide-to-mutt.xml?style=printable)
* [msmtp官方文档:Using msmtp with Mutt](http://msmtp.sourceforge.net/doc/mutt+msmtp.txt)


# History #

创建时间：2012/12/24 平安夜

最后修改：2012/12/24
