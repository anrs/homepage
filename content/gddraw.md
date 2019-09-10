Title: 无废话 GD 入门之绘制篇
Date: 2008-03-20 08:02
Category: blog
Summary:

GD::Image 提供了一系列的绘制函数，调用这些函数，就可以相当方便地绘制线条、矩形和椭圆等形状。先从点绘制开始。

setPixel 函数接收像素点的座标位置和指定的颜色句柄，并绘制该像素点。

`setPixel $img 20, 20, $color;`

上面的代码调用 setPixel 函数，并用 \$color 句柄指定的颜色绘制 $img 中座标位置为 (20, 20) 的像素点。

line 函数前四个参数，分别代表前始点座标位置和终止点座标位置，并在这两点间绘制一条线段。

`line $img 0, 0, 20, 20, $color;`

在 \$img 的 (0, 0) 点和 (20, 20) 点间用 $color 指定的颜色绘制一条直线。

相应的 rectangle 函数在起始点和终止点间绘制一个矩形。

`rectangle $img 0, 0, 10, 10, $color;`

就相当于调用四次 line 函数，绘制一个封闭的矩形。

```perl
line $img 0, 0, 0, 10, $color;
line $img 0, 0, 10, 0, $color;
line $img 0, 10, 10, 10, $color;
line $img 10, 0, 10, 10, $color;
```

filledRectangle 函数也是绘制一个矩形，不同的是 filledRectangle 还要用指定的颜色填充这个矩形。

openPolygon 函数用指定的颜色绘制一个封闭的形状，这个形状必须是封闭的，也就是说至少需要有三个端点，比如一个三角形。

```perl
my $polygon = new GD::Polygon;
addPt $polygon 0, 0;# 第一个端点
addPt $polygon 10, 0;# 第二个端点
addPt $polygon 10, 10;# 第三个端点
addPt $polygon 0, 10;# 第四个端点
openPolygon $img $polygon, $black;
```

上面代码会绘制一个与之前调用 rectangle 函数绘制出的矩形完全相同形状，当然如果是规则的形状，那么更多的情况下使用 rectangle 函数就能完全满足需要了，openPolygon 更多是用于绘制不规则的封闭图形。像 filledRectangle 函数一样，filledPolygon 函数也是用于绘制并填充一个封闭图形。