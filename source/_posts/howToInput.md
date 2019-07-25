---
title: 开心造轮子——一个输入法的输入方法查询器+码表更新
tags: [perl6, 轮子]
date: 2017-04-09 14:16:11
---


动机
----

近日学习小鹤的双拼+双型。学习曲线比较陡峭。好多时候一个字不会打，只好查码表。有的时候又觉得码表不够合理，想要自己增加或修改。然而手动的方式我是无法接受的，遂造轮子。

前言
---

我用的是fcitx，默认支持小鹤的双拼，但不支持鹤型。我使用的是贴吧某[大神](http://tieba.baidu.com/p/3627553057)做的码表。在此感谢。
码表输入法就是在查表，十分简单粗暴。所以有了码表，我们自己也可以查。它的本体(txt)大概长这样：
```
aae     阿                                                                                      
aaek    阿
```

知识点
---

控制流什么的就不写了。。。

* perl6 Opretor [//](https://docs.perl6.org/language/operators#infix_%2F%2F)
* perl6 [socket](https://docs.perl6.org/type/IO::Socket::INET)

更新码表
---

刚才展示的是码表的原文件，是txt格式，而fcitx需要的是mb文件。好在openSUSE下有工具可以相互转换。所以更新的程序很简单,没有技术含量。这里略过（虽然查询也简单）。

查询
---

简单的查询就是读入码表，正则一下。大概这样子：
``` perl6
use v6;
my @mb = 'PATH_TO_FILE.txt'.IO.lines;
sub MAIN(Str $str, :$all? = False) {
    my $regex = / (\w ** 3..4) \s+ $str $ /;
    if $all {
        $regex = / (\w ** 0..4) \s+ $str $ /;
    }
    for @mb -> $line {
        if $line ~~ $regex {
            say $line;
            last unless $all
        }
    }
}
```

因为运行得太慢，所以我加了个选项，使得只有--all的时候才查完整个文件，否则只查一个较接近全码的结果（个人需求是先学习全码）。
可是这样用了一天还是慢得不能忍受，所以我又重写了个版本。这次的思路是：一个服务端读完整个文件，在后台挂起来；一个客户端通过socket去访问它。

更快的查询
---

服务端代码如下
```perl6
# file hti_service.p6
use v6;                                                                                         

sub initMap(--> Hash) {                                                                         
    my %wordToCode;                                                                             

    for 'PATH_TO_FILE.txt'.IO.lines -> $line {
        if $line ~~ / (\w ** 3..4) \s+ (\w+) $ / {
            %wordToCode{$1}.push: $0;
        }
    }
    %wordToCode
}

sub MAIN($port) {
    my %wordToCode = initMap;
    my $listen = IO::Socket::INET.new(:listen, :localhost<localhost>, :localport($port));
    loop {
        my $conn = $listen.accept;
        while my $word = $conn.recv {
            for %wordToCode{$word} { $conn.print: $_ // "not found"}
        }
        $conn.close;
    }
}
```

`initMap`来建立hash，文字为键，输入方法的列表为值。
这里可以解释的语法只有 `//` 操作符： 如果左边的东西是个`Any`，那么就用右边的东西。

相应地客户端程序如下：
```perl6
# file hti_client.p6
use v6;

sub MAIN($str) {
    my $conn = IO::Socket::INET.new(:host<localhost>, :port(7890));
    $conn.print: $str;
    $conn.recv.say;
    $conn.close;
}
```

这个最傻，把参数直接丢给socket，回答是什么就输出什么。

最后，给他们分别套层壳子。
``` bash
# file hti

#!/bin/bash
perl6 hti_client.p6 "$@"
```

``` bash
# file hti_service

#!/bin/bash

# get pid or empty
port=7890
pid=`lsof -i:$port | tail -n 1| awk '{print $2}'`

# main
if [ $# -eq 0 ]; then
    if [ -z $pid ]; then
        perl6 hti_service.p6 "$port" &
    else
        echo "running"
    fi
elif [ $1 == "off" ]; then
    if [ -z $pid ]; then
        echo "not running"
    else
        kill -9 $pid
    fi
fi
```