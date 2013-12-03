---
layout: post
title: "Jekyll Blog使用介绍"
description: ""
category: "web"
thumbnail: "/images/thumbnail/jekyll.jpg"
tags: ["web前端"]
---
{% include JB/setup %}

## 环境配置
1. 安装`Ruby`并`libssl-dev`包:

        $ sudo apt-get install ruby1.9.1 ruby1.9.1-dev
        $ sudo apt-get install libssl-dev
2. 安装`rubygems`:  `sudo apt-get install rubygems`
>若安装失败，请在Github上拉取rubygems源码，编译并安装

        $ git clone https://github.com/rubygems/rubygems.git
        $ cd rubygems
        $ sudo ruby setup.rb
3. 查看及配置gem源：

        $ gem sources
        # 内网环境无法使用https连接，如果gem源为https，使用http协议
        $ sudo gem sources --remove https://rubygems.org
        $ sudo gem sources -a http://rubygems.org
4. 安装`jekyll`： `$ sudo gem install jekyll --no-rdoc --no-ri`
5. 安装`rake`. 类似_GNU make_的工具：`$ sudo gem install rake`
6. 安装`rdiscount`, _Markdown_解析器：`$ sudo gem install rdiscount`

## Jekyll介绍
> _TBA, Googling yourself..._

## 本地同步站点代码、添加文章
1. 拉取代码：`git clone https://github.com/TP-Mobile/TP-Mobile.github.io.git`
2. 本地访问： 源码目录下执行`jekyll --server`，访问 <http://localhost:4000> 查看效果
