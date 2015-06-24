## 17 视频编码器 ##
介绍一些当前有效的视频编码器

### libtheora ###
libtheora的封装

编译需要头和库文件，还需要利用`--enable-libtheora`在配置中允许

更多信息参考[http://www.theora.org/](http://www.theora.org/)

#### libtheora选项 ####
下面是映射给libtheora的全局选项，它们对品质和码率产生影响。


- b

    对CBR（固定码率编码）设置码率，单位bit/s，在VBR（动态码率编码）模式下本选项被忽略。
- flags

    设置是否允许`qscale标志（恒定质量模式——VBR模式下）在`pass1`和`pass2`（2次编码方式）
- g

    设置GOP尺寸
- global_quality

    设置全局质量，在 lambda工具集中是一个整数单位的倍数

    仅在VBR模式中，同时允许了 +qscale。这个值会除以`FF_QP2LAMBDA转换为QP,范围为[0 - 10]，再乘以6.3得到本地有效libtheora范围[0-63]，越大质量越高
- q

    仅作VBR模式下，设置为非负数。作为双精度浮点质量单位值，用于转换计算QP

    值范围为 [0-10] ，再乘以 6.3 将获得libtheora有效质量参数，范围[0-63]

    这个选项仅用于ffmpeg命令行工具，库接口使用`global_quality` 

#### libtheora例子 ####

- 使用最大恒定质量（VBR）编码:

    ffmpeg -i INPUT -codec:v libtheora -q:v 10 OUTPUT.ogg

- 使用CBR 1000 kbps编码 Theora视频流:

    ffmpeg -i INPUT -codec:v libtheora -b:v 1000k OUTPUT.ogg

### libvpx ###
VP8/VP9格式支持，通过libvpx

编译需要头和库文件，还需要利用`--enable-libvpx`在配置中允许

#### libvpx选项 ####
下面的选项被libvpx封装支持，部分等效的`vpxenc-XXX`类型的选项或者值列在括号中。

为了减少文件复制，只有私有的选项和一些需要特别注明（注意）的记录在这里，其他的请参考[10 编码]章节

为了了解更多关于libvpx的选项，可以在命令行中使用`ffmpeg -h encoder=libvpx`或者`ffmpeg -h encoder=libvpx-vp9`或`vpxenc --help`来获取。进一步信息可以在`libvpx API`文档中获取。

- b (target-bitrate)

    设置码率，单位bits/**注意**FFmpeg中b选项的单位是bits/s，而在vpxenc中目标码率单位是kilobits/s。
- g (kf-max-dist)
- keyint_min (kf-min-dist)
- qmin (min-q)
- qmax (max-q)
- bufsize (buf-sz, buf-optimal-sz)

    设置码率控制缓冲区大小(单位bits)。**注意**在vpxenc中是指定为多少milliseconds（毫秒），这个封装库通过下面的公式进行转换： buf-sz = bufsize * 1000 / bitrate, buf-optimal-sz = bufsize * 1000 / bitrate * 5 / 6.
- rc_init_occupancy (buf-initial-sz)

    设置解码开始前需要加载到RC的预加载数, **注意**vpxenc中指定多少 milliseconds（毫秒），这个封装库按下面公式转换: rc_init_occupancy * 1000 / bitrate.
- undershoot-pct

    设置数据下冲（min）的目标比特率
- overshoot-pct

    设置数据上冲(max)目标比特率
- skip_threshold (drop-frame)
- qcomp (bias-pct)
- maxrate (maxsection-pct)

    设置GOP最大比特率，单位 bits/s ，**注意**vpxenc描述这个为目标码率，这个封装中按如下公式计算： (maxrate * 100 / bitrate).
- minrate (minsection-pct)

    设置GOP最小比特率，单位 bits/s ，**注意**vpxenc描述这个为目标码率，这个封装中按如下公式计算： (minrate * 100 / bitrate).
- minrate, maxrate, b end-usage=cbr

    (minrate == maxrate == bitrate).
- crf (end-usage=cq, cq-level)
- quality, deadline (deadline)

    ‘best’

        使用最优质量期限，不是非常慢，这个选项指定可以有不低于`good`的输出质量（稍微慢一些）。
    ‘good’

        使用高质量期限，它在速度、质量，以及CPU使用间进行均衡。 
    ‘realtime’

        使用实时质量期限 

- speed, cpu-used (cpu-used)

    设置质量/速度 比，高的参数值将加大编码质量成本
- nr (noise-sensitivity)
- static-thresh

    设置一个变化阀值，低于它将被编码器跳过
- slices (token-parts)

    **注意**，FFmpeg指定的是切片分区总数，而vpxenc中是标记部分的log2值
- max-intra-rate

    设置最大I帧比特率作为目标比特率的百分比，0表示不限
- force_key_frames

    VPX_EFLAG_FORCE_KF
- Alternate reference frame related

    auto-alt-ref

        启用备用参考帧，只在2次编码的pass2起效 
    arnr-max-frames

        设置altref降噪的最大帧数
    arnr-type

        设置altref降噪参考过滤类型: backward, forward, centered. 
    arnr-strength

        设置altref降噪滤波强度 
    rc-lookahead, lag-in-frames (lag-in-frames)

        设置向前参考帧码率控制 

- error-resilient

    允许错误弹性
- VP9-specific options

    lossless

        允许lossless（无损）模式 
    tile-columns

        设置采用的tile columns数，**注意**这里参数是log2(tile_columns)值，例如 8 tile columns要设置 tile-columns 选项值为3. 
    tile-rows

        设置采用的tile rows数， **注意**这里参数是log2(tile_rows). 例如 4 tile rows要设置 tile-rows 选项为2. 
    frame-parallel

        允许并行可译特性
    aq-mode

        设置自适应量化模式： (0:关闭 (默认), 1: 方差 2: 复合, 3: 循环刷新). 


### libwebp ###
WebP图片编码封装

liebwebp是google提高的对于WebP图像格式的编码器，它提供任意有损/无损编码模式。有损图像本质上是对VP8框架的封装。无损图像由google单独编码器支持。

#### libwebp 像素格式 ####
当前libwebp只支持YUV420的有损图像和RGB无损。两种模式都支持透明通道。因为API限制了进行RGB有损和YUV420无损编码时像素格式会自动转换使用libwebp库中要求的格式（暨无损用RGB，有损用YUV420）。所以这样做无意义，只是提供了接口。

#### libwebp选项 ####


- -lossless boolean

    允许/禁止无损编码，默认为0（禁止）
- -compression_level integer

    对于有损，设置质量/速度比，高的值表示获取高质量（同样尺寸）需要更多编码成本（时间）。对于无损，是尺寸/速度比，高的值意味要获取小的尺寸需要更多的成本。更具体的说，就是它控制了额外算法和压缩工具的使用，这些工具的组合使用将影响编码质量/效率。它映射到libwebp选项，有效范围是0-6，默认为4
- -qscale float

    对于有损编码，控制品质，范围0-100。对于无损编码，控制资源和时间花费在压缩更多。默认值为75.**注意**使用livavcodec时它对应于`global_quality`*`FF_QP2LAMBDA`.
- -preset type

    选取预置选项。提供一些常规可用设置：

    none

        不采用预置 
    default

        默认预置 
    picture

        数码图片，例如人像拍摄、室内拍摄、
    photo

        室外图像，自然光 lighting 
    drawing

        手绘或者画线，具有高对比度的细节
    icon

        小尺寸彩色图像 
    text

        文本之类的 

### libx264,libx264rgb ###
x264 H.264/MPEG-4 AVC 编码器封装

编译需要头和库文件，还需要利用`--enable-libx264`在配置中允许

libx264提供一些令人印象深刻的特性，包括8x8和4x4自适应空间变化，自适应B帧，CAVLC/CABAC 熵编码，交织（MBAFF），无损模式，物理优化细节保留（自适应量化、psy-RD，psy-trellis）等等

大多数libx264编码器选项均是映射值ffmpeg全局编码选项，仅有少量的是私有的，他们通过libx264中函数`x264_param_parse`, `x264opts`和`x264-params`提供的单个选项或`key=value`序列的多个选项

参考[ http://www.videolan.org/developers/x264.html]( http://www.videolan.org/developers/x264.html)以了解更多x264项目内容。

libx264rgb和libx264类似，只是一个是编码RGB像素格式，一个是针对YUV像素格式的。

#### 支持的像素格式 ####
x264支持8 到 10 bit的颜色空间。确切的颜色深度在x264配置时设置，在一个特定编译版本的FFmpeg中只支持一种颜色深度，换句话就是不同位深需要多个版本的ffmpeg x264.

#### libx264 libx264rgb 选项 ####
下面的选项被libx264（libx264rgb）封装支持，所有的等效 `x264-XXX`形式的选项和值都列在括号中。

这里只列出了需要特别说明或者私有的选项，其他选项参考[10 编码选项]部分。

为了更多的了解关于libx264的选项，可以使用`x264 --full-help`（需要x264命令行工具）或者参考`libx264`文档。


- b (bitrate)

    设置码率，单位bits/s，**注意**FFmpeg的码率单位是bits/s,而x264中码率单位是kilobits/s.
- bf (bframes)
- g (keyint)
- qmin (qpmin)

    最小量化尺度
- qmax (qpmax)

    最大量化尺度
- qdiff (qpstep)

    量化尺度最大差值
- qblur (qblur)

    模糊量化曲线
- qcomp (qcomp)

    量化曲线压缩因子
- refs (ref)

    每一帧可以使用参考帧数，范围0-16.
- sc_threshold (scenecut)

    设置场景变化检测阈值
- trellis (trellis)

    执行网格量化以提高效率。默认情况下启用。
- nr (nr)
- me_range (merange)

    像素运动最大搜索范围
- me_method (me)

    设置运动估计方法。按速度递减顺序可能值：

    ‘dia (dia)’
    ‘epzs (dia)’

        半径为1菱形搜索 (fastest). ‘epzs’是‘dia’的别名 
    ‘hex (hex)’

        半径为2的正六边形搜索。
    ‘umh (umh)’

        多层次六边形搜索。 
    ‘esa (esa)’

        穷举搜索。 
    ‘tesa (tesa)’

        Hadamard(阿达玛)穷举搜索（最慢）。 

- subq (subme)

    亚像素运动估计方法。
- b_strategy (b-adapt)

    自适应B帧布局决策算法。仅第一次使用。
- keyint_min (min-keyint)

    最小 GOP 尺寸
- coder

    设置熵编码器，可能值:

    ‘ac’

        允许CABAC.
    ‘vlc’

        允许CAVLC而且禁止 CABAC. 它类似于x264中的`--no-cabac` 

- cmp

    设置全像素运动估计比较算法，可能值:

    ‘chroma’

        允许chroma
    ‘sad’

        忽略chroma，其等效于 x264中的`--no-chroma-me` 

- threads (threads)

    编码线程数
- thread_type

    设置多线程技术，可能值:

    ‘slice’

        切片多线程，它等效于x264中的`--sliced-threads` 
    ‘frame’

        基于帧的多线程 

- flags

    设置编码标志，它和`-cgop`配合可以用来关闭GOP或者打开GOP，类似于x264中的`--open-gop`
- rc_init_occupancy (vbv-init)
- preset (preset)

    设置编码预置
- tune (tune)

    设置编码参数整定
- profile (profile)

    设置配置文件的限制。
- fastfirstpass

    参数为1则当第一次编码（pass1）允许快速设置，参数为0，表示禁止快速设置（等效于x264的`--slow-firstpass`）
- crf (crf)

    设为质量恒定模式（类VBR）
- crf_max (crf-max)

    CRF模式下，防止VBV降低质量超越的阀值
- qp (qp)

    设定量化率控制方法参数。
- aq-mode (aq-mode)

    设置AQ方法，可能值

    ‘none (0)’

        禁止.
    ‘variance (1)’

        方差AQ (复杂蒙版).
    ‘autovariance (2)’

        自动方差AQ (实验). 

- aq-strength (aq-strength)

    设置AQ强度，减少阻塞平面和纹理区域模糊。
- psy

    为1表示使用视觉优化。为0则禁用（等效 x264的`--no-psy`）
- psy-rd (psy-rd)

    在psy-rd：psy-trellis中设置视觉优化强度
- rc-lookahead (rc-lookahead)

    设置向前预测参考帧数.
- weightb

    为1设置帧加权预测，否则为0表示禁止（等效于x264的`--no-weightb`）
- weightp (weightp)

    设置P帧加权预测法，可能值:

    ‘none (0)’

        禁止 
    ‘simple (1)’

        使用加权参考 
    ‘smart (2)’

        使加权文献和重复 

- ssim (ssim)

    允许在编码结束后技术输出SSIM
- intra-refresh (intra-refresh)

    为1表示使用周期内刷新代替IDR帧设置
- avcintra-class (class)

    配置编码器生成AVC-Intra，有效值50，100，200
- bluray-compat (bluray-compat)

    配置兼容蓝光标准，是 "bluray-compat=1 force-cfr=1"的简写
- b-bias (b-bias)

    设置B帧如何被影响
- b-pyramid (b-pyramid)

    设置保持一些B帧作为参考集的方法，允许值:

    ‘none (none)’

        禁用. 
    ‘strict (strict)’

        严格的分层金字塔 
    ‘normal (normal)’

        Non-strict (非蓝光兼容). 

- mixed-refs

    为1表示每个分区使用一个参考，而不是每个宏块一个参考，否则为0，其等效于x264的 `--no-mixed-refs`
- 8x8dct

    为1指采用自适应空间变换矩阵大小 (8x8变换) ，否则为0，等效于x264的`--no-8x8dct`
- fast-pskip

    为1表示早期跳过检查。等效于x264的`--no-fast-pskip`
- aud (aud)

    为1启用访问单元分隔设置
- mbtree

    为1表示允许使用宏块树，否则（为0）等效于x264的`--no-mbtree` 
- deblock (deblock)

    设置环路滤波参数，参数型为alpha:beta
- cplxblur (cplxblur)

    QP波动减少（压缩前曲线压缩）
- partitions (partitions)

    设置分区规格，参考后面逗号分隔的列表，可能值有:

    ‘p8x8’

        8x8 P帧 分区 
    ‘p4x4’

        4x4 P帧 分区 . 
    ‘b8x8’

        4x4 B帧分区 
    ‘i8x8’

        8x8 I帧分区. 
    ‘i4x4’

        4x4 I帧分区 (‘p4x4’的前提是‘p8x8’也被设置，允许‘i8x8’ 则需要设置了8x8dct被允许) 
    ‘none (none)’

        不考虑分区 
    ‘all (all)’

        考虑所有可能分区 

- direct-pred (direct)

    设置直接MV预测模式，可能值:

    ‘none (none)’

        禁止MV预测 
    ‘spatial (spatial)’

        使空间预测
    ‘temporal (temporal)’

        使时间的预测
    ‘auto (auto)’

        自动识别 

- slice-max-size (slice-max-size)

    设置每个分片的字节大小限制，单位字节，如果不设置但RTP载荷设置了就使用RTP载荷
- stats (stats)

    设置多次编码的文件名称
- nal-hrd (nal-hrd)

    设置HRD信息信号 (要求vbv-bufsize被设置). 可能值:

    ‘none (none)’

        禁用HRD信息信号
    ‘vbr (vbr)’

        可变比特率
    ‘cbr (cbr)’

        固定比特率 (MP4容器不允许). 

- x264opts (N.A.)

    设置任意的x264选项，参看x264 --fullhelp 以获取列表

    参数是一个由':'分隔的`key=value`序列。对于`filter`和`psy-rd`选项，也是有":"被','代替作为分隔符。

    例如，要指定使用libx264编码：

    ffmpeg -i foo.mpg -vcodec libx264 -x264opts keyint=123:min-keyint=20 -an out.mkv

- x264-params (N.A.)

    使用 : 分隔的 key=value 参数覆盖x264配置，

    这个选项类似x264opts，但其兼容Libav

    例如:

    ffmpeg -i INPUT -c:v libx264 -x264-params level=30:bframes=0:weightp=0:\
    cabac=0:ref=1:vbv-maxrate=768:vbv-bufsize=2000:analyse=all:me=umh:\
    no-fast-pskip=1:subq=6:8x8dct=0:trellis=0 OUTPUT

此外编码`ffpresets`还支持一些通用的选项，可以参考前述[ 预置 ]相关文档。

### libx265 ###
x265 H.265/HEVC 编码器封装

编译需要头和库文件，还需要利用`--enable-libx265`在配置中允许

#### libx265选项 ####

- preset

    设置x265预置
- tune

    设置x265可调参数
- x265-params

    使用':'分隔的`key=value`列表进行选项设置，参考 x265 --help 获取支持的选项

    例如采用libx265,并利用-x265-params进行选项设置:

    ffmpeg -i input -c:v libx265 -x265-params crf=26:psy-rd=1 output.mp4

### libxvid ###
Xvid MPEG-4 Part 2 封装

编译需要livxvidcore头和library库文件，还需要利用`--enable-libxvid --enable-gpl`在配置中允许

当前原生的`mpeg4`编码器支持MPEG-4 Part 2格式，所以不一定需要这个库了。

#### libxvid选项 ####
下面选项是libxvid封装支持的选项，其中部分只列出，而没有文档介绍是因为其同[10 编码选项]中通用选项一致，其它没有列出的通用选项则在库中无效。

- b
- g
- qmin
- qmax
- mpeg_quant
- threads
- bf
- b_qfactor
- b_qoffset
- flags

    设置编码标志，可能值:

    ‘mv4’

        对宏块使用4个运动检测
    ‘aic’

        允许高品质AC预测
    ‘gray’

        只编码灰度
    ‘gmc’

        全局运动补偿(GMC).
    ‘qpel’

        1/4像素运动补偿
    ‘cgop’

        关闭GOP.
    ‘global_header’

        在每个关键帧放置全局头extradata

- trellis
- me_method

    设置运动估计方法.按速度降低，质量增加排列的可能值:

    ‘zero’

        不使用运动估计方法 (默认).
    ‘phods’
    ‘x1’
    ‘log’

        启用16x16块和16x16块半像素细化进行菱形区域搜索， ‘x1’和‘log’是‘phods’别名
    ‘epzs’

        允许前述所有值，再加上8x8菱形区域搜索，8x8半像素细化，并在色度平面进行运动估计
    ‘full’

        允许所有的 16x16和8x8 区域搜索 

- mbd

    设置宏块选择算法，依质量提高的可能值:

    ‘simple’

        使用宏块比较函数算法 (默认).
    ‘bits’

        允许16x16块半像素和1/4像素细化失真估计
    ‘rd’

        允许上述所有可能值，再加上8x8块半像素和1/4像素细化失真估计，并采用方形图案失真估计进行搜索。

- lumi_aq

    为1允许lumi遮蔽自适应量化，默认为0 (禁止).
- variance_aq

    为1允许方差的自适应量化,默认为0 (禁止).

    如果结合lumi_aq,由此产生的质量不会比任何一个单独规定。换句话说，所得到的质量会差于单独使用任何一个选项的效果。
- ssim

    设置结构信息（SSIM）显示方法。可能的值：

    ‘off’

        禁止SSIM信息
    ‘avg’

        在编码后输出平均SSIM。格式为：

        Average SSIM: %f

        对那些不熟悉C的的用户，f表示浮点数或者小数 (例如 0.939232)
    ‘frame’

        在编码过程中输出每帧SSIM，并且在编码结束后输出平均SSIM，每帧信息格式为：

               SSIM: avg: %1.3f min: %1.3f max: %1.3f

        对那些不熟悉C的的用户，%1.3f表示3位小数的浮点数(例如0.932).

- ssim_acc

    设置SSIM精度。可用的选项参数是在0-4范围的整数，而0给出了最准确的结果和计算速度最快的4。

### mpeg2 ###
MPEG-2编码器

#### mpeg2选项 ####

- seq_disp_ext integer

    指定是否写一个 sequence_display_extension到输出

    -1
    auto

        自动检测是否写，是默认值，如果数据被写入不同于默认或指定的值则判断是否写 
    0
    never

        从不写 
    1
    always

        一直写 

### png ###
png图像编码器

#### png选项 ####

dpi integer

    设置像素的物理密度，每英寸点数，没有默认设置
dpm integer

   设置像素的物理密度，每米点数，没有默认设置

### ProRes ###
Apple ProRes编码器

FFmpeg包含2种ProRes编码器，prores-aw和prores-ks。它们可以由`-vcodec`选项指定

#### prores-ks私有选项 ####

- profile integer

    选择ProRes属性（预置）配置来编码，可能值：

    ‘proxy’
    ‘lt’
    ‘standard’
    ‘hq’
    ‘4444’

- quant_mat integer

    选择的量化矩阵,可能值：

    ‘auto’
    ‘default’
    ‘proxy’
    ‘lt’
    ‘standard’
    ‘hq’

    如果选择auto, 匹配属性的量化矩阵会被选中，如果没有设置，则选择最高质量的量化矩阵
- bits_per_mb integer

    分配的宏块位，不同的属性在200-2400间，最大值为8000
- mbs_per_slice integer

    每个切片中宏块数（1-8），默认为8，几乎是所有情况下最好值
- vendor string

    重写4字节的供应商ID。例如apl0这个自定义供应商ID会被认为是由苹果编码器产生。
- alpha_bits integer

    指定alpha分量的比特数。可能的值是0，8和16。用0禁用alpha平面编码

#### 速度考虑 ####
在默认操作模式下，编码器以高质量为目的（即在不产生超过要求的帧数据限定下，使输出质量尽可能好）。这种情况下帧内很多小的细节是很难压缩的，编码器将花更多的时间为每个片寻找合适的量化。

所以设置更高的bits_per_mb限额将提高速度。

要获取最快的编码速度，则设置qscale参数（4为推荐值）和不设置帧数据大小限制。