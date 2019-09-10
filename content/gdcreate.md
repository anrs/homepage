Title: 无废话 GD 入门之创建 Image
Date: 2008-03-18 13:19
Category: blog
Summary:

利用 new 函数创建 Image 对象

new 函数有好几种重载形式，包括

```perl
new GD::Image($width, $height, $truecolor);# 指定图像尺寸和是否应用 24 位色
new GD::Image(FILEHANDLE);# 从指定的文件句柄创建
new GD::Image($filename);# 从指定的文件创建
new GD::Image($data);# 从指定的 Image Data 创建(调用 png $img; 生成 $data)
```

在第一种重载形式中，如果省略参数调用 new 函数，GD 默认创建 64*64 尺寸的 Image 对象，而省略 truecolor 参数则默认创建 8 位色的 Image 对象。

如果 new 函数创建 Image 对象失败，那么就返回一个 undef 值

还可以调用 newFromPng 函数，以明确指定从 png 文件创建 Image 对象

`my $img = newFromPng GD::Image('test.png');`

或者从一个指定的文件句柄创建 Image 对象

```perl
open(PNG, 'test.png') || die;
my $img = newFromPng GD:Image(\*PNG) || die;
close PNG;
```

如果不为 newFromPng 函数指定 truecolor 参数，那么从 png 文件创建出的 Image 对象，其 truecolor 属性（24 位色或者是 8 位色）与源文件相同。

从 JPEG/GIF 文件创建 Image 对象也有相应的 newFromJpeg/newFromGif 函数，但是这里有些许不同。

newFromJpeg 总是返回 24 位色的 Image 对象，除非显示指定 0 作为函数的第二个参数。
newFromGif 总是返回 8 位色的 Image 对象，除非显示指定 1 作为函数的第二个参数。