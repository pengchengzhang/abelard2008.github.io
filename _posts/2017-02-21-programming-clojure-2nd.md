---
published: true
title: "Programming Clojure 2nd 笔记"
author: Dillon Peng
date: Tue Feb 21 14:28:49 CST 2017
comments: []
---
阅读该书时，有些例子直接在*cider-repl*中敲入并不能得到想要的结果

### 如何完整运行*6.4 Datatypes*中ｐ155 *(gulp vault)*
- *6.3 Protocols*最后的示例代码存入
[io.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj),
跟书上略有区别的是，为了与
[cryptovault-complete.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj)
中的*examples.protocols.io*保持一致，将*examples.io*改成*examples.protocols.io*

- *6.4 Datatypes*最后部分，从*p155*的
  `src/examples/cryptovault-complete.clj`开始的代码存入
  [cryptovault-complete.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj),
  特别注意，*cryptovault-complete.clj*最后几行为:
```clojure
(def vault (->CryptoVault "vault-file" "keystore" "toomanysecrets"))
(init-vault vault)
(expectorate vault "This is a test of the CryptoVault")
(gulp vault)
```
  
- 这时候，如果*emacs*在上面两个文件所在的目录，打开*cider-repl*,导
  入*cryptovault-complete.clj*, 会提示错误：
```sh
user> (clojure.main/load-script "cryptovault-complete.clj")
CompilerException java.io.FileNotFoundException: Could not locate examples/protocols/io__init.class or examples/protocols/io.clj on classpath., compiling:(/study/clojure/programming-clojure-2nd/chapter6/cryptovault-complete.clj:1:1) 
user> 
```
只要将下面这行：
```clojure
(clojure.main/load-script "io.clj")
```
添加到*cryptovault-complete.clj*, 问题解决：
```sh
user> (clojure.main/load-script "cryptovault-complete.clj")
"This is a test of the CryptoVault"
user> 
```


