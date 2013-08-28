---
layout: post
title: "heroku数据库配置"
description: "Heroku"
category: "heroku"
tags: ["heroku"]
---
{% include JB/setup %}

今天部署heroku的时候遇到的问题，记录下来。作为下次的参考
参照Ruby-china的上面的`heroku push`

```ruby

heroku plugins:install https://github.com/ddollar/heroku-push
 heroku push
 ```
 这样可以满足`push`上了，今天我遇到是

 ```ruby
 Running: rake assets:precompile     
 rake aborted!     
 could not connect to server: Connection refused     
 Is the server running on host "127.0.0.1" and accepting     
 TCP/IP connections on port 5432?
 ```
 解决的方法的是在`config/appliaction.rb`下面加入
 `config.assets.initialize_on_precompile = false`

 重新提交部署,又出现

 ```ruby

 rake aborted! 
 Invalid DATABASE_URL (erb):9:in rescue in <main> (erb):6:in <main>)
 ```

这个有点麻烦，搞了很长时间。


```ruby
heroku addons:add heroku-postgresql:dev --app yehyork(我的数据库名称)
#运行heroku config --app yehyork
	HEROKU_POSTGRESQL_IVORY_URL:	
postgres://ptrbsavpjbzrji:LgSxGzciRG7g1O8n0bLdpSiPyn@ec2-54-221-229-7.compute-1.amazonaws.com:5432/d928u2osvkvp10
#配置DATABASE_URL
#将刚刚得到的postgres://XXX复制
heroku config:set DATABASE_URL=postgresql://XXX
```
再次运行`heroku run rake db:migrate`
成功





