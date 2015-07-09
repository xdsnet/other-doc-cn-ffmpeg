## 37 视频滤镜 ##
在配置编译FFmpeg时可以通过`--disable-filters`来禁止所有滤镜的编译。也可以配置编译脚本来输出所有包含进编译的滤镜信息。

下面是当前可用的视频滤镜介绍。

### alphaextract ###
把输入视频作为灰度视频来提取透明通道，它通常和`alphamerge`滤镜联用。

### alphamerge ###
通过添加或者替换透明通道，让主要视频与另外一路视频混合。这里主要是使用`alphaextract`来让不支持透明通道的视频成为允许传输或存储帧透明的帧序列

例如：为了重建完整的帧，让一个普通的YUV编码视频和利用了`alphaextract`的一个单独的视频混合：

	movie=in_alpha.mkv [alpha]; [in][alpha] alphamerge [out]

因为这个滤镜专为重建帧设计，所以它的运行过程中没有考虑时间戳，而是在输入达到流结束时终止。这可能将导致编码管道丢帧，如果你仅想实现一个图像叠加到视频，建议采用`overlay`滤镜。

### ass ###
类似[`subtitles`](#subtitles)，除了它不需要`libavcodec`和`libavformat`。另一方面，它受限于ASS（Advanced Substation Alpha-高级子透明）字幕格式文件

这个滤镜除了[`subtitles`](#subtitles)滤镜选项外还接受下面选项：

- shaping

	设置图像引擎，有效的值有：

	‘auto’
	
	    默认的`libass`图像引擎，它是最合适的值
	‘simple’
	
	    快, 仅采用font-agnostic图像
	‘complex’
	
	    慢，采用OpenType。
	
	默认为auto. 

### bbox ###
在输入的亮度平面上计算非黑边缘。

这个滤镜按设定的最小亮度值计算平面上所有像素图像的编译（按亮度——即按阀值连接起来构成多个区块，阀值连线即边缘）。描述边缘的参数会记录到滤镜日志中。

滤镜接受下面的选项：

- min_val

	设置最小的亮度值（其下认为是黑的，其上为非黑），默认16

### blackdetect ###
检测视频（几乎）完全是黑色的时间间隔（段）。它可以用于检测章间的过渡、广告或者无效的录制。输出是以秒为单位的检测出黑场时间段的开始、结束和持续时间线信息。

为了显示（输出）时间线信息，你需要设置日志层次为`AV_LOG_INFO`。

滤镜接受下面的选项：

- black_min_duration, d

    以秒为单位设置最小黑场持续时间（小于这个的不被检出），它必须是非负浮点数，默认为2.0
- picture_black_ratio_th, pic_th

    设置画面黑场检验标准，即图像中黑色像素占画面所有像素比的最小值:

    nb_black_pixels / nb_pixels

    如果画面中黑色像素占比大于这个值则判断当前画面是黑场画面，默认值为0.98
- pixel_black_th, pix_th

    设置判断像素是黑色像素的标准

    采用像素亮度值来判断，只有像素亮度不大于一个设定的值（所确定的标准，见后）即为黑色像素。其依据下面的方程计算亮度:

    absolute_threshold = luminance_minimum_value + pixel_black_th * luminance_range_size

    `luminance_range_size` 和 `luminance_minimum_value `依赖于输入视频格式，范围为 对YUV全范围格式视频[0-255]，非全范围格式[16-235]

    默认值0.10（对于YUV全范围格式，则计算值为 0+0.1*255=25.5，对于非YUV全范围则为 16+0.1*(235-16)=37.9）

下面的例子设置黑像素标准为最小值，黑场时长大于等于2秒:

	blackdetect=d=2:pix_th=0.00

### blackframe ###
判断（几乎）黑帧。它可以用于检测章间的过渡、广告等。输出内容包括检测到的黑帧数、占比和文件中的偏移（对文件格式支持检测，如果不支持检测为-1）和以秒为单位的时间戳。

为了显示这些输出信息，需要设置日志层次为`AV_LOG_INFO`

它接受下面的参数：

- amount

	像素为黑在帧中的占比百分比数，默认98
- threshold， thresh

	像素为黑的判断标准，默认为32

### blend，tblend ###
混合两个视频帧

其中`blend`混合两路输出1路流，第一个输入为`top`层，二个路为`bottom`层，输出以输入短的为结束。而`tblend`（时间混合）需要从一个单独视频流的连续两帧，让新帧在上叠加在老帧上。

接受选项的介绍如下：

- c0_mode
- c1_mode
- c2_mode
- c3_mode
- all_mode

    设置混合模式（对指定像素或者所有像素——利用`all_mode`），默认值是`normal`

    当前有效的混合模式如下:

    ‘addition’
    ‘and’
    ‘average’
    ‘burn’
    ‘darken’
    ‘difference’
    ‘difference128’
    ‘divide’
    ‘dodge’
    ‘exclusion’
    ‘glow’
    ‘hardlight’
    ‘hardmix’
    ‘lighten’
    ‘linearlight’
    ‘multiply’
    ‘negation’
    ‘normal’
    ‘or’
    ‘overlay’
    ‘phoenix’
    ‘pinlight’
    ‘reflect’
    ‘screen’
    ‘softlight’
    ‘subtract’
    ‘vividlight’
    ‘xor’

- c0_opacity
- c1_opacity
- c2_opacity
- c3_opacity
- all_opacity

    设置特定像素的透明度，或者设置整个透明度（利用`all_opacity`），仅用于组合像素混合模式`blend`滤镜.
- c0_expr
- c1_expr
- c2_expr
- c3_expr
- all_expr

    设置特定像素混合表达式或所有像素混合表达式（`all_expr`），**注意**如果它们被设定，则相关模式选项被忽略

    表达式可以采用下面的变量:

    N

        进入滤镜的帧序数，从0开始计数
    X
    Y

        样本点坐标（像素坐标）
    W
    H

        整个帧画面的宽和高（原始的）
    SW
    SH

        取决于当前滤镜平面的宽和高。它对应于像素亮度平面和当前平面的比值，如对于YUV4：2：0格式，则对于亮度平面为1，1，对于色度平面则是0.5，0.5
    T

        当前帧的时间，单位秒
    TOP, A

        `top`层的视频帧上当前像素值
    BOTTOM, B

        `bottom`层的视频帧上当前像素值

- shortest

    在短输入结束时强制结束，默认为0，只用于`blend`滤镜
- repeatlast

    在结束流后继续应用底帧。值为0表明不继续应用底帧，默认为1.选项只用于`blend`滤镜 

	
#### blend，tblend例子 ####	
- 在前10秒应显示底帧:

    blend=all_expr='A*(if(gte(T,10),1,T/10))+B*(1-(if(gte(T,10),1,T/10)))'
-  显示1x1棋盘效应效果（有的显示A，有的显示B）:

    blend=all_expr='if(eq(mod(X,2),mod(Y,2)),A,B)'
- 从左到右揭开的效果:

    blend=all_expr='if(gte(N*SW+X,W),A,B)'
- 从上到下揭开效果:

    blend=all_expr='if(gte(Y-N*SH,0),A,B)'
- 从右下向左上揭开效果:

    blend=all_expr='if(gte(T*SH*40+Y,H)*gte((T*40*SW+X)*W/H,W),A,B)'
- 显示当前和前一帧之间的差异:

    tblend=all_mode=difference128

### boxblur ###

对输入视频应用边缘虚化

接受下面的参数：

- luma_radius, lr
- luma_power, lp
- chroma_radius, cr
- chroma_power, cp
- alpha_radius, ar
- alpha_power, ap

接受选项的介绍见下：

- luma_radius, lr
- chroma_radius, cr
- alpha_radius, ar

    在输入平面，设置用于边缘模糊的像素半径表达式

    这个半径必须是非负数，且必须在亮度和透明度（平面/通道上）大于min(w,h)/2，在色度平面上大于 min(cw,ch)/2 

    `luma_radius`默认为2，如果没有指定 is "2". `chroma_radius`和`alpha_radius`默认是根据`luma_radius`值的默认集

    表达式中允许下面内容：

    w
    h

        输入视频宽和高
    cw
    ch

        输入色度图像的高和宽
    hsub
    vsub

        水平和垂直采样的色彩浓度值，例如对于 "yuv422p"像素格式，`hsub`为2，`vsub`为1 

- luma_power, lp
- chroma_power, cp
- alpha_power, ap

    指定应用于相应平面的边缘虚化滤镜次数。

    默认`luma_power`为2，`chroma_power`和`alpha_power`的默认值是根据`luma_power`的对于默认值集

    如果有1个值为0则禁止了效果
#### boxblur例子 ####

- 对亮度、色度和透明都以2的半径进行边缘虚化（模糊）:

    boxblur=luma_radius=2:luma_power=1
    boxblur=2:1
- 设置亮度编译模糊半径为2，而透明和色度模糊半径为0:

    boxblur=2:1:cr=0:ar=0
- 设置亮度和色度模糊半径是视频维度的小部分（这里是1/10）:

    boxblur=luma_radius=min(h\,w)/10:luma_power=1:chroma_radius=min(cw\,ch)/10:chroma_power=1

### codecview ###
可视化展示一些编码的信息

一些编码可以在图像的一端或者其他地方输出信息。例如一些`MPEG`编码器就可以通过设置`flags2`标志为`export_mvs`来输出运动矢量检测信息

滤镜接受下面选项：

- mv

    设置运动矢量检测来可视化，

    对`mv`选项有效的标志有:

    ‘pf’

        基于P帧的前向预测 
    ‘bf’

        基于B帧的前向预测 
    ‘bb’

        基于B帧的后向预测 

#### codecview例子 ####
- 基于P帧和B帧，可视化多径向运动矢量检测

	ffplay -flags2 +export_mvs input.mpg -vf codecview=mv=pf+bf+bb

### colorbalance ###
对输入帧编辑主要的颜色（红、绿和蓝）强度

滤镜用于对输入帧调整阴影、中间调或高亮区域实现 red-cyan（红和蓝绿）, green-magenta（绿和品红） 或blue-yellow（蓝和黄）的平衡。

一个正向的调整值对应于平衡主要颜色，一个负数值则对应于补色。

滤镜接受下面的选项：

- rs
- gs
- bs

    调整red, green和blue的阴影(黑暗像素).
- rm
- gm
- bm

    调整red, green和blue的中间调 (中亮度像素).
- rh
- gh
- bh

    调整red, green和blue的高亮 (亮的像素).

    所有值的取值范围为[-1.0, 1.0]，默认为0（不调整）

#### colorbalance例子 ####
- 添加影子的红色

	colorbalance=rs=.3

### colorkey ###
RGB颜色键控

滤镜接受下面的选项：

- color

    设置被作为透明的颜色
- similarity

    设置对色键的相似性百分比（表示也作为色键的颜色范围）

    0.01匹配表示只有色键，而1表示所有颜色（相当于直接是透明了）
- blend

    混合百分比

    0.0 表示像素完全透明或者完全不透明

    更高的值导致半透明像素，对于更高的透明度相当于像素的颜色更接近于色键 

#### colorkey例子 ####
- 让所有的绿色像素透明

	ffmpeg -i input.png -vf colorkey=green out.png
- 在绿屏上显示背景图片

	ffmpeg -i background.png -i video.mp4 -filter_complex "[1:v]colorkey=0x3BBD1E:0.3:0.2[ckout];[0:v][ckout]overlay[out]" -map "[out]" output.flv

### colorlevels ###
使用层来调整输入视频

滤镜接受下面的选项：

- rimin
- gimin
- bimin
- aimin

    调整输入红、绿、蓝和透明的黑（标准），允许范围为[-1.0,1.0]，默认为0
- rimax
- gimax
- bimax
- aimax

    调整输入红、绿、蓝和透明的白（标准），允许范围为[-1.0,1.0]，默认为1.

    输入水平被用来减轻突出（明亮的色调），变暗阴影（暗色调），改变亮和暗的平衡
- romin
- gomin
- bomin
- aomin

    调整输出红、绿、蓝和透明的黑（标准），允许范围为[-1.0,1.0]，默认为0
- romax
- gomax
- bomax
- aomax

     调整输出红、绿、蓝和透明的白（标准），允许范围为[-1.0,1.0]，默认为1.

    用来手动调整输出电平范围 

#### colorlevels例子 ####
- 让视频输出暗色调

	colorlevels=rimin=0.058:gimin=0.058:bimin=0.058
- 增加对比度

	colorlevels=rimin=0.039:gimin=0.039:bimin=0.039:rimax=0.96:gimax=0.96:bimax=0.96
- 让视频更亮

	colorlevels=rimax=0.902:gimax=0.902:bimax=0.902
- 增加亮度

	colorlevels=romin=0.5:gomin=0.5:bomin=0.5

### colorchannelmixer ###
通过重新混合颜色通道调整视频输入帧

这个滤镜通过在不同颜色通道对同一个像素添加与其他通道相关的值进行颜色调整，例如为了修改红色，输出会是：

	red=red*rr + blue*rb + green*rg + alpha*ra

滤镜接受下面选项：

- rr
- rg
- rb
- ra

    为调整输出的红色通道而进行参数设置（分别对应于根据红、绿、蓝和透明通道的权值），rr默认为1，其他默认为0
- gr
- gg
- gb
- ga

    为调整输出的绿色通道而进行参数设置（分别对应于根据红、绿、蓝和透明通道的权值），gg默认为1，其他默认为0
- br
- bg
- bb
- ba

    为调整输出的蓝色通道而进行参数设置（分别对应于根据红、绿、蓝和透明通道的权值），bb默认为1，其他默认为0
- ar
- ag
- ab
- aa

    为调整输出的透明通道而进行参数设置（分别对应于根据红、绿、蓝和透明通道的权值），aa默认为1，其他默认为0

    各个值的运行范围为[-2.0, 2.0]. 

#### colorchannelmixer例子 ####
- 转换源到灰度模式

	colorchannelmixer=.3:.4:.3:0:.3:.4:.3:0:.3:.4:.3
- 模拟墨色应用

	colorchannelmixer=.393:.769:.189:0:.349:.686:.168:0:.272:.534:.131

### colormatrix ###
颜色转换矩阵（常用于颜色标准转换）

滤镜接受下面的选项：

- src
- dst

    分别指定源和目标的颜色矩阵模式。两个值都必须要设置。

    接受的值为：

    ‘bt709’

        BT.709
    ‘bt601’

        BT.601
    ‘smpte240m’

        SMPTE-240M
    ‘fcc’

        FCC 

例如要转换BT.601 到 SMPTE-240M需要：

	colormatrix=bt601:smpte240m

### copy ###
复制输入源，而不改变的输出，常用于测试。

### crop ###
裁剪输入

接受下面的参数：

- w, out_w

    输出视频的宽，默认为`iw`。这个表达式只有过滤器配置是进行一次求值
- h, out_h

    输出视频的高，默认为`iw`。这个表达式只有过滤器配置是进行一次求值
- x

    输入的水平坐标，默认为(in_w-out_w)/2，每帧都计算
- y

    输入的垂直坐标，默认为(in_h-out_h)/2，每帧都计算
- keep_aspect

    如果设置为1，则强制输出采样输入一样的长宽比例，默认为0

这里 `out_w`, `out_h`, `x`, `y`参数都可以是表达式，包含下述意义:

- x
- y

    计算x和y，通常每帧计算
- in_w
- in_h

    输入的宽和高
- iw
- ih

    同于`in_w`和`in_h`
- out_w
- out_h

    输出的（裁剪后的）宽和高
- ow
- oh

    同于`out_w`和`out_h`
- a

    同于`iw / ih`
- sar

    输入的样本点（像素）长宽比
- dar

    输入显示样本点长宽比它同于 `(iw / ih) * sar`
- hsub
- vsub

    水平和垂直颜色分量值。例如对于"yuv422p"的像素，`hsub`为2，`vsub`为1
- n

    输入帧的序数，从0计数
- pos

    输入帧在文件中的偏移，如果未知则为`NAN`
- t

    秒为单位的输入帧时间戳，如果时间戳未知则为`NAN`

#### crop例子 ####
- 从(12,34)开始裁剪出一个100x100的图像

    crop=100:100:12:34
-  使用命名选项，同上例一样:

    crop=w=100:h=100:x=12:y=34
-  在输入中心裁剪出100x100:

    crop=100:100
- 裁剪输入的2/3区域:

    crop=2/3*in_w:2/3*in_h
- 裁剪输入中心区域:

    crop=out_w=in_h
    crop=in_h
-  据边缘100裁剪出中间部分.

    crop=in_w-100:in_h-100:100:100
-  据左右10像素，上下20像素的中间部分

    crop=in_w-2*10:in_h-2*20
- 保留右下角（四分后的右下部分）

    crop=in_w/2:in_h/2:in_w/2:in_h/2
- 按黄金分割裁剪

    crop=in_w:1/PHI*in_w
- 应用抖动效果

    crop=in_w/2:in_h/2:(in_w-out_w)/2+((in_w-out_w)/2)*sin(n/10):(in_h-out_h)/2 +((in_h-out_h)/2)*sin(n/7)
- 取决于时间戳的不定照相效果:

    crop=in_w/2:in_h/2:(in_w-out_w)/2+((in_w-out_w)/2)*sin(t*10):(in_h-out_h)/2 +((in_h-out_h)/2)*sin(t*13)"
- 让x依赖于y

    crop=in_w/2:in_h/2:y:10+10*sin(n/10)

### cropdetect ###
自动检测裁剪尺寸(自动去除边缘的黑部)

它计算必要的剪切参数并通过日志系统输出推荐的参数。检测尺寸对应于输入视频的黑色区域

它接受下面的参数：

- limit

    设置更高的黑色阀值。可以选择从0到所有（255，基于8位格式）。一组强度值更大的价值被认为是黑色。它默认为24。你也可以指定一个值在0.0和1.0之间,将按比例根据像素格式的位深计算
- round

    设置可行（分割出）的宽/高分割基数，默认为16。偏移量自动调整到视频的中心，对于平面维度采用2的倍数（4：2：2视频需要的），16已经可以满足大多数编码要求了
- reset_count, reset

    设置在处理多少帧后计数器重置以重新检测最佳裁剪区域。 默认为0.

    它被用于通道标志混乱的视频。0表示不复位，并返回已播放视频的最大区域 

### curves ###
利用curves（曲线）实现颜色调整

这个滤镜模拟Adobe Photoshop 和GIMP的`curves`工具。每个分量（红、绿和蓝）都由定义的`N`个节点相互作用成为光滑的曲线来调整输入到输出的关系。其中x轴是像素从输入帧获取的值，而y轴则是对应作为输出帧的值（从而实现输入到输出的映射）。

默认原始的曲线实为（0，0）和（1，1）连接的直线。这种“变化”关系就是输出=输入，意味着没有变化。

滤镜允许根据设置的两点或者更多点产生一个新的分量关系曲线（采用自然三次样条插值法依次连接各个点形成的平滑曲线）。所有的点坐标中x和y值范围在[0,1]，超出定义域的点都被抛弃。

如果定义中没有关键点定义在x=0，则自动添加点(0,0)，类似，如果没有关键点定义在x=1，则自动添加一个（1，1）点。

下面是滤镜允许的选项：

- preset

    选定一个颜色预置（配置）。这个选项用在除了r，g，b选项的设置。在这种情况下，在后的选择作为实际预置。可用的预置有:

    ‘none’
    ‘color_negative’
    ‘cross_process’
    ‘darker’
    ‘increase_contrast’
    ‘lighter’
    ‘linear_contrast’
    ‘medium_contrast’
    ‘negative’
    ‘strong_contrast’
    ‘vintage’

    默认为none. 
- master, m

    设置主要的关键点。这些关键点用于定义第二个通过映射。它们有时调用"luminance" 或者 "value"映射。它们可以被用于r, g, b 或者 all，因为它们更像一个后处理LUT。 
- red, r

    对红色分量指定关键点 
- green, g

    对绿色分量指定关键点 
- blue, b

    对蓝色分量指定关键点
- all

    设置针对所有分量的关键点（对复合信号，但不包括主要的——即对单独设置了的分量外的其他分量的处理）， 在这种情况下，没有单独设置的分量都采用这里的设置 
- psfile

    指定一个Photoshop曲线文件(.asv)来导入设置

#### curves例子 ####
- 略有增加中间层次的蓝色:

    curves=blue='0.5/0.58'
- 复古的效果:

    curves=r='0/0.11 .42/.51 1/0.95':g='0.50/0.48':b='0/0.22 .49/.44 1/0.8'

	在这里,我们为每个分量设置以下坐标:

    red

        (0;0.11) (0.42;0.51) (1;0.95) 
    green

        (0;0) (0.50;0.48) (1;1) 
    blue

        (0;0.22) (0.49;0.44) (1;0.80) 
- 前面的例子也可以通过预置实现:

    curves=preset=vintage
- 或者更简单的:

    curves=vintage
- 采用Photoshop预置并应用于绿色分量:

    curves=psfile='MyCurvesPresets/purple.asv':green='0.45/0.53'

### dctdnoiz ###
采用2D DCT来降噪（频域滤波）

滤镜不能实时应用。

滤镜接受下面的选项：

- sigma, s

    设置固定的噪声标准差（西格玛——sigma）

    这里的`sigma`用于确定一个固定的阀值 `3*sigma`，每个DCT系数（绝对值）如果低于这个阀值则被丢弃（认为是噪声）

    如果你需要更多高级的过滤特性，则需要`expr`

    默认为 0
- overlap

    为每个块重叠的像素数。如果滤镜非常慢，你可以降低这个值，从而在较少的成本下有效过滤各种纹理

    如果重叠值不允许处理整个输入宽度或高度,将显示一个警告，且在边界不会运用

    默认为blocksize-1,它通常是一个最佳设置
- expr, e

    设置系数表达式

    对于每个DCT块的系数，表达式会计算得出一个乘数系数值

    如果这里被设置，则`sigma`选项被忽略

    系数的绝对值可以通过`c`变量访问到
- n

    按位宽设置块尺寸。1远远小于n（1<<n）,这决定于处理块的宽度和高度

    默认值是3(对应于8x8)和 4（对应于16x16），**注意**改变这个值将极大影响处理速度，同时一个更大的块大小并不一定有好的降噪效果

#### dctdnoiz例子 ####
- 以`sigma`为4.5降噪

	dctdnoiz=4.5
- 同上效果，但是采用了`expr`

	dctdnoiz=e='gte(c, 4.5*3)'
- 块大小设置为16x16

	dctdnoiz=15:n=4

### decimate ###
定期重复的帧数丢弃

接受下面的选项：

- cycle

    指示要被丢弃的帧。设置为`N`表示如果一帧已经被重复`N`次就被丢弃，默认为5
- dupthresh

    设置检验重复的阀值。如果帧间变化量小于等于阀值则被认为是重复的。默认为1.1.
- scthresh

    设置场景变化阀值，默认为15
- blockx
- blocky

    设置x或y轴大小，其用于度量计算内存。更大的内存块可以更好的实现噪声抑制，但小的运动检测效果也越差。 必须是2的幂。，默认为32
- ppsrc

    标记一个输入是有预处理的，还有一个是原始源。它允许通过输入（经各种滤镜）预处理的源和原始源来更好的无损保持内容。设置为1时,第一个输入是经过预处理的输入流,第二个流是清洁源，其保留了原始的帧（信息）。 默认为0.
- chroma

    设置颜色通道是否也用这个矩阵计算。默认为1

### dejudder ###
删除部分隔行电视电影的内容产生的颤动

颤动可被引发，例如通过[`pullup`](#pullup)滤镜，如果原始内容经`pullup`产生向上的爬动效应，`dejudder`可以引入动态帧频来消除。它可能会改变容器的帧频记录，除了这个改变，滤镜不会影响固定帧频视频的其他地方。

下面的选项被允许：
- cycle

    指定颤抖的重复窗口的长度

    接受一个大于1的整数，常用值有:

    ‘4’

        常用于把24帧电影转换为30帧的NTSC信号
    ‘5’

        用于把25帧的PAL转换为30帧的NTSC
    ‘20’

        前俩个的混合 

    默认为‘4’. 

### delogo ###
通过一个简单的插值周围像素来抑制台标。仅需要在台标周围设置一个矩形遮盖就可以消除台标（有时会出现莫名的东西）

它接受下面的参数：

- x
- y

    指定台标覆盖的x和y坐标，必须被设置
- w
- h

    指定覆盖的宽和高，必须被设置（联合前面的x和y就定义了一个矩形覆盖区）
- band, t

    指定矩形的边缘模糊厚度（添加到w和h），默认为4
- show

    但设置为1时，一个绿色的矩形被简单填充到覆盖区。默认为0，这时采用的区域内像素填充计算方法为：矩形内，画在最外层的像素将(部分)内插替换值。下一个像素的值在每个方向用于计算矩形内的内插像素值。

#### delogo例子 ####
- 采用默认方法，在坐标（0,0）采用100x77，模糊10的台标覆盖：

	delogo=x=0:y=0:w=100:h=77:band=10

### deshake ###
尝试在小的范围内去除/修复 水平/垂直变化。用于去除手持、或者旋转三脚架以及在移动车辆上拍摄的抖动（防抖效果）

接受下面的选项

- x
- y
- w
- h

    定义一个处理区域，它限定了动态监测搜索范围。它定义了有限运动矢量的左上角位置以及宽度和高度。这些参数随着[`drawbox`](#drawbox)滤镜一样有相同的意义，可以用来界定想象的边界位置

    对于同时运动（比如在车上拍摄），它十分有用，因为主题框架内混淆了本身有益的运动和无意的抖动，通过限定可以有效的区分，减少搜索难度。

    如果这些参数中的一个或者所用被设置为-1则意味着在全帧应用。这使得后来选项设置不指定的边界框运动矢量搜索

    默认为搜索整个帧
- rx
- ry

    指定最大程度x和y方向运动范围，值范围为0-64像素，默认为16.
- edge

    指定如何生成像素来填补空白的边缘。可用值:

    ‘blank, 0’

        在空白的地方填零 
    ‘original, 1’

        填充原始图像（背景部分） locations 
    ‘clamp, 2’

        在空白地方挤压边缘值 
    ‘mirror, 3’

        在空白位置反映边缘

    默认为 ‘mirror’.
- blocksize

    指导用于运动检测的块尺寸，范围4-128像素，默认8.
- contrast

    指定块的对比度阈值。只有块超过指定的对比(黑暗和轻的像素)将被考虑。范围1-255,默认125 。
- search

    指定的搜索策略。可用值:

    ‘exhaustive, 0’

        穷举搜索
    ‘less, 1’

        不详尽的搜索 

    默认‘exhaustive’.
- filename

    如果设置,那么运动搜索的详细日志写入指定的文件
- opencl

    如果设置为1时,指定使用OpenCL功能,只在编译FFmpeg时配置了`——enable-opencl`可用。默认值是0。

### detelecine ###
使用电视电影的逆操作。它需要一个预定的模板指定使用过的模式（必须同原始操作一样的参数）

滤镜接受下面的选项：

- first_field

    ‘top, t’

        顶场优先
    ‘bottom, b’

        底场优先

	默认为top. 

- pattern

    一串数字表示您想要应用的下拉模式。默认值是23。
- start_frame

    许多代表第一帧的位置对电视电影模式,这用于流被剪辑过。默认值是0。 

### drawbox ###
在输入图像上画一个颜色区块。

它接受下面的参数：

- x
- y

    指定区块的x和y坐标，可以是表达式，默认为0
- width, w
- height, h

    指定表示区块宽和高的表达式，如果为0表示整个输入的宽和高，默认为0
- color, c

    指定填充颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。如果指定的值是`invert`，则区块边缘颜色采用视频该处颜色的反亮（这样可以衬托出区域）。
- thickness, t

    设置边缘宽度的表达式，默认为3.

    各个表达式接受的值见下面的介绍。 

对于 x, y, w 和 h 以及t是表达式时，可以包括下面的内容:

- dar

    输入的显示长宽比，它即为 (w / h) * sar.
- hsub
- vsub

    水平和垂直色度量化分量值，例如对于"yuv422p"像素格式，hsub为2，vsub为1
- in_h, ih
- in_w, iw

    输入的宽和高
- sar

    输入样本点的长宽比
- x
- y

    区块的x和y坐标偏移
- w
- h

    区块的宽和高
- t

    区块边缘厚度

    这些x, y, w, h 和 t常（变）量都允许相互在表达式中引用，例如可以设置 y=x/dar 或 h=w/dar.

#### drawbox例子 ####
- 在输入图像上画一个黑块

	drawbox
- 采用50%透明的红色画一个区块

	drawbox=10:20:200:60:red@0.5
	
	也可以写为

	drawbox=x=10:y=20:w=200:h=60:color=red@0.5
- 采用`pink`填充一个区块

	drawbox=x=10:y=10:w=100:h=100:color=pink@0.5:t=max
- 画一个2像素红，有2.40:1的遮盖区域

	drawbox=x=-t:y=0.5*(ih-iw/2.4)-t:w=iw+t*2:h=iw/2.4+t*2:t=2:c=red

### drawgrid ###
在输入图像上画上网格线（外框）

接受下面的参数：
- x
- y

    指定区块的x和y坐标，可以是表达式，默认为0
- width, w
- height, h

    指定表示区块宽和高的表达式，如果为0表示整个输入的宽和高采用最小的`thickness`划线，默认为0.
- color, c

    指定填充颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。如果指定的值为`invert`，则网格线采用视频该处的反亮颜色绘制。
- thickness, t

    设置边缘宽度的表达式，默认为1.

    各个表达式接受的值见下面的介绍。 

对于 x, y, w 和 h 以及t是表达式时，可以包括下面的内容:

- dar

    输入的显示长宽比，它即为 (w / h) * sar.
- hsub
- vsub

    水平和垂直色度量化分量值，例如对于"yuv422p"像素格式，hsub为2，vsub为1
- in_h, ih
- in_w, iw

    输入的宽和高
- sar

    输入样本点的长宽比
- x
- y

    区块的x和y坐标偏移
- w
- h

    区块的宽和高
- t

    区块边缘厚度

    这些x, y, w, h 和 t常（变）量都允许相互在表达式中引用，例如可以设置 y=x/dar 或 h=w/dar.

#### drawgrid例子 ####
- 以2像素宽度绘制一个100x100的网格，颜色是红色，透明度50%

	drawgrid=width=100:height=100:thickness=2:color=red@0.5
- 在图像上绘制3x3网格，透明度50%

	drawgrid=w=iw/3:h=ih/3:t=2:c=white@0.5

### drawtext ###
在视频的上绘制文本或者描述于文件的文本块，使用了`libfreetype`库。

为了允许这个滤镜，在编译时需要配置`--enable-libfreetype`，为了允许默认的字体回调和`font`选项，还需要配置`--enable-libfontconfig`。为了允许`text_shaping`选项，还需要配置`--enable-libfribidi`

#### drawtext语法 ####
它接受如下参数：

- box

    设置是否在文本下衬一个背景颜色，1为要，0为不要，默认为0
- boxborderw

    设置背景块边缘厚度（用于在背景块边缘用`boxcolor`再围绕画一圈），默认为0
- boxcolor

    设置用于绘制文本底衬的颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。

    默认为"white".
- borderw

    使用`bordercolor`颜色绘制的文字边缘厚度，默认为0
- bordercolor

    绘制文本衬底的颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。

    默认为"black".
- expansion

    设置文本扩展模式。可以为`none`, `strftime` (已弃用了) 或 `normal` (默认). 见后面[文本扩展](#文本扩展)中的详细介绍
- fix_bounds

    如果为true，检查和修复文本坐标来避免剪切
- fontcolor

    设置文本颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。

    默认为"black".
- fontcolor_expr

    String which is expanded the same way as text to obtain dynamic fontcolor value. By default this option has empty value and is not processed. When this option is set, it overrides fontcolor option.
- font

    The font family to be used for drawing text. By default Sans.
- fontfile

    The font file to be used for drawing text. The path must be included. This parameter is mandatory if the fontconfig support is disabled.
- draw

    This option does not exist, please see the timeline system
- alpha

    Draw the text applying alpha blending. The value can be either a number between 0.0 and 1.0 The expression accepts the same variables x, y do. The default value is 1. Please see fontcolor_expr
- fontsize

    The font size to be used for drawing text. The default value of fontsize is 16.
- text_shaping

    If set to 1, attempt to shape the text (for example, reverse the order of right-to-left text and join Arabic characters) before drawing it. Otherwise, just draw the text exactly as given. By default 1 (if supported).
- ft_load_flags

    The flags to be used for loading the fonts.

    The flags map the corresponding flags supported by libfreetype, and are a combination of the following values:

    default
    no_scale
    no_hinting
    render
    no_bitmap
    vertical_layout
    force_autohint
    crop_bitmap
    pedantic
    ignore_global_advance_width
    no_recurse
    ignore_transform
    monochrome
    linear_design
    no_autohint

    Default value is "default".

    For more information consult the documentation for the FT_LOAD_* libfreetype flags.
- shadowcolor

    The color to be used for drawing a shadow behind the drawn text. For the syntax of this option, check the "Color" section in the ffmpeg-utils manual.

    The default value of shadowcolor is "black".
- shadowx
- shadowy

    The x and y offsets for the text shadow position with respect to the position of the text. They can be either positive or negative values. The default value for both is "0".
- start_number

    The starting frame number for the n/frame_num variable. The default value is "0".
- tabsize

    The size in number of spaces to use for rendering the tab. Default value is 4.
- timecode

    Set the initial timecode representation in "hh:mm:ss[:;.]ff" format. It can be used with or without text parameter. timecode_rate option must be specified.
- timecode_rate, rate, r

    Set the timecode frame rate (timecode only).
- text

    The text string to be drawn. The text must be a sequence of UTF-8 encoded characters. This parameter is mandatory if no file is specified with the parameter textfile.
- textfile

    A text file containing text to be drawn. The text must be a sequence of UTF-8 encoded characters.

    This parameter is mandatory if no text string is specified with the parameter text.

    If both text and textfile are specified, an error is thrown.
- reload

    If set to 1, the textfile will be reloaded before each frame. Be sure to update it atomically, or it may be read partially, or even fail.
- x
- y

    The expressions which specify the offsets where text will be drawn within the video frame. They are relative to the top/left border of the output image.

    The default value of x and y is "0".

    See below for the list of accepted constants and functions. 

The parameters for x and y are expressions containing the following constants and functions:

- dar

    input display aspect ratio, it is the same as (w / h) * sar
- hsub
- vsub

    horizontal and vertical chroma subsample values. For example for the pixel format "yuv422p" hsub is 2 and vsub is 1.
- line_h, lh

    the height of each text line
- main_h, h, H

    the input height
- main_w, w, W

    the input width
- max_glyph_a, ascent

    the maximum distance from the baseline to the highest/upper grid coordinate used to place a glyph outline point, for all the rendered glyphs. It is a positive value, due to the grid’s orientation with the Y axis upwards.
- max_glyph_d, descent

    the maximum distance from the baseline to the lowest grid coordinate used to place a glyph outline point, for all the rendered glyphs. This is a negative value, due to the grid’s orientation, with the Y axis upwards.
- max_glyph_h

    maximum glyph height, that is the maximum height for all the glyphs contained in the rendered text, it is equivalent to ascent - descent.
- max_glyph_w

    maximum glyph width, that is the maximum width for all the glyphs contained in the rendered text
- n

    the number of input frame, starting from 0
- rand(min, max)

    return a random number included between min and max
- sar

    The input sample aspect ratio.
- t

    timestamp expressed in seconds, NAN if the input timestamp is unknown
- text_h, th

    the height of the rendered text
- text_w, tw

    the width of the rendered text
- x
- y

    the x and y offset coordinates where the text is drawn.

    These parameters allow the x and y expressions to refer each other, so you can for example specify y=x/dar. 

#### 文本扩展 ####

#### drawtext例子 ####






