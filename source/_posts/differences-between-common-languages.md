---
title: 几种常用语言的常用语法总结 (一)
date: 2017-03-02 16:45:13
tags: programming
---

---------
说明
======
---------
<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  ```
  python
  ``` python
  ```
  perl
  ``` perl
  ```
  perl6
  ``` perl6
  ```
  Haskell
  ``` Haskell
  ```
</details>


> 列举的语言目前有c++, python2, perl, perl6, Haskell
 如果操作是修改内容的，标记为M

---------
列表操作
=======
---------

> 列表以l表示

空列表声明
---------

<details>
  <summary>show/hide</summary>

  c++
  ``` c++
  std::vector<T> l;
  ```

  python
  ``` python
  l = []
  ```

  perl
  ``` perl
  @l = ();
  ```

  perl6
  ``` perl6
  @l = ();
  ```

  Haskell
  ``` Haskell
  []
  ```
</details>

长度为n，所有元素都相同的字符串声明
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  std::vector<T> l(n,defalt_value);  
  ```
  python
  ``` python
  l = [defalt_value]*n
  ```
  perl
  ``` perl
  @l = (defalt_value) x n;
  ```
  perl6
  ``` perl6
  @l = (defalt_value,defalt_value...Inf)[0..n] # not so elegant
  ```
  Haskell
  ``` Haskell
  replicate n defalt_value
  ```
</details>

列表长度
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  l.size();
  ```
  python
  ``` python
  len(l)
  ```
  perl
  ``` perl
  scalar @l;
  ```
  perl6
  ``` perl6
  +@l;
  ```
  Haskell
  ``` Haskell
  length
  ```
</details>

列表是否为空
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  l.empty();
  ```
  python
  ``` python
  len(l) == 0
  ```
  perl
  ``` perl
  @l == 0
  ```
  perl6
  ``` perl6
  @l==Empty # or @l==() or @l.elems == 0
  ```
  Haskell
  ``` Haskell
  null
  ```
</details>

整数递增的列表
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  std::vector<int> l(n+1);
  std::iota(std::begin(l), std::end(l),0);
  ```
  python
  ``` python
  range(n+1)
  ```
  perl
  ``` perl
  (0..n);
  ```
  perl6
  ``` perl6
  (0..$n)
  ```
  Haskell
  ``` Haskell
  [0..n]
  ```
</details>

反向的列表
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  std::reverse(l.begin(),l.end()); // [M!]
  ```
  python
  ``` python
  l.reverse() # [M!]
  ```
  perl
  ``` perl
  reverse @l;
  ```
  perl6
  ``` perl6
  @l.reverse; # or reverse @l;
  ```
  Haskell
  ``` Haskell
  reverse
  ```
</details>

升序排序的列表
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  std::sort(l.begin(),l,end());
  ```
  python
  ``` python
  sorted(l)
  ```
  perl
  ``` perl
  sort {$a<=> b} @l;
  ```
  perl6
  ``` perl6
  @l.sort;    # or sort @l;
  ```
  Haskell
  ``` Haskell
  sort
  ```
</details>

遍历列表的元素
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  for (const /* or not */ auto & /* no not */ i: l) {
      // dor you stuff
  }
  ```
  python
  ``` python
  for i in l:
      # do your stuff
  ```
  perl
  ``` perl
  for my $i(@l) {
    # do your stuff
  }
  ```
  perl6
  ``` perl6
  for @l -> $i {
    # do your stuff
  }
  ```
  Haskell
  ``` Haskell
  # maybe you can use forM_ from Control.Monad
  ```
</details>

从首端插入一个元素
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  l.insert(l.begin(),value);
  ```
  python
  ``` python
  l.insert(0,value)
  ```
  perl
  ``` perl
  unshift @l, $value;
  ```
  perl6
  ``` perl6
  unshift @l, $value;
  ```
  Haskell
  ``` Haskell
  value : l
  ```
</details>

从尾端插入一个元素
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  AB.reserve( A.size() + B.size() ); // preallocate memory
  AB.insert( AB.end(), A.begin(), A.end() );
  AB.insert( AB.end(), B.begin(), B.end() );
  ```
  python
  ``` python
  l.append(value) # [M!]
  ```
  perl
  ``` perl
  push @l, $value
  ```
  perl6
  ``` perl6
  @l.push: $value; #or push @l, $value;   # [M!]
  ```
  Haskell
  ``` Haskell
  l ++ value
  ```
</details>

连接两个列表
----------------------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  std::vector<T> l;
  ```
  python
  ``` python
  l1+l2
  ```
  perl
  ``` perl
  (@l1,@l2)     # or push @l1, @l2 # [M!] for l1
  ```
  perl6
  ``` perl6
  (@l1,@l2).flat
  ```
  Haskell
  ``` Haskell
  l1 ++ l2
  ```
</details>

---------
字符串操作
=======
---------

空字符串
---------
<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  ""
  ```
  python
  ``` python
  ""
  ```
  perl
  ``` perl
  ""
  ```
  perl6
  ``` perl6
  ""
  ```
  Haskell
  ``` Haskell
  ""
  ```
</details>

字符串长度
---------
<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  str.length();
  ```
  python
  ``` python
  len(str)
  ```
  perl
  ``` perl
  length $str;
  ```
  perl6
  ``` perl6
  $str.chars;
  ```
  Haskell
  ``` Haskell
  length
  ```
</details>

字符串拼接
---------

> s1 与 s2 拼接

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  s1+s2;
  ```
  python
  ``` python
  s1 + s2
  ```
  perl
  ``` perl
  $s1.$s2;
  ```
  perl6
  ``` perl6
  $s1 ~ $s2;
  ```
  Haskell
  ``` Haskell
  ++
  ```
</details>

子串
---------

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  str.substr(l,len);
  ```
  python
  ``` python
  str[l:l+len]
  ```
  perl
  ``` perl
  substr $str, $l, $len;
  ```
  perl6
  ``` perl6
  substr $str, $l, $len;
  ```
  Haskell
  ``` Haskell
  take len $ drop l
  ```
</details>

从首部添加一个字符
---------

> ch 插入 str

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  str.insert(str.begin(),ch); // [M!]
  ```
  python
  ``` python
  ch + str
  ```
  perl
  ``` perl
  $ch.$str;
  ```
  perl6
  ``` perl6
  $ch ~ $str
  ```
  Haskell
  ``` Haskell
  char : str
  ```
</details>

从尾部添加一个字符
---------

> ch 插入 str

<details>
  <summary>show/hide</summary>
  c++
  ``` c++
  str.push_back(ch);
  ```
  python
  ``` python
  str + ch
  ```
  perl
  ``` perl
  $str.$ch;
  ```
  perl6
  ``` perl6
  $str ~ $ch
  ```
  Haskell
  ``` Haskell
  str ++ [ch]
  ```
</details>

