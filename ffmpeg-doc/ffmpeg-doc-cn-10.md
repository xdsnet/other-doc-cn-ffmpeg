## 编码选项 ##
libavcodec提供一些通用的全局选项设置，可在所有的编码器和解码器起效。另外每个编解码器可以支持所谓的私有化设置，以满足特定的编解码要求。

有时，一个全局选项会影响到特定的编解码器，而对其它编解码产生不良影响或者会不被识别，所以你需要了解这些影响编解码选项的具体意义，了解那些只对特定编码或者解码有效的选项。

这些选项大多可以`-option value`的格式在ffmpeg工具中指定，其中`-option`是选项名，`value`是要设置的选项参数值，个别是利用`AVCodecContext`选项进行额外配置，还有极个别的使用定义在`libavutil/opt.h`中的API在程序过程中配置使用。

下面是这些选项的列表(括号中表示选项有效的状态，可能是decoding-解码时,encoding—编码时，audio-音频，video-视频，subtitles-字幕 以及特定的编码名称，如果mpeg4):

b integer (encoding,audio,video)

    设置码率，单位为bits/s。默认200K。
ab integer (encoding,audio)

    设置音频码率，单位bits/s。默认128K。
bt integer (encoding,video)

    设置视频码率偏离公差（video bitrate tolerance），单位bits/s。对于1次编码模式中码率控制公差指愿意偏离目标（码率）的平均码率值，故不能由此确定最小/最大码率，即不能假定最小/最大码率是目标码率+-码率偏离公差。太低的偏离公差影响编码后的质量。
flags flags (decoding/encoding,audio,video,subtitles)

    设置常见的标志

    可能值有：

    ‘mv4’

        使用4路监控宏块运动矢量 (mpeg4). 
    ‘qpel’

        使用1/4像素补偿 
    ‘loop’

        使用循环过滤 
    ‘qscale’

        使用固定放缩qscale 
    ‘gmc’

        使用gmc. 
    ‘mv0’

        一直假定mb中mv=<0,0>。
    ‘input_preserved’
    ‘pass1’

        在第1次编码中使用内部控制段模式。 
    ‘pass2’

        在第2次编码中使用内部控制段
    ‘gray’

        只有decode/encode灰度 
    ‘emu_edge’

        不描绘边缘 
    ‘psnr’

        在设置了错误变量时仍然继续编码
    ‘truncated’
    ‘naq’

        规范的自适应量化 
    ‘ildct’

        使用交错DCT. 
    ‘low_delay’

        强制减少延迟
    ‘global_header’

        在每个关键帧的扩展数据中添加全局头 
    ‘bitexact’

        只写的关于平台、编译创建以及时间无关（platform-, build- 和time-independent） 的数据 (除了(I)DCT)。这确保了文件和数据校验和平台之间的可重复性和匹配，它主要用于回归测试。 
    ‘aic’

        H263高级帧内编码/mpeg4打开ac预测 
    ‘cbp’

        已失效/过期，使用mpegvideo提供的私有选项 
    ‘qprd’

        已失效/过期，使用mpegvideo提供的私有选项  
    ‘ilme’

        交错运动估计。
    ‘cgop’

        关闭gop 

me_method integer (encoding,video)

    设置运动估计方法

    可能值有:

    ‘zero’

        0运动估计，即不进行估计 (最快) 
    ‘full’

        full运动估计 (最慢) 
    ‘epzs’

        EPZS运动估计 (默认) 
    ‘esa’

        esa运动估计 (full的别名) 
    ‘tesa’

        tesa运动估计 
    ‘dia’

        dia运动估计(epzs的别名) 
    ‘log’

        log运动估计 
    ‘phods’

        phods运动估计
    ‘x1’

        X1运动估计 
    ‘hex’

        hex运动估计 
    ‘umh’

        umh运动估计
    ‘iter’

        iter运动估计 

extradata_size integer

    设置扩展数据尺寸
time_base rational number

    设置编码的时间基础计量

    它是时间的基本单位(秒)的帧时间戳表示。对于固定FPS内容，这个值应该是`1/frame_rate`，每次时间戳都增加1个单位的时间量。
g integer (encoding,video)

    设置一组图片的数量，默认是12
ar integer (decoding/encoding,audio)

    设置音频采样率（单位Hz)
ac integer (decoding/encoding,audio)

    设置音频通道数
cutoff integer (encoding,audio)

    设置截止带宽
frame_size integer (encoding,audio)

    设置音频帧尺寸

    除了最后一帧音频数据，否则每个音频数据帧都包含`frame_size`设定大小的数据。当编码中`CODEC_CAP_VARIABLE_FRAME_SIZE`设置了，则这个值可能是0，在这种情况下帧的大小是没有限制的。它在一些解码器中显示固定帧大小。
frame_number integer

    设置帧数量
delay integer
qcomp float (encoding,video)

    设置视频量化压缩规模(VBR)。这里用在控制方程的常数。默认rc_eq推荐范围是: 0.0-1.0.
qblur float (encoding,video)

    设置视频量化尺度模糊(VBR).
qmin integer (encoding,video)

    设置最小视频量化尺度(VBR)。 取值范围是-1-69，默认为2
qmax integer (encoding,video)

    设置最大视频量化尺度(VBR)。 取值范围是-1-1024，默认为31
qdiff integer (encoding,video)

    设置量化级之间最大的差异(VBR).
bf integer (encoding,video)

    设置最大B帧间隔

    必须是-1 -16间的数。0表示禁止B帧，如果为-1表示依据编码器进行指定，默认值是0
b_qfactor float (encoding,video)

    设置P帧和B帧之间的qp因子
rc_strategy integer (encoding,video)

    设置码率控制方法
b_strategy integer (encoding,video)

    设置I/P/B帧选择策略.
ps integer (encoding,video)

    设置RTP播放加载数据量（缓冲），单位是字节（bytes）
mv_bits integer  
header_bits integer  
i_tex_bits integer  
p_tex_bits integer  
i_count integer  
p_count integer  
skip_count integer  
misc_bits integer  
frame_bits integer  
codec_tag integer  
bug flags (decoding,video)

    解决不能自动检测/识别编码的错误（bug）

    可能值:

    ‘autodetect’
    ‘old_msmpeg4’

        一些旧lavc处理的msmpeg4v3文件(不能自动检测) 
    ‘xvid_ilace’

        Xvid交错错误 (如果强制为fourcc==XVIX则可自动检测) 
    ‘ump4’

        (fourcc==UMP4可自动检测) 
    ‘no_padding’

        填充错误(自动检测) 
    ‘amv’
    ‘ac_vlc’

        非法vlc错误 (对每个fourcc自动检测) 
    ‘qpel_chroma’
    ‘std_qpel’

        老标准的qpel (对每个fourcc/version自动检测) 
    ‘qpel_chroma2’
    ‘direct_blocksize’

        direct-qpel-blocksize错误 (对每个fourcc/version自动检测) 
    ‘edge’

        edge填充bug (对每个fourcc/version自动检测) 
    ‘hpel_chroma’
    ‘dc_clip’
    ‘ms’

        微软破解解码器上的各种缺陷 
    ‘trunc’

        截断帧

lelim integer (encoding,video)

   设置亮度单系数消除阈值（负值也考虑直流系数）。
celim integer (encoding,video)

    设置消除色度单系数阈值（负值也考虑直流系数）
strict integer (decoding/encoding,audio,video)

    指定如何遵守标准（严格程度）

    可能的值:

    ‘very’

        严格模式，遵守过时的版本规格或者依软件需求的版本规格进行处理
    ‘strict’

        严格模式，无论如何都严格按照规格处理 
    ‘normal’
    ‘unofficial’

        允许非官方扩展
    ‘experimental’

        允许非标准的实验性质解码/编码器（未完成/未测试/不工作）**注意**实验性质的解码工具可能带来安全风险，不要用这类解码器解码不可信输入 

b_qoffset float (encoding,video)

    在P帧和B帧间设置QP（帧间偏移）
err_detect flags (decoding,audio,video)

    设置错误检测标志

    可能值:

    ‘crccheck’

        嵌入CRC验证 
    ‘bitstream’

        比特流规范偏差检测 
    ‘buffer’

        不当码流长度检测 
    ‘explode’

        在出现错误时中止解码 
    ‘ignore_err’

        忽略解码错误继续解码。则对于分析视频内容是十分有用的，这时希望无论如何解码都继续工作。
    ‘careful’

        
        考虑环境支持，一个正确的编码器不能被错误停止

has_b_frames integer
block_align integer
mpeg_quant integer (encoding,video)

    使用MPEG量化代替H.263。
qsquish float (encoding,video)

   保持Qmin和Qmax之间量化器。(0 = clip, 1 = use 还可以利用函数进行微调)。
rc_qmod_amp float (encoding,video)

    设置实验/量化 
rc_qmod_freq integer (encoding,video)

    设置实验/量化调制
rc_override_count integer
rc_eq string (encoding,video)

    速率控制方程组。除了内部标准定义外，可以有以下选择：bits2qp(bits), qp2bits(qp)。还可以利用下面介绍的常数: iTex、pTex、tex ，mv fCode iCount mcVar var isl isB avgQP qComp avgIITex avgPITx avgPPTex avgBPText。
maxrate integer (encoding,audio,video)

    设置最大比特率容差（单位 比特/秒）。要求的缓冲大小被设置。
minrate integer (encoding,audio,video)

    设置最小比特率容差（单位 比特/秒）。通常用于CBR编码，否则无意义
bufsize integer (encoding,audio,video)

    设置控制缓冲区大小（单位bits）
rc_buf_aggressivity float (encoding,video)

    目前无效
i_qfactor float (encoding,video)

    设置P帧和I帧间的QP因子
i_qoffset float (encoding,video)

    设置P帧和I帧间的QP偏移
rc_init_cplx float (encoding,video)

    设置1次编码的初始复杂性
dct integer (encoding,video)

    设置DCT（数字转换）算法

    可能值:

    ‘auto’

        自动选择一个优化质量算法(默认值) 
    ‘fastint’

        偏重速度 
    ‘int’

        精准整数 
    ‘mmx’
    ‘altivec’
    ‘faan’

        浮点AAN DCT 

lumi_mask float (encoding,video)

    压缩高亮
tcplx_mask float (encoding,video)

    设置临时/时间复杂遮蔽/蒙版
scplx_mask float (encoding,video)

    设置空间复杂遮蔽/蒙版
p_mask float (encoding,video)

    设置组间遮蔽
dark_mask float (encoding,video)

    压缩暗区
idct integer (decoding/encoding,video)

    选择实施的IDCT

    可能值:

    ‘auto’
    ‘int’
    ‘simple’
    ‘simplemmx’
    ‘simpleauto’

        自动应用一个兼容的IDCT
    ‘arm’
    ‘altivec’
    ‘sh4’
    ‘simplearm’
    ‘simplearmv5te’
    ‘simplearmv6’
    ‘simpleneon’
    ‘simplealpha’
    ‘ipp’
    ‘xvidmmx’
    ‘faani’

        浮点AAN IDCT 

slice_count integer  
ec flags (decoding,video)

    设置错误隐藏策略

    可能值:

    ‘guess_mvs’

        运动矢量迭代 (MV)搜索(慢/slow) 
    ‘deblock’

        对损坏的MBs使用强壮的去块滤波
    ‘favor_inter’

        有利用从前帧预测而不是当前帧预测 

bits_per_coded_sample integer  
pred integer (encoding,video)

    设置预测方法

    可能值:

    ‘left’
    ‘plane’
    ‘median’

aspect rational number (encoding,video)

    设置样本纵横比
debug flags (decoding/encoding,audio,video,subtitles)

    输出特定调试信息.

    可能值:

    ‘pict’

        图片相关信息 
    ‘rc’

        码率控制 
    ‘bitstream’
    ‘mb_type’

        宏块 (MB)类型 
    ‘qp’

        每个块的量化参数(QP) 
    ‘mv’

        运动矢量 
    ‘dct_coeff’
    ‘skip’
    ‘startcode’
    ‘pts’
    ‘er’

        错误识别 
    ‘mmco’

        内存管理控制操作(H.264) 
    ‘bugs’
    ‘vis_qp’

        量化参数可视化 (QP),即低的QP显示为绿色 
    ‘vis_mb_type’

        块类型可视化 
    ‘buffers’

        图像缓冲区分配 
    ‘thread_ops’

        线程操作 
    ‘nomc’

        跳跃运动补偿 

vismv integer (decoding,video)

    运动矢量可视化 (MVs).

    这个选项是过时的，参考`codecview`滤镜.

    可能值:

    ‘pf’

        p帧前测MVs 
    ‘bf’

        B帧前测MVs 
    ‘bb’

        B帧后测MVs 

cmp integer (encoding,video)

    设置完整图元me压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

subcmp integer (encoding,video)

    设置局部图元me压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

mbcmp integer (encoding,video)

    设置宏块压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

ildctcmp integer (encoding,video)

    设置隔行dct压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

dia_size integer (encoding,video)

    设置运动估计区域类型和尺寸
last_pred integer (encoding,video)

    设置从前帧预测运动量
preme integer (encoding,video)

    设置预运动估计
precmp integer (encoding,video)

    设置预运动估计压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

pre_dia_size integer (encoding,video)

    设置运动估计预测的区域类型和尺寸
subq integer (encoding,video)

    设置子图元运动估计质量
dtg_active_format integer
me_range integer (encoding,video)

    设置运动矢量极限范围 (DivX是1023).
ibias integer (encoding,video)

    设置组内量化偏差
pbias integer (encoding,video)

    设置集间量化偏差
color_table_id integer
global_quality integer (encoding,audio,video)
coder integer (encoding,video)

    可能值:

    ‘vlc’

        可变长编码 / huffman编码 
    ‘ac’

        算术编码 
    ‘raw’

        raw (不进行编码) 
    ‘rle’

        游程长度编码 
    ‘deflate’

        紧缩编码 

context integer (encoding,video)

    设置上下文模型
slice_flags integer
xvmc_acceleration integer
mbd integer (encoding,video)

    设置宏块选择算法 (高质量模式).

    可能值:

    ‘simple’

        使用mbcmp，宏块比较 (默认) 
    ‘bits’

        减少数据量
    ‘rd’

        失真率优化 

stream_codec_tag integer
sc_threshold integer (encoding,video)

    设置场景变化阀值
lmin integer (encoding,video)

    设置最小拉格朗日(lagrange)因子(VBR).
lmax integer (encoding,video)

    设置最大拉格朗日(lagrange)因子 (VBR).
nr integer (encoding,video)

    设置降噪
rc_init_occupancy integer (encoding,video)

    设置解码开始前需加载到RC缓冲区的数据量
flags2 flags (decoding/encoding,audio,video)

    可能值:

    ‘fast’

        允许不符合规范的加速技巧。
    ‘sgop’

        失效, 使用mpegvideo私有选项 
    ‘noout’

        跳过比特流编码 
    ‘ignorecrop’

        忽略sps传来的遮蔽信息 
    ‘local_header’

        在全局头而不是每个关键帧放置扩展数据 
    ‘chunks’

        帧数据可被分割成多个块
    ‘showall’

        显示第一个关键帧前的所有帧（一般用于跳跃定位后的播放，默认是从最近关键帧开始显示，因为之前的帧不一定能够正确构建） 
    ‘skiprd’

        失效,使用了mpegvideo私有选项. 
    ‘export_mvs’

        支持它的编码中运动矢量导出给帧间数据(见`AV_FRAME_DATA_MOTION_VECTORS`) 。参考` doc/examples/export_mvs.c` 

error integer (encoding,video)
qns integer (encoding,video)

    失效, 使用了mpegvideo私有选项
threads integer (decoding/encoding,video)

    可能值:

    ‘auto’

        检测使用一个合适的线程数 

me_threshold integer (encoding,video)

    设置运动估计的阀值
mb_threshold integer (encoding,video)

    设置宏块阀值
dc integer (encoding,video)

    设置intra_dc_precision.
nssew integer (encoding,video)

    设置nsse权重.
skip_top integer (decoding,video)

    跳过顶部设置多个宏块行
skip_bottom integer (decoding,video)

    跳过底部设置多个宏块行
profile integer (encoding,audio,video)

    可能值:

    ‘unknown’
    ‘aac_main’
    ‘aac_low’
    ‘aac_ssr’
    ‘aac_ltp’
    ‘aac_he’
    ‘aac_he_v2’
    ‘aac_ld’
    ‘aac_eld’
    ‘mpeg2_aac_low’
    ‘mpeg2_aac_he’
    ‘mpeg4_sp’
    ‘mpeg4_core’
    ‘mpeg4_main’
    ‘mpeg4_asp’
    ‘dts’
    ‘dts_es’
    ‘dts_96_24’
    ‘dts_hd_hra’
    ‘dts_hd_ma’

level integer (encoding,audio,video)

    可能值:

    ‘unknown’

lowres integer (decoding,audio,video)

    在 1= 1/2, 2=1/4, 3=1/8 解码.
skip_threshold integer (encoding,video)

    设置跳帧阀值
skip_factor integer (encoding,video)

    设置跳帧因子
skip_exp integer (encoding,video)

    设置跳帧指数。 负值和正值除了归一化原因以外表现相同。正值存在的原因主要是兼容性，所以并不常见
skipcmp integer (encoding,video)

    设置跳帧压缩功能

    可能值:

    ‘sad’

        绝对差异总和, 最快(默认) 
    ‘sse’

        误差平方和 
    ‘satd’

        绝对Hadamard转换差异总和
    ‘dct’

        绝对DCT转换差异总和 
    ‘psnr’

        量化误差平方和 (avoid, 低品质) 
    ‘bit’

        对特定块需要的比特数 
    ‘rd’

        动态失真优化, 最慢
    ‘zero’

        0 
    ‘vsad’

        绝对垂直差异总和 
    ‘vsse’

        垂直差异平方和 
    ‘nsse’

        噪声保护的差的平方和
    ‘w53’

        5/3 小波（变换）, 只用于下雪场景 
    ‘w97’

        9/7 小波（变换）, 只用于下雪场景 
    ‘dctmax’
    ‘chroma’

border_mask float (encoding,video)

    增加接近边界宏块量化。
mblmin integer (encoding,video)

    设置最小的宏块的拉格朗日(lagrange)因子(VBR).
mblmax integer (encoding,video)

    设置最大的宏块的拉格朗日(lagrange)因子 (VBR).
mepc integer (encoding,video)

    设置运动估计比特率损失补偿(1.0 = 256).
skip_loop_filter integer (decoding,video)
skip_idct integer (decoding,video)
skip_frame integer (decoding,video)

    让解码器丢弃处理由选项值指定的帧类型

    skip_loop_filter 跳过循环帧   
	skip_idct 跳过IDCT/量化(dequantization）帧   
	skip_frame 跳过解码

    可能值:

    ‘none’

        不抛弃帧
    ‘default’

        抛弃无用帧，例如尺寸为0的帧
    ‘noref’

        抛弃所有非参考帧
    ‘bidir’

        抛弃所有双向(预测)帧
    ‘nokey’

        除了关键帧外都抛弃
    ‘all’

        抛弃所有帧 

    默认值就是‘default’.
bidir_refine integer (encoding,video)

    细化两个运动矢量用于双向宏块
brd_scale integer (encoding,video)

    对动态B帧判定是否下变换
keyint_min integer (encoding,video)

    设置IDR帧集的最小间隔
refs integer (encoding,video)

    为运动补偿设置参考帧 compensation.
chromaoffset integer (encoding,video)

    设置色度中qp对亮度的抵消
trellis integer (encoding,audio,video)

    设置比率失真优化
sc_factor integer (encoding,video)

    设置一个值（一个补偿因子）乘以`qscale`添加到每一帧的`scene_change_score`
mv0_threshold integer (encoding,video)
b_sensitivity integer (encoding,video)

    调整`b_frame_strategy`敏感性为1.
compression_level integer (encoding,audio,video)
min_prediction_order integer (encoding,audio)
max_prediction_order integer (encoding,audio)
timecode_frame_start integer (encoding,video)

    设置GOP时间码帧开始数,非丢帧格式
request_channels integer (decoding,audio)

    设置所需数字音频轨道/通道
bits_per_raw_sample integer
channel_layout integer (decoding/encoding,audio)

    可能值: 
request_channel_layout integer (decoding,audio)

    可能值: 
rc_max_vbv_use float (encoding,video)  
rc_min_vbv_use float (encoding,video)  
ticks_per_frame integer (decoding/encoding,audio,video)  
color_primaries integer (decoding/encoding,video)  
color_trc integer (decoding/encoding,video)  
colorspace integer (decoding/encoding,video)  
color_range integer (decoding/encoding,video)  
chroma_sample_location integer (decoding/encoding,video)  
log_level_offset integer  

    设置日志层次
slices integer (encoding,video)

    设置划片数，用于并行编码
thread_type flags (decoding/encoding,video)

    选择多线程方式

    使用‘frame’会导致每个线程解码延迟，所以如果客户端不提供未来帧状况就不应该使用

    可能值:

    ‘slice’

        每次解码不超过一个帧的多块数据

        划片多线程只用于视频划片编码工作
    ‘frame’

        一次解码多个帧 

    默认值是 ‘slice+frame’.
audio_service_type integer (encoding,audio)

    设置音频服务类型。

    可能值:

    ‘ma’

        主要音频服务 
    ‘ef’

        特效 
    ‘vi’

        视障 
    ‘hi’

        听障 
    ‘di’

        对话 
    ‘co’

        评论 
    ‘em’

        紧急情况 
    ‘vo’

        画外音 
    ‘ka’

        卡拉OK 

request_sample_fmt sample_fmt (decoding,audio)

    设置音频解码偏好。默认是none
pkt_timebase rational number
sub_charenc encoding (decoding,subtitles)

    设置输入的字幕字符编码
field_order field_order (video)

    设置/覆盖场序。可能值是：

    ‘progressive’

        逐行 
    ‘tt’

        隔行，顶场优先编码/显示
    ‘bb’

        隔行，底场优先编码/显示
    ‘tb’

        隔行，顶场优先编码，低场优先显示
    ‘bt’

        隔行，底场优先编码，低场优先显示

skip_alpha integer (decoding,video)

    设置为1来禁止处理透明度。不同的值可以类似一个‘灰色(gray)’蒙在画面上。默认值是0。
codec_whitelist list (input)

    "," 分隔的允许的解码器列表。 默认是都允许
dump_separator string (input)

    指定用于在命令行分隔参数、选项域的字符串。例如可以设置一个回车换行作为分隔:

    ffprobe -dump_separator "
                              "  -i ~/videos/matrixbench_mpeg2.mpg


