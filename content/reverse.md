Title: 反转序列的可读性和性能
Date: 2009-09-04 09:13
Category: blog
Summary: 

老规矩，直接上图上真相…

代码：

```python
#!/usr/bin/env python
#coding=utf-8
 
rev_by_index = lambda x: x[::-1]
rev_by_func = lambda x: [e for e in reversed(x)]
 
if __name__ == ‘__main__’:
    for i in xrange(10, 20001):
        lst = range(i)
        rev_by_index(lst)
        rev_by_func(lst)
```

结果：
screenshot_050

9.7s v.s. 0.6s … 你选择可读性还是性能？