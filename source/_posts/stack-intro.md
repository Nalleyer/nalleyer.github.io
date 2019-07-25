---
title: 2018年如何与haskell玩耍
date: 2018-09-12 20:33:30
tags: [haskell, stack]
---

前言
===

陆陆续续跟haskell玩耍了几年，自己的水平一直没有什么质的飞跃。不过摸爬滚打中，算是搞清了究竟怎么配开发环境这件十分基础的事情。索性写了这篇博客，希望新入坑的小伙伴们，不要再浪费那么多时间去倒腾。

如果你只想快速上手开始写代码，请直接根据 **短答案** 中的指引进行操作。当然如果你对其中的原理有点兴趣，可以看后面的正文。

短答案
===

1. 安装[**stack**](https://www.haskellstack.org/)。
2. 安装[atom](https://atom.io/)。
3. 找一个目录，打开命令行。
4. `stack --resolver=lts-8.13 new MyProject`。
5. `cd MyProject`。
6. atom中安装atom-haskell这个包
7. `stack build ghc-mod hoogle hasktags pointfree pointful hindent`
8. `stack build`
9. `stack exec MyProject-exe`

构建工具
===

如果你写过c++，你可能会感到拥有标准库就已经很舒服了。实在要用个什么库，下一坨代码放到工程里，include一下也基本能用。要是下了个用了新特性的库不能用，升级升级编译器也就ok。

而写haskell，会觉得基础得不能再基础的包，都不包含在Prelude里面，都要你去下。而且会发生某个包觉得你的编译器版本太新而无法使用的情况。

基于这样的不同，写haskell我并不建议在[官网](https://www.haskell.org/)下载haskell platform，或者下载ghc。请直接使用构建工具[**stack**](https://www.haskellstack.org/)。

与stack玩耍
===

LTS
---

stack存在的意义在于解决包版本的冲突问题，让构建工程顺利完成。它完成这件事的方法是维护了一堆不同的“大版本”，每个“大版本”对应一大堆各种包的特定版本，并保证在这个大版本下面，这些特定版本的包互相没有任何冲突。
在这个前提下，用户选定了这个“大版本”之后，只可能发生某个包不存在的情况，而不可能发生冲突。

这个大版本，学名叫Long Term Support (LTS)。具体的定义请参见[这个网页](https://www.stackage.org/)。

那我们在哪里配置使用哪个大版本呢？
当我们使用stack的时候，要么在一个特定的工程下，要么在“全局工程”这个特殊的工程下。每一个工程都有一个配置文件，里面说明了当前的LTS是什么版本。

基本配置
---

通过查看`STACK_ROOT`这个环境变量，可以知道stack的全局配置放在哪里。
一般windows是在`c:\sr\`，linux在`~/.stack`。

* 为了下载速度，我们首先添加tuna的源：
打开[这个网页](https://mirror.tuna.tsinghua.edu.cn/help/stackage/)，把它给的那段代码贴到`config.yaml`最后面。

* 全局工程配置：
打开`global-project/stack.yaml`，可以更改全局工程的lts版本。

命令
---

stack最常用的命令就这几个：

* `stack new xxx`创建一个工程的脚手架。
* `stack build`编译你的工程。
* `stack exec xxx-exe`运行编译出的可执行程序。这里`xxx-exe`是目标可执行文件的默认名字，同样可以在`stack.yaml`中配置。

`stack new xxx`之后，生成两个配置在xxx目录下，一个是`stack.yaml`，里面可以配置lts；另一个是`package.yaml`，里面的dependencies需要填写工程需要安装的包。
注意，`import`后面跟的名字与这里添加的包名是不同的。例如`import Data.Aeson`，但包名叫做aeson。

* `stack install xxx`这个命令，看上去像是安装一个包的常用命令，其实它并不是真正的安装。在工程中如果需要使用某个包的可执行程序，那么使用`stack build xxx`即可，使用的时候使用`stack exec xxx`即可使用。
而`stack install xxx`仅仅是将xxx拷贝到全局可访问的一个path下，让你在工程以外也能使用xxx。因此在不同的工程下（不同的lts），安装出来的xxx版本有差异。

编辑器
---

haskell的类型系统可以说是它的一个核心，在编辑器中直接查看函数的类型是一件多么美妙的事情。此外，一个配置好的编辑器，还有实时检查，重构建议等等，能提高不少生产力。

现在的各路编辑器实现haskell类型查看等等功能的插件，其逻辑基本都不是插件本身实现的，而是依赖通用的后台工具。
例如 **ghc-mod** 是起到关键作用的一个工具。我们配置编辑器的一个重点就是找到一个能用的ghc-mod插件。

而插件有两种，一种是认识stack的，一种是不认识的。认识stack的，在工程下能找到`stack build xxx`安装的xxx；不认识的就只能在全局的path里面找。所以当插件需要你安装一个工具，例如ghc-mod的时候，你在工程里面`stack build ghc-mod`就行，如果发现插件不能找到ghc-mod，就再运行一下`stack install ghc-mod`。但是这样的缺点就是在不同工程下面，如果忘记`stack install ghc-mod`，插件就可能使用了错误版本的ghc-mod。

lts-8.13
---

我建议的lts版本是8.13，如果使用默认的最新lts，会导致没有ghc-mod。你可以在[stackage](https://www.stackage.org/)里面搜索lts，在其目录下查看有没有某一个包。8.13只是一个普通的使用ghc8又包含ghc-mod的lts，你当然可以用其他的版本，但为了编辑器插件，ghc-mod必须要有。

配置编辑器
===

不论你用什么编辑器，最基本的情况，安装以下插件即可：

* 语法高亮
* ghc-mod

其他的功能多多益善。

在这里我推荐一下atom，只需要像短答案那样，安装atom-haskell这个包，然后build一堆工具就行。
代码中鼠标悬停稍等片刻就可以查看函数的类型，十分安逸。

全局玩耍
===

如果感到每次建立一个工程很麻烦，那么可以比较简单地使用`stack --resolver=8.13 install ghc`，安装一个全局的ghc。
但是现在的硬盘又不那么金贵，干活直接建个脚手架感觉代价也不那么大，还自动生成了.gitignore之类的方便的东西，何乐而不为。

后记
===

本文不是那么严谨，全是个人经验，如果不work了，可通过页面底部的联系方式告诉我。
