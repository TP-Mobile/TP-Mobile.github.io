---
layout: post
title: "测试基础培训"
description: ""
thumbnail: "/images/thumbnail/test.jpg"
category:
tags: ["测试相关", "技术讲座"]
---
{% include JB/setup %}

## 内容索引
1. [代码静态检测](#static_test)
    - [静态检测概述](#static_test_intro)
    - [FindBugs介绍](#findbug_intro)
    - [Splint介绍](#splint_intro)
2. [动态检测工具](#dynamic_test_tool)
    - [Valgrind Introduction](#valgrind_intro)
    - [Valgrind Setup](#valgrind_setup)
    - [Valgrind Usage](#valgrind_usage)
    - [常见内存问题](#common_mem_errors)
        - [使用未初始化的内存](#uninitialized_mem)
        - [内存读写越界](#mem_outrange)
        - [内存覆盖](#mem_overlap)
        - [动态内存管理错误](#mem_mgr_err)
        - [内存泄漏](#mem_leak)
    - [使用Helgrind调试多线程代码](#helgrind_usage)
3. [验收测试](#functional_test)
    - [Android工具集](#functional_android)
        - [MonkeyRunner](#monkeyrunner)
        - [Monkey](#monkey)
        - [Robotium](#robotium)
    - [Web Frontend](#functional_webfrontend)
        - [Selenium/PhantomJS](#selenium)
4. [单元测试](#unit_test)
    - [Mongoose Demo](#mongoose_demo)
    - [Android Unit Test](#unit_test_android)
5. [Repo Regression Test](#repo_reg_test)
6. [Video Format Test](#video_format_test)

## <a id="static_test"></a>代码静态检测

### <a id="static_test_intro"></a>静态检测概述
使用自动化工具软件对程序源代码进行静态检测（static program analysis），以分析程序行为的技术，应用于程序的正确性检查、安全缺陷检测、程序优化等。静态检测无需运行被测代码，仅通过分析或检查程序的语法、结果、过程、接口等检查程序正确性。静态检测有助于在项目早期发现以下问题：变量声明但未使用、变量类型不匹配、变量在使用前未定义、不可达代码、死循环、数组越界、内存泄漏等。

在软件开发过程中，静态代码分析往往优先于动态测试前进行，同时也可作为制定动态测试用例的参考。统计表明，整个开发生命周期中，30%到70%的代码逻辑设计和编码缺陷可以通过静态代码分析来发现和修复。

静态检测工具同编译器所做工作的区别：静态分析工具相比于编译器，对代码进行了更加严格的检测，甚至可以在分析工具的分析标准中定义代码的规范、在检测到不符合规范的代码时抛出警告。

>既然静态分析工具发挥了不小的作用，何不在编译器里兼备静态分析的功能？对这个问题，S.C.Johnson（Lint的作者）在1978年论文《Lint, a C Program Checker》中给出了他的答案：“Lint与C编译器在功能上的分离既有历史原因，也有现实的意义。编译器负责把C源程序快速、高效转变为可执行文件，不对代码做类型检查（特别是对分别编译的程序），有益于做到快速与高效。而Lint没有‘高效’的要求，可以花更多时间对代码更深入、仔细的检查。”

而随着计算能力的增强，集成开发环境（IDE）可以实现实时源码分析、甚至通过插件来执行更多更全面的分析，如增强代码规则或代码风格检查的插件（Checkstyle, JLint, PWD），及检查和移除冗余代码的分析器（Duplication Management Framework）

**静态代码分析的理论基础和主要技术**

- 缺陷模式匹配： <br />
    事先从代码分析经验中搜集足够多的共性缺陷模式，将待分析代码与已有的共性缺陷模式进行模式匹配，从而完成软件的安全分析。优点：简单方便，但要求内置足够多的缺陷模式，且容易产生误报。
- 类型推断： <br />
    类型推断技术指通过对代码中运算对象类型进行推理，从而保证代码中每条语句都针对正确的类型执行。这种技术首先将预定义一套类型机制，包括类型等价、类型包含等推理逻辑，而后基于这一规则进行推理计算。类型推断可以检查代码中的类型错误，简单、高效，适合代码缺陷的快速检测。
- 模型检查： <br />
    模型检查建立于有限状态自动机的概念上，这一理论将被分析代码抽象为一个自动机系统，并假设该系统是有限状态的、或可以通过抽象归结为有限状态。模型检验过程中，首先将被分析代码中的每条语句产生的影响抽象为一个有限状态自动机的一个状态，而后通过分析有限状态机从而达到代码分析的目的。模型验证主要适合检验程序并发等时序特性，但对于数据值域数据类型等方面作用较弱
- 数据流分析： <br />
    数据流分析也是一种软件验证技术，这种技术通过收集代码中引用到的变量信息，从而分析变量在程序中的赋值、引用以及传递等情况。对数据流进行分析可以确定变量的定义以及在代码中被引用的情况，同时还能检查代码数据流异常：如引用在前赋值在后、只赋值无引用等。数据流分析主要适合检验程序中的数据域特性。

### <a id="findbug_intro"></a>FindBugs介绍
FindBugs是静态分析工具，检查类或JAR文件，将字节码与一组缺陷模式进行对比以发现可能的问题。

>关于静态分析工具的局限性：提供大量并非真正问题的问题（伪问题false positivies），出现伪问题时，开发人员要学会忽略工具的输出或放弃它。

**使用FindBugs**

1. 使用Swing工具： <br />
    新建项目，添加待分析的类包和目录，可选择编译好的类所在文件夹（\bin），也可选择生成的jar包，再添加辅助类所在的文件夹（\lib），源文件所在文件夹（\src）
2. Eclipse插件： <br />
    右击工程【Find Bugs】-【Find Bugs】，打开Bug Explorer面板（Window - Show View - Bug Explorer），如果要查看某个Bug详细信息，可以选择Window - Open Perspective，然后选择FindBugs就可以打开Findbugs的Properties面板

**典型检测器实例**

1. 检测忽略方法返回值 <br />
    这个检测器查找代码中忽略了不应该忽略的方法返回值的地方，常见于调用String方法，如：

        String aStr = "bob";
        aStr.replace('b', 'p');
        if (aString.equals("pop"))
    程序员以为他已经用p替换了字符串中所有的b（line 2），但是字符串是不可变的，所有这类方法都返回一个新字符串
2. 检测NPE问题 <br />
    查找代码路径可能造成NPE的情况，还查找对null的冗余比较的情况。如：

        Person person = aMap.get("bob");
        if (person != null) {
            person.updateAccessTime();
        }
        String name = person.getName();
    如果Map中不含bob，line 5的getName()可能出现空指针异常

**忽略Android工程中R.java的伪问题**
>FindBugs is an extremely useful static analysis tool for java code that will flag many subtle bugs and point out bad practice. However, when using FingBugs with Android Projects, it will complain about the name of the auto-generated Android resource inner classes, which start with a lower-case letter, e.g. com.example.app.R$attr

By defining a filter for FindBugs, the warings can be suppressed. Create an XML-file

    <FindBugsFilter>
        <Match>
            <Class name="~.*\.R\$.*" />
            <Bug pattern="NM_CLASS_NAMING_CONVENTION" />
        </Match>
    </FindBugsFilter>

### <a id="splint_intro"></a>Splint介绍
- 安装Splint：`sudo apt-get install splint`
- 应用Splint：`splint *.c`

示例：sigsegv.c

    int main(void)
    {
        char* c = 0;
        printf("%s", *c);

        return 0;
    }
结果输出：

    $ splint sigsegv.c
    Splint 3.1.2 --- 03 May 2009

    sigsegv.c: (in function main)
    sigsegv.c:4:18: Dereference of null pointer c: *c
      A possibly null pointer is dereferenced.  Value is either the result of a
      function which may return null (in which case, code should check it is not
      null), or a global, parameter or structure field declared with the null
      qualifier. (Use -nullderef to inhibit warning)
       sigsegv.c:3:12: Storage c becomes null

    sigsegv.c:4:17: Format argument 1 to printf (%s) expects char * gets char: *c
      Type of parameter is not consistent with corresponding code in format string.
      (Use -formattype to inhibit warning)
       sigsegv.c:4:11: Corresponding format code

    Finished checking --- 2 code warnings
由输出结果可见，Splint给出两处警告：对空指针的解引用、及强制类型转换问题。

**Splint可检测类型**

- Null Dereferences（解引用空指针）
- Undefined Values（使用未定义变量）
- Types（类型转换）
- Memory Management（内存管理）
- Function Interfaces（函数接口检测）
- Control Flow（控制流检测）
- Buffer Sizes（缓存边界）
- Extensible Checking
- Macros（宏隐患检测）
- Naming Conventions（命名规范）
- Completeness
- Libraries and Header File Inclusion（库文件及头文件引用）

## <a id="dynamic_test_tool"></a> 动态检测工具

### <a id="valgrind_intro"></a>Valgrind介绍
>Valgrind是一个GPL的软件，用于Linux程序的内存调试。你可以在它的环境中运行你的程序来监视内存的使用情况，比如C语言中的malloc/free、或C++中的new/delete.

Valgrind包含以下工具集：

- Memcheck：Valgrind的默认工具，一个细粒度的内存检查器，主要检测以下程序错误：
    - 使用未初始化的内存（Use of unintialised memory）
    - 使用已经释放了的内存（Reading/Writing memory after it has been free'd）
    - 使用超过malloc分配的内存空间（Reading/Writing off the end of malloc'd blocks）
    - 对堆栈的非法访问（Reading/Writing inappropriate areas on the stack）
    - 内存泄漏（Memory leaks -- where pointers to malloc'd blocks are lost forever）
    - 申请和释放内存的匹配（Mismatched use of malloc/new/new[] vs free/delete/delete[]）
    - 踩内存（Overlapping src and dst pointers in memcpy() and related functions）
- Callgrind：主要用来检查程序中函数调用过程中出现的问题
- Cachegrind：主要用来检查程序中缓存使用出现的问题
- Helgrind：主要用来检查多线程程序中出现的竞争问题。
- Massif：主要用来检查程序中堆栈使用中出现的问题。
- Extension：可以利用core提供的功能，自己编写特定的内存调试工具。

### <a id="valgrind_setup"></a>Valgrind安装
1. 解压Valgrind源码、编译安装

        $ tar -jxvf valgrind-3.8.1.tar.bz2
        $ cd valgrind-3.8.1

        $ ./configure
        $ sudo make && sudo make install
2. 尝试执行`valgrind ls`

        ==20965== Memcheck, a memory error detector
        ==20965== Copyright (C) 2002-2012, and GNU GPL'd, by Julian Seward et al.
        ==20965== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
        ==20965== Command: ls
        ==20965==
        ==20965==
        ==20965== HEAP SUMMARY:
        ==20965==     in use at exit: 30,266 bytes in 90 blocks
        ==20965==   total heap usage: 125 allocs, 35 frees, 67,477 bytes allocated
        ==20965==
        ==20965== LEAK SUMMARY:
        ==20965==    definitely lost: 0 bytes in 0 blocks
        ==20965==    indirectly lost: 0 bytes in 0 blocks
        ==20965==      possibly lost: 0 bytes in 0 blocks
        ==20965==    still reachable: 30,266 bytes in 90 blocks
        ==20965==         suppressed: 0 bytes in 0 blocks
        ==20965== Rerun with --leak-check=full to see details of leaked memory
        ==20965==
        ==20965== For counts of detected and suppressed errors, rerun with: -v
        ==20965== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
3. 注：Valgrind运行需要带有调试信息的C库支持，若执行失败、安装缺失的包：

        $ sudo apt-cache showpkg valgrind
        ...
        Dependencies:
        libc6-dbg ...

### <a id="valgrind_usage"></a>Valgrind应用
1. 概要用法

        SYNOPSIS
            valgrind [valgrind-options] [your-program] [your-program-options]
2. 基本选项（Basic Option）

        --tool=<toolname> [default: memcheck]
            运行Valgrind工具集中名为toolname的工具
        -q --quiet
            只打印错误信息，回归测试或有其他自动化测试机制时经常使用。
        -v --verbose
            显示详细信息。在各个方面显示你的程序的额外信息，例如：共享对象加载，使用的重置，执行引擎和工具的进程，异常行为的警告信息。
        --trace-children=<no|yes> [default: no]
            是否跟踪子线程。
        --time-stamp=<no|yes> [default: no]
            增加时间戳到log信息。
        --log-file=<filename>
            指定Valgrind把它所有的信息输出到指定的文件中。被创建文件的文件名格式如<filename>.<pid> ，从而每个进程创建不同的文件。
3. Memcheck检测选项

        --leak-check=<no|summary|yes|full> [default: summary]
            对内存泄漏给出详细信息。
        --show-reachable=<yes|no> [default: no]
            当这个选项关闭时，内存泄漏检测器只显示没有指针指向的内存块，或者只能找到指向块中间的指针。当这个选项打开时，内存泄漏检测器还报告有指针指向的内存块。
        --leak-resolution=<low|med|high> [default: low]
            在做内存泄漏检查时，确定memcheck将怎么样考虑不同的栈是相同的情况。当设置为low时，只需要前两层栈匹配就认为是相同的情况；当设置为med，必须要四层栈匹配，当设置为high时，所有层次的栈都必须匹配。

### <a id="common_mem_errors"></a>常见内存问题
#### <a id="uninitialized_mem"></a>使用未初始化的内存

对于位于程序中不同段的变量，其初始值是不同的，全局变量和静态变量初始值为0，而局部变量和动态申请的变量，初始值为随机值。如果程序使用了为随机值的变量，那么程序的行为就变得不可预期。

示例：valgrind/memcheck/tests/badloop.c

    #include <stdio.h>

    int main (void) {
        int a[5];
        int i, s;
        a[0] = a[1] = a[3] = a[4] = 0;
        s = 0;
        for (i = 0; i < 5; i++)
            s += a[i];
        if (s == 377)
            printf("sum is %d\n", s);
        return 0;
    }
运行结果：

    ==24907== Memcheck, a memory error detector
    ==24907== Copyright (C) 2002-2012, and GNU GPL'd, by Julian Seward et al.
    ==24907== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
    ==24907== Command: ./badloop
    ==24907==
    ==24907== Conditional jump or move depends on uninitialised value(s)
    ==24907==    at 0x400542: main (badloop.c:11)
    ==24907==
    ==24907==
    ==24907== HEAP SUMMARY:
    ==24907==     in use at exit: 0 bytes in 0 blocks
    ==24907==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
    ==24907==
    ==24907== All heap blocks were freed -- no leaks are possible
    ==24907==
    ==24907== For counts of detected and suppressed errors, rerun with: -v
    ==24907== Use --track-origins=yes to see where uninitialised values come from
    ==24907== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
结果分析：

- 数字（24907）表示Process ID
- 输出结果表示，程序的跳转依赖于一个未初始化的变量

#### <a id="mem_outrange"></a>内存读写越界

这种情况是指：访问了你不应该/没有权限访问的内存地址空间，比如访问数组时越界；对动态内存访问时超出了申请的内存大小范围。下例中，pt是一个局部数组变量，其大小为4，p初始指向pt数组的起始地址，但在对p循环叠加后，p超出了pt数组的范围，如果此时再对p进行写操作，后果将不可预期。

示例：valgrind/memcheck/tests/badacc.cpp

    #include <stdlib.h>
    #include <stdio.h>

    int main(int argc, char *argv[]) {
        int len = 4;
        int* pt = (int*)malloc(len*sizeof(int));
        int* p = pt;

        for (int i = 0; i < len; i++)
            p++;

        *p = 5;
        printf("the value of p is: %d\n", *p);
        return 0;
    }
运行结果：

    ==25117== Command: ./badacc
    ==25117==
    ==25117== Invalid write of size 4
    ==25117==    at 0x40059A: main (badacc.cpp:12)
    ==25117==  Address 0x51f3050 is 0 bytes after a block of size 16 alloc'd
    ==25117==    at 0x4C2C66F: malloc (vg_replace_malloc.c:270)
    ==25117==    by 0x40056A: main (badacc.cpp:6)
    ==25117==
    ==25117== Invalid read of size 4
    ==25117==    at 0x4005A4: main (badacc.cpp:13)
    ==25117==  Address 0x51f3050 is 0 bytes after a block of size 16 alloc'd
    ==25117==    at 0x4C2C66F: malloc (vg_replace_malloc.c:270)
    ==25117==    by 0x40056A: main (badacc.cpp:6)
    ==25117==
    the value of p is: 5
    ==25117==
    ==25117== HEAP SUMMARY:
    ==25117==     in use at exit: 16 bytes in 1 blocks
    ==25117==   total heap usage: 1 allocs, 0 frees, 16 bytes allocated
    ==25117==
    ==25117== LEAK SUMMARY:
    ==25117==    definitely lost: 16 bytes in 1 blocks
    ==25117==    indirectly lost: 0 bytes in 0 blocks
    ==25117==      possibly lost: 0 bytes in 0 blocks
    ==25117==    still reachable: 0 bytes in 0 blocks
    ==25117==         suppressed: 0 bytes in 0 blocks
    ==25117== Rerun with --leak-check=full to see details of leaked memory
结果分析：

- 输出结果显示，在该程序的第12行，进行了非法的写操作；在第13行，进行了非法读操作。程序同时存在内存泄漏问题，16个字节的内存在程序结束时丢失（definitely lost）

#### <a id="mem_overlap"></a>内存覆盖

C 语言的强大和可怕之处在于其可以直接操作内存，C 标准库中提供了大量这样的函数，比如 strcpy, strncpy, memcpy, strcat 等，这些函数有一个共同的特点就是需要设置源地址 (src)，和目标地址(dst)，src 和 dst 指向的地址不能发生重叠，否则结果将不可预期。

下面就是一个 src 和 dst 发生重叠的例子。在 11 与 13 行中，src 和 dst 所指向的地址相差 20，但指定的拷贝长度却是 21，这样就会把之前的拷贝值覆盖。第 20 行程序类似，src(x+20) 与 dst(x) 所指向的地址相差 20，但 dst 的长度却为 21，这样也会发生内存覆盖。

示例：valgrind/memcheck/tests/badlap.cpp

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    int main(int argc, char *argv[]) {
        char x[50];
        for (int i = 0; i < 50; i++)
            x[i] = i + 1;

        strncpy(x+20, x, 20);   // OK
        strncpy(x+20, x, 21);   // Overlap
        strncpy(x, x+20, 20);   // OK
        strncpy(x, x+20, 21);   // Overlap

        x[39] = '\0';
        strcpy(x, x+20);        // OK

        x[39] = 39;
        x[40] = '\0';
        strcpy(x, x+20);        // Overlap

        return 0;
    }

运行结果：

    ==25315== Source and destination overlap in strncpy(0x7fefffec9, 0x7fefffeb5, 21)
    ==25315==    at 0x4C2D256: strncpy (mc_replace_strmem.c:472)
    ==25315==    by 0x400632: main (badlap.cpp:11)
    ==25315==
    ==25315== Source and destination overlap in strncpy(0x7fefffeb5, 0x7fefffec9, 21)
    ==25315==    at 0x4C2D256: strncpy (mc_replace_strmem.c:472)
    ==25315==    by 0x40066A: main (badlap.cpp:13)
    ==25315==
    ==25315== Source and destination overlap in strcpy(0x7fefffea0, 0x7fefffeb4)
    ==25315==    at 0x4C2D0A5: strcpy (mc_replace_strmem.c:438)
    ==25315==    by 0x4006A4: main (badlap.cpp:20)

结果分析：

- 输出结果显示，程序中第11，13，20行，源地址和目标地址设置出现重叠。

#### <a id="mem_mgr_err"></a>动态内存管理错误

常见的内存分配方式分三种：静态存储，栈上分配，堆上分配。全局变量属于静态存储，它们是在编译时就被分配了存储空间，函数内的局部变量属于栈上分配，而最灵活的内存使用方式当属堆上分配，也叫做内存动态分配了。常用的内存动态分配函数包括：malloc, alloc, realloc, new等，动态释放函数包括free, delete。

常见的内存动态管理错误包括：

1. 申请和释放不一致<br />
    由于 C++ 兼容 C，而 C 与 C++ 的内存申请和释放函数是不同的，因此在 C++ 程序中，就有两套动态内存管理函数。一条不变的规则就是采用 C 方式申请的内存就用 C 方式释放；用 C++ 方式申请的内存，用 C++ 方式释放。也就是用 malloc/alloc/realloc 方式申请的内存，用 free 释放；用 new 方式申请的内存用 delete 释放。在上述程序中，用 malloc 方式申请了内存却用 delete 来释放，虽然这在很多情况下不会有问题，但这绝对是潜在的问题。
2. 申请和释放不匹配<br />
    申请了多少内存，在使用完成后就要释放多少。如果没有释放，或者少释放了就是内存泄露；多释放了也会产生问题。上述程序中，指针p和pt指向的是同一块内存，却被先后释放两次。
3. 释放后仍然读写<br />
    本质上说，系统会在堆上维护一个动态内存链表，如果被释放，就意味着该块内存可以继续被分配给其他部分，如果内存被释放后再访问，就可能覆盖其他部分的信息，这是一种严重的错误，上述程序第13行中就在释放后仍然写这块内存。

示例：valgrind/memcheck/tests/badmac.cpp

    #include <stdlib.h>
    #include <stdio.h>

    int main(int argc, char *argv[]) {
        char* p = (char*)malloc(10);
        char* pt = p;

        for (int i = 0; i < 10; i++)
            p[i] = 'z';

        delete p;

        pt[1] = 'x';
        free(pt);
        return 0;
    }

运行结果：

    ==25465== Mismatched free() / delete / delete []
    ==25465==    at 0x4C2B1BF: operator delete(void*) (vg_replace_malloc.c:480)
    ==25465==    by 0x400648: main (badmac.cpp:11)
    ==25465==  Address 0x5a05040 is 0 bytes inside a block of size 10 alloc'd
    ==25465==    at 0x4C2C66F: malloc (vg_replace_malloc.c:270)
    ==25465==    by 0x40060C: main (badmac.cpp:5)
    ==25465==
    ==25465== Invalid write of size 1
    ==25465==    at 0x400651: main (badmac.cpp:13)
    ==25465==  Address 0x5a05041 is 1 bytes inside a block of size 10 free'd
    ==25465==    at 0x4C2B1BF: operator delete(void*) (vg_replace_malloc.c:480)
    ==25465==    by 0x400648: main (badmac.cpp:11)
    ==25465==
    ==25465== Invalid free() / delete / delete[] / realloc()
    ==25465==    at 0x4C2B5D9: free (vg_replace_malloc.c:446)
    ==25465==    by 0x40065F: main (badmac.cpp:14)
    ==25465==  Address 0x5a05040 is 0 bytes inside a block of size 10 free'd
    ==25465==    at 0x4C2B1BF: operator delete(void*) (vg_replace_malloc.c:480)
    ==25465==    by 0x400648: main (badmac.cpp:11)
结果分析：

- 输出显示，第11行分配和释放函数不一致；第13行发生非法写操作，也就是往释放后的内存地址写值；第14行释放内存函数无效。

#### <a id="mem_leak"></a>内存泄漏

内存泄露（Memory leak）指的是，在程序中动态申请的内存，在使用完后既没有释放，又无法被程序的其他部分访问。内存泄漏是最难发现的常见错误之一，因为除非用完内存或调用 malloc失败，否则都不会导致任何问题。实际上，使用像C 或C++这类没有垃圾回收机制的语言时，你一大半的时间都花费在处理如何正确释放内存上。如果程序运行时间足够长，一个小小的失误也会对程序造成重大的影响。防止内存泄露要从良好的编程习惯做起，另外重要的一点就是要加强单元测试（Unit Test），而memcheck就是这样一款优秀的工具。

下面是一个比较典型的内存泄露案例。main函数调用了mk函数生成树结点，可是在调用完成之后，却没有相应的函数：nodefr释放内存，这样内存中的这个树结构就无法被其他部分访问，造成了内存泄露。

在一个单独的函数中，每个人的内存泄露意识都是比较强的。但很多情况下，我们都会对malloc/free 或new/delete做一些包装，以符合我们特定的需要，无法做到在一个函数中既使用又释放。这个例子也说明了内存泄露最容易发生的地方：即两个部分的接口部分，一个函数申请内存，一个函数释放内存。并且这些函数由不同的人开发、使用，这样造成内存泄露的可能性就比较大了。这需要养成良好的单元测试习 惯，将内存泄露消灭在初始阶段。

示例：tree.h

    #ifndef _BADLEAK_
    #define _BADLEAK_

    typedef struct _node {
        struct _node *l;
        struct _node *r;
        char v;
    }node;

    node *mk(node *l, node *r, char val);

    void nodefr(node *n);

    #endif
tree.cpp

    #include <stdlib.h>
    #include "tree.h"

    node *mk(node *l, node *r, char val) {
        node* f = (node*)malloc(sizeof(*f));
        f->l = l;
        f->r = r;
        f->v = val;
        return f;
    }

    void nodefr(node *n) {
        if (n) {
            nodefr(n->l);
            nodefr(n->r);
            free(n);
        }
    }
badleak.cpp

    #include <stdio.h>
    #include <stdlib.h>
    #include "tree.h"

    int main() {
        node *tree1, *tree2, *tree3;
        tree1 = mk(mk(mk(0,0,'3'),0,'2'),0,'1');
        tree2 = mk(0,mk(0,mk(0,0,'6'),'5'),'4');
        tree3 = mk(mk(tree1,tree2,'8'),0,'7');
        return 0;
    }

运行结果：

    ==25771== HEAP SUMMARY:
    ==25771==     in use at exit: 192 bytes in 8 blocks
    ==25771==   total heap usage: 8 allocs, 0 frees, 192 bytes allocated
    ==25771==
    ==25771== LEAK SUMMARY:
    ==25771==    definitely lost: 24 bytes in 1 blocks
    ==25771==    indirectly lost: 168 bytes in 7 blocks
    ==25771==      possibly lost: 0 bytes in 0 blocks
    ==25771==    still reachable: 0 bytes in 0 blocks
    ==25771==         suppressed: 0 bytes in 0 blocks
    ==25771== Rerun with --leak-check=full to see details of leaked memory
结果分析：

- 从上述输出可以看出，所有的内存泄露都被发现。 Memcheck将内存泄露分为两种，一种是可能的内存泄露（Possibly lost），另外一种是确定的内存泄露（Definitely lost）。Possibly lost 是指仍然存在某个指针能够访问某块内存，但该指针指向的已经不是该内存首地址。Definitely lost 是指已经不能够访问这块内存。而Definitely lost又分为两种：直接的（direct）和间接的（indirect）。直接和间接的区别就是，直接是没有任何指针指向该内存，间接是指指向该内存的指针都位于内存泄露处。在上述的例子中，根节点是directly lost，而其他节点是indirectly lost。

### <a id="helgrind_usage"></a>使用Helgrind调试多线程代码

Helgrind用于检测C/C++下使用POSIX pthreads程序的线程同步错误。Helgrind可以检测以下三类错误：

- POSIX pthreads API的错误使用（Misuses of the POSIX pthreads API）
- 由加锁和解锁顺序引起的潜在死锁（Potential deadlocks arising from lock ordering problems）
- 数据竞争：在没有锁或同步机制下访问内存（Data races - accessing memory without adequate locking or synchronisation）

示例：valgrind/helgrind/tests/hg04_race.c

    #include <pthread.h>
    #include <unistd.h>

    static int shared;

    static void *th(void *v)
    {
        shared++;
        return 0;
    }

    int main()
    {
        pthread_t a, b;

        pthread_create(&a, NULL, th, NULL);
        sleep(1);       /* force ordering */
        pthread_create(&b, NULL, th, NULL);

        pthread_join(a, NULL);
        pthread_join(b, NULL);

        return 0;
    }
运行结果`valgrind --tool=helgrind ./hg04_race`：

    ==3931== Helgrind, a thread error detector
    ==3931== Copyright (C) 2007-2012, and GNU GPL'd, by OpenWorks LLP et al.
    ==3931== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
    ==3931== Command: ./hg04_race
    ==3931==
    ==3931== ---Thread-Announcement------------------------------------------
    ==3931==
    ==3931== Thread #3 was created
    ==3931==    at 0x5145C8E: clone (clone.S:77)
    ==3931==    by 0x4E3BF6F: do_clone.constprop.4 (createthread.c:75)
    ==3931==    by 0x4E3D57F: pthread_create@@GLIBC_2.2.5 (createthread.c:256)
    ==3931==    by 0x4C2E84A: pthread_create_WRK (hg_intercepts.c:255)
    ==3931==    by 0x4C2E9CC: pthread_create@* (hg_intercepts.c:286)
    ==3931==    by 0x400659: main (hg04_race.c:21)
    ==3931==
    ==3931== ---Thread-Announcement------------------------------------------
    ==3931==
    ==3931== Thread #2 was created
    ==3931==    at 0x5145C8E: clone (clone.S:77)
    ==3931==    by 0x4E3BF6F: do_clone.constprop.4 (createthread.c:75)
    ==3931==    by 0x4E3D57F: pthread_create@@GLIBC_2.2.5 (createthread.c:256)
    ==3931==    by 0x4C2E84A: pthread_create_WRK (hg_intercepts.c:255)
    ==3931==    by 0x4C2E9CC: pthread_create@* (hg_intercepts.c:286)
    ==3931==    by 0x400634: main (hg04_race.c:19)
    ==3931==
    ==3931== ----------------------------------------------------------------
    ==3931==
    ==3931== Possible data race during read of size 4 at 0x601040 by thread #3
    ==3931== Locks held: none
    ==3931==    at 0x4005FC: th (hg04_race.c:10)
    ==3931==    by 0x4C2E9B5: mythread_wrapper (hg_intercepts.c:219)
    ==3931==    by 0x4E3CE99: start_thread (pthread_create.c:308)
    ==3931==
    ==3931== This conflicts with a previous write of size 4 by thread #2
    ==3931== Locks held: none
    ==3931==    at 0x400605: th (hg04_race.c:10)
    ==3931==    by 0x4C2E9B5: mythread_wrapper (hg_intercepts.c:219)
    ==3931==    by 0x4E3CE99: start_thread (pthread_create.c:308)
    ==3931==
    ==3931== ----------------------------------------------------------------
    ==3931==
    ==3931== Possible data race during write of size 4 at 0x601040 by thread #3
    ==3931== Locks held: none
    ==3931==    at 0x400605: th (hg04_race.c:10)
    ==3931==    by 0x4C2E9B5: mythread_wrapper (hg_intercepts.c:219)
    ==3931==    by 0x4E3CE99: start_thread (pthread_create.c:308)
    ==3931==
    ==3931== This conflicts with a previous write of size 4 by thread #2
    ==3931== Locks held: none
    ==3931==    at 0x400605: th (hg04_race.c:10)
    ==3931==    by 0x4C2E9B5: mythread_wrapper (hg_intercepts.c:219)
    ==3931==    by 0x4E3CE99: start_thread (pthread_create.c:308)
结果分析：

- 可以看到竟态访问的地址和大小，还有调用的堆栈信息。可以看到，是第10行对shared的修改可能导致竞争；采用同步机制以保证同一时间只有独立的访问。

## <a id="functional_test"></a>验收测试
验收测试也称功能测试，是测试和检验应用程序是否能按照涉众（stakeholder）的功能性需求、非功能性需求和其他需求来运行的一种方法。验收测试是单元测试和组合测试的补充，后两者通常使用xUnit框架便写。

验收测试同单元测试区别：

- 应用程序是作为一个完整的端到端实体来测试（而不像单元测试，只测试一个类或一组类）
- 编写TC的人不一定知道应用程序的内部结构、即黑盒测试

### <a id="functional_android"></a>Android

#### <a id="monkeyrunner"></a>MonkeyRunner
>[MonkeyRunner](http://developer.android.com/tools/help/monkeyrunner_concepts.html) provides an API for writing programs that control an Android device or emulator from outside of Android code. The monkeyrunner tool is primarily designed to test apps and devices at the functional/framework level and for running unit test suites.

MonkeyRunner工具使用Jython（使用Java的一种Python实现），Jython允许monkeyrunner API与Android框架轻松的进行交互。使用Jython，可以使用Python语法来获取API中的常量、类以及方法。

MonkeyRunner Features:

- _Functional Testing:_ <br />
    MonkeyRunner can run an automated start-to-finish test of an Android application. You provide input values with keystrokes or touch events, and view the results as screenshots.
- _Regression Testing:_ <br />
    MonkeyRunner can test application stability by running an application and comparing its output screenshots to a set of screenshots that are known to be correct.
- _Extensible Automation:_ <br />
    Since monkeyrunner is an API toolkit, you can develop an entrie system of Python-based modules and programs for controlling Android devices. Besides using the monkeyrunner API itself, you can use the standard Python OS and subprocess modules to call Android tools such as ADB.

测试代码示例：

    #! /usr/bin/env monkeyrunner

    import sys
    from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice, MonkeyImage

    # Connects to the current device, returning a MonkeyDevice object
    device = MonkeyRunner.waitForConnection()
    if not device:
    print >> sys.stderr, "Couldn't get connection"
    sys.exit(1)

    device.startActivity(component='com.example.aatg.tc/.MainActivity')
    MonkeyRunner.sleep(3.0)

    device.type("123")

    # Takes a screenshot
    MonkeyRunner.sleep(3.0)
    result = device.takeSnapshot()

    # Writes the screenshot to a file
    result.writeToFile('/tmp/device.png', 'png')
    device.press('KEYCODE_BACK', 'DOWN_AND_UP')

#### <a id="monkey"></a>Monkey
>[Monkey](http://developer.android.com/tools/help/monkey.html)向系统发送伪随机的用户事件（如按键输入、触摸屏输入、手势输入等），实现对正在开发的应用程序进行压力测试。如果应用程序崩溃或者接收到任何失控异常，或出现ANR错误，Monkey将停止并报错。

常用参数：

    -v
        verbosity level 执行Monkey的反馈信息级别

    -p <allowed-package-name>
        执行Monkey限定的包，仅启动这些包中的Activity
        Note: 如果你的应用程序还需要访问其它包里的Activity(如选择取一个联系人)，那些包也需要在此同时指定。

    -c <main-category>
        执行Monkey限定的Category
        Note: 对于一些场景，如某个App的某个Activity仅能通过Back键来退出，一旦执行Monkey时进入了该Activity，此后系统产生的随机事件都会仅限于该Activity——直至一个Back Key Event被产生。
        这可能达不到我们想要的测试目标，解决办法有两种：增加syskeys事件的概率（也增加了Back键被触发的概率）；或者对于主要的、待测的Activity添加CATEGORY_MONKEY，这样可以确保Monkey执行在你所指定的一些Activity中。

    --pct-syskeys
        指定syskeys事件的比例
        比如，仅测试应用响应系统按键的事件（Home, Back, Start Call, End Call, or Volume controls），可指定参数 --pct-syskeys 100 （即事件类型100%为syskeys）

    --monitor-native-crashes
        监控Native Code的崩溃事件

    --throttle <milliseconds>
        添加随机事件执行延迟（缺省情况下无延迟）

执行Monkey，并保存反馈信息及log:

    $ adb logcat -c && adb shell monkey -p {package_name} -v -v -v 500>./monkey_report.txt && adb logcat -d>./logcat.txt

_Monkey Source Code:_ `$ANDROID/development/cmds/monkey`

#### <a id="robotium"></a>Robotium
>Robotium is a test framework created to simplify the writing of tests requiring minimal knowledge of the application under test. Robotium is mainly oriented write powerful and robust automatic black-box test cases for Android applications. It can cover funciton, system, and acceptance test scenarios, even spanning multiple Android activities of the same application automatically. <br />
Robotium can also be used to test applications that we don't have the source code for, or even pre-installed applications.

### <a id="functional_webfrontend"></a>Web Frontend

#### <a id="selenium"></a>Selenium/PhatomJS _(TBA)_

## <a id="unit_test"></a>单元测试

### <a id="mongoose_demo"></a>Mongoose Demo
{mongoose}/test/unit_test.c

    #define FATAL(str, line) do {                     \
      printf("Fail on line %d: [%s]\n", line, str);   \
      abort();                                        \
    } while (0)
    #define ASSERT(expr) do { if (!(expr)) FATAL(#expr, __LINE__); } while (0)

    static void test_base64_encode(void) {
        const char *in[] = {"a", "ab", "abc", "abcd", NULL};
        const char *out[] = {"YQ==", "YWI=", "YWJj", "YWJjZA=="};
        char buf[100];
        int i;

        for (i = 0; in[i] != NULL; i++) {
            base64_encode((unsigned char *) in[i], strlen(in[i]), buf);
            ASSERT(!strcmp(buf, out[i]));
        }
    }

    int __cdecl main(void) {
        test_base64_encode();
        ...
    }

{mongoose}/Makefile

    un:
    $(CC) test/unit_test.c -o unit_test -I. -I$(LUA) $(LUA_SOURCES) \
          $(CFLAGS) -g -O0
    ./unit_test

### <a id="unit_test_android"></a>Android UnitTest

$ANDROID/packages/apps中为每个app提供了单元测试的包tests

1. 编译测试工程：

        . build/envsetup
        mmm packages/apps/Camera/tests
2. 安装测试apk：

        adb install out/target/product/generic/data/app/CameraTests.apk
3. 查看已安装的测试用例：

        adb shell pm list instrumentation
4. 通过`adb shell`执行测试用例：

        adb shell am instrument -w com.android.camera.tests/android.test.InstrumentationTestRunner
测试用例集合的名称对应于tests工程中manifest文件中定义的Instrumentation的android name属性

        <instrumentation android:name="com.android.camera.CameraLaunchPerformance"
            android:targetPackage="com.android.camera"
            android:label="Camera Launch Performance" />

        <instrumentation android:name="com.android.camera.stress.CameraStressTestRunner"
            android:targetPackage="com.android.camera"
            android:label="Camera Launch Performance" />

        <instrumentation android:name="android.test.InstrumentationTestRunner"
            android:targetPackage="com.android.camera"
            android:label="Tests for Camera application."/>

refs: [Android Testing Framework](http://developer.android.com/tools/testing/index.html)

示例：Camera工程unittest（注：AOSP Gingerbread）

_$ANDROID/packages/app/Camera/tests/src/com/android/cameras/UnitTests.java_

    package com.android.camera;

    import android.test.suitebuilder.UnitTestSuiteBuilder;
    import junit.framework.Test;
    import junit.framework.TestSuite;

    /** TestSuite for all Camera unit tests. */
    public class UnitTests extends TestSuite {

        public static Test suite() {
            return new UnitTestSuiteBuilder(UnitTests.class)
                .includePackages("com.android.camera.unittest",
                        "com.android.camera.gallery")
                .named("Camera Unit Tests")
                .build();
        }
    }
_$ANDROID/packages/app/Camera/tests/src/com/android/cameras/unittest/CameraTest.java_

    package com.android.camera.unittest;

    import com.android.camera.Camera;
    import android.test.suitebuilder.annotation.SmallTest;
    import junit.framework.TestCase;

    @SmallTest
    public class CameraTest extends TestCase {
        public void testRoundOrientation() {
            assertEquals(0, Camera.roundOrientation(0));
            assertEquals(0, Camera.roundOrientation(0 + 44));
            assertEquals(90, Camera.roundOrientation(0 + 45));
            assertEquals(90, Camera.roundOrientation(90));
            assertEquals(90, Camera.roundOrientation(90 + 44));
            assertEquals(180, Camera.roundOrientation(90 + 45));
            assertEquals(180, Camera.roundOrientation(180));
            assertEquals(180, Camera.roundOrientation(180 + 44));
            assertEquals(270, Camera.roundOrientation(180 + 45));
            assertEquals(270, Camera.roundOrientation(270));
            assertEquals(270, Camera.roundOrientation(270 + 44));
            assertEquals(0, Camera.roundOrientation(270 + 45));
            assertEquals(0, Camera.roundOrientation(359));
        }
    }

## <a id="repo_reg_test"></a>repo Regression Test

对于一些试错成本较高的修改发布，引入回归测试是一个很好的办法，可以提高发布信心、确保工具在整个维护周期都保持基本功能正常。

对于repo工具，为确保加入ACL后基本功能正常，测试如下用例：

1. `repo init`，判断repo文件目录是否符合预期
2. `repo sync`，判断本地版本库文件目录是否符合预期
3. `repo list`，判断输出是否符合manifest中定义的project
4. `repo start/repo branch`，判断新建分支操作是否符合预期
5. `repo abandon`，判断删除分支操作是否符合预期
6. `repo status`，判断版本库状态查看是否符合预期
7. `repo forall`，判断针对所有Git库执行命令是否符合预期
8. `repo upload`，提示手动测试（assertFail）

**repo**是一个脚本工具集，上述每一个TC的预期输入就是执行相应repo命令；判断用例是否执行成功，则可以通过如对比目录结构、或判断repo操作的stdout/stderr是否符合预期。

注：测试开发可以使用多种语言、甚至无需xUnit框架。测试用例的核心在于调用程序的命令行接口，通过断言_assert_来判断用例_pass/fail_.  命令行接口（CLI）可以理解为程序内部API的前端_frontend_，对于一些没有提供CLI的待测工程，可以写一个如main入口来支持面向黑盒的测试。

refs: [PyUnit](http://docs.python.org/2/library/unittest.html)框架

为了实现本地repo操作，首先需要一个桩模块_MockRemoteRepo_、以模拟_Remote Git Server_，测试执行时从该Mock版本仓库中同步代码。

    class MockRemoteRepo():
        """
        Create a Mock Remote Repository(remote git server),
        containing manifest Git repository, and 4 Git repository.
        """

        def setUp(self):
            """Setup TestCases Environment
            """
            pass

        def cleanUp(self):
            """Clean Mock Repository
            """
            pass

        def createManifest(self):
            """Create a manifest repo with default.xml
            """
            pass

        def createRepo(self, repoName="repo1", commitMsg="test1"):
            """Create each git repo, this is our mock source repo.
            """
            pass

测试用例类（RepoStagingTestCase），继承unittest模块中TestCase类
>TestCase类的实例是可以运行test method / set-up / tidy-up代码的对象。<br />
TestCase实例的测试代码必须是自包含的(self-contained)，换言之，它可以单独运行或与其它任意数量用例共同运行。

    class RepoStagingTestCase(unittest.TestCase):

        def setUp(self):
            """Setup test environment.
            """
            pass

        def tearDown(self):
            """Tidy up everything.
            """
            pass

        def repoSync(self):
            """Some repo commands rely on Sync first.
            """
            pass

        def test_repo_init(self):
            """Test for 'repo init -u url' command.
            """
            pass

        def test_repo_sync(self):
            """Test for 'repo sync' command.
            """
            pass

        def test_repo_list(self):
            """Test for 'repo list' command.
            """
            pass

        def test_repo_start_branch(self):
            """Test for 'repo start' and 'repo branch' command.
            """
            pass

        def test_repo_abandon(self):
            """Test for 'repo abandon' command.
            """
            pass

        def test_repo_status(self):
            """Test for 'repo status' command.
            """
            pass

        def test_repo_forall(self):
            """Test for 'repo forall -c {git command}'.
            """
            pass

        def test_repo_upload(self):
            """Test for 'repo upload' command.
            """
            pass
>其中setup可以理解为hook method，初始化每次测试所需依赖的环境。每个TC在用例类中对应一个test开头的方法，框架会自动执行测试。 <br />
要注意的是，每个test method间是不应该有依赖关系的，因为框架执行顺序不是由代码顺序决定的（和AndroidInstrumentation/Junit等等都一样）。 <br />
tearDown会在每个test method执行完进行。

测试用例示例：

_TC-repo init_

    def test_repo_init(self):
        """Test for 'repo init -u url' command.
        """
        # Check for .repo
        _dirExpect = ['manifests.git', 'manifests', 'manifest.xml', 'repo']
        assert os.listdir(".repo").sort() == _dirExpect.sort()

_TC-repo forall_

    def test_repo_forall(self):
        """Test for 'repo forall -c {git command}'
        """
        # repo sync first
        self.repoSync()

        # repo forall
        repo_forall_cmd1 = 'repo forall -c git ls-files'
        ps = Popen(repo_forall_cmd1, shell=True, stdout=PIPE, stderr=PIPE)
        output, err = ps.communicate()
        _resultExpect1 = 'test\ntest\ntest\ntest\n'
        assert _resultExpect1 == output
>若assertion为假，抛出AssertionError异常，且测试框架会认为用例执行失败。其他非assert检查抛出的异常被认为是errors.

聚合成测试套件（Test Suite）并执行测试：

    if __name__ == "__main__":
        suite = unittest.makeSuite(RepoStagingTestCase, 'test')
        runner = unittest.TextTestRunner()
        runner.run(suite)

测试结果示例：

    FAIL: test_repo_start_branch (__main__.RepoStagingTestCase)
    Test for 'repo start' and 'repo branch' command.
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "test_repo.py", line 175, in test_repo_start_branch
        assert fout == _resultExpect
    AssertionError

    ======================================================================
    FAIL: test_repo_upload (__main__.RepoStagingTestCase)
    Test for 'repo upload' command
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "test_repo.py", line 243, in test_repo_upload
        assert False, "Please Test Repo Upload manually"
    AssertionError: Please Test Repo Upload manually

    ----------------------------------------------------------------------
    Ran 8 tests in 113.711s

    FAILED (failures=2)

## <a id="video_format_test"></a>音视频自动化测试脚本

#### 测试需求
- 测试动机： <br />
    多媒体资源库中有600多个音视频测试资源文件，对于快速迭代过程中的回归测试（如Hot-fix软件发布），脚本实现媒体库所有资源文件的自动化测试需求。
- 测试步骤： <br />
    上传待测媒体文件至SDCard、调用播放器播放、记录是否播放成功、删除媒体文件。
- 测试定位： <br />
    支持我司T881、T882、TP-Mini、T881v2、T882v2

#### 数据结构
对于不同平台分别用字典声明播放器包名、Activity名：

    t881 = {
        'platform'  :'t881',
        'pkg_name'  :'com.cooliris.media',
        'cls_name'  :'com.cooliris.media/.MovieView'
        }
    t882 = {
        'platform'  :'t882',
        'pkg_name'  :'com.android.gallery3d',
        'cls_name'  :'com.android.gallery3d/.app.MovieActivity'
    }
测试资源声明：

    test_resources = [
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\视频分辨率和码率',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\视频分辨率和帧率',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\视频和音频组合',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\RGB',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\视频档次',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\1080P视频',
        r'\\172.31.130.17\MobilePublic\软件部\多媒体资源库\超高码率1080P视频'
        ]
脚本递归测试该目录中每一个媒体文件；当添加或更新媒体库测试项或测试资源时，改写全局变量test_resources即可。

#### 功能说明
测试活动核心操作通过adb shell实现，如判断设备就绪、上传文件、调用图库播放视频、判断Log等

- 判断设备就绪：

        def checkDevice():
            var = os.popen('adb devices').read().strip()
            if not var.endswith("device"):
                print "Devices Not Available"
                sys.exit()
- 上传文件：

        def uploadFile(filepath, fileName):
            cmd_push = r'adb push "' + filepath + os.sep + fileName + r'" /sdcard/'
            os.popen(cmd_push)
            filelist = os.popen('adb shell ls /sdcard/').read().strip()
            if fileName in filelist:
                return True
            else:
                return False
- 控制Activity生命周期：

        # Start Activity
        cmd_start = r'adb shell am start -W -n ' + cls_name + ' -d ' + uri
        os.popen(cmd_start)

        # Kill process
        pid_list = os.popen(r'adb shell ps').read().strip().split()
        if pkg_name in pid_list:
            index = pid_list.index(pkg_name) - 7
            cmd_stop = 'adb shell kill ' + pid_list[index] # Debug mode needed!
            os.popen(cmd_stop)
    测试完成后中止播放器进程，`adb shell kill <pid>`需要Debug Mode下执行 <br />
    uri的获取，是在完整文件名路径字符串前加上`file://`协议：

        uri = r'file://mnt/sdcard/' + "'" + filename + "'"
- 判断是否播放成功： <br />
    需要对VideoPlayer进行定制，在onError回调中输出定制的Log信息，以便脚本在shell环境获取Log进而判断是否播放成功：

        @Override
        public boolean onError(MediaPlayer mp, int what, int extra) {
            Log.e(TAG, ERROR_MSG);
            ...
        }
    脚本中捕获Log并判断：

        # Clear logcat first
        cmd_clear_log = 'adb logcat -c'
        os.popen(cmd_clear_log)

        cmd_log_filter = 'adb logcat -d -s \'MoviePlayer\''
        log_msg = os.popen(cmd_log_filter).read()

        if not log_msg.find('Unable to play video') is -1:
            print '\t --- 该视频无法播放! ---\n'
            if auto_test:
                fd.write(',不支持\n')
        else:
            if auto_test:
                print '\t --- 该视频可以播放! ---\n'
                fd.write('\n')
