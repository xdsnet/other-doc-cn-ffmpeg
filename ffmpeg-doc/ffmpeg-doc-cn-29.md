## 放缩选项 ##
视频支持下面的一些选项。

选项可以在ffmpeg工具集中采用`-option value`的形式进行设置，或者在`aresample`滤镜中以`option=value`形式设置，也可以通过`libavutil/opt.h`的API或明确设置在`SwrContext`选项中。

- sws_flags

    Set the scaler flags. This is also used to set the scaling algorithm. Only a single algorithm should be selected.

    It accepts the following values:

    - ‘fast_bilinear’

        Select fast bilinear scaling algorithm.
    - ‘bilinear’

        Select bilinear scaling algorithm.
    - ‘bicubic’

        Select bicubic scaling algorithm.
    - ‘experimental’

        Select experimental scaling algorithm.
    - ‘neighbor’

        Select nearest neighbor rescaling algorithm.
    - ‘area’

        Select averaging area rescaling algorithm.
    - ‘bicublin’

        Select bicubic scaling algorithm for the luma component, bilinear for chroma components.
    - ‘gauss’

        Select Gaussian rescaling algorithm.
    - ‘sinc’

        Select sinc rescaling algorithm.
    - ‘lanczos’

        Select lanczos rescaling algorithm.
    - ‘spline’

        Select natural bicubic spline rescaling algorithm.
    - ‘print_info’

        Enable printing/debug logging.
    - ‘accurate_rnd’

        Enable accurate rounding.
    - ‘full_chroma_int’

        Enable full chroma interpolation.
    - ‘full_chroma_inp’

        Select full chroma input.
    - ‘bitexact’

        Enable bitexact output. 

- srcw

    Set source width.
- srch

    Set source height.
- dstw

    Set destination width.
- dsth

    Set destination height.
- src_format

    Set source pixel format (must be expressed as an integer).
- dst_format

    Set destination pixel format (must be expressed as an integer).
- src_range

    Select source range.
- dst_range

    Select destination range.
- param0, param1

    Set scaling algorithm parameters. The specified values are specific of some scaling algorithms and ignored by others. The specified values are floating point number values.
- sws_dither

    Set the dithering algorithm. Accepts one of the following values. Default value is ‘auto’.

    - ‘auto’

        automatic choice
    - ‘none’

        no dithering
    - ‘bayer’

        bayer dither
    - ‘ed’

        error diffusion dither
    - ‘a_dither’

        arithmetic dither, based using addition
    - ‘x_dither’

        arithmetic dither, based using xor (more random/less apparent patterning that a_dither).

