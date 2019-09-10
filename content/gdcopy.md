Title: 无废话 GD 入门之图像拷贝篇
Date: 2008-03-29 09:50
Category: blog
Summary:

GD::Image 提供了数个操作函数，以从源图拷贝内容到目标图。

copy 函数是最其中最简单的一个，这个函数拷贝源图的指定区域到目标图（即调用 copy 函数的对象）。

`$img->copy($src, 0, 0, 50, 50, 100, 100)`

上面的代码便是从 \$src 对象的 (50, 50) 位置拷贝 100*100 大小的矩形内容到当前 $img 对象的 (0, 0) 处。

copyMerge 函数和 copy 函数非常相似，在极端情况下两者甚至是执行了完全相同的动作，但 copyMerge 函数更有用，copyMerge 函数与 copy 函数的入参唯一不同的就是在最末多了一个参数，该参数指定了从源图来的内容在多大程度上合并到目标图，其取值范围从 0-100，100 即表示完全覆盖，这时的 copyMerge 函数运行结果就完全等同于 copy 函数了。

`$img->copyMerge($src, 0, 0, 50, 50, 100, 100, 100)`

等同于上面的 $img->copy 函数了。

另一个非常有用的函数是 copyResized 函数，这个函数的入参同时包括源图待拷贝内容的大小和要复制到的目标图内容的大小，如果源和目标的大小不一致，copyResized 会自动对目标进行缩放操作。

文字绘制

最简单最常用的 string 方法在指定位置用指定颜色绘制指定大小的指定文字。

`$img->string(gdTinyFont, 0, 0, 'test', $black)`

在 (0, 0) 位置用 $black 指定的颜色绘制文字 test，gdTinyFont 是 GD 常量之一，相应的字体还有 gdGiantFont, gdLargeFont, gdMediumBoldFont 和 gdSmallFont。这几个常量唯一区别便是绘制出来的大小不同。但是更灵活的做法是调用字体文件获得字体。