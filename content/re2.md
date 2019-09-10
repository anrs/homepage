Title: PCRE/Python 下的 re 细节(3) — Unicode
Date: 2009-09-30 03:47
Category: blog
Summary: 

本篇是概念流，但个人认为这个 Recipe 2.7 的内容比 Mastering Regular expressions 中关于 Unicode 的部分讲得更精到，相当有看点的说；再就是对 unicode 的完美支持大多集中在 PCRE/Perl 上，所以下面例出的手法在除 Perl 外的大多数语言中都是屠龙之技。(怒了，Perl 6 杀很大，你到底准备在哪年圣诞节君临天下啊…)

代码点(code point)

一个代码点就是 Unicode 字符集中的一个条目，虽然说多数字符都唯一映射到一个对应的代码点，但一个代码点还是不能说完全等同于一个字符，因为有些语言中的单个字符是映射到 Unicode 中的多个代码点组合上的，比如字符 å 就映射到 U+0061 和 U+030A 两个代码点。

可以用 \uXXXX 和 \x{XXXX} 匹配一个代码点，使用 \u 还是 \x 取决于所使用的语言流派，前者在 Python 中使用，后者则应用在 PCRE/Perl 之中。此外，二者还有个区别是 \u 后面的 XXXX 必须是固定的4位16进制数值，范围从 U+0000 到 U+FFFF；而 \x 后面的 XXXX 则支持变长的16进制数，取值从 U+000000 到 U+FFFFFF，即 \x{0000E0} 等价于 \x{E0}。另外就是代码点在字符组(character classes)内外都可以使用。

属性(property)和分类(category)

每个代码点都有一个属性或者说属于某个分类，这个只是说法不同，二者想要表达的意思是完全一样的，这里是所有 Unicode 分类的列表，总共7大类30小类。所谓分类其实就是把所有 Unicode 代码点按其意义分组并形成多个不同的类别。比如 Sc 这个分类下面就全是跟货币符号相关的代码点，分类 Sm 下面全是跟数学符号相关的代码点。PCRE/Perl 下用 \p{Sc} 和 \p{Sm} 来匹配前面提到的两组分类，可惜的是 Python re 不支持按 Unicode 分类来匹配代码点，怒~~~

块(block)

块和分类的概念差不多，也是把代码点划分为 105 个不同的类别，但划分得更细腻，块名字也更适合人类阅读，块名字显示了这个块下面的代码点的主要用途。比如上面的 \p{Sc} 等价的块匹配就是 \p{InCurrency} 明显 InCurrency 比 Sc 可读性高上一百倍呀一百倍。然后让我们继续回到鄙视 Perl 的大路上了，块匹配更是连 PCRE 都不支持了，可挨千刀的 Perl 继续在完美的支持它，\p{InBlockName} 和 \p{IsBlockName} 这两种语法都可以工作，但推荐使用 \p{InBlockName} 这种语法，因为 Is 可能会和 \p{IsScript} 产生混淆。

Script

这个 Script 我真不知道该怎么翻译了，保持原样好了。Script 和 block/category 也都差不多，它按自然语言的不同将代码点分成了汉语(Han)、片假名(Katakana)、希腊语(Greek)等 N 多个分组。但这种划分并不是完全一一对应到自然语言的，有的 Script 下可能包括多种自然语言中的字符，比如 Latin Script；而有的自然语言下的字符所对应的代码点又会被划分到多个不同的 Script 中，比如 Hiragana Script(平假名)、Katakana Script(片假名)、Han Script(汉语)和 Latin Script 这4种 Script 下面均有日文字符的代码点。\p{Script} 和 \p{IsScript} 都用来匹配某个 Script，但推荐使用前者，以避免和 \p{IsBlockName} 发生冲突。是的，你没有猜错，这个不知道是什么东西的 Script 在 Python 下也是不支持的，它仅在 PCRE/Perl 下面生效。

字形(grapheme)

前面有提到代码点和字符并全是一一对应的关系，会有一个字符对应到多个代码点组合的情形发生，比如字符 å 就对应到 U+0061 和 U+030A 两个代码点。当使用点号去匹配 Unicode 字符时，其实只匹配得到一个代码点，也就是只匹配得到 U+0061，这明显不是期望中的结果。要解决这个问题，就需要用到 \X 模式，这里有我以前写的一个例子，现在就不多说了。

最后一句，所有 \p 都有对应的 \P 模式，\P 的意思就是匹配所有非 XXX 的情况。比如 \P{Sc} 就是匹配所有非 Currency 代码点。然后是支持 \p 的流派也都支持 \P 的，所以那些流派可以用 \P{M}\p{M}* 来模拟 PCRE/Perl 的 \X 模式，但由于 Python 连 \p 都不支持，所以建议各位用 Python 童鞋继续望 Perl 兴叹吧，Perl 6深情的凝望…