Title: 无废话 GD 入门之颜色管理
Date: 2008-03-19 07:51
Category: blog
Summary:

用 colorAllocate 函数创建颜色句柄

```perl
my $r = 0;
my $g = 0;
my $b = 0;
my $black = colorAllocate $img $r, $g, $b;
```

上面的代码用先前创建的 Image 对象生成 RGB 值分别为 0, 0, 0 的一个颜色句柄

或者还可以用 colorAllocateAlpha 函数创建一个指定透明程度的颜色句柄

```perl
my $alpha = 0;
my $black = colorAllocateAlpha $img 0, 0, 0, $alpha;
```

第四个参数 \$alpha 指定透明程度，聚值范围 0 到 127，0 表示完全不透明，127 则是完全透明

用 colorDeallocate 函数消除一个已存在的颜色句柄，之后就可以重新设置该句柄，该函数不存在返回值。

```perl
my $black = colorAllocate $img 255, 255, 255;
colorDeallocate $img $black;
$black = colorAllocate $img 0, 0, 0;
```

通过 rgb 函数从指定的颜色句柄返回相应的 RGB 值

`my ($r, $g, $b) = rgb $img $black;`

上面的代码获取了 \$black 颜色句柄具体的 RGB 值。如果再配合 getPixel 函数，可以得到指定 Image 对象的指定位置的 RGB 值。

```perl
my $color = getPixel $img 100, 100;
my ($r, $g, $b) = rgb $img $color;
```

上面代码就演示了获取 \$img 的 (100, 100) 位置像素点的 RGB 值