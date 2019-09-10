Title: PCRE 基础之“最早匹配”
Date: 2008-03-03 07:18
Category: blog
Summary:

昨个有朋友问我 s/a|b/c/i 为什么只能替换 a 或者 b 而不能同时将 a, b 都替换成 c？
关于这个问题，最简单的回答就是“忽略了最早匹配原则”

首先解释一下什么是“最早匹配原则”

```perl
my $txt = 'ksharp kplus';
$txt =~ s/k/c/i;
print $txt;
```

打印结果是 csharp kplus

根据 Perl Regex “优先选择最早出现的匹配结果”原则
/k/ 会匹配 $txt 中单词 ksharp 中打头的 k，因为这个 k 是整个字符串中最早匹配 /k/ 的结果，这也就是所谓的“最早匹配原则”

多选分支中的匹配也符合该原则

```perl
$txt = 'csharp cplus ruby perl java';
$txt =~ s/java|cplus/lisp/i;
print $txt;
```

打印结果是 csharp lisp ruby perl java

虽然 /java|cplus/ 中 java 出现在 cplus 之前，但是 Regex 会在每一个位置尝试所有的匹配，也就是说每一次 Regex 引擎都会同时尝试 /java/ 和 /cplus/，根据“优先选择最早出现的匹配结果”原则，当匹配 /cplus/ 成功后，引擎就会退出，而不再尝试匹配 /java/，这就有点类似于 open(FH, $file) || die 'error' 中的 || 短路操作，当 open 成功的话，就不再执行 die 操作了。

这还有一个稍微特殊一点的例子来说明最早匹配原则

```perl
$txt = 'abcd';
$txt =~ s/a|ab|abc|abcd/r/i;
print $txt;
```

打印结果是 rbcd

从这个例子可以看到多选分支是存在顺序问题的，左边的多选分支先匹配，右边的多选分配后匹配，如果忽略了顺序问题，那么就有可能出现非预期的匹配结果。（由于多选分支存在顺序问题，所以并不会和“最长匹配”原则相冲突）