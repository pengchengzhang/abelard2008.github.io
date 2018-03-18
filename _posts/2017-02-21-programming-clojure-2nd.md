---
layout: post
status: publish
published: true
title: "Programming Clojure 2nd 笔记"
author: Dillon Peng
date: Tue Feb 21 14:28:49 CST 2017
comments: []
---

书中一些例子直接在*cider-repl*中敲入并不能得到想要的结果,[https://github.com/abelard2008/programming-clojure-2nd](https://github.com/abelard2008/programming-clojure-2nd) 
### 如何完整运行*6.4 Datatypes*中p155 *(gulp vault)*
- *6.3 Protocols*最后的示例代码存入
[io.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj),
跟书上略有区别的是，为了与
[cryptovault-complete.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj)
中的*examples.protocols.io*保持一致，将*examples.io*改成*examples.protocols.io*

- *6.4 Datatypes*最后部分，从*p155*的 `src/examples/cryptovault-complete.clj` 开始的代码存入
  [cryptovault-complete.clj](https://github.com/abelard2008/programming-clojure-2nd/blob/master/chapter6/io.clj),
  特别注意，为了演示*(gulp vault)*, *cryptovault-complete.clj*最后几行为:


<?prettify?>
<pre class="lang-java" id="java_lang">
package foo;

import java.util.Iterator;

/**
 * the fibonacci series implemented as an Iterable.
 */
public final class Fibonacci implements Iterable<Integer> {
  /** the next and previous members of the series. */
  private int a = 1, b = 1;

  @Override
  public Iterator<Integer> iterator() {
    return new Iterator<Integer>() {
      /** the series is infinite. */
      public boolean hasNext() { return true; }
      public Integer next() {
        int tmp = a;
        a += b;
        b = tmp;
        return a;
      }
      public void remove() { throw new UnsupportedOperationException(); }
    };
  }

  /**
   * the n<sup>th</sup> element of the given series.
   * @throws NoSuchElementException if there are less than n elements in the
   *   given Iterable's {@link Iterable#iterator iterator}.
   */
  public static <T>
  T nth(int n, Iterable<T> iterable) {
    Iterator<? extends T> it = iterable.iterator();
    while (--n > 0) {
      it.next();
    }
    return it.next();
  }

  public static void main(String[] args) {
    System.out.print(nth(10, new Fibonacci()));
  }
}
</pre>

<?prettify?>
<pre>
    $(function() {
    window.prettyPrint && prettyPrint();
    });

    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
    (def vault (->CryptoVault "vault-file" "keystore" "toomanysecrets"))
    (init-vault vault)
    (expectorate vault "This is a test of the CryptoVault")
    (gulp vault)
</pre>




  
- 这时候，如果*emacs*在上面两个文件所在的目录，打开*cider-repl*,导
  入*cryptovault-complete.clj*, 会提示错误：
```sh
user> (clojure.main/load-script "cryptovault-complete.clj")
CompilerException java.io.FileNotFoundException: Could not locate examples/protocols/io__init.class or examples/protocols/io.clj on classpath., compiling:(/study/clojure/programming-clojure-2nd/chapter6/cryptovault-complete.clj:1:1) 
user> 

只要将下面这行：
```clojure
(clojure.main/load-script "io.clj")
```
添加到*cryptovault-complete.clj*的第一行, 问题解决：
```sh
user> (clojure.main/load-script "cryptovault-complete.clj")
"This is a test of the CryptoVault"
user> 
```


