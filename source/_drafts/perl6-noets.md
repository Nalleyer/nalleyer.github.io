---
title: perl6_noets
tags: [note, perl6]
---

p6系列知识点的草稿。同时供自己查阅
------

OO
====

* method vs submethods 

method就是该类的方法。可以共有或私有。

与之相对地，submethod不能被衍生的类继承，适合做一些特别的事情。

例： 构造函数BUILD应该是submethod。

``` perl
class C {
    method ai {
        say "i'm public";
    }
    method !mai {
        say "i'm private";
    }
    submethod mii {
        say "i'm submethod";
    }
}
class CC is C {
}
say ">> in C";
my $aC = C.new;
$aC.ai;
# output: i'm public
#$aC.mai;
# output: No such method 'mai' for invocant of type 'C'
$aC.mii;
# output: i'm submethod
say ">> in CC";
my $aCC = CC.new;
$aCC.ai;
# output: i'm public
#$aCC.mai;
# output: No such method 'mai' for invocant of type 'CC'
#$aCC.mii;
# output: No such method 'mii' for invocant of type 'CC'
```
