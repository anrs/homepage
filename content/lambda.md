Title: 无聊 — 比较下线性递归和尾递归、函数调用和 lambda 演算
Date: 2009-08-20 19:56
Category: blog
Summary:

不多话，直接上图上真相…

代码：

```python
#!/usr/local/bin/python2.6
#coding=utf-8
 
fib1 = lambda n: 1 if n < 3 else fib1(n – 2) + fib1(n – 1)
 
def fib2(n):
    if n < 3:
        return 1
    return fib2(n – 2) + fib2(n – 1)
 
def fib3(n):
    f = (lambda i, first, second:
             second if i >= n else f(i + 1, second, first + second))
    return f(1, 0, 1)
 
def fib4(n):
    def _fib4(i, first, second):
        if i >= n:
            return second
        return _fib4(i + 1, second, first + second)
    return _fib4(1, 0, 1)
 
 
if __name__ == ‘__main__’:
    for i in range(1, 41):
        print fib1(i)
        print fib2(i)
        print fib3(i)
        print fib4(i)
```

结论：

1. fib1/fib2 的 40 次调用累计 5 亿次以上的递归，总耗时 653 秒多；fib3 的 lambda 和 _fib4 的 40 次调用累计 820 次递归，总耗时不到 0.1 秒

2. 函数调用与 lambda 演算区别不大，5 亿次以上的递归调用，差异仍然保持在 20 秒内

3. 能用尾递的还用线递，都是虎逼青年…