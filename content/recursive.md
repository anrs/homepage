Title: 双递归和树形递归
Date: 2008-09-13 09:23
Category: blog
Summary:

双递归

双递归过程在过程体内两次引用其自身；而树形递归过程的计算则在过程体中分裂为 N 路分支，各自引用自身以完成运算，最后汇总得出过程本身的计算结果。可以想见，当 N 等于 2 时，那树形递归过程也是个双递归过程，但双递归过程也可能是个树形递归过程吗？这个就不一定，还是得看具体的计算进展方式了。比如 Ackermann 函数。

```erlang
ackermann(X, Y) ->
  if ->
    Y == 0 ->
      0;
    X == 0 ->
      2 * Y;
    Y == 1 ->
      2;
    true ->
      f(X - 1, f(X, Y - 1))
  end.
```

从 ackermann 函数的过程定义来看，很明显的这是个双递归过程，接着来分析具体的计算进展方式以观察它是否也是个树形递归。对于 ackermann(1, 5) 这次过程调用，计算进展方式如下：

```erlang
ackermann(0, ackermann(1, 4)
ackermann(0, ackermann(0, ackermann(1, 3))
ackermann(0, ackermann(0, ackermann(0, ackermann(1, 2))))
ackermann(0, ackermann(0, ackermann(0, ackermann(0, ackermann(1, 1)))))
ackermann(0, ackermann(0, ackermann(0, ackermann(0, 2))))
ackermann(0, ackermann(0, ackermann(0, 4)))
ackermann(0, ackermann(0, 8))
ackermann(0, 16)
32
```

整个计算进展方式呈现出了线性递归的先扩张后收缩的特征，并没有分裂为多路分支，那 ackermann 函数就不是树形双递归过程。（事实上，Ackermann 函数在 X 值等于 1 和大于 1 的不同情况下，会展现出不同的函数计算特性，在 X 大于 1 时，运算结果也会突飞猛进，最后遁入破碎虚空）

树形递归

典型的树形递归当属 Fibonacci 数列无疑，最简单的 Fibonacci 函数如下：

```erlang
fibonacci(0) ->
  0;
fibonacci(1) ->
  1;
fibonacci(N) ->
  fibonacci(N - 1) + fibonacci(N - 2).
```

过程定义自然分裂为两路分支，对传入的 N 值，一路计算 fibonacci(N - 1) 另一路则计算 fibonacci(N - 2)，最后相加得出 fibonacci(N) 的结果。这个文字描述干瘪无味，观察 fibonacci(4) 的计算进展方式的图形描述就直观了许多。

从上图还可以发现树形递归的一个重大问题－－效率。在上图中求解 fibonacci(3) 的计算过程几乎占到了全部计算过程一多半的运算量，但它却整整被运行了两次，如此可以看出，当 N 值足够大时，整个求解过程中重复的运算量有多少了！同样的，也可以把这个树形递归改造成迭代计算进展方式的尾递归过程。

```erlang
fibonacci(N) ->
  if
    N < 1 ->
      exit(argerror);
    true ->
      fib_iter(1, 0, N)
  end.

fib_iter(Product, LastProduct, Counter) ->
  if
    Counter == 1 ->
      Product;
  true ->
    fibonacci(Product + LastProduct, Product, Counter - 1)
  end.
```

同样的，对 fibonacci(4) 的调用，整个计算进展方式变化为：

```erlang
fibonacci(1, 0, 4)
fibonacci(1, 1, 3)
fibonacci(2, 1, 2)
fibonacci(3, 2, 1)
```

改进后的 fibonacci 过程充分展现了尾递归的高效率，在我的机器上尾递归的版本迟滞不到 1 秒即可求出 N = 100000 时的结果值，而树形递归在求 N = 50 时就已经麻木不仁了（想想 N ＝ 50 时过程体内会分裂出多少路分支计算也就释然了）～～

虽然计算进展方式变得更简单了，空间占用更低，但过程体的代码却变得复杂了许多，不如树形递归简洁明了，适合人类阅读；而树形递归过程甚至只是自然而然的、直觉的反映了我们大脑中对算法的想象，所以还是有其实际应用意义的。再者依靠编译器优化措施也可以解决树形递归大部分的效率问题，比如为树形递归的运算结果维护一张对应表，每次计算时都到表中查找是否有相应的计算结果，没有才进行实际的递归运算，运算结果出来后再将这个结果存入对应表，以便将来查询。比如对于 fibonacci(4) 的调用来说，第一次执行完 fibonacci(2) 之后便将计算结果存入对应表，第二次执行前先查看对应表中是否存在相应的计算结果，发现已存在的话便直接取出来应用就可以了。

p.s. Ackermann 函数的破碎虚空，想想这个函数的时间和空间的增长阶吧。

```erlang
ackermann(2, 4)
ackermann(1, ackermann(2, 3))
ackermann(1, ackermann(1, ackermann(2, 2)))
ackermann(1, ackermann(1, ackermann(1, ackermann(2, 1))))
ackermann(1, ackermann(1, ackermann(1, 2)))
ackermann(1, ackermann(1, ackermann(0, ackermann(1, 1))))
ackermann(1, ackermann(1, ackermann(0, 2)))
ackermann(1, ackermann(1, 4))
ackermann(1, ackermann(0, ackermann(1, 3)))
ackermann(1, ackermann(0, ackermann(0, ackermann(1, 2))))
ackermann(1, ackermann(0, ackermann(0, ackermann(0, ackermann(1, 1)))))
ackermann(1, ackermann(0, ackermann(0, ackermann(0, 2))))
ackermann(1, ackermann(0, ackermann(0, 4)))
ackermann(1, ackermann(0, 8))
ackermann(1, 16)
```

...（根据上面的求解可知 ackermann(1, N) 即 2 ^ N，因此省略接下来的计算进展方式）
65536

```erlang
ackermann(3, 3)
ackermann(2, ackermann(3, 2))
ackermann(2, ackermann(2, ackermann(3, 1)))
ackermann(2, ackermann(2, 2))
ackermann(2, ackermann(1, ackermann(2, 1)))
ackermann(2, ackermann(1, 2))
ackermann(2, ackermann(0, ackermann(1, 1)))
ackermann(2, ackermann(0, 2))
ackermann(2, 4)
ackermann(1, ackermann(2, 3))
ackermann(1, ackermann(1, ackermann(2, 2)))
ackermann(1, ackermann(1, ackermann(1, ackermann(2, 1))))
ackermann(1, ackermann(1, ackermann(1, 2)))
ackermann(1, ackermann(1, ackermann(0, ackermann(1, 1))))
ackermann(1, ackermann(1, ackermann(0, 2)))
ackermann(1, ackermann(1, 4))
ackermann(1, ackermann(0, ackermann(1, 3)))
ackermann(1, ackermann(0, ackermann(0, ackermann(1, 2))))
ackermann(1, ackermann(0, ackermann(0, ackermann(0, ackermann(1, 1)))))
ackermann(1, ackermann(0, ackermann(0, ackermann(0, 2))))
ackermann(1, ackermann(0, ackermann(0, 4)))
ackermann(1, ackermann(0, 8))
ackermann(1, 16)
ackermann(0, ackermann(1, 15))
```

...（同样滴，根据上面的求解可知 ackermann(0, ackermann(1, 15)) 即 ackermann(0, 2 ^ 15)，因此省略接下来的计算进展方式，直接给成运算结果）
65536

