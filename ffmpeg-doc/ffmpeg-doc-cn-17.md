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
