Title: PCRE 基础之“最长匹配”、“最短匹配”
Date: 2008-03-03 18:17
Category: blog
Summary:

量词的最长匹配原则

*, + 和 {m, n} 这样的量词都遵循最长匹配原则，也就是说可能出现多种匹配情况时，Regex 总是选择最长的匹配结果。

```perl
my $txt = 'javascript and ecmascript and perlscript';
$txt =~ /(.*)script/i;
print $1;
```

打印结果是 javascript and ecmascript and perl

.*script 可能的匹配结果包括

`javascript`
`javascript and ecmascript`
`javascript and ecmascript and perlscript`

而第三种匹配结果是最长的，所以 (.*) 的内容就包括了 javascript and ecmascript and perl，这就是所谓的最长匹配原则。同理，

```perl
my $txt = 'VeryVeryVery';
$txt =~ /((?:very){1,3})/i;
print $1;
```

打印结果是 VeryVeryVery，因为这个匹配结果也是最长的。

可以用 ? 忽略最长匹配原则，因为 ? 总是告诉 Regex 引擎“我要最短匹配”

```perl
my $txt = 'VeryVeryVery';
$txt =~ /((?:very){1,3}?)/i;
print $1;
```
因为 ? 忽略了最长匹配原则，所以打印结果是 Very

再看一下极端情况下的 ? 最短匹配

```
my $txt = 'VeryVeryVery';
$txt =~ /((?:very){0,3}?)/i;
print $1;
```

打印结果是空字符串，因为 {0,3} 的最短匹配就是空字符串，一定一定一定要注意的是匹配空并不等价于匹配失败，空字符串也是一种有效的成功匹配的结果。
