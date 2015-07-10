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
    ffplay -f lavfi "color,drawtext=text=TEST:fontsize=50:fontfile=FreeSerif.ttf:fontcolor_expr=ff0000%{eif\\\\: clip(255*(1*between(t\\, $DS + $FID\\, $DE - $FOD) + ((t - $DS)/$FID)*between(t\\, $DS\\, $DS + $FID) + (-(t - $DE)/$FOD)*between(t\\, $DE - $FOD\\, $DE) )\\, 0\\, 255) \\\\: x\\\\: 2 }"

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

