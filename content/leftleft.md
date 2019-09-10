Title: << 文本
Date: 2008-03-12 10:07
Category: blog
Summary:

Perl << 会原样呈现分界符中的内容，这就有点类似于 C# 中加了 @ 前缀的字符串。

```perl
print <<END
      what's this
END
;
```

打印结果是 "    what's this"

其中 END 就称为 << 分界符，该分界符可以被引号引起，根据不同的引号 << 文本就会相应的具有（不具有）变量内插的能力，甚至是成为 shell 命令执行的能力。省略引号的裸分界符被视为被双引号引起。

结束分界符需要是一行中唯一的内容，并且要顶行书写。

```perl
my $txt = "what's this";
if (shift) {
  print <<END
    $txt
  END
  ;
}
```

上面的代码会收到一个“未找到结束分界符”的错误提示。正确的写法应该是

```perl
my $txt = "what's this";
if (shift) {
   print <<END
     $txt
END
   ;
}
```

打印结果"    what's this"

由于 << 文本虽然被书写到了不同的行，但本质上它仍然是一条语句，所以结束分界符下面一行有一个独立的分号。如果觉得独立分号影响美观的话，也可以将其写到起始分界符的末尾。

```perl
my $txt = "what's this";
if (shift) {
   print <<END;
     $txt
END
}
```

当用反引号引起起始分界符时，Perl 会将 << 文本依序发送到 shell 执行。

```perl
print <<`END`;
      cd
      ls
END
```

上面代码会打印出当前登录用户家目录下的内容。

可以用空字符作为分界符，这样当遇到第一个独立空行（包括空格在内的任何内容都没有的行）时 << 就会退出。

```perl
print <<"";
      what's this

#
```

如果使用空字符作为分界符，最好将其用引号引起，否则如果打开 warnings 开关就会收到一个警告消息。空字符作为分界符时 << 后面可以跟一个或多个空格。

```perl
print << "";
      what's this

# 合法

print << END;
      what's this
END

# 非法
```

还可以将 << 与 Regex 联合使用。

```perl
my $capture= <<END =~ /(?:mod_)(.*)/i;
   apache
   mod_perl
   mod_python
END
print $1;
```

打印结果 perl

ps. 当用到 << 时，GNU Emacs 21.3.1 版本默认的 perl_mode 会有一个很大的问题
