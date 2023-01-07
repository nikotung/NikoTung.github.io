---
title: "利用 Letsencrypt 部署 https"
description: "得益于 Letsencrypt ，可以快速的给自己的网站加上https"
categories: "2016-11"
tags: [Letsencrypt , https]
---

得益于 [Letsnecrypt](https://letsencrypt.org/about/)，我们可以快速免费的给自己的网站加上 https。对于个人网站，个人博客来说绝对是好事一桩,只需要简单的按照文档非常简单的就能给自己加上全站https。它本身就是一个 CA,给我们提供免费证书，还能自动更新。只需要有自己的一台vps ，一个域名，就可以了。

简单来说就是：

* 生成证书
* 配置https
* 自动更新证书

三部曲，就可以拥有 https 了。


##### 生成证书

这一步在 [Letsencrypt]()文档中有非常详细的说明。它提供了一个证书机器人[certbot](https://certbot.eff.org/),可以方便一键生成证书。

以我自己的 vps ，装的是 ubuntu，为例:

	#先安装工具
	sudo apt-get install letsencrypt 

因为要验证你是否拥有这个域名，需要在服务器上配置一个路径，能够给certbot 在生成证书的过程中进行验证。需要配置一下 nginx,因为我的机器是用 nginx 做转发。

	server {
	  listen              80;
	  server_name         yourdomain.com;
	  location '/.well-known/acme-challenge' {
	  	default_type "text/plain";
	    	root        /var/www/example;
	  }

	#生成证书
	letsencrypt certonly --webroot -w /var/www/example -d yourdomain.com 

这样生成了你自己域名的证书,看到成功后的提示就知道证书存放的位置。


##### 配置https

	server {
	    listen 443 ssl;
	    ssl_certificate         /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
	    ssl_certificate_key     /etc/letsencrypt/live/yourdomain.com/privkey.pem;
	    ssl_trusted_certificate /etc/letsencrypt/live/yourdomain.com/chain.pem;
	    server_name yourdomain.com;
	    location / {
	        proxy_set_header   X-Forwarded-For $remote_addr;
	        proxy_set_header   Host $http_host;
	        proxy_pass         http://127.0.0.1:3000;
	    }

	}


然后 reload 一下 nginx 

	sudo nginx -s reload

这样就能访问 : https://yourdomain.com

也要可以配置一下自动把 http 请求转化成 https

	server {
	  listen              80;
	  listen              [::]:80;
	  server_name         yourdomain.com;
	  location '/.well-known/acme-challenge' {
	  	default_type "text/plain";
	    	root        /var/www/example;
	  }

	  location / {
	    return              301 https://$host$request_uri;
	  }
	}

##### 自动更新证书

因为 Letsencrypt 提供的证书有效期为90天，所以我们要自动更新证书，加一个定时任务。

	crontab -e 

然后输入

	@monthly cd /usr/bin/letsencrypt && ./letsencrypt renew && /etc/init.d/nginx -s reload

	