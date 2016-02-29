## 20 格式选项
`libavformat`库提供一些常规的全局选项，它们都可被混合器/分离器设置。一些混合器/分离器还支持附加的私有选项，这些都在其组件处介绍。

ffmpeg工具中选项通过特定的`-option value`进行设置，或者通过`AVFormatContext`选项设置，或者通过`libavutil/opt.h`中的API设置

下面是一些被支持的选项：

- avioflags flags (input/output)

    可能值:

    ‘direct’

        减少缓冲 

- probesize integer (input)

    设置probing（探测）尺寸，单位bytes, 即加载数据的大小来分析得到的数据流信息。较高的值会使检测的更多信息，分散到流，但会增加延迟。必须有一个不小于32的整数。它的默认值是5000000。
- packetsize integer (output)

    设置包尺寸
- fflags flags (input/output)

    设置格式标志

    可能值:

    ‘ignidx’

        忽略索引 
    ‘fastseek’

        允许快速定位，但只有个别格式有效 
    ‘genpts’

        常规PTS. 
    ‘nofillin’

        不填补缺失值，则可以精确计算 
    ‘noparse’

        禁止AVParsers，它要和 +nofillin联用. 
    ‘igndts’

        忽略DTS. 
    ‘discardcorrupt’

        丢弃损坏的帧 
    ‘sortdts’

        尝试按DTS输出去交织
    ‘keepside’

        不合并侧数据 
    ‘latm’

        允许 RTP MP4A-LATM有效载荷 
    ‘nobuffer’

        通过可选缓冲减少延迟 
    ‘bitexact’

        仅写平台、编译和时间独立的数据。这保证了文件和数据校验和平台之间的可重复性和匹配，它主要用在回归测试

- seek2any integer (input)

    为1表示分离器中允许定位到非关键帧，默认为0，表示禁止
- analyzeduration integer (input)

    指定多少微秒分析探头输入。较高的值会使检测更准确的信息，但会增加延迟。默认为5000000毫秒= 5秒。
- cryptokey hexadecimal string (input)

    设置解密密钥
- indexmem integer (input)

    设置最大内存使用时间戳索引（每流）。
- rtbufsize integer (input)

    设置用于缓冲实时帧的最大内存。
- fdebug flags (input/output)

    打印特定调试信息

    可能值:

    ‘ts’

- max_delay integer (input/output)

    设置分离或者混合时的最大延迟，单位microseconds.
- fpsprobesize integer (input)

    设置用于探测fps的帧数
- audio_preload integer (output)

    设置音频包在交错前时间（微秒）量
- chunk_duration integer (output)

    设置块时间，单位microseconds
- chunk_size integer (output)

    设置块尺寸，单位bytes
- err_detect, f_err_detect flags (input)

    设置错误描述标志，f_err_detect已过时，仅用于ffmpeg工具
    可能值:

    ‘crccheck’

        验证嵌入的CRCs. 
    ‘bitstream’

        比特流的规范偏差检测。 
    ‘buffer’

        不当的码流长度检测。 
    ‘explode’

        在检测到错误时中断 
    ‘careful’

        违反规范但并不作为错误
    ‘compliant’

        违反规范都作为错误 
    ‘aggressive’

        一个智能编码器不会作为错误的

- max_interleave_delta integer (output)

    设置最大缓冲时间交错。时间是用微秒，默认值为1000000（1秒）。

    这将确保流交错（混合）的正确。libavformat会在写输出时确保每个流至少有1个数据包。但当一些流十分稀疏（即有很大间隔才需要一个流数据）这将导致过度的缓冲

    这里就指定了一个混合在一起的序列中其中流时间的差值不超过设定数，而不管是否已经在数据序列中已经有所有流信息了

    如果设置为0，libavformat将缓冲数据包，直到每个流至少有一个数据包，而不管数据包间时间差值
- use_wallclock_as_timestamps integer (input)

    使用时钟作为时间戳。
- avoid_negative_ts integer (output)

    可能值:

    ‘make_non_negative’

        转移的时间戳，使其非负。也请注意，这仅影响领导负的时间戳，而不非单调负时间戳。
    ‘make_zero’

        转移的时间戳，使其从0开始 
    ‘auto (default)’

        目标格式要求时转移时间戳
    ‘disabled’

        禁止转换时间戳 

    当时间戳转换允许时，所有基于时间戳的流（音频、视频和字幕）都会被转换，以保证相对时间不变（时间基准统一）
- skip_initial_bytes integer (input)

    设置为1则读取头和帧前跳过数字节（对解析帧无关），默认为0
- correct_ts_overflow integer (input)

    为1则允许单时间溢出，默认为1
- flush_packets integer (output)

    为1则每个包都清除底层I/O流。默认为1，可以减少延迟，为0则稍微增加延迟，但可以改善某些情况下的性能
- output_ts_offset offset (output)

    设置输出时间偏移

    `offset`必须是时间持续描述值，参考`ffmpeg-utils`中关于时间持续的章节（在ffmpeg-utils(1)手册中）

    这个便宜量将在混合时加到时间戳上进行输出

    指定一个正偏移意味着相应的流延迟bt（basetime 基准）时间，默认值为0，表示没有偏移
- format_whitelist list (input)

    "," 分隔的一个列表用于分离器，默认是所有的都允许
- dump_separator string (input)

    在命令行中指定分离器流参数的域，例如用换行和缩进的独立域:

    ffprobe -dump_separator "
                              "  -i ~/videos/	matrixbench_mpeg2.mpg

### 格式流指定（说明） ###
格式流指定允许选择1个或者多个流匹配特定的属性

可能的流指定形式：

- stream_index

    有索引匹配流
- stream_type[:stream_index]

    联合流类型和流索引指定流，流类型有: ’v’ 对应视频 , ’a’对应音频, ’s’ 对应字幕, ’d’对应数据和’t’ 对应附加. 如果流索引` stream_index`被设置，则匹配指定类型的流索引，否则匹配所有该类型流
- p:program_id[:stream_index]

    如果流索引被设置，其匹配程序Id匹配的处理中索引的流，否则匹配该程序中所有流
- #stream_id

    通过格式指定ID匹配流


声明在`libavformat/avformat.h`中的avformat_match_stream_specifier() 用来精确的完成格式流指定（说明）