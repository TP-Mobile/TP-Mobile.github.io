---
layout: post
title: "Web前端知识架构"
description: ""
thumbnail: "/images/thumbnail/web.jpg"
category: "web"
tags: ["web前端", "技术分享"]
---
{% include JB/setup %}

## Web前端概述
>从职能来看，前端是面向用户的第一道关口，与设计、产品等用户的使用体验直接相关；前端是后端数据呈现的载体，与后端、运维也联系紧密。<br />从领域来看，前端与数据可视化、富互联网应用、第三方投放等方向都有很好的结合点

<a href="/images/webfrontend/web-frontend.png"><img src="/images/webfrontend/web-frontend.png"/></a>

## 1. 开发语言
- HTML/CSS/JavaScript (专题)
    - 实现页面原型
    - 页面重构
    - 前端代码优化
- NodeJs/PHP/Python/Perl/Ruby
    - 实现诸如页面埋点分析、测试开发、流程化管理等非传统意义的“后端”需求

## 2. 移动终端
- 响应式布局 (Responsive Layout)：
    - 页面根据用户行为及设备环境（系统平台、屏幕尺寸、屏幕定向等）自适应调整。 <br />具体的实践方式由多方面组成，包括弹性网格和布局、图片、CSS Media query的使用等。
- Web移动端开发：
    - PhoneGap：开源的开发框架，使用HTML、CSS和JavaScript构建跨平台的移动应用程序
    - jQuery Mobile/Sencha Touch：面向移动端的WebUI库，可以为Web App呈现Native App的UI控件库

参考资料:

- [Bootstrap](http://getbootstrap.com/)
- [HTML5 in mobile devices](http://en.wikipedia.org/wiki/HTML5_in_mobile_devices)
- [PhoneGap/Cordova](http://cordova.apache.org/)
- [Getting Started with jQuery Mobile](http://learn.jquery.com/jquery-mobile/getting-started/)

## 3. 前端框架
- 前端开发中最常见的问题大致分为：封装原生API和常用任务、基础架构、复应用架构、视觉交互、以及工具链
- JavaScript的原生API存在以下问题：
    - 对象、方法名烦琐
    - 缺少一些常用任务的语法糖
    - 浏览器兼容性不佳
- 封装的对象包含以下类别：
    - DOM的选择和操作：如jQuery的链式API
    - DOM事件处理：简化事件的delegation，即利用事件冒泡机制在父元素上用一个侦听函数侦听触发在多个子元素上的事件
    - Ajax：简化烦琐的XMLHttpRequest API，并加强其语义性
    - 语言增强：主要提供一些对数组和对象进行操作的便利函数。
- 效果和动画：用原生JavaScript实现动画是一个比较烦琐的过程，尤其是当需要精确的时间和缓动（easing）处理时。因此一些框架如jQuery提供一个API简洁的动画引擎，并封装了常见的动画效果。
- UI组件库：更多关注CSS层面的前端框架，如Bootstrap/jQuery-UI

参考资料:

- [开源前端框架纵横谈](http://www.programmer.com.cn/15552)
- [Using jQuery Core](http://learn.jquery.com/using-jquery-core/)
- [jQuery UI](http://learn.jquery.com/jquery-ui/)

## 4. 安全
- CSRF/XSS（专题）
- ...

## 5. 测试
- [JSHint](http://en.wikipedia.org/wiki/Jshint)/CSSHint：
    - JSHint is static code analysis tool used in software development for checking if JavaScript source code compiles with coding rules.
- [Selenium](http://en.wikipedia.org/wiki/Selenium_(software\)):
    - Selenium is a portable software testing framework for web applications. Selenium provides a record/playback tool for authoring tests without learning a test scripting language (Selenium IDE). It also provides a test domain-specific language (Selenese) to write tests in a number of popular programming languages, including Java, C#, Groovy, Perl, PHP, Python and Ruby. The tests can then be run against most modern web browsers.
- [PhantomJS](www.phantomjs.org):
    - PhantomJS is a headless WebKit scriptable with JavaScript or CoffeeScript.
- PhantomJS Use Cases:
    - **Headless web testing**: Lightning-fast testing without the browser is now possible! Various [test frameworks](https://github.com/ariya/phantomjs/wiki/Headless-Testing) such as Jasmine, Capybara, QUnit, Mocha, WebDriver, YUI Test, BusterJS, FuncUnit, Robot Framework, and many others are supported.
    - **Page automation**: [Access and manipulate](https://github.com/ariya/phantomjs/wiki/Page-Automation) web pages with the standard DOM API, or with usual libraries like jQuery.
    - **Screen capture**: Programmatically [capture web contents](https://github.com/ariya/phantomjs/wiki/Screen-Capture), including CSs, SVG and Canvas. Build server-side web graphics apps, from a screenshot service to a vector chart rasterizer.
    - **Network monitoring**. Automate performance analysis, track [page loading](https://github.com/ariya/phantomjs/wiki/Network-Monitoring) and export as standard HAR format.
