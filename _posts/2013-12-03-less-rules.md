---
layout: post
title: "Less规范"
description: ""
category: "web"
tags: ["技术分享", "最佳实践"]
---
{% include JB/setup %}

##页面样式文件添加
各页面的样式用less来写，放在less/pages/下面。主要是方便使用variebles.less和mixins.less中的变量和混合，提供编写效率。
这里介绍一下less文件结构:
    less文件夹
        -   variables.less存放着全局使用的变量，包括有字体大小、颜色、背景等，各元素使用的重要的样式变量都存放在这里，如果要简单定制样式，如改变整体页面字体大小，只需要改变@baseFontSize这个变量即可。
        -   mixins.less存放全局使用的混合，混合简单的理解就是将一些你觉得相关的样式放在一起，可以使用参数，使用的时候传入需要的参数，继承它。
        -   style.less中存放的是需要编译的less文件，一般样式在前，特殊样式在后。
        -   reset.less是帮助统一各浏览器间的样式。可以不用太关注。
        -   其他的less文件都是针对不同的元素的样式定制。

##less规范
1. 页面级的样式尽量写在各个页面对应的less文件中，如果觉得可以全局通用，需要放在variebles.less或mixins.less中的话，请写在文件的最下面，并做好注释。
2. class名采用“xxx-xxx”这样的形式，全部小写。尽量是名词在前，形容词、动词在后
   id的命名采用 xxxYyy这样的形式。
3. 样式书写模板为

        .class {
            background: xxx;
            padding: xxx;
        }
缩进为4个空格；左大括号跟在selector后，且与selector用一个空格隔开；多条样式分行写，且属性值与属性名间的冒号靠近属性名，与属性值用一个空格隔开。
4. 属性值为0的时候，0后面不需要加单位。如
    font-size: 0;
5. 如果属性值为0.xxx时，省略0。如`background-color: rgba(0, 0, 0, .3);`
6. 尽量使用shorthand书写方法。如`padding: 10px 5px;`
7. 组织样式的时候，尽量先提取出一个抽象、通用的样式class，然后在其之上进行特殊样式的扩展。
比如.btn和.btn-success\.btn-big这样的，后续若要修改btn的大小，就只需要改变.btn中的padding，所有button的大小就会改变；若要添加一个新的button颜色，可以再定义一个.btn-xxx就可以了。
8. 页面级的样式若无法通用，非常个例，可以使用#id selector。
9. 利用好可以继承的样式。  比如font样式是可以继承的，那么在一个div中的所有子元素都会从父div中继承这个样式，而不需要在每个子元素中都定义一下font属性。
10. 将样式定义在父级。如
    <ul class="list-featured">
        <li></li>
        <li></li>
        <li></li>
    </ul>
    样式可以写为：
    .list-featured {
    ...
    }
    .list-featured > li {
    ...
    }
    不要写成是
    <ul>
        <li class="item"></li>
        <li class="item"></li>
        <li class="item"></li>
    </ul>
11. 父级嵌套样式不要超过4层。如 .nav ul > li > a，不能再多了，影响查找速度。
12. 使用less的嵌套时，嵌套不超过2层。

