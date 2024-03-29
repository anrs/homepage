Title: 正则表达式的匹配过程
Date: 2009-07-05 23:58
Category: blog
Summary: 

“匹配过程”应该是学习正则表达式的第一堂课，但国内出版的相关图书大多对这一主题语焉不详，结果导致很多使用正则表达式的程序员都知其然却不知其所以然，想要对表达式进行优化也完全不知道从何入手。所以下面尽量用简单的例子来把匹配过程描述清楚，欢迎大家拍砖。

首先是最简单的情况，用表达式 a 匹配目标文本 django 的过程。正则引擎首先用 a 和目标文本中的 d 进行比较，此时匹配失败，引擎会将当前待匹配目标字符前推到第2个字符 j 后再进行比较，结果还是匹配失败，引擎继续前推当前待匹配目标字符到 a 并进行比较，这回匹配成功，于是引擎返回匹配的内容并退出。当前待匹配目标字符就是第一次等待匹配的实际参与者，如果匹配失败，引擎就会将当前待匹配目标字符符前推到目标文本的下一个字符，然后再与当前待匹配表达式字符进行比较（在第1个例子中，当前待匹配表达式字符始终都是 a。当前待匹配目标字符的说法实在太绕舌，下面分别将之简称为当前表达式和当前字符），一旦目标文本已经结束却依然没有匹配成功，那么引擎就会报告整个正则表达式匹配失败并退出。

接下来将原来的表达式扩展为 an，目标文本依然是 django，首先将当前表达式 a 和当前字符 d 进行比较，匹配失败，所以前推当前字符到 j 并与当前表达式 a 进行比较，还是失败，继续前推当前字符到 a，这次匹配成功，于是将当前表达式和当前字符都向前推进，得到当前表达式 n 和当前字符 n，进行比较时还是匹配，再继续前推当前表达式时，引擎发现表达式已经结束，于是返回匹配的内容并退出。当正则表达式包含多个字符时，如果当前匹配成功，就同时前推当前表达式和当前字符并进行下一次比较，否则就只前推当前字符并将之继续与当前表达式进行比较。

再考虑以 an 匹配 there is a django boy 的情形。首先是当前表达式 a 和当前字符 t 进行比较，匹配失败，前推当前字符，继续比较，还是失败，直到前推当前字符到第1个 a 时，才匹配成功。这时因表达式尚未结束，所以除了前推当前字符到 a 以外，还同时前推当前表达式到 n，结果匹配失败。这时，正则引擎会做两件事，重置当前表达式为整个正则表达式的第1个字符，其次是将当前字符设置为第1次成功匹配字符的下1个字符，这里的第1次成功匹配字符是 a，a 的下一个字符就是空格符，然后再重新开始匹配，也就是将 a 与空格符进行比较，匹配失败，于是前推当前字符，比较、前推，再比较、再前推，直到当前表达式 a 再次与 django 中的 a 匹配成功之后，引擎又同时前推当前表达式与当前字符并进行比较，匹配成功后继续前推时发现表达式已经结束，于是引擎返回匹配的内容并退出。这里的关键点是如果当前表达式和当前字符同时前推后匹配失败，那么当前表达式重置到整个正则表达式的第1个字符，当前字符重置为第1次成功匹配字符的下1个字符，并继续进行比较。

上面几个都是不断前推并进行比较的例子，在遇到需要进行最长匹配的量词元字符时，正则引擎则可能会不断后退当前字符并和当前表达式进行比较，还是以实例来说话，考虑以正则表达式 a.*g 匹配目标文本 there is an elegant django 的情况。当 a 和点号元字符都匹配成功，前推当前表达式后发现是需要最长匹配的元字符 *，于是直接将当前字符彻底前推到整个目标文本的末尾结束符（由于结束符不可见，可以将其想成 C 中的 ‘\0′），并将中间忽略过的字符统统设为匹配成功，这时引擎侦测到表达式并未结束，前推当前表达式到 g 时发现与结束符并不匹配，于是引擎回退当前字符到 o 后再进行比较，还是匹配失败，于是当前字符继续回退到 g，终于匹配成功，之后引擎发现整个正则表达式已经结束，于是返回匹配的内容并退出，匹配的内容就是 an elegant djang

(这里顺便感谢 .rex 同学今天送我 O’Reilly 最新出版的”Regular Expressions Cookbook“一书，你大无畏的奉献精神深深感动了我…)