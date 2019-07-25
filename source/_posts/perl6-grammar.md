---
title: perl6笔记系列： grammar
tags: [perl6, note]
---

> prel6笔记系列记录我学习perl6过程中学到的一些 **要点**， 笔记系列不强调理论严谨，只求容易理解。
>
> 必要的基础知识将列于笔记的开头处，并附链接（我知道的话）

依赖知识
-------

* perl6 [面向对象](https://docs.perl6.org/language/classtut)
* perl6 [regex / token / rule](https://docs.perl6.org/language/regexes)


动机
-----

有时我们需要解析文本，首先想到的就是使用正则。但是，perl6中提供了更强大的工具：grammar。使用grammar可以以及简介（几乎接近文法或范式）的方式书写程序，轻松地解析文本。

结构
----

grammar一般需要两部分： grammar和action。
grammar负责匹配和解析，action负责使用匹配到的文本来构建数据结构。

接下来举一个简单的例子： 只有加法的表达式。规范一点的写法是：

```
Expr -> Expr + Var
    | Var
```

grammar
----

写成grammar可以是这样的：

``` perl6
grammar ExprAdd {
    token TOP  { ^ <Expr> $ };
    token Expr { [ <Expr> \s* '+' \s* <Var> ] | <Var> }; 
    token Var  { <alpha> };
}
```

可以这样运行：
``` perl6
    my $s = "a + b + c";
    say ExprAdd.parse($s);
```

当然这里会得到Nil。使用一下Debug工具：
``` perl6
use Grammar::Debugger;
```

容易发现出现了[左递归](https://en.wikipedia.org/wiki/Left_recursion)，但这是文法的问题，可以容易地解决：

``` perl6
grammar ExprAdd {
    token TOP  { ^ <Expr> $ };
    token Expr { [ <Var> \s* '+' \s* <Expr> ] | <Var> }; 
    token Var  { <alpha> };
}
```

运行程序可以看到这样的结果：
``` perl6
｢a + b + c｣
 Expr => ｢a + b + c｣
  Var => ｢a｣
   alpha => ｢a｣
  Expr => ｢b + c｣
   Var => ｢b｣
    alpha => ｢b｣
   Expr => ｢c｣
    Var => ｢c｣
     alpha => ｢c｣
```

这个结果显示了匹配的情况，但是这个结果没有什么实际用途。不过现在我们可以根据匹配结果做我们想要做的事了。

接下来可以从匹配的结果做很多事，例如化简表达式，或者建立一个数据结构。不过一般先建立数据结构，再做别的工作比较优雅。

建立数据结构（或直接干别的事）就是action的工作了。

action
-----

action是一个类，它的每个方法的名字对应grammar的匹配项目（token,regex,rule）的名字，并且参数是固定的$/，这个参数的值是该项目匹配到的字符串。

举例说明吧。假如我想把上面的表达式的变量存储到一个列表里。我可以这么写：
``` perl6
class ExprAddAction0 {
    method TOP ($/) { make $<Expr>.made; }

    method Expr ($/) {
        make [|$<Var>.made,|$<Expr>.made];
    }
    method Var ($/) {
        make [$/.Str];
    }
}
```

在parse的参数里加入这个acion：
``` perl6
my $m = ExprAdd.parse($s, actions => ExprAddAction0.new);
```

先解释一下action的语法。我们在`token TOP`中使用了`<Expr>`，把问题丢给`token Expr`，所以action中我们做同样的事。`make`可以当作是必须书写的句子（详细见[官网](https://docs.perl6.org/type/Match#method_make)）。`$<Expr>`代表`TOP`中被`<Expr>`匹配到的字符串。`$<Expr>.made`代表这些内容被`method Expr`处理后的结果。

再看`method Expr`， 它返回一个列表，第一个元素是`$<Var>.made`后面是展开的`$<Expr>.made`。这自然地形成了递归调用，生成匹配到的所有Var的列表。

`method Var`简单地返回匹配到的字符串。

运行发现，列表的最后有一个Nil（<s>这很Haskell</s>），可以简单地在`method TOP`里使用列表操作去掉它，但这并不优雅。

仔细查看action，可以发现，`token Expr`里面使用了`|`来进行选择，当匹配到`Var`时，就不存在`$<Expr>`这个变量，所以action里面必然有一个Nil。

解决办法是把`token Expr`的两种情况拆开，并使用一个类似函数重载的方式书写，这样就可以为不同情况写不同的action了。
``` perl6
grammar ExprAdd {                                                                               
    token TOP  { ^ <Expr> $ };                                                                  
    proto token Expr {*};
          token Expr:sym<just_a_var> { <Var> };     # <-- here
          token Expr:sym<a_expression> { <Expr0> };     # <-- here
    token Expr0 { <Var> \s* '+' \s* <Expr> };
    token Var  { <alpha> };
}

class ExprAddAction0 {
    method TOP ($/) { make $<Expr>.made; }

    method Expr:sym<a_expression> ($/) { make $<Expr0>.made; }     # <-- here
    method Expr:sym<just_a_var> ($/) { make $<Var>.made; }     # <-- here
    method Expr0 ($/) { make [|$<Var>.made,|$<Expr>.made]; }
    method Var ($/) { make [$/.Str]; }
}
```

注意sym后面的尖括号里的内容只是重载函数的标识符，用于区分不同的重载函数。与匹配到的内容无关，这里我专门写了不同的文字来说明这个问题。

总结
---

perl6的grammar很强大，写起来很爽很简单。