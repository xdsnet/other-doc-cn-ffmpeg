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

    用于计算获得动态`fontcolor`值的字符串表达式。默认为控，即不处理。当被设置时将把计算结果覆盖`fontcolor`选项
- font

    设置选用的字体，默认为Sans.
- fontfile

    指定字体文件。信息中包括路径。这个参数在`fontconfig`被禁用时必须设置
- draw

    这个选项已不存在，参考`timeline`系统（时间线）
- alpha

    设置绘制的文本透明度，值范围为0.0-1.0。而且还可以接受x和y的变量。请参阅`fontcolor_expr`
- fontsize

    设置字体大小，默认为16 drawing text. The default value of fontsize is 16.
- text_shaping

    如果设置为1，则试图整理文本排列顺序（例如阿拉伯语是按从右到左排序），否则按给定的顺序从左到右排，默认为1
- ft_load_flags

    这些标志用于加载字体

    这些标志对应于`libfreetype`支持的标志，并结合下面的值:

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

    默认值为 "default".

    要了解更多信息，请参考文档中`libfreetype`标志的`FT_LOAD_*` 部分。
- shadowcolor

    阴影颜色。语法详见[颜色/color](ffmpeg-doc-cn-07.md#颜色color)中的介绍。

    默认为"black".
- shadowx
- shadowy

    这里的x和y是字阴影对于字本体的偏移。可以是正数或者负数（决定了偏移方向），默认为0
- start_number

    起始帧数，对于`n/frame_num`变量。 默认为0
- tabsize

    用于呈现的区域数量空间大小，默认为4
- timecode

    设置初始的时间码，以"hh:mm:ss[:;.]ff"格式。它被用于有或者没有`text`参数，此时`timecode_rate`必须被指定
- timecode_rate, rate, r

    设置时间码`timecode`的帧率（在`timecode`指定时）
- text

    要被绘制的文本。文本必须采用`UTF-8`编码字符。如果没有指定`textfile`则这个选项必须指定
- textfile

    一个文本文件，其内容将被绘制。文本必须是一个`UTF-8`文本序列

    如果没有指定`text`选项，则必须设定。

    如果同时设定了`text`和`textfile`将引发一个错误
- reload

    如果设置为1，`textfile`将在每帧前加载。一定要自动更新它，或者它可能是会被读取的或者失败
- x
- y

    指定文本绘制区域的坐标偏移。是相对于图像顶/左边的值

    默认均为"0".

    下面是接受的常量和函数 

对于x和y是表达式时，它们接受下面的常量和函数:

- dar

    输入显示接受的长宽比，它等于 (w / h) * sar
- hsub
- vsub

    水平和垂直色度分量值。例如对于"yuv422p"格式像素，`hsub`为2，`vsub`为1
- line_h, lh

    文本行高
- main_h, h, H

    输入的高
- main_w, w, W

    输入的宽
- max_glyph_a, ascent

    从基线到最高/上字形轮廓高点（所有内容的最高点）的最大距离。必须是一个正值，因为网格与Y轴方向关系使然
- max_glyph_d, descent

    从基线到最低/下字形轮廓高点（所有内容的最高点）的最大距离。必须是一个负值，因为网格与Y轴方向关系使然
- max_glyph_h

    最大字形高度，限定了所有单个字的高度，如果设置值小于实际值，则输出可能覆盖到其他行上
- max_glyph_w

    最大字形宽度，限定了所单个字显示的宽度，如果设置值小于实际值，则发生字重叠
- n

    输入帧序数，从0计数
- rand(min, max)

    返回一个min和max间的随机数
- sar

    输入样本点的长宽比
- t

    以秒计的时间戳。如果无效则为`NAN`
- text_h, th

    渲染文本的高
- text_w, tw

    渲染文本的宽
- x
- y

    文本的x和y坐标。

    所有参数都允许`x`和`y`被引用，例如`y=x/dar` 

#### 文本扩展 ####
如果`expansion`设置为`strftime`，则滤镜会接受`strftime()`序列提供的文本并进行相应扩展。检查`strftime（）`的文档。这个特性现在是弃用的。

如果`expansion`设置为`none`，则文本都是直接打印文本（即直接以文本内容不扩展进行输出）

如果`expansion`设置为`normal`（它是默认值），将应用下面的扩展规则。

序列形式`${...}`的内容将被扩展。大括号之间的文本是一个函数的名字，可能紧随其后是一些用`：`隔开的参数。如果包含特殊字符或分隔符（这里是`:`或者`}`），它们应该被转义。

**注意**对于在作为滤镜参数的`text`选项值，或者滤镜链图中的参数（多个滤镜连接时）以及是在`shell`环境中使用，则可能需要4层转义。使用文本文件可以避免这些问题（减少转义的使用）。

下面是有效的函数（功能）：

- expr, e

    计算结果的表达式

    它必须有一个参数来计算，接受计算x和y相同的常数和函数。**注意**并不是所有的常数都适合，例如`text_w`和`text_h`在此时还是一个未定义值（因为这两个值依赖于这里计算结果）
- expr_int_format, eif

    把表达式求值和输出格式化为整数

    第一个参数是用于计算的表达式，就像是`expr`函数（包括了变量/常量等），第二个参数指定输出格式，允许‘x’, ‘X’, ‘d’ 和 ‘u’，其意义同于`printf`函数（C语言）中的意义。第三个参数是可选的，用来设置格式化为固定位数，左边可以用0来填补。
- gmtime

    设置滤镜运行时间，是UTC时间。它接受一个`strftime()`格式字符串参数
- localtime

    滤镜运行的本地时间，它以本地时区表示的时间。它接受一个`strftime()`格式字符串参数
- metadata

    帧的元数据。它必须有一个指定元数据键的参数
- n, frame_num

    帧序数，从0开始计数
- pict_type

    一个字符描述当前图片类型
- pts

    当前帧的时间戳。它可以有2个参数

    第一个参数是时间戳格式，默认为`flt`是秒格式，其有微秒级精度；`hms`则代表`[-]HH:MM:SS.mmm`格式时间戳表示，其也有毫秒精度。

    第二个参数是添加到时间戳的偏移量

#### drawtext例子 ####
- 以FreeSerif字体绘制 "Test Text" ，其他可选参数均为默认值

    drawtext="fontfile=/usr/share/fonts/truetype/freefont/FreeSerif.ttf: text='Test Text'"
-  以FreeSerif字体，24大小在x=100 和y=50位置 绘制 ’Test Text’ ，字体采用黄色且还有红色边缘，字体和衬底军为20%透明度

    drawtext="fontfile=/usr/share/fonts/truetype/freefont/FreeSerif.ttf: text='Test Text':\
              x=100: y=50: fontsize=24: fontcolor=yellow@0.2: box=1: boxcolor=red@0.2"

    **注意**如果不是参数列表则不一定采用双引号来标识范围
- 把字显示在视频中间（计算位置）

    drawtext="fontsize=30:fontfile=FreeSerif.ttf:text='hello world':x=(w-text_w)/2:y=(h-text_h-line_h)/2"
- 在视频帧的最后一排从右向左滑动显示一个文本行。假设文件LONG_LINE仅包含一行且没有换行。

	drawtext="fontsize=15:fontfile=FreeSerif.ttf:text=LONG_LINE:y=h-line_h:x=-50*t"

- 从下向上显示文本行内容

    drawtext="fontsize=20:fontfile=FreeSerif.ttf:textfile=CREDITS:y=h-20*t"
- 在视频中间以绿色绘制单独的字符"g"。文本基线放置在视频半高位置

    drawtext="fontsize=60:fontfile=FreeSerif.ttf:fontcolor=green:text=g:x=(w-max_glyph_w)/2:y=h/2-ascent"
-  每3秒显示文本1秒

    drawtext="fontfile=FreeSerif.ttf:fontcolor=white:x=100:y=x/dar:enable=lt(mod(t\,3)\,1):text='blink'"
- 采用`fontconfig`来设置字体。**注意**冒号需要转义

    drawtext='fontfile=Linux Libertine O-40\:style=Semibold:text=FFmpeg'
- 输出实时编码日期（参考`strftime(3)`）:

    drawtext='fontfile=FreeSans.ttf:text=%{localtime\:%a %b %d %Y}'
- 以淡入淡出显示文本 (显示/消失）

    #!/bin/sh  
    DS=1.0 # display start  
    DE=10.0 # display end  
    FID=1.5 # fade in duration  
    FOD=5 # fade out duration  
    ffplay -f lavfi  "color,drawtext=text=TEST:fontsize=50:fontfile=FreeSerif.ttf:fontcolor_expr=ff0000%{eif\\\\: clip(255*(1*between(t\\, $DS + $FID\\, $DE - $FOD) + ((t - $DS)/$FID)*between(t\\, $DS\\, $DS + $FID) + (-(t - $DE)/$FOD)*between(t\\, $DE - $FOD\\, $DE) )\\, 0\\, 255) \\\\: x\\\\: 2 }"

关于`libfreetype`, 参考[http://www.freetype.org/](http://www.freetype.org/)

对于`fontconfig`,参考[http://freedesktop.org/software/fontconfig/fontconfig-user.html](http://freedesktop.org/software/fontconfig/fontconfig-user.html).

对于`libfribidi`, 参考[http://fribidi.org/](http://fribidi.org/). 

### edgedetect ###
检测边缘。滤镜采用精明边缘检测(Canny Edge Detection)算法

接受下面的选项：

- low
- high

    设置检测算法边缘探测的低和高的阀值。

    `high`阀值选择`强有力`的边缘像素，然后通过8向邻接检测`软弱`到`low`阀值，从而标注出边缘

    `low`和`high`的值范围为[0,1]，且`low`<=`higt`

    默认`low`为20/255, `high`为50/255.
- mode

    定义绘制(边缘)模式

    ‘wires’

        绘制一个 white/gray间隔线在黑背景上
    ‘colormix’

        混合颜色创建一个油漆/卡通效果 

    默认为 wires. 

#### edgedetect例子 ####
- 采用了自定义滞后阀值的标准边缘检测

	edgedetect=low=0.1:high=0.4
- 绘画效果没有阈值（因为high设置为0，low必须小于等于high则只能为0了）

	edgedetect=mode=colormix:high=0

### eq ###
设置亮度、对比度、饱和度和近似伽马(gamma)调整

滤镜支持下面选项：

- contrast

    设置contrast表达式，值必须是一个-2.0-2.0间的浮点数，默认为0
- brightness

    设置brightness表达式.值必须是一个-1.0-1.0间的浮点数，默认为0
- saturation

    设置saturation表达式. 值必须是一个0-3.0间的浮点数，默认为1
- gamma

    设置gamma表达式 ，值必须是一个0.1-10.0间的浮点数，默认为1
- gamma_r

    设置gamma表达式，对红色. 值必须是一个0.1-10.0间的浮点数，默认为1
- gamma_g

    设置gamma表达式，对绿色. 值必须是一个0.1-10.0间的浮点数，默认为1
- gamma_b

    设置gamma表达式，对蓝色. 值必须是一个0.1-10.0间的浮点数，默认为1
- gamma_weight

    设置gamma权重表达式.它可以用来减少高伽马值在明亮的图像区域影响,例如只是普通的白色放大，而其它保持不变。值必须是一个在0.0到1.0范围的浮点数。值为0.0时把伽马校正效果最强，为1.0没有效果。默认设置是“1”。
- eval

    设置brightness, contrast, saturation 和 gamma是表达式时的计算模式S

    它接受下面值:

    ‘init’

        仅在滤镜初始化或者命令被处理时计算
    ‘frame’

        每帧计算 

    默认为‘init’. 

下面是表达式中接受的参数:

- n

    帧序数，从0计数
- pos

    当前包在文件中的偏移，如果没有定义则为`NAN`
- r

    输入视频帧率，如果无效则为`NAN`
- t

    以秒计的时间戳，如果输入时间戳无效则为`NAN`

#### eq命令 ####
滤镜也接受下面的命令：

- contrast

    设置contrast表达式
- brightness

    设置brightness表达式
- saturation

    设置saturation表达式
- gamma

    设置gamma表达式
- gamma_r

    设置gamma_r表达式
- gamma_g

    设置gamma_g表达式
- gamma_b

    设置gamma_b表达式
- gamma_weight

    设置gamma_weight表达式

    命令接受对应选项中相同的语法

    如果指定的表达式是无效的，则保持当前值

### extractplanes ###
从输入流分离单独的颜色通道成为灰度视频流

滤镜接受下面选项：

- planes

    设置要提取的通道

    接受下面的值（标识通道）:

    ‘y’
    ‘u’
    ‘v’
    ‘a’
    ‘r’
    ‘g’
    ‘b’

    选择无效的值会产生错误。这也意味着同时你只能选择`r, g, b和y`，或者`y, u, v`

#### extractplanes例子 ####
-  提取亮度和`u`和`v`颜色分量到3个灰度输出。

	ffmpeg -i video.avi -filter_complex 'extractplanes=y+u+v[y][u][v]' -map '[y]' y.avi -map '[u]' u.avi -map '[v]' v.avi

### elbg ###
应用多色调分色印效果，使用了ELBG(增强型LBG)算法。（构建颜色模板）

对每个输入图像，滤镜会对于给定编码长度计算最优的从输入到输出的的映射，它们对应于不同的输出颜色的数量

滤镜接受下面的选项：

- codebook_length, l

    设置编码长度,值必须是正整数，代表不同的输出颜色数量，默认为256
- nb_steps, n

    设置计算最优映射的最大迭代数。值越高，结果越好，但越耗时。默认为1
- seed, s

    设置一个随机种子，必须是0-UINT32_MAX间的整数，如果不指定，或者设置为-1 ，则会自动选择一个好的随机值

### fade ###
应用淡入/淡出

它接受下面参数：

- type, t

    指定类型是`in`代表淡入，`out`代表淡出，默认为`in`
- start_frame, s

    指定应用效果的开始时间，默认为0.
- nb_frames, n

    应用效果的最后一帧序数。对于淡入，在此帧后将以本身的视频输出，对于淡出此帧后将以设定的颜色输出，默认25.
- alpha

    如果设置为1，则只在透明通道实施效果（如果只存在一个输入），默认为0
- start_time, st

    指定按秒的开始时间戳来应用效果。如果`start_frame`和`start_time`都被设置，则效果会在更后的时间开始，默认为0
- duration, d

    按秒的效果持续时间。对于淡入，在此时后将以本身的视频输出，对于淡出此时后将以设定的颜色输出。如果`duration`和`nb_frames`同时被设置，将采用`duration`值。默认为0（此时采用`nb_frames`作为默认）
- color, c

    设置淡化后（淡入前）的颜色，默认为"black". 

#### fade例子 ####
-  30帧开始淡入

    fade=in:0:30
- 等效上面

    fade=t=in:s=0:n=30
- 在200帧视频中从最后45帧淡出

    fade=out:155:45
    fade=type=out:start_frame=155:nb_frames=45
- 对1000帧的视频25帧淡入，最后25帧淡出:

    fade=in:0:25, fade=out:975:25
- 让前5帧为黄色，然后在5-24淡入:

    fade=in:5:20:color=yellow
- 仅在透明通道的第25开始淡入

    fade=in:0:25:alpha=1
- 设置5.5秒的黑场，然后开始0.5秒的淡入:

    fade=t=in:st=5.5:d=0.5

### fftfilt ###
在频域内应用任意表达式于样品

- dc_Y

    调整亮度dc值（增益），范围0-1000，默认为0
- dc_U

    调整色度第1分量dc值（增益），范围0-1000，默认为0
- dc_V

    调整色度第2分量dc值（增益），范围0-1000，默认为0
- weight_Y

    设置对于亮度的频域权重表达式
- weight_U

    设置对于色度第1分量的频域权重表达式
- weight_V

    设置对于色度第2分量的频域权重表达式

    滤镜接受下面的变量: 
- X
- Y

    对应当前样本点的坐标
- W
- H

    当前图像的宽和高 

#### fftfilt例子 ####
- 高通:

    fftfilt=dc_Y=128:weight_Y='squish(1-(Y+X)/100)'
- 低通:

    fftfilt=dc_Y=0:weight_Y='squish((Y+X)/100-1)'
- 锐化:

    fftfilt=dc_Y=0:weight_Y='1+squish(1-(Y+X)/100)'

### field ###
使用步算法从隔行图像中提取单个场来避免浪费CPU时间。标记为逐行输出帧。

滤镜接受下面选项：

- type

	指定是`top`（或者0）还是`bottom`（或者1）类型的场

### fieldmatch ###
场匹配用于反转电视（隔行）电影。它为了从电视流中建立起逐帧电影，需要过滤保留部分重复帧，为了更好的对压缩转换电视`fieldmatch`滤镜需要跟随一个[`decimate`](#decimate)滤镜之类的抽取滤镜。

为了更好的分离和抽取，需要在两个滤镜间插入一个反交错滤镜。如果源是混合了电视电影和现实的内容，则单独的`fieldmatch`滤镜不足以分离出交错内容，所以需要一个诸如[`yadif`](#yadif)之类的反交错滤镜来进一步标记剩余的交错内容，以便于后面的抽取。

除了各种配置选项，`fieldmatch`可以通过`ppsrc`可选项启用第二个流。如果被允许，将基于第二个流进行帧的重建。它允许第一个流作为预处理来帮助各种滤镜算法实现无损输出（减少能正确匹配）。通常一个`field-aware`降噪，或者亮度/对比度调整可以实现这一的帮助。

**注意**滤镜使用如TIVTC/TFM （AviSynth项目）和 VIVTC/VFM（VapourSynth项目）的相同算法。`fieldmatch`有一点克隆TFM的意味，除了一些行为和选项不同外，语义和用法很接近。

当前[`decimate`](#decimate)滤镜仅工作在固定帧率视频。在输入视频是混合了电视电影和逐场内容且是变帧率时不能使用`fieldmatch`和`decimate`

滤镜接受下面的选项：

order

    指定输入流的场序。可用值:

    ‘auto’

        自动探测 (采用FFmpeg内部校验值). 
    ‘bff’

        设置为下场优先 
    ‘tff’

        设置为上场优先 

    **注意**不要太信任流的宣称值（即需要尝试探测）

    默认为auto.
mode

    设置匹配模式或采用的策略. `pc`模式是最安全的，不会产生抖动，但其对于错误编辑或者混合会被输出，而实际有更好的匹配结果（即检测不出最好匹配结果）。在其他处理中`pcn_ub`模式最有可能出现创建抖动的危险，但如果存在最好匹配则一定可以检测出。其他模式介于二者之间。

    更多关于p/c/n/u/b 的有效性见 p/c/n/u/b 意义部分。

    有效值有:

    ‘pc’

        2路匹配 (p/c) 
    ‘pc_n’

        2路匹配，并尝试第3路 (p/c + n) 
    ‘pc_u’

        2路匹配，并尝试第3路 (同`order`)对应 (p/c + u) 
    ‘pc_n_ub’

        2路匹配，并尝试第3路, 并可尝试第4/第5匹配(p/c + n + u/b) 
    ‘pcn’

        3路匹配 (p/c/n) 
    ‘pcn_ub’

        3路匹配，并尝试第4/第5匹配 (p/c/n + u/b) 

    括号中的模式匹配是假设order=tff (场序是自动或者上场优先).

    pc模式最快，而pcn_ub则最慢

    默认为pc_n.
ppsrc

    标记主要输入作为预处理输入，而且允许第二个输入流作为干净源。参考滤镜介绍以了解更多详细信息。它类似于VFM/TFM 的clip2特性.

    默认为0 (表示禁止).
field

    设置场序，它建议设置为同`order`，除非你尝试的结果是失败。在某些情况下改变设置可以产生很大的匹配性能影响，可用值有:

    ‘auto’

        自动 (同于`order`中的介绍). 
    ‘bottom’

        下场优先 
    ‘top’

        上场优先 

    默认为auto.
mchroma

    设置色度信号是否包含在比较判断中。大多数情况下建议不专门设置。仅当影片中包含坏的色度问题如有大片的彩虹或者其他工件时才该设置为0。设置为0也可以加快处理但会造成精度的降低

    默认为1.
y0
y1

    这些定义用来明确一个范围，以除去（忽略）字幕、台标或者其他可能干扰匹配的东西。其中y0设置扫描的开始行，y1设置扫描的结束行（包括y0和y1，之外的部分被忽略），如果y0和y1设置为相同的值将禁用这个特性。y0和y1都默认为0
scthresh

    设置在亮度上场景变化最大百分比数, 好的值范围为[8.0, 14.0]，场景变化检测只是在combmatch=sc有效. 可用值范围[0.0, 100.0].

    默认为12.0.
combmatch

    当设置为非`none`, `fieldmatch`会在更多的情况中梳理出合适的结果。有效值:

    ‘none’

        梳理多种可能但没有最优结果 
    ‘sc’

        在采用了场景改变检测基础上梳理结果
    ‘full’

        全时梳理结果 

    默认为sc.
combdbg

    强制`fieldmatch`对某些指标匹配并输出。这个设置在TFM/VFM中被称为`micout`，有效值有:

    ‘none’

        没有强制 
    ‘pcn’

        强制p/c/n  
    ‘pcnub’

        强制p/c/n/u/b. 

    默认none.
cthresh

    用于控制梳理检测的阀值。它实质上控制“强烈”或“可见”的交错必须被检测到。大的值意味着必须有更多差异才能被检验到，小的值则允许很少的交错（量）就会被检测到。有效值为-1（像素级的检测）到255（没有像素会被作为交错检验出来），这基本上是一个像素值的差异，好的范围是[8, 12].

    默认为9.
chroma

    决定是否把色度检测包含在匹配模式中。如果源中存在色度问题（彩虹）则必需要禁用。实际上设置`chroma=0`通常就足够可靠了，除非你确实在源中包含了色度需要被检测

    默认为0.
blockx
blocky

    设置检测窗的x和y轴尺寸。它协同`combpel`领域像素一起来声明梳理框架。参考`combpel`介绍了解更多信息。可能值是2的幂数，从4到512。

    默认为16.
combpel

    在梳理检测窗中梳理像素的数量。虽然`cthresh`控制了必须是“可见”的才被梳理（即精度），而这个参数控制了在局部（检测窗）中有“多少”则被检出，最小为0，最大为`blocky x blockx`（它们定义了整个检测窗）。它类似于TFM/VFM中的MI

    默认为80. 

#### p/c/n/u/b的意义 ####

##### p/c/n #####
我们假定有下如下的电视电影流：

	Top fields:     1 2 2 3 4
	Bottom fields:  1 2 3 4 4

对应的数字对应需处理的场。在这里前2帧是需要的，3和4则被梳理，等等

当`fieldmatch`被配置运行在`field=bottom`时，输入流被如下传输：

	输入流:
		T     1 2 2 3 4
	    B     1 2 3 4 4   <-- 匹配参考
	
	匹配检测:  c c n n c
	
	输出流:
	    T     1 2 3 4 4
	    B     1 2 3 4 4

作为一个场匹配的结果，我们可以看到一些帧被复制了。为了完整实现电视电影的逆转换，你需要依靠后续的滤镜去除掉重复的帧。参考[`decimate`](#decimate)滤镜。

如果相同的处理但配置为`field=top`，结果会：


	输入流:
		T     1 2 2 3 4		<-- 匹配参考
	    B     1 2 3 4 4   
	
	匹配检测:  c c p p c
	
	输出流:
	    T     1 2 2 3 4
	    B     1 2 2 3 4

在这些例子里，我们可以看出`p`、`c`和`n`的意义。基本上，它们指出参考帧的位置（关系）

- p，匹配于前一帧
- c，匹配于当前帧
- n，匹配于下一帧

##### u/b #####
这里的`u`和`b`是在匹配帧基础上的位级描述。在下面的例子中，我们假定当前匹配了2个帧（Top:2 ,bottom:2）,根据匹配，一个`x`是在每个匹配场景的上方和下方：

对于`field=bottom`的匹配

	 匹配:           c         p           n          b          u
	
	                 x       x               x        x          x
	  Top          1 2 2     1 2 2       1 2 2      1 2 2      1 2 2
	  Bottom       1 2 3     1 2 3       1 2 3      1 2 3      1 2 3
	                 x         x           x        x              x
	
	输出帧:
	                 2          1          2          2          2
	                 2          2          2          1          3

对于`field=top`的匹配

	 匹配:           c         p           n          b          u
	
	                 x         x           x        x              x
	  Top          1 2 2     1 2 2       1 2 2      1 2 2      1 2 2
	  Bottom       1 2 3     1 2 3       1 2 3      1 2 3      1 2 3
	                 x       x               x        x          x
	
	输出帧:
	                 2          2          2          1          2
	                 2          1          3          2          2

#### fieldmatch例子 ####
- 简单的IVTC 有上场优先的视频电影流：

	fieldmatch=order=tff:combmatch=none, decimate
- 高级IVTC，由后续[`yadif`](#yadif)继续处理：

	fieldmatch=order=tff:combmatch=full, yadif=deint=interlaced, decimate


### fieldorder ###
改变输入的场序

接受下面的参数：

- order

	输出场序。有效值有`ttf`对应的上场优先和`bff`对应的下场优先

默认是`tff`

转换是将图像内容向上或者向下一行，并填充其余部分以符合图像内容。该方法适合大多数广播电视的场序转换

如果输入视频没有标记为交错，或者已经标记为所需场序，则滤镜不会改变输入视频

它常用于转换到PAL DV格式，它是下场优先的。

例如：

	ffmpeg -i in.vob -vf "fieldorder=bff" out.dv

### fifo ###
缓冲输入并在需要时送出

它常用于`libavfilter`的自动插入（保证一些连接有效）

它没有参数

### find_rect ###
找到一个矩形对象

它接受下面的选项：

- object

    对象图像的文件路径，需要是gray8.
- threshold

    探测阀，默认0.5.
- mipmaps

    最小的贴图, 默认为3.
- xmin, ymin, xmax, ymax

    指定矩形对象检测的范围（xmin,ymin）(xmax,ymax)定义的一个矩形中

#### find_rect例子 ####
-  对视频生产一个调色板视频

	ffmpeg -i file.ts -vf find_rect=newref.pgm,cover_rect=cover.jpg:mode=cover new.mkv

### cover_rect ###
覆盖一个矩形对象

它接受下面的选项：

- cover

    用作覆盖的图像路径，需要是yuv420.
- mode

    设置覆盖模式

    接受下面的值:

    ‘cover’

        蒙在表面 
    ‘blur’

        同周边插值来覆盖 

    默认blur. 

#### cover_rect例子 ####
-  对视频生产一个调色板视频

	ffmpeg -i file.ts -vf find_rect=newref.pgm,cover_rect=cover.jpg:mode=cover new.mkv
	
###  format ###
转换输入视频为指定的像素格式。`libavfilter`尝试为下一个滤镜输入选择一个合适的输出（而自动采用）。

它接受下面的参数：

-  pix_fmts

	一个用`|`分隔的像素格式名列表，例如"pix_fmts=yuv420p|monow|rgb24".

####  format例子 ####
- 转换输出为`yuv420p`格式

	format=pix_fmts=yuv420p

	转换输入到列表的任何格式之一

	format=pix_fmts=yuv420p|yuv444p|yuv410p

### fps ###
通过复制或者丢弃帧来把帧率调整为一个近似固定值

它接受下面的参数：

- fps

	目标帧率，默认25
- round

    舍入方法.

    可能值:

    zero

        对0舍入（去小数），绝对值最小 
    inf

        从0开始的园 
    down

        向负无穷舍入（去除小数） 
    up

        向正无穷舍入（见小数加1） 
    near

        舍入到近似值（命名的频率） 

    默认为near.
- start_time

    假设第一个`PTS`是一个按秒的给定值。它允许填补/去除流的开始。默认没有需要填补/去除。例如设置为0，将会在视频流后于音频流时在前面添加黑帧，否则去除早于音频流的负帧。

另外，选项可以指定为一个平面字符串形式：fps[:round]

参考[`setpts`](#setpts)滤镜

#### fps例子 ####
- 输出25帧频

	fps=fps=25
- 输出24帧频，使用了帧频缩写名和舍入方法为最接近

	fps=fps=film:round=near

### framepack ###
包两个不同的视频流到立体视频,设置适当的元数据支持的编解码器。两个视角视频需要有相同的尺寸与帧频以及以短的视频为停止时间。**注意**你可能需要预先通过[`scale`](#scale)和[`fps`](#fps)调整。

接受下面的参数：

- format

    设置包装格式，支持:

    sbs

        一个视图左另一个在右(默认).
    tab

        一个视图在上，一个视图在下
    lines

        视图按线交错（按行）
    columns

        视图按列交错
    frameseq

        视图都暂时交错

一些例子：
- 把左/右视图以 `frameseq`模式打包成立体视频

	ffmpeg -i LEFT -i RIGHT -filter_complex framepack=frameseq OUTPUT
- 把视图以 `sbs`模式交错，还进行了放缩预处理

	ffmpeg -i LEFT -i RIGHT -filter_complex [0:v]scale=w=iw/2[left],[1:v]scale=w=iw/2[right],[left][right]framepack=sbs OUTPUT

### framestep ###
每`N`帧选择1帧

它接受下面选项：

- step

	设置间隔的`N`值。必须大于0，默认为1（这样相当于不处理，完全输出）

### frei0r ###
对视频采用`frei0r`效果

编译配置参数` --enable-frei0r`

接受下面的参数：

- filter_name

    设置加载的`frei0r`效果名。如果环境变量`FREI0R_PATH`被定义，则将在其所指目录搜索。`FREI0R_PATH`是有`，`分隔的多个路径。通常`frei0r`的搜索路径是: HOME/.frei0r-1/lib/, /usr/local/lib/frei0r-1/, /usr/lib/frei0r-1/.
- filter_params

    一个由’|’分隔的参数列表来传递给`frei0r`效果

	一个`frei0r`效果的参数是布尔值（为`y`或者`n`）、双精度数、颜色值（以R/G/B形式描述，其中R、G和B是0.0-1.0间的浮点数）或者在[颜色/color](ffmpeg-doc-cn-07.md#颜色Color)中定义的颜色名、位置量（以X/Y形式描述，X和Y均是浮点数）和/或 字符串

	参数的数量和类型要根据加载的效果，如果一个效果参数没有指定则选用默认设置

#### frei0r例子 ####
- 采用`distort0r`效果，参数有2个值。

	frei0r=filter_name=distort0r:filter_params=0.5|0.01
- 应用`colordistance`效果，有一个颜色值作为参数（下面3个形式等效）

	frei0r=colordistance:0.2/0.3/0.4
	frei0r=colordistance:violet
	frei0r=colordistance:0x112233
- 应用`perspective`效果，指定了图像的左上和右上位置

	frei0r=perspective:0.2/0.2|0.8/0.2

关于`frei0r`的更多信息参考[http://frei0r.dyne.org](http://frei0r.dyne.org)

### fspp ###
应用快速简单的后处理，这是[`spp`](#spp)滤镜的快速版本

它分离 (I)DCT为水平/垂直 来传递，不同于简单的 `post`滤镜，其中每个块执行一次，而不是每个像素，则允许更快的速度。

滤镜接受下面的选项：

- quality

    设置品质水平，它定义了平均水平数，接受4-5的整数，默认为4
- qp

    强制包含一个不断量化参数，它接受0-63间的整数，如果不设定，滤镜会使用视频流的`QP`（如果有效）
- strength

    设置滤镜强度。它接受-15-30间的整数。越低表示更多细节但需要更多工作，高的值则图像平滑（模糊）也更快，默认值为0——PSNR最佳
- use_bframe_qp

    为1则允许从B帧使用QP。使用它在大的QP时可能导致B帧闪烁。默认为0（不允许）

### geq ###
滤镜接受下面的选项：

- lum_expr, lum

    设置亮度表达式 
- cb_expr, cb

    设置色度分量中蓝色表达式 
- cr_expr, cr

    设置色度分量中红色表达式 
- alpha_expr, a

    设置透明通道表达式Set the alpha expression. 
- red_expr, r

    设置红色表达式 
- green_expr, g

    设置绿色表达式 
- blue_expr, b

    设置蓝色表达式 

根据指定的选项来确定颜色空间。如果`lum_expr`, `cb_expr`, 或者`cr_expr`中的一个被定义，则滤镜自动选择`YCbCr`颜色空间，如果`red_expr`, `green_expr`,或 `blue_expr`中有一个被定义则选择`RGB`颜色空间

如果其中一个颜色分量选项没有被定义，则它等于前一个谷底值。如果`alpha_expr`没有被定义则认为是不透明的。如果没有任何颜色分量被定义，它将只计算亮度表达式

表达式接受下面变量和函数:

- N

    帧序数，从0开始计数 from 0.
- X
- Y

    当前样本坐标
- W
- H

    图像宽和高
- SW
- SH

    依赖当前滤镜的放缩宽和高。它根据当前像素亮度数和当前平面的比例。例如对于YUV4:2:0给我饿死，这个值是1,1对应于亮度还有0.5，0.5 的颜色分量
- T

    按秒当前帧时间
- p(x, y)

    返回当前帧平面（x,y）点的像素值
- lum(x, y)

    返回当前帧平面（x,y）点的像素亮度值
- cb(x, y)

    返回当前帧平面（x,y）点的像素色度分量差蓝色值,0表示没有该分量
- cr(x, y)

    返回当前帧平面（x,y）点的像素色度分量差红色值,0表示没有该分量
- r(x, y)
- g(x, y)
- b(x, y)

    返回当前帧平面（x,y）点的像素红/绿/蓝值，为0表示没有该颜色
- alpha(x, y)

    返回当前帧平面（x,y）点的像素透明通道值，为0表示没有该值

对于函数，如果x和y超出了范围，则值自动由影片边缘值代替 

#### geq例子 ####
- 水平翻转图像
	geq=p(W-X\,Y)
- 生成一个二维的正弦波,角π/ 3和100像素的波长:

    geq=128 + 100*sin(2*(PI/100)*(cos(PI/3)*(X-50*T) + sin(PI/3)*Y)):128:128
- 生成一个花哨的神秘的光:

    nullsrc=s=256x256,geq=random(1)/hypot(X-cos(N*0.07)*W/2-W/2\,Y-sin(N*0.09)*H/2-H/2)^2*1000000*sin(N*0.02):128:128
-  生成一个快速浮雕效果:

    format=gray,geq=lum_expr='(p(X,Y)+(256-p(X-4,Y-4)))/2'
- 根据像素的位置修改RGB分量:

    geq=r='X/W*r(X,Y)':g='(1-X/W)*g(X,Y)':b='(H-Y)/H*b(X,Y)'
- 创建一个径向渐变,是相同的大小作为输入(也见[`vignette`](#vignette)滤镜):

    geq=lum=255*gauss((X/W-0.5)*3)*gauss((Y/H-0.5)*3)/gauss(0)/gauss(0),format=gray
- 创建一个线性渐变使用作为另一个滤镜的蒙版,然后用叠加组成。在本例中,视频会从底部到顶部的y轴定义的线性梯度逐渐变得更加模糊:

    ffmpeg -i input.mp4 -filter_complex "geq=lum=255*(Y/H),format=gray[grad];[0:v]boxblur=4[blur];[blur][grad]alphamerge[alpha];[0:v][alpha]overlay" output.mp4

### gradfun ###
解决条带效果，这是由于对于8位颜色深度有时会被就近截断成平面（化），通过插入渐变来柔化它们

这个滤镜仅用于回放。在有损压缩前也不要使用它，因为压缩本身就会损伤细节（渐变）而成为条带

它接受下面的参数:

- strength

    滤镜对任何像素最大的改变值。这也是检测(颜色)平坦区域的阀值。取值范围是.51 至64，默认为1.2.超出范围的值会被被裁减为有效值
- radius

    指定合适的修正梯度。一个大的`radius`值会产生更平滑的过渡，也防止滤镜修改处理区域附近的像素。接受的范围为8-32，默认为16，超出范围的值会被裁减以符合

另外，选项可以采用平面字符串的形式指定： `strength[:radius]`

#### gradfun例子 ####
- 以3.5的`strength`和8的`radius`值应用滤镜:

    gradfun=3.5:8
- 指定`radius`，省略`strength` (会采用默认值为1.2):

    gradfun=radius=8
    
### haldclut ###
对视频流采用`Hald CLUT`

第一个输入是要处理的视频，第二个则是`Hald CLUT`，这个`Hald CLUT`输入可以是一张简单的图片或者复合视频信号

滤镜接受下述选项:

- shortest

    强制以最短输入来总作为整个数出。默认为0
- repeatlast

    在结束后继续以`CLUT`处理。值为0则禁止。默认为1. 

`haldclut`也有类似[`lut3d`](#lut3d)相同的选项（两个滤镜共享相同的内部结构）。 

关于`Hald CLUT`可以通过`Eskil Steenberg`（Hald CLUT的作者）的网站，在[http://www.quelsolaar.com/technology/clut.html](http://www.quelsolaar.com/technology/clut.html) 

#### haldclut工作流例子 ####
##### Hald CLUT视频流 #####

- 生成一个`Hald CLUT`的流 ，流改变各种效果。

	ffmpeg -f lavfi -i haldclutsrc=8 -vf "hue=H=2*PI*t:s=sin(2*PI*t)+1, curves=cross_process" -t 10 -c:v ffv1 clut.nut

	**注意**：确认你选用的是无损编码

- 然后把滤镜（随`haldclut`）应用在随机流。

	ffmpeg -f lavfi -i mandelbrot -i clut.nut -filter_complex '[0][1] haldclut' -t 20 mandelclut.mkv

	这里`Hald CLUt`被用于前10秒（持续时间由clut.nut定义）。然后其最后的CLUT图片应用到继续的的[`mandelbrot`](#mandelbrot)流上

##### 带预览的Hald CLUT #####

一个`Hald CLUT`支持作为有多层（`Level*Level*Level`）像素的多层（`Level*Level*Level`）图像。 对于一个给定的`Hald CLUT`，FFmpeg尽可能在图像的左上开始选择最大可能， 将选择尽可能最大的广场在图片的左上角开始。剩下的填充像素(底部或右)将被忽略。这个区域可以用来添加一个预览`Hald CLUT`。.

通常，下面会利用[`haldclutsrc`](ffmpeg-doc-cn-38.md#haldclutsrc)生成一个支持`haldclut`滤镜的`Hald CLUT`图:

	ffmpeg -f lavfi -i haldclutsrc=8 -vf "
	   pad=iw+320 [padded_clut];
	   smptebars=s=320x256, split [a][b];
	   [padded_clut][a] overlay=W-320:h, curves=color_negative [main];
	   [main][b] overlay=W-320" -frames:v 1 clut.png

它包含原始和CLUT的预览效果：SMPTE颜色条被显示在右上，其下显示相同的颜色处理的颜色变化

然后,这Hald CLUT效果可以可视化:

	ffplay input.mkv -vf "movie=clut.png, [in] haldclut"
	
### hflip ###

水平翻转输入视频

例如利用fmpeg水平翻转输入视频:

	ffmpeg -i in.avi -vf "hflip" out.avi
	
### histeq ###
这个过滤器适用于每帧的基础上的全局颜色直方图均衡化

它被用于产生压缩了像素强度的正确视频。滤镜在强度范围内重新分配像素强度分布。它可被视为“自动调整对比度滤镜"。滤镜只适用于纠正退化或者较差质量的视频采集

接受下面的选项:

- strength

    确定的数量均衡。随着参数的降低，像素强度的分布在输入帧中越来越多。值为浮点数，范围为[0,1]，默认0.200.
- intensity

    设置在生成的输出中最大可能强度。`strength`设置表面了期望，而`intensity`的设置强调了限制，从而避免了出现错误。值为浮点数，范围为[0,1]，默认0.210.
- antibanding

    设置antibanding级别。如果启用，滤镜将通过随机小批量改变输出像素的亮度直方图避免产生条带。允许的值有`none`, `weak` 或`strong`，默认为`none` 。

### histogram ###
对输入视频计算并绘制一个颜色分布直方图

它计算的直方图代表了各种颜色分量在图像中的分布情况。

滤镜接受下面的选项:

- mode

    设置直方图模式.

    有下面的可能值:

    ‘levels’

        显示图像颜色分量的标准直方图。它显示每个颜色分量图形。依据输入视频可以显示当前帧的`Y, U, V, A `或者`R, G, B`分量图形。其下是每个图像颜色分量的规模计
    ‘color’

        在二维图（这被称为矢量监视器）中显示色度通道值 (U / V颜色位置) 。像素矢量表示亮度，每个点对应代表输入中的各个像素（都有其色度分量值）。V分量为水平（X）坐标，最左表示`V=0`,最右为`V=255`，U分量为垂直（Y）坐标，最上标识`U=0`，最下`U=255`.

        图中白色像素的位置对应一个像素的色度值输入。因此该图可以用来了解色相(颜色味道)和饱和度(颜色)的情况其反映了原始图像的主导色调。颜色的色调变化围绕着一个区域(监视器面)，在区域的中心饱和度是0，则意味着对应的像素没有颜色（白色，只有亮度值）。如果图像中一个特定的颜色的数量增加(而其他颜色不变)，即饱和度的增加,则在矢量监视器中显示为整体偏向边缘。
    ‘color2’

        类似color以矢量监视器显示，不过增加了实际色度值的显示。
    ‘waveform’

        每行（row）/列（colum）颜色分量图形。在row模式，图形左边表示颜色分量为0，右边为255，在column默认，顶部表示颜色分量值为0，底部为255

    默认为`levels`模式
- level_height

    在`levels`模式中设置图形高，默认200，允许[50, 2048].
- scale_height

    在`levels`设置颜色放缩高，默认12，允许[0, 40].
- step

    对`waveform`模式设置步长。小的值用于更多了解在相同亮度下颜色分布情况，默认10，允许 [1, 255] 。
- waveform_mode

    对`waveform`设置`row`或者`column`，默认`row`
- waveform_mirror

     对`waveform`设置镜像模式，0表示不镜像（默认），1表示镜像。在镜像模式中，对`row`高值在左边，对`column`高值在上面
- display_mode

    设置`waveform`和`levels`的显示模式，它接受:

    ‘parade’

        依次排列各种颜色图，`waveform`的`row`是从左到右，`column`是从上到下。`levels`中是从上到下。

        `waveform`中应用这种模式可以容易的通过比较顶部与底部轮廓波形来区分高光和阴暗的颜色图像，因为白、灰和黑的特点是完全相等的红、绿和蓝的混合。中性的图片区域应该是这三种波形的大致相等的宽/高度。如果不是，则可以通过水平调整这三个波形来很容易的实现校正。
    ‘overlay’

        除了表示颜色的图形组件直接叠加在一起外，与`parade`模式展示的信息相同

        在`waveform`中这样的显示模式更易于发现颜色的相对差异或相似。因为重叠区域的分量应该是相同的，如中性的白色、灰和黑

    默认为`parade`
- levels_mode

    对`levels`设置模式，可以是`linear`或`logarithmic`，默认`linear` 

#### histogram例子 ####
- 计算并绘制柱状图:

    ffplay -i input -vf histogram

### hqdn3d ###
这是一个高精度/质量的3D降噪滤镜。它的目的是减少图像噪声,产生平滑的图像和让静止图像保存原样。它可以提高压缩率。

接受下面可选参数:

luma_spatial

    非负浮点数来指明亮度强度。默认为4.0
chroma_spatial

    非负浮点数来指明亮色强度，默认为`3.0*luma_spatial/4.0`.
luma_tmp

    一个浮点数指明亮度临时强度。默认为`6.0*luma_spatial/4.0`
chroma_tmp

    一个浮点数指明色度临时强度。默认为`luma_tmp*chroma_spatial/luma_spatial` 

### hqx ###
应用一个高质量的像素放大滤镜。这个滤镜最初由 Maxim Stepin创建。

它接受下面的选项:

- n

    设置缩放尺度。2 对应hq2x, 3 对应hq3x，4对应hq4x，默认为3。
    
### hue ###
编辑或者设定颜色的饱和度

接受下面的参数:

- h

    指定色度角的度数，接受表达式，默认为0
- s

    指定饱和度，范围[-10,10]，接受表达式，默认为"1".
- H

    指定色调角的弧度，接受表达式，默认为"0".
- b

    指定亮度，范围[-10,10]。接受表达式，默认为"0". 

`h`和`H`互斥，不能同时设定

其中`b`, `h`, `H`和`s`表达式允许下面内容:

- n

    从0开始的帧序数
- pts

    按时间基单位的输入帧时间戳
- r

    输入视频帧率，如果无效则为`NAN`
- t

    按秒的时间码，如果无效则为`NAN`
- tb

    输入视频时基 

#### hue例子 ####
- 设置色度角90度，饱和度为1:

    hue=h=90:s=1
- 同上，但以色度角弧度值进行设置:

    hue=H=PI/2:s=1
- 旋转色相,以及让饱和度在0-2间变化:

    hue="H=2*PI*t: s=sin(2*PI*t)+1"
- 从0开始应用一个3秒饱和度淡入效果:

    hue="s=min(t/3\,1)"
- 一般淡入表达式可以为:

    hue="s=min(0\, max((t-START)/DURATION\, 1))"
- 从5秒开始的饱和度淡出:

    hue="s=max(0\, min(1\, (8-t)/3))"
- 一般淡出表达式为:

    hue="s=max(0\, min(1\, (START+DURATION-t)/DURATION))"

#### hue命令 ####
滤镜还支持下面的命令：

- b
- s
- h
- H

    它们分别编辑色度 和/或 饱和度 和/或 亮度。命令接受对应选项一样的语法。

    如果指定的表达式是无效的，则采用当前值（不变化）
    
### idet ###
检测视频交错类型。

这个滤镜试图检测如果输入帧交错,逐行,顶部或底部优先（对交错视频）。它还将尝试检测相邻帧之间的字段重复(电视电影的标志)。

单帧检测时只考虑当前与相邻帧每一帧类型。多帧检测结合的之前的帧类型历史。

滤镜输出日志有下面的元数据值:

- single.current_frame

    对当前帧单帧检测结果，有如下值:“tff” (上场优先), “bff” (下场优先), “progressive”（逐行）, 和“undetermined”（不能检测出）
- single.tff

    累积的以单帧检测检测出的上场优先数
- multiple.tff

    累积的以多帧检测检测出的上场优先数
- single.bff

    累积的以单帧检测检测出的下场优先数
- multiple.current_frame

    对当前帧多帧检测结果，有如下值:“tff” (上场优先), “bff” (下场优先), “progressive”（逐行）, 和“undetermined”（不能检测出，不定）
- multiple.bff

    累积的以多帧检测检测出的下场优先数
- single.progressive

    累积的以单帧检测检测出的逐行帧数
- multiple.progressive

    累积的以多帧检测检测出的逐行帧数
- single.undetermined

    累积的以单帧检测检测出的不定帧数
- multiple.undetermined

    累积的以多帧检测检测出的不定帧数
- repeated.current_frame

    指示当前帧是从最近帧的重复情况,为“neither”(表示不是重复), “top”,或者 “bottom”.
- repeated.neither

    累积的`neither`重复情况.
- repeated.top

    累积的`top`重复情况
- repeated.bottom

    累积的`bottom`重复情况 

滤镜接受下面选项:

- intl_thres

    设置交错阀值 
- prog_thres

    设置逐行阀值 
- repeat_thres

    重复检测阀值 
- half_life

    设定指定的帧数之后,一个给定的数据帧的贡献是减半（即只占0.5的类型）。默认为0意味着所有的帧永远是有1.0的权重
- analyze_interlaced_flag

    当设置为非0数时，`idet`将使用指定的帧数来确定交错标记是准确的，它不会对不能检测帧计数。如果发现标记是正确的使用它没有进一步的计算，即使发现标记不正确，也只是将它(标记)清除而没有进一步的计算。它插入`idet`滤镜作为低计算方法来清除交错标记。



### il ###
反转非交错或者交错

这个滤镜可以把让非交错图像变成交错的，把交错图像变成非交错的。非交错的图像被分裂为2部分（称作半图），其中奇数行移到输出图像的上部，偶数行移到下半部分。你可以利用滤镜连续处理2次（相当于没有效果）。

滤镜接受下面的选项：

- luma_mode, l
- chroma_mode, c
- alpha_mode, a

    对`luma_mode`, `chroma_mode`和`alpha_mode`的可能值有:

    ‘none’

        什么都不做
    ‘deinterleave, d’

        非交错部分，放置在其它上
    ‘interleave, i’

        交错部分，反向非交错效果

    默认值为`none`
- luma_swap, ls
- chroma_swap, cs
- alpha_swap, as

    交换 luma/chroma/alpha 部分。交换奇数和偶数行。 默认0. 

### interlace ###

简单的从逐行转交错滤镜。它把奇数帧交错上（或下）行，同时把偶数帧作为下（或上）行。同时减半帧率和保持图像高度。

	Original        Original             New Frame
   	Frame 'j'      Frame 'j+1'             (tff)
	==========      ===========       ==================
    Line 0  -------------------->    Frame 'j' Line 0
    Line 1          Line 1  ---->   Frame 'j+1' Line 1
    Line 2 --------------------->    Frame 'j' Line 2
    Line 3          Line 3  ---->   Frame 'j+1' Line 3
     ...             ...                   ...

新的帧 + 1 会以帧'j+2'和帧'j+3'这样依次生成

它接受下面的选项参数:

- scan

    它决定了交错帧模式是`even`(tff - 默认) 还是`odd` (bff) 
- lowpass

    允许(默认)或禁止垂直低通滤波器来避免`twitter`交错和减少波纹。

### kerndeint ###
通过Donald Graft的自适应内核deinterling反交错视频。其使交错视频转换成逐行视频

下面是接受参数的介绍.

- thresh

    设置一个滤镜阀值用于确定哪些像素会被处理。值为[0,255]间整数，默认为10.如果为0则处理所有像素
- map

    设置如果超过阀值的处理模式，为1则为白色，默认为0
- order

    设置字段顺序。如果为1则交换字段，否则为0，默认为0.
- sharp

    为1则附加锐化，默认为0.
- twoway

    为1则尽量锐化，默认为0

#### kerndeint例子 ####
- 以默认值应用:

    kerndeint=thresh=10:map=0:order=0:sharp=0:twoway=0
- 允许附加锐化:

    kerndeint=sharp=1
- 超阀值后刷成白色

    kerndeint=map=1

### lenscorrection ###
径向透镜畸变修正

这个滤镜可以用来修正广角镜头产生的径向畸变，从而修正图像。要找到合适的参数，可以使用工具，例如`opencv`或者简单的多次试错尝试。利用`opencv`源码中的校准样例（在 `samples/cpp`）并且从结果矩阵中提取`k1`和`k2`系数。

**注意**相同效果滤镜在开源KDE项目工具`Krita`和`Digikam`中同样是有效的。

这个滤镜还可以同[`vignette`](#vignette)滤镜联合使用，来补偿透镜错误，它修正滤镜处理图像的失真，而[`vignette`](#vignette)滤镜纠正亮度分布，所以你有时可能需要同时使用两个滤镜，不过你还要注意点二者的顺序，即到底是哪个滤镜先使用

#### lenscorrection选项 ####
滤镜接受下面选项：

- cx

    图像的焦点相对x坐标,从而扭曲的中心。这个值区间[0,1],用分数表示图像的宽度比 
- cy

    图像的焦点相对y坐标,从而扭曲的中心。这个值区间[0,1],用分数表示图像的高度比
- k1

    二次修正系数。0.5意味着没有修正
- k2

    双二次修正系数。0.5意味着没有修正。

生成修正的公式:

	r_src = r_tgt * (1 + k1 * (r_tgt / r_0)^2 + k2 * (r_tgt / r_0)^4) 

这里`r_0`是减半的图像对角，`r_src`和`r_tgt`分别是源和目标图像中相对于焦点的距离。

### lut3d ###
对视频应用3D LUT滤镜。

滤镜接受下面的选项：

- file

    设置3D LUT文件名，该文件支持的类型有：

    ‘3dl’

        AfterEffects 的类型
    ‘cube’

        Iridas 的类型
    ‘dat’

        DaVinci 的类型
    ‘m3d’

        Pandora 的类型

- interp

    选择插值模式，有效值有：


    ‘nearest’

        选用离定义点最近的
    ‘trilinear’

        采用8点多维设置来定义插入值 
    ‘tetrahedral’

        使用一个四面体插入值

### lut, lutrgb, lutyuv ###
根据每个像素的分量数据查表选择输出值的滤镜。

其中`lutyuv`应用于`YUV`输入视频，而`lutrgb`应用于`RGB`输入视频

滤镜接受下面的参数：

- c0

    设置第一个像素分量表达式
- c1

    设置第二个像素分量表达式set second pixel component expression 
- c2

    设置第三个像素分量表达式 
- c3

    设置第四个像素分量表达式
- r

    设置红色分量表达式 
- g

    设置绿色分量表达式 
- b

    设置蓝色分量表达式
- a

    设置透明分量表达式
- y

    设置亮度（Y）分量表达式 
- u

    设置色度U/Cb分量表达式 
- v

    设置色度V/Cr分量表达式 

它们每个人都指定使用的表达式计算像素对应分量（查找表方法）。

具体的分量关联到每个`c*`选项的输入格式。

其中`lut`滤镜允许输入像素格式是`YUV`或者`RGB`，`lutrgb`只允许`RGB`格式，`lutyuv`只允许`YUV`格式

表达式支持下列常量和函数:

- w
- h

    输入的宽和高
- val

    输入像素分量值
- clipval

    输入值，并且裁剪到`minval-maxval`范围内
- maxval

    输入像素分量的最大值
- minval

    输入像素分量的最小值 component.
- negval

    “负片”表示的分量值，它被裁剪到`minval-maxval`范围内，对应于`maxval-clipval+minval`.
- clip(val)

    对于`val`的计算值,裁剪到 `minval-maxval`范围内
- gammaval(gamma)

    伽马校正计算值像素分量的值，裁剪到 `minval-maxval`范围内，其对应于`pow((clipval-minval)/(maxval-minval)\,gamma)*(maxval-minval)+minval`

所有表达式默认为"val"

#### lut, lutrgb, lutyuv例子 ####
- 输入图像的负片效果

    lutrgb="r=maxval+minval-val:g=maxval+minval-val:b=maxval+minval-val"
    lutyuv="y=maxval+minval-val:u=maxval+minval-val:v=maxval+minval-val"

    等效于:

    lutrgb="r=negval:g=negval:b=negval"
    lutyuv="y=negval:u=negval:v=negval"
- 亮度负片效果

    lutyuv=y=negval
- 移除色度分量，转换成灰度图像:

    lutyuv="u=128:v=128"

- 应用一个亮度燃烧效果:

    lutyuv="y=2*val"
- 移除绿色和蓝色分量（红色灰度图）:

    lutrgb="g=0:b=0"
- 设定固定的透明通道效果:

    format=rgba,lutrgb=a="maxval-minval/2"
- 以系数0.5进行伽玛亮度矫正:

    lutyuv=y=gammaval(0.5)
- 丢弃的亮度低有效位（减少细节，亮块化）:

    lutyuv=y='bitand(val, 128+64+32)'

### mergeplanes ###
从一些视频流中混合颜色通道

滤镜最多接受4路输入流，然后混合选用的(颜色)平面来输出。

滤镜接受下面的选项：

- mapping

    设置输入到输出颜色映射，默认为0 mapping. Default is 0.

    这个映射由一个位映射指定，它被描述为一个格式为`0xAa[Bb[Cc[Dd]]]`的十六进制数。其中`Aa`描述第一个用作输出的（颜色）平面，`A`设置采用那个输入流（0-3），`a`指定是输入流的那个分量（0-3，因为一个输入视频流最多有4个输入分量）。此后的`Bb`、`Cc`和`Dd`以此类推，分别指定第2到第4个输出映射关系。
- format

    设置输出像素格式，默认为`yuva444p` 

#### mergeplanes例子 ####
- 从三个灰度视频流混合为单个视频流（有相同的图像尺寸）:

    [a0][a1][a2]mergeplanes=0x001020:yuv444p
- 混合第一路的`yuv444p`和第二路的灰度视频到一个`yuva444p`:

    [a0][a1]mergeplanes=0x00010210:yuva444p
- 在yuva444p交换Y和A通道:

    format=yuva444p,mergeplanes=0x03010200:yuva444p
- 在`yuv420p`交换U和V输出到`yuv420p`:

    format=yuv420p,mergeplanes=0x000201:yuv420p

- 转换`rgb24`到`yuv444p`

    format=rgb24,mergeplanes=0x000102:yuv444p

### mcdeint ###
应用反交错运动补偿

它每帧需要一个输入字段，所以必须同`yadif=1/3`或者等效 一同使用。

率接受下面的选项：

- mode

    设置反交错模式，有下列有效值：

    ‘fast’
    ‘medium’
    ‘slow’

        使用迭代的运动估计 
    ‘extra_slow’

        类似‘slow’,但使用多个参考帧

    默认‘fast’.
- parity

    设置输入视频图片字段校验。它必须以下值之一:

    ‘0, tff’

        对应上场优先 
    ‘1, bff’

        对应下场优先 

    默认 ‘bff’.
- qp

    设置内部使用的编码器按块量化参数设置(QP)

    更高的值会导致一个更平滑的运动向量场但不最优个体向量。默认值是1。

### mpdecimate ###
减少对前帧变化不大的帧,以减少帧速率

这个滤镜的主要作用是对非常低码率的编码进行预处理（例如面向拨号调整解调器应用的），不过理论上还可以用于修复发转的电视电影影片（影片转换成电视又转换回来）

下面是选项介绍：

- max

    设置连续帧的最大数量,可以删除(如果正),或删除帧之间的最小间隔帧(如果负的) ，如果为0，则删除帧和前面删除帧没有关联（关系）

    默认为0.
- hi
- lo
- frac

    设置删除阀值。

    其中`hi`和`lo`是对于一个`8x8`像素块和代表实际像素值差异，所以阀值64对应与每个像素都有1个单位的差异，或者被完全不同的块覆盖

    一个帧作为候选（被丢弃）则它没有超过`hi`个不同的`8x8`块和没有超过`frac`量的（1意味着整个图像）超过`lo`个不同的`8x8`块

    默认值为`hi`是`64*12`,`lo`为`64*5`,`frac`为`0.33` 

### negate ###
否定输入视频

它接受一个整数（参数）,如果非零它否定透明分量(如果可用)。默认值是0

### noformat ###
强制`libavfilter`不采用指定的像素格式来输入到下一个滤镜

接受的参数为：

- pix_fmts

    是一个由 ’|’分隔的像素格式名列表，例如pix_fmts=yuv420p|monow|rgb24"

#### noformat例子 ####
- 强制不采用`yuv420p`的像素格式来输出到`vfilp`滤镜

	noformat=pix_fmts=yuv420p,vflip
- 转换输入到不是指定的像素格式

	noformat=yuv420p|yuv444p|yuv410p

### noise ###
给视频添加噪点

滤镜接受下面的选项：

- all_seed
- c0_seed
- c1_seed
- c2_seed
- c3_seed

    对指定像素分量或者整个像素设置噪点种子，默认为123457.
- all_strength, alls
- c0_strength, c0s
- c1_strength, c1s
- c2_strength, c2s
- c3_strength, c3s

    对指定像素分量或者整个像素设置噪点强度，默认为0，值范围[0, 100].
- all_flags, allf
- c0_flags, c0f
- c1_flags, c1f
- c2_flags, c2f
- c3_flags, c3f

    设置分量或者所有的标志，可能的标志有:

    ‘a’

        平均时间的噪点(平滑) 
    ‘p’

        混合随机噪点(半)规律
    ‘t’

        时间噪点（噪点模式在帧间变化） 
    ‘u’

        统一噪点 (gaussian外的) 

#### noise例子 ####
- 添加时间和统一的噪点给图像

	noise=alls=20:allf=t+u

### null ###
不改变输入进行输出

### ocv ###
申请使用`libopencv`进行视频转换。

要启用需要处理编译参数`--enable-libopencv`，且系统中要有`libopencv`的头和库。

接受下面的参数：

- filter_name

    指定要使用的`libopencv`滤镜名
- filter_params

    对滤镜指定参数，如果不指定则采用具体滤镜默认参数

在[`libopencv`官方文档](http://docs.opencv.org/master/modules/imgproc/doc/filtering.html)了解更多`libopencv`精确信息

在`libopencv`中下面的滤镜被支持。
#### dilate ####
这个滤镜使用特定结构化的元素来扩张图像，其对应于`libopencv`中的函数`cvDilate`

它接受的参数形式为:`struct_el|nb_iterations`

其中`struct_el`代表了一个结构化元素，语法为：`colsxrows+anchor_xxanchor_y/shape`，其中`cols`和`rows`是结构元素的行和列数，`anchor_x`和`anchor_y`是锚点的坐标值，`shape`是结构化元素的图形，`shape`可能值是"rect"、 "cross"、 "ellipse"和 "custom"。如果`custom`被设置，它还必须跟一个格式为`=filename`来指定一个位图，其每个可打印字符对应于一个亮的像素。当定制的图形被使用，`cols`和`rows`被忽略，行和列的数有读取的文件决定。

`struct_el`的默认值为`3x3+0x0/rect`

`nb_iterations`指定迭代的次数，默认为1

一些例子：
- 采用默认值

	ocv=dilate
- Dilate采用了一个5x5的结构元素，且迭代2次

	ocv=filter_name=dilate:filter_params=5x5+2x2/cross|2

- 从文件`diamond.shape`读取图像，迭代2次

	其中`diamond.shape`文件的内容为：
```	
  *
 ***
*****
 ***
  *
```
	因为描述为定制，所以原来指定的行和列值被忽略：

	ocv=dilate:0x0+2x2/custom=diamond.shape|2

#### erode ####
通过使用特定的结构化元素侵蚀一个图像。它对应于`libopencv`中的`cvErode`函数。

它接受参数为：`struct_el:nb_iterations`，解释同[`dilate`](#dilate)

#### smooth ####
平滑输入视频

滤镜接受的参数为：`type|param1|param2|param3|param4`

其中`type`是应用的平滑类型，有 "blur", "blur_no_scale", "median", "gaussian", 或者"bilateral".默认为"gaussian"

接下来的参数`param1`, `param2`, `param3`和`param4`依赖于所选的平滑类型，其中`param1`, `param2`接受正整数或0，`param3`和`param4`接受浮点数。其中`param1`的默认为3，其他的默认为0.

这些参数对应于`libopencv`中的`cvSmooth`函数。

### overlay ###
把一个视频覆盖在另外一个上面

它有两个输入，其中第一个输入是主要的输入会覆盖到第二个输入上

它接受下面的参数（介绍见下）：

- x
- y

    设置主要视频（在被覆盖视频上）的x和y坐标表达式，默认为0，如果表达式无效则会设置为一个巨大的值（在这种情况下相当于没有覆盖，即覆盖出现在非可视区域）
- eof_action

    在操作时遇到第二路输入的`EOF`信号时的处理，它接受下面的值之一：:

    repeat

        重复最后一帧（默认）
    endall

        同时结束两个流 
    pass

        把第一路输入作为输出 

- eval

    它设置何时计算x和y坐标值

    它接受下面的值：

    ‘init’

        仅在滤镜初始化或命令处理时计算一次
    ‘frame’

        每帧计算 

    默认‘frame’.
- shortest

    如果设置为1，强制以最短的输入时间终止输出，默认为0
- format

    设置输出格式，允许：

    ‘yuv420’

        强制为YUV420
    ‘yuv422’

        强制为YUV422
    ‘yuv444’

        强制为YUV444
    ‘rgb’

        强制为RGB 

    默认 ‘yuv420’.
- rgb (弃用的)

    如果设置为1，强制滤镜接受输出是RGB颜色空间。默认为0，它已经被弃用，而是使用格式来代替。
- repeatlast

    如果设置为1，强制滤镜绘制最后一个覆盖帧直到结束流。如果设置为0则禁止这样的行为，默认为1 

值`x`和`y`的表达式中接受下面的参数：.

- main_w, W
- main_h, H

    主要输入流的宽和高
- overlay_w, w
- overlay_h, h

    被覆盖流的宽和高
- x
- y

    对于`x`和`y`的计算值，它在每帧中都计算
- hsub
- vsub

    输出格式中的水平和垂直色度通道分量值，例如对于`yuv422p`像素格式，`hsub`是2 ，`vsub`是1
- n

    帧序数，从0开始计数
- pos

    输入帧在文件当中的偏移，如果未知则为`NAN`
- t

    时间码，以秒计。如果时间码未知则为`NAN`

#### overlay命令 ####
滤镜支持下面的命令：

- x
- y

    修改覆盖输出的`x`和`y`坐标。它接受的语法同于前面对应的选项

    如果指定的表达式无效，则保持当前值

#### overlay例子 ####
- 在据右下10像素位置绘制主要视频:

    overlay=main_w-overlay_w-10:main_h-overlay_h-10

    采用名字选项，则上面的变成:

    overlay=x=main_w-overlay_w-10:y=main_h-overlay_h-10
- 插入一个透明的PNG标记到右下。协同`ffmpeg`工具集利用`-filter_complex`选项来实现:

    ffmpeg -i input -i logo -filter_complex 'overlay=10:main_h-overlay_h-10' output
- 插入2个不同的透明PNG标志，（第二个在右下角）:

    ffmpeg -i input -i logo1 -i logo2 -filter_complex 'overlay=x=10:y=H-h-10,overlay=x=W-w-10:y=H-h-10' output
- 在视频上面添加一个透明颜色层，其中`WxH`指定了输入视频的宽和高:

    color=color=red@.3:size=WxH [over]; [in][over] overlay [out]
- 同时播放原始和过滤版（协同了`deshake`滤镜）:

    ffplay input.avi -vf 'split[a][b]; [a]pad=iw*2:ih[src]; [b]deshake[filt]; [src][filt]overlay=w'

    同上效果，但利用命令完成

    ffplay input.avi -vf 'split[b], pad=iw*2[src], [b]deshake, [src]overlay=w'
- 从2秒开始一个从左到右的滑动叠加:

    overlay=x='if(gte(t,2), -w+(t-2)*20, NAN)':y=0
- 组合2个输入，一个放置在一边:

    ffmpeg -i left.avi -i right.avi -filter_complex "
    nullsrc=size=200x100 [background];
    [0:v] setpts=PTS-STARTPTS, scale=100x100 [left];
    [1:v] setpts=PTS-STARTPTS, scale=100x100 [right];
    [background][left]       overlay=shortest=1       [background+left];
    [background+left][right] overlay=shortest=1:x=100 [left+right]
    "

- 在10-20秒应用一个`delogo`滤镜

    ffmpeg -i test.avi -codec:v:0 wmv2 -ar 11025 -b:v 9000k
    -vf '[in]split[split_main][split_delogo];[split_delogo]trim=start=360:end=371,delogo=0:0:640:480[delogoed];[split_main][delogoed]overlay=eof_action=pass[out]'
    masked.avi
- 串联多个`overlays`滤镜:

    nullsrc=s=200x200 [bg];
    testsrc=s=100x100, split=4 [in0][in1][in2][in3];
    [in0] lutrgb=r=0, [bg]   overlay=0:0     [mid0];
    [in1] lutrgb=g=0, [mid0] overlay=100:0   [mid1];
    [in2] lutrgb=b=0, [mid1] overlay=0:100   [mid2];
    [in3] null,       [mid2] overlay=100:100 [out0]

### owdenoise ###
应用超完备小波降噪

滤镜接受下面的选项：

- depth

    设置深度

    大的值将在低频部分降噪明显，但速度很慢

    值范围8-16，默认为8
- luma_strength, ls

    设置亮度强度

    为0-1000的双精度值，默认为1.0
- chroma_strength, cs

    设置色度强度

    为0-1000的双精度值，默认为1.0

### pad ###
在原始输入的`x`和`y`坐标上填充输入图像（多处部分用颜色填充）

它接受下面参数：

width, w
height, h

    指定输出尺寸的表达式。如果值为0，则输入图像尺寸作为输出尺寸

    在`width`表达式中可以引用`height`值，反之亦然

    默认都是0
x
y

    指定输入图像在输出中放置的坐标据上边和左边的值

    其中`x`可以引用`y`值计算，反之亦然

    默认为0.
color

    指定添加区域的颜色。语法参考[颜色/color](ffmpeg-doc-cn-07.md#颜色Color)章节的介绍

    默认为"black". 

对于`width`, `height`, `x`和`y`选项的表达式，可以包含下面的常量:

in_w
in_h

    输入宽和高
iw
ih

    同于`in_w`和`in_h`
out_w
out_h

    输出的宽和高（输出添加的区域），它由`width`和`height`表达式指定
ow
oh

    同于`out_w`和`out_h`
x
y

    指定的x和y的偏移（在另外一个表达式中），如果不指定则为`NAN`
a

    同于`iw / ih`
sar

    输入样本点长宽比
dar

    输入视频长宽比，它等于`(iw / ih) * sar`
hsub
vsub

    水平和垂直色度分量值，例如对`yuv422p`像素格式，`hsub`为2，`vsub`为1 

#### pad例子 ####
- 在（0，40）填充`violet`颜色到输入视频，输出视频为`640x480`

	pad=640:480:0:40:violet

    其等效于：

    pad=width=640:height=480:x=0:y=40:color=violet
- 把图像放置在原始输入扩大3/2倍的衬底中间位置:

    pad="3/2*iw:3/2*ih:(ow-iw)/2:(oh-ih)/2"
- 输出到一个正方形衬底上，衬底的变长是输入图像宽和高中的大的一个值，放置在正中:

    pad="max(iw\,ih):ow:(ow-iw)/2:(oh-ih)/2"
- 输出成为有16:9的长宽比的衬底上，水平中置图像（其他比例视频转16:9视频，但不拉伸放缩的效果）:

    pad="ih*16/9:ih:(ow-iw)/2:(oh-ih)/2"
- 在合成视频的情况下，为了正确设置显示，有必要利用`sar`设置表达式:

    (ih * X / ih) * sar = output_dar
    X = output_dar / sar

    因此需要修正前例为:

    pad="ih*16/9/sar:ih:(ow-iw)/2:(oh-ih)/2"
- 双倍输入视频尺寸，把输入放置在右下区域（占输出区域的右下1/4）:

    pad="2*iw:2*ih:ow-iw:oh-ih"

### palettegen ###
对整个视频产生一个调色板

它接受下面的选项：

- max_colors

    设置调色板最大数量。**注意**调色板仍包含256个色彩，只是没有用到的色彩都被设置为黑色了
- reserve_transparent

    创建一个255色调色板，最后一个存储颜色。GIF优化保留透明的颜色是非常有用的。如果没有设置,那么最大的颜色调色板将达到256。你可能对于某个独立的图像想要禁用该选项。默认设置是`default`
- stats_mode

    设置统计模式

    接受下面值:

    ‘full’

        计算全帧直方图
    ‘diff’

        只计算与前一帧的差异部分。这将更关注输入中变化的部分（运动的部分）,如果背景是静态的。

    默认为full. 

#### palettegen例子 ####
- 生成一个调色板

	ffmpeg -i input.mkv -vf palettegen palette.png

### paletteuse ###
使用一个调色板来处理输入视频流中的样本点转换

滤镜接受2个输入，一个视频输入流和一个调色板。调色板必须是256个像素的图像（即有256个颜色）

它接受下面选项：

- dither

    选择抖动模式，可用算法有:

    ‘bayer’

        顺序8x8 bayer抖动(确定的) 
    ‘heckbert’

        由Paul Heckbert定义与1982年的抖动算法（简单的错误扩散）**注意**这个抖动有时被认为是“错误”的，而仅作为参考 
    ‘floyd_steinberg’

        Floyd 和Steingberg抖动(错误扩散) 
    ‘sierra2’

        Frankie Sierra抖动第二版(错误扩散) 
    ‘sierra2_4a’

        Frankie Sierra抖动第二版的"简化" (错误扩散) 

    默认为sierra2_4a.
- bayer_scale

    当bayer抖动算法被选取，这个选项将定义用作调色板规模（多少交叉——混合是可见的）。一个小的值意味着更多可见的且更少的条纹，高的值意味更少可见可能更多的条纹。

    值范围为[0,5]，默认为2
- diff_mode

    如果设置，定义区域过程

    ‘rectangle’

        只有改变的矩形区域会进行再加工. 这类似于GIF裁剪/抵消压缩机制。它用来加快速度，对于只要部分改变时，如用例，只有一个矩形移动（边界）区域，且限制误差范围抖动（它会导致更多的确定性产出，如果现场没有太大的改变,它可以减少噪音且更好的GIF压缩移动） 

    默认为none. 

#### paletteuse例子 ####
- 利用`palette`（调色板可由[`palettegen`](#palettegen)产生）编码输出GIF

	ffmpeg -i input.mkv -i palette.png -lavfi paletteuse output.gif

### perspective ###
采用正确的视角记录视频，而不是垂直于平面

接受参数的介绍如下：

- x0
- y0
- x1
- y1
- x2
- y2
- x3
- y3

    对左上、右上、左下和右下各点设置表达式。默认为`0:0:W:0:0:H:W:H`表示不改变视角。如果场景选项是对源设置（`sense=0`），那么将发送指定点给目标。如果是对目标设置的（`sense=1`），则对应的源将被送到指定坐标

    表达式接受下面的变量:

    W
    H

        视频帧的宽和高 

- interpolation

    为透视校正设置插值

    允许下面的值:

    ‘linear’
    ‘cubic’

    默认为‘linear’.
- sense

    设置协调选项的解释.

    它接受值为:

    ‘0, source’

        表明前面`x0y0x1y1x2y2x3y3`是对源设置的
    ‘1, destination’

        表明前面`x0y0x1y1x2y2x3y3`是对目标设置的

       	默认为‘source’. 

### phase ###
延迟隔行视频一段时间以便现场秩序变化

用于解决PAL制下电影到视频转换中的场序问题

接受参数介绍见下：

- mode

    设置相位方法，它允许的值为：

    ‘t’

        捕获是上场优先，要转换为下场优先3，滤镜会延迟下场
    ‘b’

        捕获是下场优先，要转为上场优先，滤镜会延迟上场。
    ‘p’

        捕获和输出相同场序。这个模式存在文档中引用的其他选项，如果你真的选择它，滤镜将什么也不做。
    ‘a’

        捕获字段自动确定场序标志，转换则相反。滤镜根据相应标识在逐帧的基础上自动选择' t '和' b '。如果没有包含有效的指示字段，则同于`u`
    ‘u’

        捕获模式未知或者不定，转换时则相反。滤镜通过分析自动选择 ‘t’ 和 ‘b’实现最佳匹配
    ‘T’

        捕获是上场优先，转换未知或者不定。滤镜通过图像分析选择`t`和`p`
    ‘B’

        捕获是下场优先，转换未知或者不定。滤镜通过图像分析选择`b`和`p`
    ‘A’

        捕获根据标志确定，滤镜由此选择`t`、`b`和`p`，如果没有有效的标志信息，则同于`U`，这是默认模式
    ‘U’

        捕获和转换都未知，根据检测自动选择`t`、`b`和`p`以使最佳匹配

### pixdesctest ###
像素格式检测测试滤镜，主要用于内部测试。输出视频等于输入视频

例如：

	format=monow，pixdesctest

可以用来测试monowhite像素格式描述是否符合定义

### pp ###
使用指定的`libpostproc`后处理`subfilters`链。这个库会自动选择一个`GPL`编译（--enable-gpl）。`subfilters`必须是由`/`分隔，可以利用`-`来禁用。每个`subfilter`有长或短的选项名，例如`dr/dering`

滤镜接受下面的选项：

- subfilters

    指定subfilters字符串 

所有subfilters有共同选项来确定其范围，它们是:

a/autoq

    对subfilter的质量等级
c/chrom

    同时做色差和亮度(默认).
y/nochrom

    只做亮度过滤 (无色差处理).
n/noluma

    只做色差过滤 (无亮度处理). 

这些选项可以通过`|`附加在`subfilter`名后面

有效的`subfilter`有：

hb/hdeblock[|difference[|flatness]]

    水平解封滤镜

    difference

        差异因素,高值意味着更多的解封(默认值:32)。
    flatness

        平面度阈值,降低值意味着更多的解封(默认值:39)。

vb/vdeblock[|difference[|flatness]]

    垂直解封滤镜

    difference

        差异因素,高值意味着更多的解封(默认值:32)。
    flatness

        平面度阈值,降低值意味着更多的解封(默认值:39)。

ha/hadeblock[|difference[|flatness]]

    准确的水平解封滤镜

    difference

        差异因素,高值意味着更多的解封(默认值:32)。
    flatness

        平面度阈值,降低值意味着更多的解封(默认值:39)。

va/vadeblock[|difference[|flatness]]

    准确的垂直解封滤镜

    difference

        差异因素,高值意味着更多的解封(默认值:32)。
    flatness

        平面度阈值,降低值意味着更多的解封(默认值:39)。

水平和垂直解封过滤器共享`difference`和`flatness`,因此不能设置平面度值不同的水平和垂直的阈值。

h1/x1hdeblock

    实验的水平解封滤镜
v1/x1vdeblock

    实验的垂直解封滤镜
dr/dering

    去振铃滤镜
tn/tmpnoise[|threshold1[|threshold2[|threshold3]]], temporal noise reducer

    threshold1

        larger -> stronger filtering /大->强滤镜
    threshold2

        larger -> stronger filtering /大->强滤镜
    threshold3

        larger -> stronger filtering /大->强滤镜

al/autolevels[:f/fullyrange], automatic brightness / contrast correction

    f/fullyrange

        拉伸亮度到0-255范围 

lb/linblenddeint

    通过`(1 2 1)`滤镜线性混合`deinterlacing`滤镜的去交错块 
li/linipoldeint

    每秒通过线性插值`deinterlacing`滤镜的去交错块
ci/cubicipoldeint

    每秒通过立方插值`deinterlacing`滤镜的去交错块
md/mediandeint

    每秒通过中值滤波`deinterlacing`滤镜的去交错块
fd/ffmpegdeint

    每秒通过线性(-1 4 2 4 -1)滤波来处理`deinterlacing`滤镜的去交错块
l5/lowpass5

    通过`(-1 2 6 2 -1)`滤波处理`deinterlacing`滤镜的去交错块
fq/forceQuant[|quantizer]

    覆盖指定一个从输入到输出指示不变的量化器

    quantizer

        使用的Quantizer（量化器） 

de/default

    默认的`pp`滤镜组合 (hb|a,vb|a,dr|a)
fa/fast

    快的`pp`滤镜组合 (h1|a,v1|a,dr|a)
ac

    高品质`pp`滤镜组合(ha|a|128|7,va|a,dr|a) 


#### pp例子 ####
- 应用水平和垂直解封，去交错和自动亮度/对比度:

    pp=hb/vb/dr/al
- 应用默认的滤镜但不包括自动亮度/对比度:

    pp=de/-al
- 应用默认滤镜且包括瞬时降噪A:

    pp=default/tmpnoise|1|2|3
- 应用亮度解封，自动根据可用的CPU时间开关垂直解封:

    pp=hb|y/vb|a

### pp7 ###
应用`Postprocessing`滤镜7.它是[`spp`](#spp)滤镜的变通，类似于 spp =6 或者7的点DCT。是有中心样本使用者IDCT后

滤镜接受下面的选项：

- qp

    强制设置的量化参数，接受一个0-63的整数，如果没有设置，将使用视频流的`QP`值（如果可用）
- mode

    设置阀值模式，可用的是:

    ‘hard’

        设置为硬阀值 
    ‘soft’

        设置为软阀值（更好的de-ringing效果,但可能变得更模糊）
    ‘medium’

        设置中度阀值 (最好的效果，默认)

### psnr ###
两个输入视频之间的平均,最大和最小峰值信噪比(PSNR-Peak Signal to Noise Ratio)表示。

滤镜接受2路输入视频，其中第一个输入被认为是“主要”的源，将不改变的进行输出，第二输入作为“参考”视频进行PSNR计算

两个视频必须有相同的分辨率和像素格式才能正常工作。而且假定了有相同的输入帧来进行比较。

获得的平均PSNR会被输出到日志系统

滤镜存储每帧积累MSE（均方误差），并在处理末尾记录帧平均值，其公示为：

	PSNR = 10*log10(MAX^2/MSE)

这里`MAX`是图像中每个分量最大值的平均值。

滤镜接受参数的介绍见下：

- stats_file, f

    如果指定，滤镜将采用指定的文件来保存每个帧的PSNR值

在`stats_file`指定的文件中将包含一个`key/value`序列，其是每两个比较帧的相应值

`stats_file`指定的文件中各个key的介绍如下：:

- n

    帧序数，从1计数
- mse_avg

    对比帧逐像素均方误差，平均值是基于整个图像所有分量
- mse_y, mse_u, mse_v, mse_r, mse_g, mse_g, mse_a

    对比帧逐像素分量均方误差，后缀就是分量类型
- psnr_y, psnr_u, psnr_v, psnr_r, psnr_g, psnr_b, psnr_a

    对比帧分量峰值信噪比，后缀表明分量类型

例如：

	movie=ref_movie.mpg, setpts=PTS-STARTPTS [main];
	[main][ref] psnr="stats_file=stats.log" [out]

在这个例子中，输入和`ref_movie.mpg`文件比较，每个单独帧的`PSNR`存储在`stats.log`中

### pullup ###
下拉转换（逆电视电影）滤镜。能够处理混合`hard-telecine`，24000/1001帧率逐行和30000/1001帧率逐行内容。

这个`pullup`滤镜利用上下文进行决策。它是无状态的，不锁定模式，但不期待有以下字段可以进行匹配识别和重建逐行帧。

为了能产生一个包含偶数帧率的内容，在`pullup`后插入一个滤镜，如果输入帧率是29.97fps使用`fps=24000/1001`，如果输入是30fps或者25fps则`fps=24`

滤镜接受下面的选项：

- jl
- jr
- jt
- jb

    这些选项设置“垃圾”部分（会被忽略的左、右、上和下部分），左和右是8像素单位的整数倍，上和下是2行的整数被。默认每边8像素
- sb

    设置严格的打断。设置为1将减少滤镜生成一个偶尔不匹配框架的机会，但也可能在高运动序列下导致帧数下降。设置为-1将更容易过滤匹配，则可以帮助处理视频内轻微的模糊，但也可能导致输出交错帧，默认为0
- mp

    设置度量标准平面（通道），允许值有:

    ‘l’

        采用亮度通道
    ‘u’

        采用色差蓝通道
    ‘v’

        采用色差红 

    这个选项被设置为使用指定的通道作为滤镜计算平面，而替换默认的亮度通道计算。它可能会提高精度（如果源材料十分干净），但更可能是降低精度，特别是色度噪声（彩虹效应）或灰度视频（无色差信号——这时结果是随机的）。设置它的主要目的是减少CPU负载，使得`pullup`在慢的计算机上可以减速停止

为了最好的效果（而不在输出中复制帧）且尽量让输出帧率接近，例如要反转一个NTSC输入：

	ffmpeg -i input -vf pullup -r 24000/1001 ...

### qp ###
改变视频的量化参数（QP）

滤镜接受下面选项：
- qp

	设置量化参数表达式

这个表达式通过`eval` API计算，可以包含以下常量：

- known

    1表示索引不是129，否则为0
- qp

    从-129到128的时序索引

#### qp例子 ####
- 一个方程：

	qp=2+2*sin(PI*qp)

### removelogo ###
抑制台标，使用一个图像文件来确定哪些像素组成台标。它通过采用台标相近像素来填充台标位置。

率接受下面选项：

- filename， f

	设置一个过滤位图文件，它指示了要抑制的台标图像信息，格式可以是`libavformat`中任何有效格式。图像的宽和高必须匹配要处理的视频流。

像素依据这样的规则指示台标：为0的位置不属于台标部分，非0的值则是台标的部分。如果使用白色（255）黑色（0）来标志是最安全的。过滤位图建议是带黑底的台标截屏，然后使用一个阀值标志台标周围的虚化。

若必要，可以手动固定小斑点。**记住**如果台标像素没有被标注到，则滤镜质量会大大降低。标记过多的像素不会有太多伤害，但会增加需要模糊处理信息，对于一个大的图标过多额外像素会减慢处理。

### repeatfields ###
这个滤镜根据其价值使用视频`ES`头中和重复字段中的`repeat_field`标志

### rotate ###
任意旋转视频角度（以弧度值表示）

可选参数介绍如下：

- angle, a

    设置一个表示要旋转角度的弧度表达式（中心旋转）。负值表示逆时针旋转，默认为0

    这个表达式会每帧计算
- out_w, ow

    设置输出宽表达式，默认为"iw",表达式只在配置时计算一次
- out_h, oh

    设置输出高表达式，默认为"iw",表达式只在配置时计算一次
- bilinear

    如果为1则允许双线性插值，为0禁用，默认为1
- fillcolor, c

    设置填补颜色（图像旋转后水平），颜色的语法见[颜色/color](ffmpeg-doc-cn-07.md#颜色Color)章节的介绍，如果指定为`none`表示没有背景色输出（例如背景色不显示——相当于透明）

    默认为"black". 

角度和输出大小的表达式可以包含以下常量和函数:

- n

    输入帧序数，从0计数（开始滤镜时），如果早于滤镜第一帧则为`NAN`
- t

    输入帧按秒时间，当滤镜被配置时为0，早于则为`NAN`
- hsub
- vsub

    水平和垂直色度分量。例如对像素格式`yuv422p`，`hsub`为2，`vsub`为1
- in_w, iw
- in_h, ih

    输入的视频宽和高
- out_w, ow
- out_h, oh

    输出的宽和高，它指定了要填补的宽和高表达式
- rotw(a)
- roth(a)

    计算要完整包含旋转`a`弧度图像的最小的宽和高

    它只能用于计算`out_w`和`out_h`的表达式 

#### rotate例子 ####
- 顺时针旋转PI/6:

    rotate=PI/6
- 逆时针旋转PI/6 :

    rotate=-PI/6
- 顺时针旋转45度:

    rotate=45*PI/180
- 应用一个恒定选择周期T,开始角度为PI/3:

    rotate=PI/3+2*PI*t/T
- 让输入旋转振荡T秒，有A弧度振幅:

    rotate=A*sin(2*PI/T*t)
- 旋转视频，输出能包含所有内容:

    rotate='2*PI*t:ow=hypot(iw,ih):oh=ow'
- 旋转视频，减少输出大小且没有背景:

    rotate=2*PI*t:ow='min(iw,ih)/sqrt(2)':oh=ow:c=none

#### rotate命令 ####
滤镜接受下面的命令：
- a, angle

	设置表示角度的弧度表达式，命令接受同对于选项相同的语法。

	如果指定的表达式无效，则保持当前值（不旋转）

### sab ###
应用形状自适应模糊

滤镜接受下面选项：

- luma_radius, lr

    设置亮度模糊强度，值范围0.1-4.0,默认1.0。更大的值会导致图像更模糊，但更慢
- luma_pre_filter_radius, lpfr

    设置亮度预处理半径，值范围为0.1-2.0，默认值1.0.
- luma_strength, ls

    设置被认为是同一像素的最大亮度区别。值范围0.1-100.0，默认1.0.
- chroma_radius, cr

    设置色差模糊强度，值范围0.1-4.0,默认1.0。更大的值会导致图像更模糊，但更慢
- chroma_pre_filter_radius, cpfr

    设置色差预处理半径，值范围为0.1-2.0
- chroma_strength, cs

    设置被认为是同一像素的最大色差区别。值范围0.1-100.0

每个色度选项值，如果没有明确指定，则选用对应亮度选项值

### scale ###
对输入视频放缩（修改尺寸），利用了`libswscale`库。

这个`scale`滤镜强制输出与输入有相同的长宽比，但改变输出样本点长宽比。

如果输入图像格式不同于下一个滤镜要求的格式，这个`scale`滤镜可以转换以符合要求。

#### scale选项 ####
滤镜接受下面介绍的选项或任何`libswscale`放缩支持的选项。

参考 (ffmpeg-scaler)ffmpeg-scaler手册中以了解完整的放缩选项。

- width, w
- height, h

    设置输出视频尺寸表达式，默认是输入尺寸。

    如果值为0，则输入宽被用作输出

    如果其中一个值被设置为-1，`scale`滤镜将以另外一个值在保持输入长宽比的基础上计算该值。如果两个都设置为-1则以输入尺寸作为输出

    如果一个值设置为-n，且n>1，滤镜将以另外一个值为基础，保持输入长宽比进行计算，然后确保计算出的值是可整除n的最接近值

    看下面对于表达式中允许内容的介绍
- interl

    设置交错模式，允许:

    ‘1’

        强制交错放缩
    ‘0’

        不应用交错放缩Do not apply interlaced scaling.
    ‘-1’

        根据输入源帧标志中是否有`interlaced`来决定是否采用交错放缩 

    默认为‘0’.
- flags

    设置`libswscale`放缩标志，参考ffmpeg-scaler文档来了解完整的值列表。如果没有显式指定将采用默认值
- size, s

    设置视频尺寸，语法同于[视频尺寸（分辨率）](ffmpeg-doc-cn-07.md#视频尺寸（分辨率）)
- in_color_matrix
- out_color_matrix

    设置输入/输出 `YCbCr`颜色空间类型

    这将设定一个值来覆盖且强制用于输出和编码器

    如果不指定，则颜色空间类型依赖于像素格式。

    可能值:

    ‘auto’

        自动选择.
    ‘bt709’

        格式符合国际电信联盟( International Telecommunication Union-ITU) 推荐标准 BT.709.
    ‘fcc’

        格式符合美国联邦通信委员会（United States Federal Communications Commission-FCC)的美国联邦法规（Code of Federal Regulations-CFR) 的Title 47 (2003) 73.682 (a).
    ‘bt601’

        设置颜色空间符合：

            格式符合国际电信联盟( International Telecommunication Union-ITU) 推荐标准 BT.601
            ITU-R Rec. BT.470-6 (1998)系统的B, B1, 和 G 以及电影与电视工程师学会（
            Society of Motion Picture and Television Engineers-SMPTE)的 ST 170:2004 

    ‘smpte240m’

        颜色空间符合SMPTE ST 240:1999. 

- in_range
- out_range

    设置输入/输出`YCbCr`样本范围

    它设置一个值来覆盖且强制用于输出和编码。如果不知道，则范围依赖于输入像素格式，可能值是:

    ‘auto’

        自动选择.
    ‘jpeg/full/pc’

        设置完整的范围 (在8-bit亮度上是0-255).
    ‘mpeg/tv’

        设置"MPEG"范围(在8-bit亮度上是16-235 ). 

- force_original_aspect_ratio

    设置在减少或者增加视频高/宽时是否保持原来的宽高比模式。可能值:

    ‘disable’

        禁用这个特性而完全依据设置的宽/高尺寸
    ‘decrease’

        输出视频尺寸将自动减少（以保持宽高比）
    ‘increase’

        输出视频尺寸将自动增加（以保持宽高比）

    关于这个选项的有用例子是：当你知道一个特定设备的最大允许尺寸时，可以使用这个来限制输出视频，同时保持宽高比。例如设备允许1280x720，你的视频是1920x800，使用这个选项（选择‘decrease’），且设定输出是1280x720，将得到一个1280x533的实际结果。

    **请注意**它与让w或者h为-1有些不同，要使用这个选项，你需要完整指定w和h为实际需要的尺寸才能工作

在上述表达式中允许下面的内容：:

- in_w
- in_h

    输入的宽和高
- iw
- ih

    同于`in_w`和`in_h`.
- out_w
- out_h

    输出 (放缩后)宽和高
- ow
- oh

    同于`out_w`和`out_h`
- a

    等于`iw / ih`
- sar

    输入的样本点宽高比
- dar

    输入的宽高比，计算自`(iw / ih) * sar`
- hsub
- vsub

    水平和垂直输入色差分量值，例如对于`yuv422p`像素格式，`hsub`为2，`vsub`为1
- ohsub
- ovsub

    水平和垂直输出色差分量值，例如对于`yuv422p`像素格式，`hsub`为2，`vsub`为1

#### scale例子 ####
- 放缩输入为200x100

    scale=w=200:h=100

    等效于:

    scale=200:100

    或者:

    scale=200x100
- 指定输出为缩写尺寸名:

    scale=qcif

    也可以写为:

    scale=size=qcif
- 放大为输入的2倍:

    scale=w=2*iw:h=2*ih
- 上面等效于:

    scale=2*in_w:2*in_h
- 放大2倍，且采用交错放大模式:

    scale=2*iw:2*ih:interl=1
- 缩小为一半:

    scale=w=iw/2:h=ih/2
- 增大宽度，且高度保持:

    scale=3/2*iw:ow
- 黄金分割比:

    scale=iw:1/PHI*iw
    scale=ih*PHI:ih
- 增加高度，且让宽是高的3/2:

    scale=w=3/2*oh:h=3/5*ih
- 增加尺寸，让尺寸是一个色差分量的整数倍:

    scale="trunc(3/2*iw/hsub)*hsub:trunc(3/2*ih/vsub)*vsub"
- 增加最大为500像素，保持输入的宽高比:

    scale=w='min(500\, iw*3/2):h=-1'

### separatefields ###
这个`separatefields`滤镜以视频帧为基础，将每个帧分成多个分量，产生一个半高而两有两倍帧率和帧数。

这个滤镜默认使用在帧中的场序标志信息来决定那些先输出。如果结果不对，在之前使用[`setfield`](#setfield)

### setdar, setsar ###
其中`setdar`滤镜设置输出的宽高比

是按下式通过改变指定的样本（即像素）宽高比来实现的：

	DAR = HORIZONTAL_RESOLUTION / VERTICAL_RESOLUTION * SAR

记住，`setdar`滤镜不修改视频帧的像素尺寸（画面尺寸，即还是WxH）。设定的显示长宽比也可能改变其后的滤镜链中的滤镜，例如在放缩滤镜或者另一个“setdar”或“setsar”滤镜。

而`setsar`滤镜设置输出视频的像素宽高比

**注意**有用`setsar`滤镜的应用，输出显示的宽高比将根据前面的方程变化。

**记住**这里改变了样本（像素）宽高比将影响到后续滤镜，如另外一个“setdar”或“setsar”滤镜。

它接受下面参数：

- r, ratio, dar (setdar only), sar (setsar only)

    设置指定的宽高比

    可以是浮点数或者表达式或者一个`num:den`形式的字符串（其中分别对应分子和分母，表示`num/den`）。如果没有指定则假定为0。在使用`num:den`形式时要注意特殊字符转义问题。
- max

    设置可改变的最大整数的值，其用于保证宽高比为设置值，而允许在分子（对应宽）和分母（对应高）减少的数.默认为100

参数表达式接受下面的内容:

- E, PI, PHI

    一些可用常量，其中`E`为自然对数的底, `PI`为π值, 以及`PHI`为黄金比例
- w, h

    输入的宽和高
- a

    等于`w / h`
- sar

    输入的样本点宽高比
- dar

    输入的宽高比，计算自`(iw / ih) * sar`
- hsub, vsub

    水平和垂直输入色差分量值，例如对于`yuv422p`像素格式，`hsub`为2，`vsub`为1

#### setdar, setsar例子 ####
- 设置输出为16：9，可以是

	setdar=dar=1.77777
	setdar=dar=16/9
	setdar=dar=1.77777
- 设置样本点宽高比为10：11

	setsar=sar=10/11
- 设置输出为16：9，并设置了`max`值，采用命令

	setdar=ratio=16/9:max=1000

### setfield ###
强制输出帧的场序

这里`setfield`把交错场提供给输出帧，它不改变输入帧，只是设置相应的属性，影响到后续的滤镜（如果 [`fieldorder`](#fieldorder)和 [`yadif`](#yadif)）

滤镜接受下面选项：

- mode

    有效的值是:

    ‘auto’

        保持输入属性.
    ‘bff’

        设置为下场优先
    ‘tff’

        设置为上场优先
    ‘prog’

        设置为逐行 

### showinfo ###
不改变输入而在行中显示每帧信息。

显示的信息以`key/value`的序列形式给出

下面是将显示在输出中的值：

- n

    帧序数，从0开始计数
- pts

    输入帧的时间戳，以时基为单位，时间依赖于输入
- pts_time

    按秒计的时间戳
- pos

    输入帧在输入流中的偏移定位，-1表示信息不可用和/或无意义（例如合成视频中）
- fmt

    像素格式名
- sar

    输入帧的宽高比，表示为`num/den`格式
- s

    输入帧尺寸，语法同于[视频尺寸（分辨率）](ffmpeg-doc-cn-07.md#视频尺寸（分辨率）)
- i

    交错模式 ("P"对应 "逐行", "T" 对应上场优先, "B"为下场优先t)
- iskey

    为1表示是关键帧，0则不是
- type

    输入帧图片类型 ("I"对应I帧, "P" 对应P帧, "B" 对应B帧,或者 "?"对应未知类型).参考定义与`libavutil/avutil.h`中的`av_get_picture_type_char`函数和`AVPictureType`枚举
- checksum

	输入帧所有信息内容的 Adler-32校验值 (以16进制输出)
- plane_checksum

    输入帧所有信息内容的 Adler-32校验值 (以16进制输出), 以格式"[c0 c1 c2 c3]"显示 

### showpalette ###
显示每个帧的256色模板。滤镜是关于`pal8`像素格式帧的。

它接受下面选项：

-s 

	设置调色板中颜色数量宽，默认为30（对应于30x30`pixel`）

### shuffleplanes ###
重排和/或复制视频通道

它接受下面参数：

- map0

    被用作第一个输出通道的输入通道
- map1

    被用作第二个输出通道的输入通道
- map2

    被用作第三个输出通道的输入通道
- map3

    被用作第四个输出通道的输入通道

第一个通道序数为0，默认是保持输入不变。

交换第二和第三通道：

	ffmpeg -i INPUT -vf shuffleplanes=0:2:1:3 OUTPUT

### signalstats ###
评估各种视觉指标,协助确定问题与模拟视频媒体的数字化。

默认滤镜会以日志记录这些元数据值：

- YMIN

    显示输入帧中最小的Y值（亮度），范围为[0-255]
- YLOW

    显示10%(表示有10%的像素点Y值低于这个值)的Y值,范围[0-255]
- YAVG

    显示输入帧的平均Y值，范围[0-255]
- YHIGH

    显示90%(表示有90%的像素点Y值低于这个值)的Y值，范围[0-255]
- YMAX

    显示输入帧中最大Y值，范围[0-255]
- UMIN

    显示输入帧中最小的U值（色差U），范围为[0-255]
- ULOW

    显示10%(表示有10%的像素点U值低于这个值)的U值,范围[0-255]
- UAVG

    显示输入帧的平均U值，范围[0-255]
- UHIGH

    显示90%(表示有90%的像素点U值低于这个值)的U值，范围[0-255]
- UMAX

    显示输入帧中最大U值，范围[0-255]
- VMIN

    显示输入帧中最小的V值（色差V），范围为[0-255]
- VLOW

    显示10%(表示有10%的像素点V值低于这个值)的V值,范围[0-255]
- VAVG

    显示输入帧的平均V值，范围[0-255]
- VHIGH

    显示90%(表示有90%的像素点U值低于这个值)的U值，范围[0-255]
- VMAX

    显示输入帧中最大V值，范围[0-255]
- SATMIN

    显示输入帧最小色包含度值，范围[0-~181.02].
- SATLOW

    显示输入帧10%(表示有10%的像素点该类值低于这个值)色包含度值，范围[0-~181.02].
- SATAVG

    显示输入帧最平均包含度值，范围[0-~181.02].
- SATHIGH

    显示输入帧90%(表示有90%的像素点该类值低于这个值)色包含度值，范围[0-~181.02].
- SATMAX

    显示输入帧最大色包含度值，范围[0-~181.02].
- HUEMED

    显示输入帧中的颜色中值，范围[0-360].
- HUEAVG

    显示输入帧中颜色中值平均，范围[0-360].
- YDIF

    显示当前帧与前帧的像素平均Y值差，范围[0-255].
- UDIF

    显示当前帧与前帧的像素平均U值差，范围[0-255].
- VDIF

    显示当前帧与前帧的像素平均V值差，范围[0-255].

滤镜接受下面的选项:

- stat
- out

    `stat`指定一个额外形式的图像分析， `out`则对指定类型像素在输出视频中高亮（突出）显示

    这两个选项都接受下面的值：:

    ‘tout’

        识别时间离群值（瞬时差异）像素。 一个时间离群是像素不同于其周围的像素有同样的分量特性（趋势）。例如视频的中断、头阻塞、磁带跟踪问题
    ‘vrep’

        识别垂直线重复。垂直线重复即帧内包含类似的行像素。在数码视频中垂直线重复较常见的,但这种情况在数字化模拟源时是罕见的，当出现则表明信号失落补偿出现了问题。  
    ‘brng’

        识别像素超出范围的

- color, c

    指定高亮的颜色，默认为黄色 

#### signalstats例子 ####
- 输出各种数据指标

	ffprobe -f lavfi movie=example.mov,signalstats="stat=tout+vrep+brng" -show_frames

- 输出每帧Y的最小和最大值:

	ffprobe -f lavfi movie=example.mov,signalstats -show_entries frame_tags=lavfi.signalstats.YMAX,lavfi.signalstats.YMIN
	
- 播放视频，并以红色突出标识超出范围的像素

	ffplay example.mov -vf signalstats="out=brng:color=red"

- 播放视频，并与标志元数据一同展示：

	ffplay example.mov -vf signalstats=stat=brng+vrep+tout,drawtext=fontfile=FreeSerif.ttf:textfile=signalstat_drawtext.txt

	使用的命令后signalstat_draw.text的内容:
```
time %{pts:hms}
Y (%{metadata:lavfi.signalstats.YMIN}-%{metadata:lavfi.signalstats.YMAX})
U (%{metadata:lavfi.signalstats.UMIN}-%{metadata:lavfi.signalstats.UMAX})
V (%{metadata:lavfi.signalstats.VMIN}-%{metadata:lavfi.signalstats.VMAX})
saturation maximum: %{metadata:lavfi.signalstats.SATMAX}
```
### smartblur ###
在不影响轮廓的基础上模糊视频

它接受下面的选项：

- luma_radius, lr

    设置亮度半径，为浮点数，范围[0.1,5.0],用于指示高斯滤波模糊的方差值（越大越慢），默认为1.0
- luma_strength, ls

    设置亮度强度。为浮点数，范围[-1.0,1.0],用于配置模糊强度，在[0.0,1.0]内的值将模糊图像，而[-1.0,0.0]内的值将锐化图像，默认为1.0
- luma_threshold, lt

    设置亮度阀值作为系数来判断一个像素是否该模糊。为[-30,30]内的整数，如果值在[0,30]则过滤平面，如果值在[-30,0]则过滤边缘。默认为0
- chroma_radius, cr

    设置色度半径，为浮点数，范围[0.1,5.0],用于指示高斯滤波模糊的方差值（越大越慢），默认为1.0
- chroma_strength, cs

    设置色度强度。为浮点数，范围[-1.0,1.0],用于配置模糊强度，在[0.0,1.0]内的值将模糊图像，而[-1.0,0.0]内的值将锐化图像，默认为1.0
- chroma_threshold, ct

    设置色度阀值作为系数来判断一个像素是否该模糊。为[-30,30]内的整数，如果值在[0,30]则过滤平面，如果值在[-30,0]则过滤边缘。默认为0

如果色度（差）选项没有显式设置，将采用对应的亮度值来设定

### stereo3d ###
在不同的立体图像格式之间进行转换

滤镜接受下面选项：

- in

    设置输入格式类型

    有效的是:

    ‘sbsl’

        并排，左眼在左，右眼在右
    ‘sbsr’

        并排，右眼在左，左眼在右
    ‘sbs2l’

        半宽并排，左眼在左，右眼在右
    ‘sbs2r’

        半宽并排，右眼在左，左眼在右
    ‘abl’

        上下排布，左眼上，右眼下
    ‘abr’

        上下排布，右眼上，左眼下
    ‘ab2l’

        半高上下排布，左眼上，右眼下
    ‘ab2r’

        半高上下排布，右眼上，左眼下
    ‘al’

        交替帧，左眼前，右眼后
    ‘ar’

        交替帧，右眼前，左眼后

        默认是‘sbsl’. 

- out

    设置输出排布类型：

    有效的在输入格式基础上还有:

    ‘arbg’

        浮雕红/蓝通道 (红色左眼，蓝色右眼)
    ‘argg’

        浮雕红/绿通道 (红色左眼，绿色右眼)
    ‘arcg’

        浮雕红/青通道 (红色左眼，青色右眼)
    ‘arch’

        浮雕红/青一半(红色左眼，青色右眼)
    ‘arcc’

        浮雕红/青通道 (红色左眼，青色右眼)
    ‘arcd’

        浮雕红/青最小二乘优化杜布瓦投影 (红色左眼，青色右眼)
    ‘agmg’

        浮雕绿/品红通道 (绿色左眼，品红色右眼)
    ‘agmh’

        浮雕绿/品红半色通道 (绿色左眼，品红色右眼)
    ‘agmc’

        浮雕绿/品红通道 (绿色左眼，品红色右眼)
    ‘agmd’

        浮雕绿/品红最小二乘优化杜布瓦投影 (绿色左眼，品红色右眼)
    ‘aybg’

        浮雕黄/蓝通道 (黄色左眼，蓝色右眼)
    ‘aybh’

        浮雕黄/蓝半色通道 (黄色左眼，蓝色右眼)
    ‘aybc’

        浮雕黄/蓝通道 (黄色左眼，蓝色右眼)
    ‘aybd’

        浮雕黄/蓝最小二乘优化杜布瓦投影 (黄色左眼，蓝色右眼)
    ‘irl’

        隔行扫描的行 (左眼上行，右眼下一行)
    ‘irr’

        隔行扫描的行 (右眼上行，左眼下一行)
    ‘ml’

        单独输出左眼
    ‘mr’

        单独输出右眼 

    默认是‘arcd’. 

#### stereo3d例子 ####
- 将并排3d格式转换为浮雕黄/绿通道混合3d:

    stereo3d=sbsl:aybd
- 将上下排布3d转换为并排3d

    stereo3d=abl:sbsr

### spp ###
应用一个简单的后处理滤镜，压缩或者解压缩图像（或-`quality`为6）变化提升平均结果。

滤镜接受下面选项：

- quality

    设置质量系数。它是一个平均水平数，范围0-6，为6意味着有最高质量，每个增量大约对应处理速度2倍降低，默认为3
- qp

    强制量化参数。如果不设置，将从输入流中提取`QP`值（如果可用）
- mode

    设置阀值模式，允许：

    ‘hard’

        硬阀值 (默认) 
    ‘soft’

        软阀值 (更好的效果，也可能更模糊). 

- use_bframe_qp

    如果为1将支持使用B帧的QP，使用这个选项如果B帧有大的QP则可能导致闪烁默认为0（不允许）

### subtitles ###
利用`libass`库在输入视频上添加字幕

编译配置选项`--enable-libass`。滤镜还需要编译包含`libavcodec`和` libavformat`以支持把字幕文件转换成ASS(Advanced Substation Alpha) 字幕格式

滤镜接受下面选项：

- filename, f

    设置要读取的字幕文件，它必须指定
- original_size

    指定原始视频分辨率，这个视频是要组合ASS文件的。其语法见[视频尺寸（分辨率）](ffmpeg-doc-cn-07.md#视频尺寸（分辨率）)部分。为了应对ASS中长宽比例不良设计，放缩字体长宽比是必要的
- charenc

    设置输入字符编码，只对字幕过滤，仅当编码不是UTF-8时使用
- stream_index, si

    设置字幕流索引数，仅在`subtitles`滤镜中使用
- force_style

    覆盖`subtitles`的默认样式或脚本信息参数。它接受一个有`，隔开的`KEY=VALUE`字符串

如果第一个键没有指定，则假定第一个值是描述`filename`

例如渲染字幕文件sub.srt到输入视频顶部采用命令:

	subtitles=sub.srt

它等效于:

	subtitles=filename=sub.srt

渲染mkv文件中的字幕:

	subtitles=video.mkv

渲染从文件活动的字幕文件中提取的字幕:

	subtitles=video.mkv:si=1

为字幕添加一个透明的绿`DejaVer`衬线:

	subtitles=sub.srt:force_style='FontName=DejaVu Serif,PrimaryColour=&HAA00FF00'

### super2xsai ###
使用滤镜来平滑放大2倍（放大和插入）像素扩展算法。

通常用于在不降低锐度基础上放大图像。

### swapuv ###
交换U和V分量值

### telecine ###
对视频应用`telecine`处理

滤镜接受下面选项：

- first_field

	可能值：

    ‘top, t’

        上场优先（默认） 
    ‘bottom, b’

        下场优先 

- pattern

    要应用的下拉模式。默认为23

一些典型的模式是 

NTSC输出(30i):

	27.5p: 32222
	24p: 23 (classic)
	24p: 2332 (preferred)
	20p: 33
	18p: 334
	16p: 3444

PAL输出(25i):

	27.5p: 12222
	24p: 222222222223 ("Euro pulldown")
	16.67p: 33
	16p: 33333334

### thumbnail ###
在一个给定的连续帧序列中选定一个最具代表性的帧（缩略图，抽帧效果形成跳帧序列）。

滤镜接受下面选项：

- n

	设置要分析的帧批量大小。设置为`n`帧，则滤镜会从中挑选出一帧，然后处理下面`n`帧，直到最后，默认为100

因为滤镜跟踪整个帧序列，一个更大的`n`将占用更多的内存，所有不推荐设置太高的值 

#### thumbnail例子 ####
-  每50帧抽取一个

	thumbnail=50
-  应用ffmpeg来输出一个缩略图序列

	ffmpeg -i in.avi -vf thumbnail,scale=300:200 -frames:v 1 out.png

### tile ###
让连续几帧磁贴拼接

滤镜接受下面选项：

- layout

    设置网格布局模式（行和列数）。语法同于[视频尺寸（分辨率）](ffmpeg-doc-cn-07.md#视频尺寸（分辨率）)部分
- nb_frames

    设置最大用作渲染的区域尺寸，必须小于或者等于 `wxh`（这里是`layout`值），默认为0，表示最大（所有）尺寸
- margin

    设置外边界边缘像素
- padding

    设置内边界边缘厚度（布局渲染后两个帧间的像素个数）。更大的填补选项（表示有更多的值用于边缘）将由滤镜填充（颜色）
- color

    设置未使用区域的颜色。描述语法同于[颜色/Color](ffmpeg-doc-cn-07.md#颜色Color)部分。默认为"black". 

#### tile例子 ####
- 产生8x8的PNG关键帧标题（`-skip_frame nokey`）

	ffmpeg -skip_frame nokey -i file.avi -vf 'scale=128:72,tile=8x8' -an -vsync 0 keyframes%03d.png

	这里`-vsync 0`是必要的，以防止ffmpeg以原始帧率复制每个输出帧

- 每个区域显示5个图片（在3x2的`layout`值下），间隔7像素且有2像素的外边缘，使用了省略（关键名）和命名选项

	tile=3x2:nb_frames=5:padding=7:margin=2

### tinterlace ###
执行各种类型的时间交错

帧计数从1开始，所以第一帧为奇数帧。

滤镜接受下面选项：

- mode

    指定交错模式。这个选项也可以单独指定一个值。关于值的列表见下。

    有效的值:

    ‘merge, 0’

        移动奇数帧到上场，偶数帧到下场，产生双倍高度的帧，但是只要一半的帧率.

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444

        输出:
        11111                           33333
        22222                           44444
        11111                           33333
        22222                           44444
        11111                           33333
        22222                           44444
        11111                           33333
        22222                           44444

    ‘drop_odd, 1’

        只输出偶数帧，奇数帧被丢弃，不改变高但只要一半帧率

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444

        输出:
                        22222                           44444
                        22222                           44444
                        22222                           44444
                        22222                           44444

    ‘drop_even, 2’

        只输出奇数帧，丢弃了偶数帧，不改变高，但只有一半帧率

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444

        Output:
        11111                           33333
        11111                           33333
        11111                           33333
        11111                           33333

    ‘pad, 3’

        扩展每个帧的高度，每行用黑色间隔，生成有2倍的高，和相同的帧率

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444
        11111           22222           33333           44444

        输出:
        11111           .....           33333           .....
        .....           22222           .....           44444
        11111           .....           33333           .....
        .....           22222           .....           44444
        11111           .....           33333           .....
        .....           22222           .....           44444
        11111           .....           33333           .....
        .....           22222           .....           44444

    ‘interleave_top, 4’

        奇数帧在上保留奇数行，偶数帧在下保留偶数行的交错，不改变高，有一半的帧率

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111<-         22222           33333<-         44444
        11111           22222<-         33333           44444<-
        11111<-         22222           33333<-         44444
        11111           22222<-         33333           44444<-

        输出:
        11111                           33333
        22222                           44444
        11111                           33333
        22222                           44444

    ‘interleave_bottom, 5’

        偶数帧在上保留奇数行，奇数帧在下保留偶数行的交错，不改变高，有一半的帧率

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222<-         33333           44444<-
        11111<-         22222           33333<-         44444
        11111           22222<-         33333           44444<-
        11111<-         22222           33333<-         44444

        输出:
        22222                           44444
        11111                           33333
        22222                           44444
        11111                           33333

    ‘interlacex2, 6’

        双倍帧率不改变高，帧间插入一个两帧的混合，混合模式（依赖于`top_field_first`标志）可是后帧的奇数行和前帧的偶数行交错。这对没有同步的隔行视频显示有好处

         ------> time
        输入:
        Frame 1         Frame 2         Frame 3         Frame 4

        11111           22222           33333           44444
         11111           22222           33333           44444
        11111           22222           33333           44444
         11111           22222           33333           44444

        输出:
        11111   22222   22222   33333   33333   44444   44444
         11111   11111   22222   22222   33333   33333   44444
        11111   22222   22222   33333   33333   44444   44444
         11111   11111   22222   22222   33333   33333   44444

    单独的数字值被弃用，但这里给出是为了向后兼容

    默认为`merge`.
- flags

    指定标记影响筛选过程.

    有效值对`flags`是:

    low_pass_filter, vlfp

        使用垂直低通滤波。在从逐行源（且包含高频垂直细节）创建交错视频时为了减少’twitter’和云纹图案现象需要使用这个滤波器 

        垂直低通滤波仅在`interleave_top`和`interleave_bottom`模式时允许使用

### transpose ###
对输入转置行和列来可选的翻转（图像）

它接受下面的参数：

- dir

    指定翻转方向，可以采用下面的值:

    ‘0, 4, cclock_flip’

        逆时针90度和垂直翻转（默认）,它的效果是:

        L.R     L.l
        . . ->  . .
        l.r     R.r

    ‘1, 5, clock’

        顺时针90度旋转，效果是:

        L.R     l.L
        . . ->  . .
        l.r     r.R

    ‘2, 6, cclock’

        逆时针90度旋转，效果是:

        L.R     R.r
        . . ->  . .
        l.r     L.l

    ‘3, 7, clock_flip’

        顺时针90度旋转且垂直翻转，效果是:

        L.R     r.R
        . . ->  . .
        l.r     l.L

    对于4-7的值，表示转换是对于肖像或风景的。这些值都是被弃用的，应该采用`passthrough`设定来替代

    数值是弃用,应该被删除,取而代之的是符号常量
- passthrough

    如果输入的几何匹配（超出）一个指定的形式部分将不进行转换。有效值是:

    ‘none’

        一直适用 
    ‘portrait’

        保护肖像的几何(对于 height >= width) ，指裁剪一定的转换宽度，保护高度不变（等比例）
    ‘landscape’

        保护景观几何 (对于 width >= height)，表示裁剪一定的转换高度，保持宽度不变（等比例）

    默认为none. 

例如，要顺时针旋转90度且保护肖像布局:

	transpose=dir=1:passthrough=portrait

以命令的形式实现同样效果:

	transpose=1:portrait

### trim ###
减少输入，输出包含一个连续输入的组成部分

它接受下面参数：

- start

    指定开始部分时间的，即帧时间戳开始将输出第一帧
- end

    指定结束部分时间，即帧的时间戳达到的前一帧是输出的最后一帧。
- start_pts

    同于`start`，只是以时基为时间单位替代秒
- end_pts

    同于`end`，只是以时基为时间单位替代秒
- duration

    按秒最大持续时间 seconds.
- start_frame

    开始的帧序数，该帧开始被输出
- end_frame

    结束的帧序数，该帧开始被丢弃（不被输出）

其中`start`、`end`和`duration`采用持续时间表示格式，其语法见[持续时间](ffmpeg-doc-cn-07.md#持续时间) 

**注意**`start`/`end`开头的选项和`duration`都仅关注帧的时间戳，而带`_frame`后缀的版本则仅仅通过帧简单计数。**还要注意**滤镜并不改变时间戳，如果你想输出是以0开始的时间戳，则需要在滤镜后接一个`setpts`滤镜来处理。

如果有多个开始或者结束时间，滤镜将采取贪婪算法，视图在匹配至少一个约束（开始/结束）的情况下尽量多的输出。想保留多个约束部分组成，需要链式采用多个裁剪过滤器（可能还需要结合其他滤镜来拼接）

默认是保留所有输入。所以它可以仅设置一个结束值来保留之前的一切。

例如：

- 只保留第二分钟

	ffmpeg -i INPUT -vf trim=60:120
- 保留第一秒内的

	ffmpeg -i INPUT -vf trim=duration=1

### unsharp ###
锐化或者模糊输入视频

它接受下面的参数：

- luma_msize_x, lx

    设置亮度矩阵水平尺寸。它必须是3-63的奇数值，默认5
- luma_msize_y, ly

    设置亮度矩阵垂直尺寸，它必须是3-63的奇数值，默认5
- luma_amount, la

    设置亮度效果强度，合理值为-1.5 - 1.5的浮点数（可超出前范围）。

    负数值表明视频会被模糊，正数值则会被锐化，0则没有效果

    默认为1.0.
- chroma_msize_x, cx

    设置色度矩阵水平尺寸。它必须是3-63的奇数值，默认5
- chroma_msize_y, cy

    设置色度矩阵垂直尺寸。它必须是3-63的奇数值，默认5.
- chroma_amount, ca

    设置色度效果强度，合理值为-1.5 - 1.5的浮点数（可超出前范围）。

    负数值表明视频会被模糊，正数值则会被锐化，0则没有效果

    默认为0.0.
- opencl

    如果为1，指定采用OpenCL能力, 要求编译时启用了`--enable-opencl`，默认0.

所有参数都是可选的默认值等效于’5:5:1.0:5:5:0.0’字符串 
#### unsharp例子 ####
- 加强亮度锐化效果

	unsharp=luma_msize_x=7:luma_msize_y=7:luma_amount=2.5
- 加强亮度和色度的模糊

	unsharp=7:7:-2:7:7:-2

### uspp ###
应用超慢/简单的后处理,压缩和解压图像(或对应于`quality`中水平为8的完全处理)变化和平均结果。

它不同于`spp`，实际上`uspp`编码和解码每个`libavcodec`块（Snow），而`spp`使用一个内部简化的8x8 DCT，其相似于MJPEG的DCT

滤镜接受下面选项：

- quality

    设置质量水平值。它是平均水平值数字，范围0-8，如果为0，则滤镜没有效果，设置为8将有最好的效果。每增加1级大约速度减慢2倍，默认为3
- qp

    强制设定质量参数，如果不设置，将采用输入流中的QP值（如果可用）

### vidstabdetect ###
分析视频的静止/不晃动，两步过程中的第1步，下一步是[`vidstabtransform`](#vidstabtransform)。

这个滤镜生成一个文件，指定相对平移和旋转变换后续帧的信息，它用于[`vidstabtransform`](#vidstabtransform)滤镜

为了编译支持它需要设置`--enable-libvidstab`

滤镜接受下面选项：

- result

    指定保存转换信息的文件路径。默认为 is transforms.trf.
shakiness

    设置摄像头如何快速设置来满足晃动的视频,值范围是1-10整数，1意味着很小的晃动，10意味着强烈晃动，默认为5
accuracy

    设置检测过程的准确性,值范围为1-15，1表示低精度，15表示高精度。默认15
stepsize

    设置搜索过程的间隔值（扫描尺度）。最低是1像素分辨率扫描，默认为6
mincontrast

    设置最低对比度。低于这个值一个本地测量领域会被丢弃。为范围在0-1的浮点数，默认为0.3.
tripod

    设置参考帧数三脚架模式

    如果允许，对帧运动的比较将以一个参考过滤流相比进行，从中指定一个。这样可以补偿或多或少的静态帧中的所有动作，保持相机视图绝对静止

    如果设为0则禁用，帧数从1开始计数
show

    显示字段和转换生成的帧，接受一个0-2间的整数，默认为0，它禁止任何可视内容。 

#### vidstabdetect例子 ####
- 使用默认值:

    vidstabdetect
- 分析晃动视频的强度，把结果放置在mytransforms.trf:

	vidstabdetect=shakiness=10:accuracy=15:result="mytransforms.trf"
- 把内部转换生成的视频显示出来（可视化）:

    vidstabdetect=show=1
- 在ffmpeg中分析中等强度晃动:

    ffmpeg -i input -vf vidstabdetect=shakiness=5:show=1 dummy.avi

### vidstabtransform ###
视频静止/不晃动，两步过程的第二步，其第一步是[`vidstabdetect`](#vidstabdetect)

从一个文件读取每一帧需要应用/补偿的信息，与[`vidstabdetect`](#vidstabdetect)一起使用来稳定视频，参看[http://public.hronopik.de/vid.stab](http://public.hronopik.de/vid.stab)来了解更多。见下，它对于使用[`unsharp`](#unsharp)是很重要的。

为了使用它需要允许编译设置`--enable-libvidstab`
#### vidstabtransform选项 ####

- input

    设置读取转换信息的文件，默认为transforms.trf.
- smoothing

    设置帧数，其值以表达式 (value*2 + 1)用作低通来滤除摄像机运动，默认为10.

    例如对于设置为10则意味着21帧被使用（过去10帧和接下来10帧）来平滑摄像机移动。更大的值可以得到一个更平滑视频，但限制摄像机加速度(平底锅摇/倾斜 移动)。0表示摄像机是静止的
- optalgo

    设置相机路径优化算法

    接受值:

    ‘gauss’

        镜头运动采用高斯低通滤波器内核(默认) 
    ‘avg’

        转换平均值 

- maxshift

    设置帧中最大转换像素值，默认为-1，表示没有限制
- maxangle

    设置最大帧旋转角度（弧度值，度*PI/180），默认为-1，表示没有限制
- crop

    指定如何处理边界,由于运动补偿可能可见

    有效值:

    ‘keep’

        从以前帧保持图像信息 (默认) 
    ‘black’

        填充黑色边 

- invert

    为1则转化转换。默认值为0
- relative

    为1表示转换是相对于前帧，0表示绝对的（不和前帧相关），默认为0
- zoom

    设置放大比例。正数则相对于推进效果，负数相当于拉远效果，默认为0（不变）
- optzoom

    设置最佳缩放以避免边界

    可能值:

    ‘0’

        禁止 
    ‘1’

        确定最优静态缩放值(只有很强的运动将导致可见边界)(默认)
    ‘2’

        确定最优自适应缩放值(没有边界可见),参见`zoomspeed` 

    **注意**这里的`zoom`值被添加到一个计算中
- zoomspeed

    设置每帧放大的最大百分比限度值（当`optzoom`被设置为2时），范围为0-5，默认为0.25
- interpol

    指定插值类型

    有效值是:

    ‘no’

        不插值 
    ‘linear’

        水平线性插值 
    ‘bilinear’

        在两个方向上线性插值（默认）
    ‘bicubic’

        在两个方向上立方插值(慢) 

- tripod

    如果为1启用虚拟三脚架模式，其等效于`relative=0:smoothing=0`默认为0 Default value is 0.

    它要求在`vidstabdetect`中也启用`tripod`
- debug

    为1增加日志记录按冗长形式。也检测全局运动写入到临时文件 global_motions.trf，默认为0 

#### vidstabtransform例子 ####
- 帧ffmpeg使用默认典型的稳定系数:

    ffmpeg -i inp.mpeg -vf vidstabtransform,unsharp=5:5:0.8:3:3:0.4 inp_stabilized.mpeg

    **注意**一直建议使用[`unsharp`](#unsharp)
- 从给定文件加载转换数据来放大一点:

    vidstabtransform=zoom=5:input="mytransforms.trf"
- 使视频更平滑:

    vidstabtransform=smoothing=30

### vfilp ###
让输入垂直翻转

例如：利用ffmpeg垂直翻转视频

	ffmpeg -i in.avi -vf "vflip" out.avi

### vignette ###
使或扭转自然渐晕效应

滤镜接受下面选项：

- angle, a

    以弧度表示的镜头组角度

    值范围为 [0,PI/2]

    默认为: "PI/5"
- x0
- y0

    设置中心坐标表达式，默认分别是"w/2" and "h/2" 
- mode

    设置向前/向后模式

    有效值为：

    ‘forward’

        中心点的距离越大,图像的颜色越深
    ‘backward’

        中心点的距离越大,图像越亮。这可以用于扭转装饰图案效果,虽然没有自动检测提取镜头角度和其他设置。它也可以用来创建一个燃烧的效果。

    默认为‘forward’.
- eval

    设置表达式计算模式(对于angle, x0, y0).

    有效值为:

    ‘init’

        只在初始化时计算一次
    ‘frame’

        每帧计算，它的速度远低于`init`模式，因为它需要每帧计算所有表达式，但这允许了先进的动态表达式（完成一些特效）

    默认为‘init’.
- dither

    为1（默认）则启用抖动减少循环条带效应
- aspect

    设置插图像素长宽比。此设置将允许调整插图形状，设置值对于输入`SAR`（样本长宽比）将调整矩形光损失后的尺寸

    默认为1/1. 

#### vignette表达式 ####
这里有`angle`（原文误为alpha）, x0 和 y0表达式允许包含的参数

- w
- h

    输入的宽和高
- n

    输入帧序数，从0开始计
- pts

    以时基单位计的PTS (作品时间戳)，未定义则为`NAN`
- r

    输入视频帧率，未知则为`NAN`
- t

    以秒计的PTS (作品时间戳)，未定义则为`NAN`
- tb

    输入视频时基 

#### vignette例子 ####
- 应用简单的强大的渐晕效应:

    vignette=PI/4
- 做一个闪烁的光损失:

    vignette='PI/4+random(1)*PI/50':eval=frame

### w3fdif ###
反交错的输入视频(“w3fdif”代表“韦斯顿3场反交错滤波器——Weston 3 Field Deinterlacing Filter”)。

基于英国广播公司（BBC R&D）的马丁•韦斯顿（Martin Weston）研发，并由吉姆·伊斯特布鲁克（Jim Easterbrook）实现的反交错算法。这个滤镜使用的滤波系数是BBC研发的

它有两组滤波系数，被称为"simple"（简单）和 "complex"（复杂）。使用那个滤波系数可以通过参数设置。

- filter

    设置采用的滤波系数，允许值为：

    ‘simple’

        简单滤波器系数. 
    ‘complex’

        复杂滤波器系数 

    默认‘complex’.
- deint

    指定帧反交错，接受值为:

    ‘all’

        反交错所有帧 
    ‘interlaced’

        仅反交错设置为交错的帧 

    默认‘all’. 

### xbr ###
对像素应用一个xBR高质量放大滤镜，它遵循一套边缘检测规则，详情见[http://www.libretro.com/forums/viewtopic.php?f=6&t=134](http://www.libretro.com/forums/viewtopic.php?f=6&t=134)

接受选项：

- n

	设置放缩尺寸， 2对应于2xBR，3对应于3xBR，4对应于4xBR，默认为3

### yadif ###
反交错输入视频（`yadif`意味着另外一个反交错滤镜）

它接受下面的参数：

- mode

    采用隔行扫描模式。它接受下列值之一:

    0, send_frame

        对每帧都输出 
    1, send_field

        对每场都输出一帧 
    2, send_frame_nospatial

        类似`send_frame`,但跳过交错检查
    3, send_field_nospatial

        类似`send_field`,但跳过交错检查 

    默认为`send_frame`
- parity

    假定输入隔行视频的模式，它接受下列值:

    0, tff

        假定为上场优先 
    1, bff

        假定为下场优先 
	-1, auto

        自动侦测

    默认为`auto`,如果交错模式未知或者不能正确处理则假定为`tff`
- deint

    指定哪些帧需要反交错，接受下列值:

    0, all

        所有帧 
    1, interlaced

        仅标记为交错的帧 

    默认为所有

### zoompan ###
应用放大和摇镜头效果

滤镜接受下面选项：

- zoom, z

    设置放大系数表达式，默认为1
- x
- y

    设置x和y表达式，默认为0
- d

    设置持续帧数，这设置有多少数量的帧受到影响
- s

    设置输出图像尺寸，默认为 ’hd720’. 
每个表达式接受下列参数:

- in_w, iw

    输入的宽
- in_h, ih

    输入高
- out_w, ow

    输出宽
- out_h, oh

    输出高
- in

    输入帧计数
- on

    输出帧计数
- x
- y

    最后计算的x和y对于当前输入帧的x和y表达式。
- px
- py

    之前输入帧对应的最后输出帧最后计算’x’ 和 ’y’，或者为0（第一个输入帧）
- zoom

    当前输入帧对应的最后`z`表达式计算得出的放大系数
- pzoom

    前一输入帧前最后输出帧计算的放大系数
- duration

    当前输入帧对应的输出帧数。对每个输入帧计算`d`值
- pduration

    前一输入帧之前创建输出帧的数量
- a

    有理数 = iw/ih
- sar

    样本长宽比
- dar

    显示长宽比

#### zoompan例子 ####
- 推近到1.5 并且同时在中心附近摇的效果:

    zoompan=z='min(zoom+0.0015,1.5)':d=700:x='if(gte(zoom,1.5),x,x+1/a)':y='if(gte(zoom,1.5),y,y+1)':s=640x360
- 推近到1.5 并且同时以中心摇的效果:

    zoompan=z='min(zoom+0.0015,1.5)':d=700:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)'




