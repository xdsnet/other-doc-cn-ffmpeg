## 28 重采样选项
音频重采样支持下面一些选项。

选项可以在ffmpeg工具集中采用`-option value`的形式进行设置，或者在`aresample`滤镜中以`option=value`形式设置，也可以通过`libavutil/opt.h`的API或明确设置在`SwrContext`选项中。

- ich, in_channel_count

    设置输入通道序数。默认为0。如果`in_channel_layout`被设置，则并不强制要求设置这个值。
- och, out_channel_count

    设置输出通道序数，默认为0。如果`out_channel_layout`被设置，则并不强制要求设置这个值。
- uch, used_channel_count

    设置使用的输入通道序数，默认为0。这个选项仅用于指定重新映射
- isr, in_sample_rate

    设置输入采样率，默认为0
- osr, out_sample_rate

    设置输出采样率，默认为0
- isf, in_sample_fmt

    设置输入采样格式，默认为none
- osf, out_sample_fmt

    设置输出采样格式，默认为none
- tsf, internal_sample_fmt

    设置内部采样格式。默认值为none，当不显式设置时它会自动被选中
- icl, in_channel_layout
- ocl, out_channel_layout

    设置输入/输出通道布局

    参考FFmepg工具集(ffmpeg-utils)通道布局章节手册（ffmpeg-utils(1) ）以了解要求语法。
- clev, center_mix_level

    设置中心混合水平。这是个用分贝（deciBel）表示的值，范围在 [-32,32].
- slev, surround_mix_level

    设置环绕混合水平。这是个用分贝（deciBel）表示的值，范围在 [-32,32].
- lfe_mix_level

    设置LFE混合成非LFE水平。它表示输入为LFE，但输出没有LFE的处理。这是个用分贝（deciBel）表示的值，范围在 [-32,32]
- rmvol, rematrix_volume

    设置`rematrix`值（声音的放缩处理），默认1.0.
- rematrix_maxval

    设置`rematrix`处理的最大值。它用来防止声音放缩为1.0时被裁剪
- flags, swr_flags

    设置为采用转换的标志，默认为0.

    它支持下面的标志:

    - res

        强制重采样。这个标志将强制重采样，即使输入和输出的采样频率一样。 

- dither_scale

    设置抖动率。默认1.
	**注**关于`dither`：Dither是数字音乐处理上非常神奇的技巧，目的是通过用少数的Bit达到与较多Bit同样的听觉效果，方法是在最后一个Bit(LSB)上动“手脚”。例如用16Bit记录听起来好似20Bit的信息，听到原先16Bit无法记录的微小信息。举例来说，现在我有个20Bit的采样信息，现在想将其存为16Bit的信息格式，最简单的转换方式就是直接把后面4个Bit去掉，但是这样就失去用20Bit录音/混音的意义。比较技巧性的方法是在第17~20Bit中加入一些噪音，这段噪音就叫做Dither。这些噪音加入后，可能会进位而改变第16个Bit的信息，然后我们再把最后4个Bit删掉，这个过程我们称为redithering，用意是让后面4个Bit的数据线性地反映在第16个Bit上。由于人耳具有轻易将噪音与乐音分离的能力，所以虽然我们加入了噪音，实际上我们却听到了更多音乐的细节。 
- dither_method

    设置抖动方法，默认 0.

    支持的值:

    - ‘rectangular’

        选择rectangular抖动方法 
    - ‘triangular’

        选择triangular抖动方法 
    - ‘triangular_hp’

        对高频采用triangular抖动方法 
    - ‘lipshitz’

        选择lipshitz噪音塑造抖动方法 
    - ‘shibata’

        选择shibata噪音塑造抖动方法 
    - ‘low_shibata’

        选择低shibata噪音塑造抖动方法
    - ‘high_shibata’

        选择高shibata噪音塑造抖动方法 
    - ‘f_weighted’

        选择f-weighted噪音塑造抖动方法 
    - ‘modified_e_weighted’

        选择modified-e-weighted噪音塑造抖动方法 
    - ‘improved_e_weighted’

        选择improved-e-weighted噪音塑造抖动方法

- resampler

    设置重采样技术。默认为 swr.

    支持的值:

    - ‘swr’

        选择原生SW重采样; 滤镜选项`precision`和`cheby`在这种情况下不可用 
    - ‘soxr’

        选择SoX重采样（如果可用）。 `compensation`, 和滤镜选项`filter_size`, `phase_shift`, `filter_type` 和 `kaiser_beta`在这种情况下不可用

- filter_size

    仅对`swr`有效，设置重采样滤镜尺寸，默认为32
- phase_shift

    仅对`swr`有效，设置重采样相移，默认为10，范围 [0,30].
- linear_interp

    如果为1则采用线性插值。默认为0
- cutoff

    设置截止频率 (swr: 6dB ; soxr: 0dB ) 比例;必须是一个0到1的浮点数。默认为0.97（对swr模式）和0.91（对soxr）（这意味着采样率为44100，就可以保存整个20KHz的频带）
- precision

    仅对soxr有效,重采样信号精度位计算。默认值为20（配合合适的抖动，适合一个16bit位深的目标），适合于高品质的`SoX`，可以设置为28以获得非常高品质的`SoX`
- cheby

    仅对soxr有效, 选择通频带滚边切除(Chebyshev)和高精度近视’无理数’比率。默认为0
- async

    仅对`swr`有效, 为1则可以采用伸展、挤压、填充和修剪等方法实现同步，默认为0，表示没有任何补偿用于同步音频时间戳
- first_pts

    仅对`swr`有效,假定第一个`pts`（这里表示数据包时间戳）是这个值。时间单位是1/采样率。 这允许对流填充/切边。默认，没有假设第一帧预期时间，所以没有填充或者切边。例如这个值设置为0，表示如果有编码器延迟，音频流与视频流本身是同步的，则先静音直到二者同步开始
- min_comp

    仅对`swr`有效, 设置时间戳与音频数据最小差值，单位秒，以此来触发拉伸/压缩/填充或调整的数据匹配的时间戳。默认值为(min_comp = FLT_MAX)，表示禁用拉伸/压缩/填充或调整的数据匹配到时间戳
- min_hard_comp

    仅对`swr`有效, 设置时间戳与音频数据最小差值，单位秒，以此来触增加/抛弃以匹配时间戳。它设置了一个阀值来选择有效的硬（裁剪/填充）补偿和软（压缩/拉伸）补偿.**注意**补偿默认是禁止的。而这个选项默认值是0.1.
- comp_duration

    仅对`swr`有效, 设置拉伸/压缩数据匹配的时间戳的持续时间。 必须是非负双精度浮点数，默认为1.0.
- max_soft_comp

    仅对`swr`有效, 设置在拉伸/压缩以匹配时间戳的最大系数。必须是非负双精度浮点数，默认为0.
- matrix_encoding

    选择立体声编码矩阵

    接收如下值:

    - ‘none’

        选择none 
    - ‘dolby’

        选择Dolby 
    - ‘dplii’

        选择Dolby Pro Logic II 

    默认为none.
- filter_type

    仅对`swr`有效,选择重采样滤镜类型，仅用于重采样操作。

    接收如下值:

    - ‘cubic’

        选择cubic 
    - ‘blackman_nuttall’

        选择Blackman Nuttall Windowed Sinc 
    - ‘kaiser’

        选择Kaiser Windowed Sinc 

- kaiser_beta

    仅对`swr`有效, 设置Kaiser Window Beta值，必须是整数，范围[2,16],默认为9.
- output_sample_bits

    仅对`swr`有效,采用输出采样抖动。必须为整数，范围 [0,64], 默认为0，表示不采用
