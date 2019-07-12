---
layout: post
title: "关于图片的一些知识点"
date: 2018-12-26
comments: true
categories: [iOS]
tags: [图片渲染]
keywords: [图片渲染]
publish: true
## description: 图片渲染
---

如果说之前的项目中哪个 bug 让我记忆犹新，我会毫不犹豫的说是内存溢出（OOM），因为当时无论从 dSYM 还是第三方的报错信息中我都找不出问题是所在，而且开发过程中也极少遇到，现在知道当时遇到的是高分辨率的图片集中渲染导致的 OOM。

内存溢出从字面上就很好理解，传统意义上的 OOM 就是当前使用的 App 达到了 “high water mark”，也就是达到了系统对单个 App 的内存限制，系统会把这个应用杀掉（由 [Jetsam](https://medium.com/@davion/ios-daemon-journey-e8c1b26e11e1) 执行）。简单的说完关于 OOM 的知识点，接下来是本文要探讨一些关于图片渲染的一些知识点。

现在大家都应该知道图片从读取到最终渲染都会经历解压的过程，大致过程如下（图片来自于[Image and Graphics Best Practices](https://developer.apple.com/videos/play/wwdc2018/219)）

![AboutImage-8](http://lc.yardwill.top/AboutImage-8.png)

Decode 过程简单说就是把图片转化成 Bitmap，那么 Bitmap 具体是什么？



### Bitmap

在 [Wikipedia](https://en.wikipedia.org/wiki/Bitmap) 有这么一段解释

>In [computing](https://en.wikipedia.org/wiki/Computing), a **bitmap** is a mapping from some domain (for example, a range of integers) to [bits](https://en.wikipedia.org/wiki/Bit). It is also called a [bit array](https://en.wikipedia.org/wiki/Bit_array) or [bitmap index](https://en.wikipedia.org/wiki/Bitmap_index).The more general term **pix-map** refers to a map of [pixels](https://en.wikipedia.org/wiki/Pixel), 

通俗点讲 bitmap 就是像素图，通过以下方法我们可以得到一张图片的 bitmap 信息

```swift
extension UIImage {
    
    var decodeData: Data? {
        guard let cgimage = cgImage
            , let dataProvider = cgimage.dataProvider
            , let rawData = dataProvider.data as Data? else {
                return nil
        }
        return rawData
    }
}
```

拿下面这张 48 * 48 的图片为例

![image-1](http://lc.yardwill.top/AboutImage-1.png)



通过上面提供的方法最终获取到的 bitmap 信息前一小段是这样的（这样子分割开来的十六进制数据一共有 48 * 48 个），事实上这些数据对应的就是各个像素上要显示的颜色信息

```
ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff fffbf9ff ffd5c8ff ffb19bff ff9b7eff ff8765ff ff7e59ff ff7750ff ff7750ff ff7e59ff ff8765ff ff9b7eff ffb19bff ffd5c8ff fffbf9ff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff fff6f3ff ffcbbcff ff9f84ff ff7953ff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff734bff ff7953ff ff9f84ff ffcbbcff fff6f3ff ffffffff ffffffff ffffffff
```

验证一下，前十七个像素点都是白色，这符合预期，因为左上角都是白色区域，到了第十八个像素点的时候变成了 fffbf9ff，我们把图片放大后对比下



![AboutImage-2](http://lc.yardwill.top/AboutImage-2.png)



第十八个像素点是`fffbf9ff`，具体的，这个色值就是就是上面所显示的浅红色，最后两位 ff 代表的 alpha 值，到现在算是搞清楚了 bitmap 到底是什么。



### 内存占用

回到文章开头提到的高分辨率图片的解压导致 OOM 问题，首先一个问题是：一张图片解压需要消耗多少内存？答案是 **水平象素 × 垂直象素 × 4 ** 单位是 byte。我们来验证一下，这次换一张稍大的彩色 PNG 图片，它的分辨率是 400 * 350，所以理论上解压需要的内存大小是 400 * 350 * 4 = 547k，测试设备是 iPhone X 12.3.0，通过 Instruments 的 Allocations 检测到结果是 560k 这非常接近于理论值



![AboutImage-3](http://lc.yardwill.top/AboutImage-3.png)



为什么要强调彩色图片呢，或许你已经猜到了解压一张彩色图片和解压一张只有黑白组成的照片是所消耗的性能是不一样的，对的，比如下面这张图片同样是 400 * 350 的 PNG 图片

![AboutImage-4](http://lc.yardwill.top/AboutImage-4.png)

解压消耗的内存只要288k，计算方法 400 * 350 * 2 = 273k

![AboutImage-5](http://lc.yardwill.top/AboutImage-5.png)

这时候可能有些小伙伴会想，这样子也可以的话，那是不是纯红，纯绿或者纯蓝消耗的内存大小也一样的，抱歉，不是的，要解释这个问题需要引入颜色空间的概念，苹果目前支持的颜色空间有下面[这种方式](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html)

- Gray spaces, used for grayscale display and printing; see [Gray Spaces](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCICIEED)
- RGB-based color spaces, used mainly for displays and scanners; see [RGB-Based Color Spaces](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCIFIGHI)
- CMYK-based color spaces, used mainly for color printing; see [CMY-Based Color Spaces](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCIDBFBB)
- Device-independent color spaces, such as L*a*b, used mainly for color comparisons, color differences, and color conversion; see [Device-Independent Color Spaces](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCIGHHEJ)
- Named color spaces, used mainly for printing and graphic design; see [Named Color Spaces](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCIHEFEH)
- Heterogeneous HiFi color spaces, also referred to as multichannel color spaces, primarily used in new printing processes involving the use of red-orange, green and blue, and also for spot coloring, such as gold and silver metallics; see [Color-Component Values, Color Values, and Color](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BABHGDIB)

具体到 iOS 有这[几个方式](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/dq_context/dq_context.html#//apple_ref/doc/uid/TP30001066-CH203-BCIBHHBB)

| CS   | **Pixel format and bitmap information constant**             | Availability  |
| ---- | ------------------------------------------------------------ | ------------- |
| Null | 8 bpp, 8 bpc, [kCGImageAlphaOnly](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphaonly) | Mac OS X, iOS |
| Gray | 8 bpp, 8 bpc, [kCGImageAlphaNone](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphanone) | Mac OS X, iOS |
| Gray | 8 bpp, 8 bpc, [kCGImageAlphaOnly](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphaonly) | Mac OS X, iOS |
| RGB  | 16 bpp, 5 bpc, [kCGImageAlphaNoneSkipFirst](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/noneskipfirst) | Mac OS X, iOS |
| RGB  | 32 bpp, 8 bpc, [kCGImageAlphaNoneSkipFirst](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/noneskipfirst) | Mac OS X, iOS |
| RGB  | 32 bpp, 8 bpc,  [kCGImageAlphaNoneSkipLast](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphanoneskiplast) | Mac OS X, iOS |
| RGB  | 32 bpp, 8 bpc,  [kCGImageAlphaPremultipliedFirst](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphapremultipliedfirst) | Mac OS X, iOS |
| RGB  | 32 bpp, 8 bpc,  [kCGImageAlphaPremultipliedLast](https://developer.apple.com/documentation/coregraphics/cgimagealphainfo/kcgimagealphapremultipliedlast) | Mac OS X, iOS |

回到前面的计算方式，在表格中可以看到每个颜色空间（CS）对应的像素格式（Pixel format）和一些常量（bitmap information constant），包括每种颜色空间对应的每像素总 bit 数(bpp)等 ，对于彩色图片来说，解压它所需要用到的必然是 RGB ，对应的就是 32 bpp（关于 16 bpp 后面会提到）， 32 bits / 8 = 4 bytes，所以计算方式就是 **水平象素 × 垂直象素 × 4 **。

那么对于只有黑白组成的图片，对应的就是灰度颜色空间（Gray），[官方解释](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/csintro_colorspace/csintro_colorspace.html#//apple_ref/doc/uid/TP30001148-CH222-BCIFIGHI)

> Gray spaces typically have a single component, ranging from black to white, as shown in Figure 2-1. Gray spaces are used for black-and-white and grayscale display and printing. A properly plotted gray space should have a fifty percent value as its midpoint.

![image](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/csintro/art/scul01_2x.png)

所以内存占用计算方式就是 **水平象素 × 垂直象素 × 2 **（为什么表格给的是 8 bpp），这种解释方式同样适用于 UILabel 的渲染，不信可以试试红色文字的显示和黑白文字的显示需要占用的内存。

另外，或许你会想到 UIColor 的有个通过 HSB 颜色空间初始化的方法，貌似违背以上的说法

```swift
public init(hue: CGFloat, saturation: CGFloat, brightness: CGFloat, alpha: CGFloat)
```

HSB 确实是一种颜色空间，但是也是基于 RGB，在[维基百科](https://en.wikipedia.org/wiki/Color_space#Generic)有相应的解释，以及[官方文档](https://developer.apple.com/documentation/uikit/uicolor/1621931-init)也提到最终还是 RGB。



### RGB

前面提到的 RGB 颜色空间，每像素拥有的总 bit 数并不一定都是 32 bits，也有一种特殊情况是 16 bits。首先 32 bpp 的意思就是，在 R G B 三个颜色上都用 8 bits 去表示，比如 Red 颜色有 2^8 个数去表示，也就是 0 - 255 个数值，当然剩下的 8 bits 留给了 alpha。那么 16 bpp 就是在 R G B 上分别使用 5 6 5 位去去表示，至于为什么 G 分到 6 位而不是其他的，据说是因为人类的眼睛对绿色比较敏感。所以它们还有另外一种表述形式叫做 RGB888 和 RGB565。

RGB888 示意图

![AboutImage-6](http://lc.yardwill.top/AboutImage-6.jpg)



RGB565 示意图

![AboutImage-7](http://lc.yardwill.top/AboutImage-7.jpg)

比如我们需要创建一个 bitmap 来表示 RGB565，以十六进制 0x001f 表示其中的一个像素，转化为二进制就是 11111，这时候它表示并不是红色，而是蓝色，因为如果高位不够就会用 0 来补，左边 -> 右边 就是高位 -> 低位，所以最终其实是 0000000000011111 来表示一个像素的颜色，也就是蓝色。知道这些后我们可以动手创建一个RGB565 的 bitmap

```swift
    static func makeData() -> UnsafeMutablePointer<UInt16> {
        let capacity = 200 * 200 * 2
        let imageBuffer = UnsafeMutablePointer<UInt16>.allocate(capacity: capacity)
        for row in 0..<200 {
            let color: UInt16 = row >= 100 ? 0x001f : 0x7e0
            for col in 0..<200 {
                imageBuffer[row * 200 + col] = color
            }
        }
        free(imageBuffer)
        return imageBuffer
    }
```

这里设置 200 * 200 分辨率的图片上半部分是 0x7e0 也就是 11111100000 绿色，下半部分是 0x001f 就是刚才说的蓝色，然后通过 CGImage 生成图片。

```swift
static var RGB565Image: UIImage? {
        let width = 200
        let height = 200
        let rawData = makeData()
        guard let data = Data(bytes: rawData, count: width * height * 2) as CFData?
            , let provider = CGDataProvider(data: data) else {
                return nil
        }
        
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let bitmapInfo =  CGBitmapInfo(rawValue: CGImageAlphaInfo.noneSkipFirst.rawValue | CGBitmapInfo.byteOrder16Little.rawValue)
        let imageRef = CGImage(width: width,
                               height: height,
                               bitsPerComponent: 5,
                               bitsPerPixel: 16,
                               bytesPerRow: width * 2,
                               space: colorSpace,
                               bitmapInfo: bitmapInfo,
                               provider: provider,
                               decode: nil,
                               shouldInterpolate: false,
                               intent: CGColorRenderingIntent.defaultIntent
        )
        
        guard let cgImage = imageRef else {
            return nil
        }
        
        let finalImage = UIImage(cgImage: cgImage)
        return finalImage
    }
```

注意 CGBitmapInfo 的参数，不需要 alpha 通道 `noneSkipFirst`，以及 16 位小端模式 `byteOrder16Little`，这里不做过多介绍，有兴趣的可以搜一搜。



## 未完待续



