Title: Python descriptor 特性应用一例
Date: 2010-11-11 11:13
Category: blog
Summary: 

首先，这篇文章很水，只是表达了 descriptor 的一个实际应用。其次，写本篇的原因只是因为前两天赖总刚写了另一篇用 descriptor 解决变态问题的文章（其实赖总那篇也很水的哈），我来凑凑趣而已。

需要祭出 Python descriptor 才能解决问题的场景的确不多，我写过、看过的代码中，对 descriptor 的典型应用要算 django 的 LazyUser 手法最漂亮（位置在 django.contrib.auth.middleware.LazyUser）。当 view 中真正用到了 request.user 时（在 AuthenticationMiddleware 的 process_request 时给 request 赋上了 LazyUser 的实例），django 才会实际调用 django.contrib.auth.get_user 函数来获取 user 的数据（这也是叫 LazyUser 的原因）。

我的需求其实很简单，我在 doubandb 里存储了一个哈希表，需要在运行时映射到对象实例上，并且希望这个哈希表的每个 item 都能作为一个单独的属性对外暴露。

最简单直接的手法是为每个 item 单独编写 getter/setter，这样做在 item 数量很多时工作量也会快速增长；第2个方法是编写一个通用的函数来处理访问请求，但这的话所有访问又都会变成 obj.get_item(’key’)/obj.set_item(’key’)，因此也不符合我的需求；第3种办法是在 \_\_getattribute__ 函数中做手脚，但此种情况下需要在 \_\_getattribute__ 中判断本次访问的属性到底是从哈希表中取数据还是从 \_\_dict__ 中取数据，而且如果我有多个不同的 class 的话，那每个 class 都要添加 \_\_getattribute__ 处理，任务也挺重的。综上，最懒的办法还是定义一个 descriptor 来曲线救国。

代码很简单，先看引用 descriptor 的部分：

```python
class Employee(PropsMixin):
    name = PropsItem(’name’, ”)
    age = PropsItem(’age’, 18)
    ob = PropsItem(’job’)
```

这样定义和使用都完全跟普通的属性没什么两样了。唯一需要解释的是哈希表在 PropsMixin 中定义，并且 PropsMixin 提供了对哈希表单个 item 的访问和设置函数。

下面是 PropsItem 的定义部分：

```python
class PropsItem(object):
    def __init__(self, name, default=None):
        self.name = name
        self.default = default

    def __get__(self, obj, objtype):
        return obj.get_props_item(self.name, self.default)

    def __set__(self, obj, value):
        obj.set_props_item(self.name, value)
```

代码很简单吧，欢迎大家写信告诉我其它能用 descriptor 手法使代码更性感的场景。