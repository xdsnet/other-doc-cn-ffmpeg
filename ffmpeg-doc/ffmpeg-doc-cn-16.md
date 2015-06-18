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

#### AC-3元数据控制选项 ####

- `-per_frame_metadata boolean`

    允许每个框架的元数据。指定编码器应该检测每帧变化的元数据

    0

        初始化的元数据用于每帧（不再管变化，默认) 
    1

        每帧都要检测元数据改变 

#### AC-3中置混合水平 ####
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

#### AC-3音频制作信息 ####
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

#### 其他AC-3元数据选项 ####



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

