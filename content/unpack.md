Title: unpack 的三种实现
Date: 2009-09-08 09:13
Category: blog
Summary:

解析文件或者网络数据时，基本的操作就是 unpack 流内容，下面演示了三种不同的 unpack 手法，分别使用了 re, struct.unpack 和 list slice，最后是三种手法的性能比较。

代码：

```python
#!/usr/bin/env python

def split_by_unpack(txt):
    import struct
    baseformat = ‘5s 3x 8s 8s’
    format = ‘%s %ds’ % (baseformat, len(txt) – struct.calcsize(baseformat))
    return struct.unpack(format, txt)
 
def _split_by_re():
    import re
    regex = re.compile(r‘(.{5})(?:.{3})(.{8})(.{8})(.*)’, re.I)
    def __split_by_re(txt):
        r = regex.match(txt)
        return (r.group(1), r.group(2), r.group(3), r.group(4))
    return __split_by_re
split_by_re = _split_by_re()
 
def split_by_slice(txt):
    cuts = [5, 8, 16, 24]
    pieces = [txt[i:j] for i, j in zip([0] + cuts, cuts + [None])]
    return (pieces[0], pieces[2], pieces[3], pieces[4]) 
 
if __name__ == ‘__main__’:
    txt = ‘12345###abcdefghABCDEFGH###’
    for i in xrange(1000000):
        split_by_slice(txt)
        split_by_unpack(txt)
        split_by_re(txt)
```

上面代码演示了将文本 unpack 为长度分别是 5, 3(忽略), 8, 8, k 的子串，下面是对三个函数进行百万次调用后的性能分析：

screenshot_052

其实三种手法还是有一些差异的，struct.unpack 是基于 bytes 来拆分，而 re 和 list slice 是基于字符拆分的，所以在非 ASCII 的情况下要特别注意。