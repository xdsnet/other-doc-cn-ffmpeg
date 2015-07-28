## 29 放缩选项
视频支持下面的一些选项。

选项可以在ffmpeg工具集中采用`-option value`的形式进行设置，或者在`aresample`滤镜中以`option=value`形式设置，也可以通过`libavutil/opt.h`的API或明确设置在`SwrContext`选项中。

- sws_flags

    设置放缩标志。也用于设置放缩算法，仅有一个算法能被选中。

    接受如下值:

    - ‘fast_bilinear’

        快速双线性缩放算法
    - ‘bilinear’

        双线性缩放算法
    - ‘bicubic’

        双三次的缩放算法.
    - ‘experimental’

        实验缩放算法.
    - ‘neighbor’

        近邻取样缩放算法
    - ‘area’

        平均区域尺度缩放算法.
    - ‘bicublin’

        对亮度采用双三次的缩放算法，对色度采用双线性缩放算法
    - ‘gauss’

        高斯缩放算法
    - ‘sinc’

        辛格缩放算法
    - ‘lanczos’

        兰索斯分块缩放算法
    - ‘spline’

        自然双三次的样条插值缩放算法
    - ‘print_info’

        允许输出/调试日志
    - ‘accurate_rnd’

        允许精度舍入
    - ‘full_chroma_int’

        允许完整的色度插值
    - ‘full_chroma_inp’

        选择完整的浓度输入
    - ‘bitexact’

        允许bitexact（位精确算法 ）输出 

- srcw

    设置源宽度
- srch

    设置源高度
- dstw

    设置目标宽度
- dsth

    设置目标高度
- src_format

    设置源像素格式 (必须表示为整数).
- dst_format

    设置目标像素格式 (必须表示为整数).
- src_range

    选择源区域范围
- dst_range

    选择目标区域范围
- param0, param1

    设置缩放算法参数。指定的值是特定缩放算法适用的可能被别的算法忽略。值为浮点数
- sws_dither

    设置抖动算法。接收如下值，默认为 ‘auto’.

    - ‘auto’

        自动选择
    - ‘none’

        没有抖动
    - ‘bayer’

        bayer抖动
    - ‘ed’

        error diffusion（误差扩散）抖动
    - ‘a_dither’

        arithmetic（算术）抖动,基于加法
    - ‘x_dither’

        arithmetic（算术）抖动, 基于xor（异或） (比`a_dither`有更多的随机性/更少的模式化).

