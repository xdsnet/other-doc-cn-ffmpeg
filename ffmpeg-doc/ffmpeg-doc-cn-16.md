## 音频编码器 ##
介绍当前可用的音频编码器

### aac ###
AAC（Advanced Audio Coding ）编码器

当前原生（内置）编码器还处于实验阶段，而且只能支持AAC-LC（低复杂度AAC）。要使用这个编码器，必须选择 ‘experimental’或者'lower'

因为当前还处于实验期，所以很多意外可能发生。如果需要一个更稳定的AAC编码器，参考`libvo-aacenc`,然而它也有一些负面报告。

#### aac选项 ####

b

    设置码率，单位是bits/s，是自动恒定比特率(CBR)模式的码率
q

    设置为可变比特率（VBR）模式。此选项仅用于ffmpeg命令行工具。库接口是`global_quality`
stereo_mode

    设置立体声编码模式，可能值有:

    ‘auto’

        在编码时自动判断
    ‘ms_off’

        禁止中端的（即2.0声道，而不是2.1声道）编码，这时默认值
    ‘ms_force’

        强制中端编码，即强制2.1声道 encoding. 

aac_coder

    设置AAC编码方法，可能值:

    ‘faac’

        FAAC-启发方法.

        这个方法是显示方法的简化实现版，它为频带能量比设置阀值，降低所有量化步骤找到合适的量化失真阀值，对低于频带阀值的频带进行编码

        这种方法的质量稍微好于下面介绍的两回路搜索法，但很慢
    ‘anmr’

        基于网格的ANMR（平均噪音Average noise to mask ratio)掩比方案

        理论上它效果最好，但最慢
    ‘twoloop’

        双环搜索(Two loop searching)法

        该方法首先根据波段阀值量化并试图通过添或调整个别量化点减去一个特征值得到一个最佳组合

        这种方法和FAAC方法质量相当，是默认值
    ‘fast’

        固定量化法

        该方法设置所有带定量化，这是最快的方法，但质量最差

### ac3和ac3修订版 ###
AC-3音频编码器

这一编码器定义在 ATSC A/52:2010 和ETSI TS 102 366,以及RealAudio 3 (通过dnet)

AC3编码器使用浮点运算，而ac3_fixed编码器仅用定点整数的数学运算。这并不意味着一个人总是更快，只是一个或另一个可能更适合一个特定的系统。浮点编码通常会产生一个给定的比特率，更好的音频质量。ac3_fixed编码器没有任何输出格式的默认编码，所以它必须显式使用选项`-acodec ac3_fixed`指定。

#### AC-3元数据 ####
AC-3元数据选项用于设置音频参数的描述，它们大多数情况下不影响音频编码本身。这些选项不直接影响比特流或影响解码播放，只是提供信息。几个选项会增加音频数据比特输出流，从而影响输出质量。这将在下面的选项列表中注记出来：

下面文档介绍了几个公开文件文档：

- [A/52:2010 Digital Audio Compression (AC-3) (E-AC-3) Standard](http://www.atsc.org/cms/standards/a_52-2010.pdf)
- [A/54 Guide to the Use of the ATSC Digital Television Standard](http://www.atsc.org/cms/standards/a_54a_with_corr_1.pdf)
- [Dolby Metadata Guide](http://www.dolby.com/uploadedFiles/zz-_Shared_Assets/English_PDFs/Professional/18_Metadata.Guide.pdf)
- [Dolby Digital Professional Encoding Guidelines](http://www.dolby.com/uploadedFiles/zz-_Shared_Assets/English_PDFs/Professional/46_DDEncodingGuidelines.pdf) 

##### AC-3元数据控制选项 #####

- `-per_frame_metadata boolean`

    允许每个框架的元数据。指定编码器应该检测每帧变化的元数据

    0

        初始化的元数据用于每帧（不再管变化，默认) 
    1

        每帧都要检测元数据改变 

##### AC-3中置混合水平 #####
- `-center_mixlev level`

	AC-3中置混合水平。该值决定编码时根据立体声产生中置音量的标准。它只会写入存在中置通道的输出中。该值为规模因子，有3个有效值:

    0.707

        应用-3dB增益 
    0.595

        应用-4.5dB增益(默认值) 
    0.500

        应用-6dB增益

- `-surround_mixlev level`

    环绕混合水平。适用于环绕声道增益。它只会写入存在环绕声通道的输出中。该值为规模因子，有3个有效值:

    0.707

        应用-3dB增益 
    0.500

        应用-6dB增益(默认值) 
    0.000

        静默环绕声道(即没有环绕) 

##### AC-3音频制作信息 #####
音频制作信息描述了可选的混合环境信息，应用中要么都没有，要么同时有两个（即下面两个需要同时设置/或不设置）


- `-mixing_level number`

    混合水平. 指定在环境中混合的峰值声压级（SPL-Specifies peak sound pressure level）。 有效值是80 - 111或者-1（表示未知）或不指定。 默认值为-1，但如果`room_type`不为默认值，则`mixing_level`不能为-1.
- `-room_type type`

    空间类型。介绍了混音环境。是按大房间还是按小房间。如果没有指定`mixing_level`则写入默认值

    0
    notindicated

        没有指定 (默认) 
    1
    large

        大房间 
    2
    small

        小房间 

##### 其他AC-3元数据选项 #####



- `-copyright boolean`

    版权指示。

    0
    off

        不包含 版权(默认iansbaq) 
    1
    on

        版权信息 

- `-dialnorm value`

    对话常态化。表明对于低于平均值的音量保持原样（0dBFS）。 这个参数决定了匹配源的目标音量。值过小会导致相对于源没有变化。有效值为整数，范围为-31至-1，-31是默认值。
-dsur_mode mode

    杜比环绕模式。指定是否使用杜比环绕立体声信号。只对音频流是立体声的输出有效。使用了这个选项并不意味着实际处理会产生杜比环绕。

    0
    notindicated

        未指定 (默认) 
    1
    off

        不采用 
    2
    on

        采用杜比环绕解码 

-original boolean

    原始流指示器。指音频是原始源而不是副本。

    0
    off

        非原始源 
    1
    on

        原始源 (默认) 

#### 其他扩展比特流信息 ####
这些扩展比特流选项都被定义在A/52:2010标准的附录D中。它分为2个部分（组）。如果组中任意一个参数被指定，则组中所有的值将以默认值写入到流中。如果`mixing levels`被设置，则对支持备用比特流语法（ Alternate Bit Stream Syntax）的解码器将采用这个值以替代`center_mixlev`和`surround_mixlev`选项定义。

##### 其他扩展比特流信息 第一部分 #####
- `-dmix_mode mode`

    优化立体声缩混模式。允许在Lt/Rt (杜比环绕)或者Lo/Ro (常规立体声) 作为优化立体声缩混模式

    0
    notindicated

       	未指定(默认) 
    1
    ltrt

        Lt/Rt 缩混优化 
    2
    loro

        Lo/Ro 缩混优化 

- `-ltrt_cmixlev level`

    Lt/Rt模式下中置混合层次。在Lt/Rt模式下解码器输出中置通道的增益

    1.414

        使用+3dB增益 
    1.189

        使用 +1.5dB增益
    1.000

        使用 0dB  
    0.841

        使用 -1.5dB  
    0.707

        使用 -3.0dB  
    0.595

        使用 -4.5dB 增益 (默认值) 
    0.500

        使用 -6.0dB  
    0.000

        禁用中置通道 


- `-ltrt_surmixlev level`

    Lt/Rt模式下环绕增益。在Lt/Rt模式下解码器输出环绕通道的增益

    0.841

        使用 -1.5dB  
    0.707

        使用 -3.0dB  
    0.595

        使用 -4.5dB 增益 (默认值) 
    0.500

        使用 -6.0dB  
    0.000

        禁用环绕通道  


- `-loro_cmixlev level`

    Lo/Ro模式下中置混合层次。在Lo/Ro模式下解码器输出中置通道的增益.

    1.414

        使用+3dB增益 
    1.189

        使用 +1.5dB增益
    1.000

        使用 0dB  
    0.841

        使用 -1.5dB  
    0.707

        使用 -3.0dB  
    0.595

        使用 -4.5dB  
    0.500

        使用 -6.0dB  增益(默认值) 
    0.000

        禁用中置通道 


- `-loro_surmixlev level`

    Lo/Ro模式下中置混合层次。在Lo/Ro模式下解码器输出环绕通道的增益.

    0.841

        使用 -1.5dB  
    0.707

        使用 -3.0dB  
    0.595

        使用 -4.5dB  
    0.500

        使用 -6.0dB  增益(默认值) 
    0.000

        禁用环绕通道  

##### 其他扩展比特流信息 第二部分 #####
- `-dsurex_mode mode`

    Dolby环绕EX模式. 标识是否使用Dolby环绕EX模式（7.1矩阵转5.1）.使用了此选项并不意味着编码器将实际应用Dolby环绕EX模式

    0
    notindicated

        未标识 (default) 
    1
    on

        Dolby环绕EX模式关闭 
    2
    off

        Dolby环绕EX模式打开 

- `-dheadphone_mode mode`

    杜比耳机模式。标识编码为杜比耳机（多通道矩阵合成为2个声道）使用这个选项并不意味着实际应用了杜比耳机模式。

    0
    notindicated

        未标识 (default)
    1
    on

        Dolby 耳机模式关闭
    2
    off

        Dolby 耳机模式打开



- `-ad_conv_type type`

    A/D（模数转换） 转换类型。标识音频需要HDCD A/D 转换。

    0
    standard

        标准 A/D转换 (默认) 
    1
    hdcd

        HDCD A/D 转换 

#### 其他AC-3编码器选项 ####

- `-stereo_rematrixing boolean`

    Stereo 再混（Rematrixing）。通过Enables/Disables 来对应立体声输入。它是可选功能，通过选择左/右而当作立体声输出，从而提高输出效果。默认是启用的。因为该编码器会增加程序现象，所以只建议用于测试。

#### 浮点AC-3编码特有选项 ####
这些选项只在浮点AC-3编码时有效，整形AC-3时是不起作用的。

- `-channel_coupling boolean`

    Enables/Disables通道的耦合。这是一个可选的音频选项，它从多个通道中获取高频带信息整合输出到一个通道中。这允许更多的比特位用于较低频率音频的同时保持足够的信息重建高频部分。这个选项对浮点AC-3来说主要用于测试或者提高编码速度。

    -1
    auto

        由编码器选择 (默认) 
    0
    off

        禁用通道耦合 
    1
    on

        允许通道耦合

- `-cpl_start_band number`

    耦合开始带。设置通道耦合的开始带，从1-15可选。如果设置的值大于通道数，则处理为需耦合最后通道减一。如果auto（自动）被设置，则开始带将根据码率、通道布局、采样率有编码器自动计算。如果通道耦合设置为禁用，则本选项失效。

    -1
    auto

        由编码器 选择(默认) 

### flac ###
FLAC(自由低损失音频编码——Free Lossless Audio Codec)编码器
#### flac选项 ####
下面是FFmpeg中flac编码可用选项
- `compression_level`

    设置压缩级别，如果没有显式设置即采用默认值

- `frame_size`

    设置各个通道的帧大小

- `lpc_coeff_precision`

    设置LPC系数精度，有效值从1到15, 15是默认值

- `lpc_type`

    设置第一阶段LPC算法

    ‘none’

        不采用LPC
    ‘fixed’

        整数LPC
    ‘levinson’
    ‘cholesky’

- `lpc_passes`

    用Cholesky分解LPC的次数

- `min_partition_order`

    最小分区顺序

- `max_partition_order`

    最大分区顺序

- `prediction_order_method`

    ‘estimation’
    ‘2level’
    ‘4level’
    ‘8level’
    ‘search’

        强力搜索
    ‘log’

- `ch_mode`

    通道模式

    ‘auto’

        自动模式，对每帧自动匹配通道 
    ‘indep’

        通道独立编码 
    ‘left_side’
    ‘right_side’
    ‘mid_side’

- `exact_rice_parameters`

    是精确还是近似.如果设置为1表示精确，会减慢编码速度以提高压缩率。

- `multi_dim_quant`

    多维量化。如果设置为1，那么第二阶段LPC应用第一阶段结果进行算法调整。这很慢，但可以提高压缩率

### libfaac编码 ###
libfaac 是AAC（Advanced Audio Coding）编码器的再封装

要使用它需要libfaac头文件和库存在配置。你还需要在编译ffmpeg时通过`--enable-libfaac` `--enable-nonfree`进行配置。

该编码器高质量版本参考[aacenc]

对更多信息，参考libfaac项目介绍[http://www.audiocoding.com/faac.html/](http://www.audiocoding.com/faac.html/)

#### libfaac相关选项 ####
下面是ffmpeg工具编码时的可用选项

下面的选项适用于libfaac封装，`faac-XXXX`等效选项列在括号中


- b (-b)

    设置ABR（平均码率），单位bits/s。如果码率没有特别指定，会自动匹配所选特性（属性配置）。 faac比特率单位是 kilobits/s.

    **注意**libfaac不支持CBR(Constant Bit Rate——固定码率)，只支持ABR (Average Bit Rate——平均码率).

    如果VBR模式设置为允许，则本选项被忽略
- ar (-R)

    设置音频采样率，单位Hz
- ac (-c)

    设置音频通道数
- cutoff (-C)

    设置截至频率。如果没有设置或者设置为0，则自动根据库计算。默认为0
- profile

    设置音频特性（属性配置）文件

    下面的音频特性文件有效:

    ‘aac_main’

        主要的AAC (Main)
    ‘aac_low’

        低复杂度AAC (LC)
    ‘aac_ssr’

        可扩展采样 (SSR)
    ‘aac_ltp’

        长期预测(LTP——Long Term Prediction ) 

    如果没有指定则表示为‘aac_low’.
- flags +qscale

    设置VBR(动态码率Variable Bit Rate)下的品质
- global_quality

    设置VBR下的品质，其为一个数字或者lambda表达式

    仅仅在VBR模式，且 flags +qscale 被设置为有效才起作用。它将被`FF_QP2LAMBDA`转换成QP值，并应用于libfaac。QP值的范围为[10-500]，越大品质越好
- q (-q)

    允许VBR模式，但设置为负数值，则值作为双精度浮点数

	值应用于libface。值的可能范围是[10-500]，数字越大QP值越高。

	选项只用于ffmpeg命令行，或者通过`global_quality`属性描述问中rs。 

#### libfaac例子 ####
- 使用`ffmpeg`把一个音频转换为ABR 128kbps AAC编码格式流放置在M4A（MP4音频）文件中
    
    ffmpeg -i input.wav -codec:a libfaac -b:a 128k -output.m4a

- 使用`ffmpeg`把一个音频转换为VBR AAC编码（采用LTP AAC）格式流放置在M4A（MP4音频）文件中

	ffmpeg -i input.wav -c:a libfaac -profile:a aac_ltp -q:a 100 output.m4a

### libfdk aac ###libfdk-aac
libfdk-aac的再封装

该库只用于Fraunhofer  FDK AAC 编码格式

要使用，必须有libfdk-aac有头和预配的库，并在编译ffmpeg时用配置选项`--enalbe-libfdk-aac`启用。这个库可能不兼容于GPL，如果你要使用GPL，你必须`--enable-gpl --enable-nonfree --enable-libfdk-aac`

这个编码器被认为品质高于 内置的[AACenc] 和[libfaac]

VBR编码可以通过 `vbr` 或者`flags + qscale`选项启用，它们是实验性质的，只适合于某些参数组合。

libfdk-aac 0.1.3或者更高版本支持7.1声道

更多信息请参考fdk-aac项目[http://sourceforge.net/p/opencore-amr/fdk-aac/](http://sourceforge.net/p/opencore-amr/fdk-aac/)

#### libfdk-aac选项 ####
下面是可用的一些选项：


- b

    设置码率，如果未指定，则根据属性特性自动匹配

    如果工作于VBR模式，本选项被忽略、
- ar

    设置采样率 (单位Hz).
- channels

    设置通道数
- flags +qscale

    可以调整品质，VBR (Variable Bit Rate)模式。**注意** 当vbr为正表示VBR模式被隐含启用
- cutoff

    设置截止频率，如果没有设置或者为0，表示自动根据库计算默认为0
- profile

    设置音频属性预设文件，可以有：

    ‘aac_low’

        低复杂度AAC(LC)
    ‘aac_he’

        高效率AAC (HE-AAC)
    ‘aac_he_v2’

        高效率AAC版本2 (HE-AACv2)
    ‘aac_ld’

        低延迟AAC(LD)
    ‘aac_eld’

        增强低延迟AAC (ELD) 

    如果没有特别指定则为‘aac_low’. 

下面是libfdk_aac私有选项

- afterburner

    设置为1表示允许助力，否则为0，它可以增强品质，但要求更多处理能力。

    默认为1
- eld_sbr

    1表示允许对ELD采样SBR (Spectral Band Replication-频带复制)，否则为 0.

    默认为 0
- signaling

    设置SBR/PS 指令方式

    接受下面的值：

    ‘default’

        选择含蓄信号模式 (默认明确为 hierarchical, 如果全局头设置为禁止，则隐式表达)
    ‘implicit’

        隐式向后兼容指令信号
    ‘explicit_sbr’

        明确为SBR, 隐式PS信号
    ‘explicit_hierarchical’

        明确为hierarchical信号 

    默认为‘default’.
- latm

    设为1表示输出LATM/LOAS封装数据，否则为0

    默认为0
- header_period

    设置StreamMuxConfig 和 PCE 重复周期 (在帧上)， 把LATM/LOAS包含着配置发送缓冲中

    必须为16bit的非负整数

    默认为 0.
- vbr

    设置VBR模式，从1最低品质（但仍足够好）, 5是最高品质。如果值为0表示禁用VBR，而是采用CBR（固定码率）

    当前只有 ‘aac_low’属性预设支持VBR

    VBR模式1-5代表的平均码率:

    ‘1’

        32 kbps/channel 
    ‘2’

        40 kbps/channel 
    ‘3’

        48-56 kbps/channel 
    ‘4’

        64 kbps/channel 
    ‘5’

        about 80-96 kbps/channel 

    默认0. 

#### libfdk_aac 例子 ####
- 转换为VBR AAC M4A

    ffmpeg -i input.wav -codec:a libfdk_aac -vbr 3 output.m4a
- 转换为CBR 64k AAC，使用高效率AAC属性预设
    
    ffmpeg -i input.wav -c:a libfdk_aac -profile:a aac_he -b:a 64k output.m4a

### libmp3lame ###
LAME (Lame Ain’t an MP3 Encoder) MP3 编码器封装

需要在编译时配置 libmp3lame 头和库，并且显式设置-`-enable-libmp3lame`

参考`libshine` 这个整数修正MP3编码器（虽然质量较低）

#### libmp3lame选项 ####
下面是支持的选项（lame-XXX等效选项列在括号中）:


- b (-b)

    设置码率CBR/ABR，单位为bits/s，LAME的码率为kilobits/s\
- q (-V)

    设置VBR下的品质。它只用于ffmpeg命令行工具，对于库接口，使用`global_quality`
- compression_level (-q)

    设置算法品质。通过0-9的参数，表示不同的品质，0最高但最慢，9最低但最快
- reservoir

    为1（默认）表示允许bit池，否则为0. LAME 也是默认允许但可以被`--nores`覆盖
- joint_stereo (-m j)

    1表示允许在（每帧）中编码L/R立体声或者mid/side立体声。默认为1 Default value is 1.
- abr (--abr)

    为1表示允许ABR，lame `--abr`设置为一共的码率，这里只是表示采用ABR，码率还是由`b`设置

