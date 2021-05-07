现实中人类的眼睛所能看到亮度比的范围是[公式] 左右，照相机和摄像机可以捕捉到 HDR 的影响，渲染过程中可以产生 HDR 的画面。这样问题就出现了， [公式] 或者更高数量级的亮度只能存在电脑里，而一般的显示器只能表示 [公式] 个亮度数量级，如果用 256 个数字的 LDR 设备来显示 HDR 所表示的信息，就需要一个称为 tone mapping 的过程来改变动态范围。在进行了 tone mapping 之后，为了针对人对自然亮度的非线性感知以及显示器 de-gamma 特性，我们会进行 gamma 校正，最终输出对应颜色到显示器。

本文将会以下面五部分大致的讲解自然界捕捉亮度到显示器最终输出亮度处理过程中的 Tone Mapping 和 Gamma Correction：

## 一、HDR 与 LDR

1. HDR(High Dynamic Range)
   什么是 HDR，首先我们来理解 DR——Dynamic Range（动态范围）：Dynamic Range 是一种用数学方式来描述某个给定场景的亮度层次范围的技术术语。指图像中所包含的从“最亮”至“最暗”的比值，也就是图像从“最亮”到“最暗”之间灰度划分的等级数；动态范围越大，所能表示的层次越丰富，所包含的色彩空间也越广。最通常的解释有两种：

- (1)一种是摄影界通常所说的 D 值（以对数值表示的场景最高亮度和最低亮度比的相对数值），通常由 0-4 之间的很精确的数字来表示。D 值的计算公式为：Dynamic Range=log10(Max Intensity / Min Intensity)。公式中 intensity 是指光照强度，我们对最大亮度除以最低亮度的结果取对数，得到的结果就是动态范围的相对数值——摄影界所说的 D 值。各种景物、底片和照片都有其各自特定的 D 值范围。

- (2)另一种是计算机图形学中通常使用的直接以场景最高亮度和最低亮度的亮度比表述的方法，如 255:1。 在数字图像领域一般都采用这第二种比值的表述方式来评述场景的动态范围。亮度的单位以每平方米的烛光来表示（cd/m2）。太阳自身的亮度大约为 1,000,000,000 cd/m2。阳光照射下的景物的亮度可达 100,000 cd/m2，而星光的亮度大约在 0.001 cd/m2 以下，二者亮度比达亿倍以上。现实中人类的眼睛所能看到亮度比的范围是 [公式] 左右，相对于 255:1 来说，我们称之为高动态范围，即 HDR。

2. LDR(Low Dynamic Range Image)

什么是 LDR？它所采用的色彩模型是目前通用的图像描述模型——RGB 模型。每种色彩都可以用三原色（红、绿、蓝）加上适当的亮度来表示，三原色的亮度梯度各为 256 级。选定每色 256 级是在电脑硬件性能、照片级真彩图片需要和电脑 2 进制方案综合考虑后的结果。这就是目前我们非常熟习的观看、编辑、交换和处理数字图像的软硬件环境。这种 8 比特位元 RGB 低动态范围图像描述模型是将场景最高亮度和最低亮度的亮度比限定为 255 比 1，计算得出的动态范围 D 值即为 2.4。

## 二、Tone Mapping

采用 HDR 渲染出来的亮度值会超过显示器能够显示最大值，此时我们需要将光照结果从 HDR 转换为显示器能够正常显示的 LDR，这一过程我们通常称之为 Tone Mapping。用 Tone mapping 压缩以后，我们所得到的 HDR 影像就能很好地在显示器上显示了。下图是对采用 HDR 渲染的图片使用了 Tone Mapping 和没有使用 Tone Mapping 的对比结果：

![](https://pic2.zhimg.com/80/v2-ed17388c48eefa33655f2521b8494e4d_720w.jpg)

可以在图中的上面部分看到未经 tone mapping 处理的图有地方出现过曝的情况，简单的线性转换这些颜色值并不能很好的解决这个问题，因为明亮的地方会显得更加显著。我们可以用一个比较合适的非线性的曲线来转换这些 HDR 值到 LDR 值，来解决我们对于场景的亮度的完全掌控。

随着 GPU、游戏和电影特效的突飞猛进，tone mapping 也经历了一系列的进化历程。例如基于经验模型的 Reinhard tone mapping，到后来基于 tone mapping 的结果来进行拟合出来的 Filmic tone mapping，再到现在比较常用的 Academy Color Encoding System(AECS)等方法，具体可以参考下面文章：

- [hdr_photographic](http://www.cmap.polytechnique.fr/~peyre/cours/x2005signal/hdr_photographic.pdf)
- [Filmic Tonemapping Operators](https://link.zhihu.com/?target=http%3A//filmicworlds.com/blog/filmic-tonemapping-operators/)
- [ACES](https://link.zhihu.com/?target=https%3A//www.oscars.org/science-technology/sci-tech-projects/aces)

现在越来越多的游戏或者引擎例如 Rise of the Tomb Raider、UE 4 等采用 ACES，它是一套颜色编码系统，或者说是一个新的颜色空间。它是一个通用的数据交换格式，一方面可以不同的输入设备转成 ACES，另一方面可以把 ACES 在不同的显示设备上正确显示。不管你是 LDR，还是 HDR，都可以在 ACES 里表达出来。这就直接解决了 VDR 的问题，不同设备间都可以互通数据。下面是 AECS 的表达式：

```cpp
float3 ACESToneMapping(float3 color, float adapted_lum)
{
       const float A = 2.51f;
       const float B = 0.03f;
       const float C = 2.43f;
       const float D = 0.59f;
       const float E = 0.14f;
       return (color * (A * color + B)) / (color * (C * color + D) + E);
}
```

ACES 映射的曲线如下：

![](https://pic4.zhimg.com/80/v2-e8ce3b4c044a5698637ca2553b3cf993_720w.jpg)

由映射曲线可以看出，在做映射时，较小和较大的 color 映射到相对较小的范围，而中间强度的颜色值获得比较大的范围区间，这样使得显示场景暗的地方不至于太暗，亮的地方不至于过曝。通过这条 tone mapping 曲线我们就可以将 HDR 亮度值转换到[0,1]，然后乘以 255，就转到了我们 LDR 设备的显示区域[0,255]了。当然在转到[0,255]之前，我们还需要做另外一个操作——Gamma Correction。

## 三、Gamma Correction

在查找相关资料的过程中，看到了很多人争论，出现 Gamma Correction 到底是人对自然亮度的感知原因，还是由于早期的 CRT 显示器存在非线性输出的问题。在研究了一些 Gamma Correction 资料后，个人认为早期 Gamma Correction 的出现是由于原来的 CRT 显示器存在非线性输出的问题，而在显示器完全可以解决非线性输出的问题的今天，Gamma Correction 的存在更多的是由于人对自然亮度的感知。

早期的 CRT 显示器存在非线性输出的问题,简单来说,你给 CRT 显示器输入(input)一个 0.5(输入范围为[0,1]), CRT 显示器的输出(output)并不是 0.5,而是约等于 0.218,输入与输出间存在一个指数大概为 2.2 的幂次关系:
![](https://pic1.zhimg.com/80/v2-8d410eeea31b29f5b522e7629e471a20_720w.jpg)

其中 2.2 这个指数即为伽马(gamma)值,而显示器的这种非线性输出过程则称为伽马展开(display gamma).

为了能够得到正确的输出,我们必须对输入进行补偿,方法就是对输入进行一次指数为 1 / 2.2 的幂次运算,这个补偿的过程便是伽马编码(encoding gamma)我们也称 gamma correction :

![公式](https://www.zhihu.com/equation?tex=input%5Crightarrow+input%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D)

经过伽马校正后,显示器便能够正确显示我们的输入了 :

![公式](https://www.zhihu.com/equation?tex=input%5Crightarrow+input%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D)

![公式](https://www.zhihu.com/equation?tex=output%5Capprox+%28input%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D%29%5E%7B2.2%7D+%3D+input)

所以为了让显示器正确输出 0.5, 我们需要对 0.5 进行伽马校正,实际给显示器的输入约为 0.73 :

![公式](https://www.zhihu.com/equation?tex=0.5%5Crightarrow+0.5%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D+%5Capprox+0.73)

看到这里你可能会有个疑问：既然伽马校正起源于早期 CRT 显示器的非线性输出问题,而我们现在基本已经淘汰掉这些显示器,并且当今的显示器已经可以做到线性输出了(输入 0.5,输出也是 0.5),那么我们是不是可以直接废弃伽马校正了呢?

答案可能有些出人意料 : 我们仍然需要进行伽马校正!

原因有些巧合 : 伽马校正除了可以解决早期 CRT 显示器的非线性输出问题, 同时还可以帮助我们"改善"输出的图像质量，下图是自然界中的亮度以及对应的人所感受的亮度值 :

![](https://pic4.zhimg.com/80/v2-5da4673b8149b499df7202f54317e2d3_720w.jpg)

由上图可以看出，人眼对于较暗(接近 0)的亮度值比较敏感,对于较亮(接近 1)的亮度值则不太敏感，你可以理解为人眼更能辨别较暗的亮度值发生的变化，因此颜色在存储时，我们应该更多的保存较暗部分的颜色值。对于自然界中大约 0.218 的亮度值，人感受亮度值约为 0.5，因此我们有了下面的 Gamma Correction 和 CRT gamma 曲线。

![](https://pic3.zhimg.com/80/v2-cf44aa0c6c91317c67dce58e5276d5ca_720w.jpg)

假设我们现在使用一个字节(能够表达整数范围[0,255])来存储亮度值,并且我们要存储 0.240 和 0.243 这两个亮度值,如果不进行伽马校正,则有:

value1=0.240∗255=61.2⟶ 取整为 61

value2=0.243∗255=61.965⟶ 取整为 61

可以看到 0.240 和 0.243 的存储数值都是 61,所以这两个输入的实际显示效果其实是一样的(细节差异丢失了).

但如果我们进行一次伽马校正,则有:

![公式](https://www.zhihu.com/equation?tex=value1+%3D+0.240%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D+%2A255+%5Capprox+133.3+%5Crightarrow+%E5%8F%96%E6%95%B4%E6%95%B0%E4%B8%BA133+)

![公式](https://www.zhihu.com/equation?tex=value2+%3D+0.243%5E%7B%5Cfrac%7B1%7D%7B2.2%7D%7D+%2A255+%5Capprox+134.1+%5Crightarrow+%E5%8F%96%E6%95%B4%E6%95%B0%E4%B8%BA134)

0.240 和 0.243 的存储数值变为了 133 和 134,所以这两个输入的实际显示效果便区分开了(细节差异保留了).

实际上,伽马校正增大了较暗数值的表示精度,而减小了较亮数值的表示精度,人眼又恰好对较暗数值比较敏感,对较亮数值不太敏感,于是从视觉角度讲,输出的图像质量就被伽马校正"改善"了.基于这个原因,我们仍然需要进行伽马校正,而既然我们进行了伽马校正,当今的显示器也便保留了非线性输出(伽马展开)的功能,颇有些因果倒置的意思.

## 四、总结

总的来说，Tone Mapping 和 Gamma Correction 二者都是为了更好的在 LDR 设备上显示图片，将图片的颜色值从一个范围分布变换到另一个范围分布。而不同的是，Tone Mapping 是根据相应的算法将颜色值从一个大的范围映射到了较小的范围，而 Gamma Correction 则是从[0,1]映射到[0,1],映射范围并没有改变，只是改变了不同亮度值颜色的分布情况。这两种变换需求的本质是由于我们的 LDR 设置只支持 8 位的颜色值造成的，如果有一天我们的显示设备支持 16 位或者更高位的颜色值，也许它们就可以退出了。
