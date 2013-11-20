---
layout: post
title: "GruntJS介绍"
description: ""
category: "web"
tags: ['web前端', '技术分享', '项目文档']
---
{% include JB/setup %}

## 安装 Grunt CLI
安装：`sudo npm install -g grunt-cli`

1. 要求Node.js版本大于0.8.0 (Node安装见文档底部)
2. 配置npm使用http连接（https内网受限）：

        $ npm config set strict-ssl false
        $ npm config set registry "http://registry.npmjs.org/"

## Grunt项目结构
一个标准的配置过程只需要给项目添加两个文件：package.json和Gruntfile.js

**package.json**：这个文件被用来存储已经作为npm模块发布的项目元数据（也就是依赖模块）。你将在这个文件中列出你的项目所依赖的Grunt（通常我们在这里配置Grunt版本）和Grunt插件（相应版本的插件）。

**Gruntfile**：通常这个文件被命名为Gruntfile.js或者Gruntfile.coffee，它是用于配置或定义Grunt任务和加载Grunt插件的。

示例`package.json`：

    {
      "name": "TpGLoRy",                   // => 项目名称
      "version": "0.0.1",
      "author": "TP-Mobile Web Team",
      "devDependencies": {                 // => Grunt依赖插件
          "grunt": "*",
          "grunt-contrib-less": "*",
          "grunt-contrib-watch": "*",
          "grunt-contrib-uglify": "*",
          "grunt-contrib-copy": "*",
          "grunt-contrib-jshint": "*",
          "grunt-contrib-qunit": "*"
      }
    }

示例`Gruntfile.js`：

    module.exports = function(grunt) {      // => 封装Grunt配置的包装函数
        grunt.initConfig({                  // => 初始化Grunt项目配置

            // => 将package.json中存储的JSON元数据导入Grunt配置
            pkg: grunt.file.readJSON('package.json'),

            // => 定义banner变量，即要插入到每个JS文件的文件注释
            banner: '/* =============================================================\n' +
                    ' * Copyright (c) 2013 TP-LINK Technologies Co., Ltd.\n' +
                    ' *\n' +
                    ' * @version: <%= pkg.version%>\n' +
                    ' * @author: <%= pkg.author%>\n' +
                    ' * @date: <%= grunt.template.today("dd/mm/yyyy HH:MM") %> Released \n' +
                    ' * =============================================================*/\n\n',

            // => uglify任务，合并压缩JS文件
            uglify: {
                options: {
                    banner: '<%= banner %>',
                    report: 'min'
                },
                TpGLoRy: {
                    files: {
                        'dist/libs.min.js': [
                            'assets/js/lib/jQuery.js',
                            'assets/js/lib/jQuery.validator.js'
                        ],
                        'dist/<%= pkg.name %>.min.js': [
                            'assets/js/panel/dhcpSettings.js',
                            'assets/js/panel/wanSettings.js',
                            'assets/js/panel/lanSettings.js'
                        ]
                    }
                }
            },

            // => less任务，编译LESS文件为CSS，并压缩
            less: {
                production: {
                    options: {
                        paths: ["assets/less"],
                        cleancss: true      // => 启用CSS压缩
                    },
                    files: {
                        'dist/<%= pkg.name %>.css': "assets/less/style.less"
                    }
                }
            },

            // => JavaScript静态检测
            jshint: {
                files: ['Gruntfile.js', 'assets/**/*.js', 'test/**/*.js'],
                options: {      // => 定义JSHint选项
                    globals: {
                        jQuery: true,
                        console: true,
                        module: true
                    }
                }
            },

            // => 监控项目工程变化
            watch: {
                scripts: {
                    files: ['assets/**/*.less'],
                    tasks: ['less']     // => 一旦LESS文件发生变化，执行less任务
                                        // => 即自动化编译、压缩CSS文件
                }
            }
        });

        // => 加载各个Grunt插件
        grunt.loadNpmTasks('grunt-contrib-less');
        grunt.loadNpmTasks('grunt-contrib-uglify');
        grunt.loadNpmTasks('grunt-contrib-watch');
        grunt.loadNpmTasks('grunt-contrib-copy');
        grunt.loadNpmTasks('grunt-contrib-jshint');
        grunt.loadNpmTasks('grunt-contrib-qunit');

        // => 定义'test'任务，执行grunt test
        grunt.registerTask('test', ['jshint']);
        // => 缺省任务，即执行grunt会依次运行grunt less, grunt uglify
        grunt.registerTask('default', ['less', 'uglify']);
    };


## 执行Grunt

1. 执行`npm install`安装项目相关的依赖
2. 执行`grunt`. 如下，执行了less及uglify任务，即编译打包LESS为CSS，以及合并压缩JS、并添加文件头注释

<img src="/images/gruntjs/grunt-exec.png"/>

`grunt watch`可以监控工程文件的变化，并定义一些响应事件。最常见的就是自动编译LESS为CSS：

<img src="/images/gruntjs/grunt-watch1.png"/>

如下，开发者新创建了一个LESS文件，触发了`watch`任务中定义的`less`目标，就会自动重新编译打包LESS

<img src="/images/gruntjs/grunt-watch2.png"/>

## Grunt Qunit单元测试
首先于Gruntfile中声明加载qunit插件，并定义qunit任务：

        qunit: {
            files: ['test/**/*.html']
        },
        ...

        grunt.loadNpmTasks('grunt-contrib-qunit');
        ...

        grunt.registerTask('test', ['jshint', 'qunit']);
        ...

执行测试时候，运行`grunt qunit`或者`grunt test`

测试用例定义在`test/tests.js`下，比如对历史悠久的common.js进行测试：

<img src="/images/gruntjs/qunit-test.png"/>

测试结果，可以看到28个测试用例里面有2个失败了

<img src="/images/gruntjs/qunit-ret.png"/>

回到common.js中查看源码（QUnit报告的堆栈打印），可以找到问题所在：`is_ipaddr`检查了传参是否为空，而`is_gatewayaddr`和`is_dnsaddr`没有（典型的NPE问题）

<img src="/images/gruntjs/qunit-src.png"/>

查看测试结果，可以通过直接打开`test/index.html`查看。开发过程中更为便利的方法，是在工程目录下执行`grunt qunit`，如下：

<img src="/images/gruntjs/qunit-grunt.png"/>

## 项目开发规范
每次提交代码前要执行`grunt test`进行静态检测以及运行测试用例。执行`grunt`并本地验证确认修改有效，才可以push到Gerrit评审

<img src="/images/gruntjs/grunt_norm1.png"/>
提示"Done, without errors."表示工程项目编译OK.

而如果JSHint检测存在ERROR，则禁止提交。
<img src="/images/gruntjs/grunt_norm2.png"/>

## 备注
### Node.js
>[Node.js](http://en.wikipedia.org/wiki/NodeJS) 是服务器端的Javascript运行环境，它的设计初衷是以一种简单的方式创建可伸缩性的网络程序。Node.js具有异步I/O和事件驱动等特性，充分利用了Javascript的闭包特性和事件处理机制。

>这里主要使用npm作为包管理工具，以及Node运行时环境以支持下文提到的工具集

安装：

    # 拉取代码
    git clone http://github.com/joyent/node.git

    # 由于版本依赖关系，将node切换至0.9.9-release的发布Tag
    git checkout v0.9.9-release

    # 编译安装
    ./configure
    make && sudo make install
