## 34 音频滤镜
当你配置编译FFmpeg时，先采用`--disable-filters`可以禁止所有的滤镜，然后显式配置想要支持的滤镜。

下面是当前可用的音频滤镜

### adelay ###
延迟一个或者多个音频通道

它接受如下选项：
- delays

	参数是以`|`分隔的列表字符串，分别用于指明对应各个通道延迟的微秒（milliseconds）数。应提供至少一个大于0的延迟。未使用的延迟将被静默忽略。如果延迟值数量小于通道数量，则剩余通道不会被延迟。
#### adelay例子 ####
- 第一通道延迟1.5秒，第三通道0.5秒（其它通道均不延迟变化）

	adelay=1500|0|500

### aecho ###
重复应用于音频输入（回声效果）滤镜

回声反射声音,可以自然发生在山区大型建筑(有时)谈话时,或者大叫时，数字回声效果模拟这种行为，通常用来帮助填补一个乐器或声音的（回声）。原始信号和发射信号的时差就是`delay`（延迟），而反射信号的响度是`decay`（衰减）。多个回声可以有不同的延迟和衰减。

要描述一个回声效果需要如下的参数值（注意下面的参数之间用`：`分隔）：

- in_gain

    设置输入获得的反射信号强度，默认0.6.
- out_gain

    设置输出增加反射信号强度，默认0.3.
- delays

    一个由`|`分隔原始信号和反射作用的指代延迟时间的字符串列表，单位是微秒（milliseconds）。每个延迟允许范围（0-90000.0），默认为1000
- decays

    设置一个由`|`分隔的反映信号响度衰减值的列表，每个衰减值范围是（0-1.0），默认为0.5. 

#### aecho例子 ####
- 让一个声音听起来像两倍

	aecho=0.8:0.88:60:0.4
- 如果延迟十分短，那听起来像一个机器人（金属）音乐

	aecho=0.8:0.88:6:0.4
- 一个十分长延迟（回声）的声音好像在一个空旷山谷里听露天音乐会。

	aecho=0.8:0.9:1000:0.3
- 同上，但不只一座山的效果（多次反射回音）

	aecho=0.8:0.9:1000|1800:0.3|0.25
	
### aeval ###
根据指定的表达式修改（改变、变化）一个音频信号

这个滤镜接受一个或者多个表达式（对每个通道），这些表达式计算用于相应的音频信号。

它接受下面的参数：

- exprs

    设置一个用`|`分隔的对应于各个通道的表达式。如果输入通道数量比表达式数量大，则最后指定的表达式用于其余通道
- channel_layout, c

    设置输出通道布局。如果不指定，通道布局采用通道布局数值表达式。如果设置为`same`则采用输入通道相同的布局（这是默认值）

帧各通道的计算表达式中，下面的项目被允许。:

- ch

    当前表达式对应通道索引 expression
- n

    评估样本数量，从0开始
- s

    采样率
- t

    一秒内采样点数量。
nb_in_channels
nb_out_channels

    输入和输出通道索引
val(CH)

    the value of input channel with number CH 

**注意**这个滤镜比较慢，要快速处理你可能需要`dedicated`滤镜

#### aeval例子 ####
- 一半音量

    aeval=val(ch)/2:c=same
- 转化相位的第二个通道::

    aeval=val(0)|-val(1)
    
### afade ###
对输入音频应用淡入淡出效果

下面是跟上来的滤镜参数：

- type, t

    指定滤镜效果，可以是`fade-in`,或者`fade-out` 
- start_sample, ss

    指定的数量开始样品开始应用褪色的效果。默认是0S
- nb_samples, ns

    指定实现淡入/淡出效果的样品数量，最终淡入效果输出的音频音量同于输入，而淡出将静音。默认音频采样率为44100。
- start_time, st

    指定淡入/淡出效果开始的时间，默认为0。这个值必须被以[持续时间](#ffmpeg-doc-cn-07.md#持续时间)语法来描述。它可以用来替代`start_sample`选项。
- duration, d

    指定淡化效果持续时间。以[持续时间](#ffmpeg-doc-cn-07.md#持续时间)语法来描述。在效果的最后，淡入使得音量同于输入音频，淡出则静音。默认持续时间由`nb_samples`定义。 这里设置了就替代了`nb_samples`
- curve

    设置曲线过渡衰减，接受下面的值：

    tri

        选择三角形,线性斜坡(默认)
    qsin

        选择正弦波
    hsin

        选择正弦波的一半 
    esin

        选择指数正弦 
    log

        选择对数 
    ipar

        选择倒抛物线 
    qua

        选择二次
    cub

        选择立方
    squ

        选择平方根 
    cbr

        选择立方根 
    par

        选择抛物线
    exp

        选择指数
    iqsin

        选择正弦波反季
    ihsin

        选择倒一半的正弦波
    dese

        选择双指数插值 
    desi

        选择双指数S弯曲

#### afade例子 ####
- 15秒的一个音频淡入

	afade=t=in:ss=0:d=15
- 25秒的音频淡出

	afade=t=out:st=875:d=25

### aformat ###
让输入音频约束成为指定格式。该框架会采用最合适（少）的格式转换

它接受下面的参数：

- sample_fmts

    一个用`|`分隔的列表，列出了采样格式
- sample_rates

    一个用`|`分隔的列表，列出了采样率
- channel_layouts

    一个用`|`分隔的列表指定通道布局.

    参考[通道布局](ffmpeg-doc-cn-07.md#通道布局)了解通道布局相关语法。

如果一个参数被省略，所有的值都是允许的。

强制输出为8位 或者16位 立体声

	aformat=sample_fmts=u8|s16:channel_layouts=stereo

### allpass ###
应用一个两极（two-pole）全通（all-pass）滤波器的中心频率(Hz)的`frequency`,和filter-width值`width`。一个`allpass`滤镜可以改变音频的频率相位关系而不改变其频率振幅关系。（可以实现移相）

滤镜接受下面的选项：

- frequency, f

    设置频率，单位Hz.
- width_type

    设置带宽滤波器的带宽单位，有下面的类型

    h

        Hz 
    q

        Q-Factor 
    o

        octave——8度 音阶
    s

        slope 

- width, w

    指定一个过滤器的带宽width_type单位 
    
### amerge ###
合并两个或两个以上的音频流到一个多通道流

滤镜接受下面的选项：

- inputs
	
	设置输入数量，默认为2

如果输入的通道布局是不相交的,因此可兼容,输出将设置相应的通道布局和渠道，并在必要时重新排序。如果输入的通道布局是不可分离的，则输出将会是第一个输入的所有通道，然后第二个输入的所有通道，在这种顺序下，输出的通道布局将默认通道数设为总数。

例如：如果第一个输入是`2.1`（`FL+FR+LF`）和第二个输入为`FC+BL+BR`，则输出是`5.1`通道布局，并且按：`a1，a2,b1,a3,b2,b3`设置输出通道布局（这里a1是第一个输入的第一个通道`FL`，b1是第二个输入的第一个通道`FC`）

在另外的应用中，如果两个输入都是立体声，则输出会默认为：`a1,a2,b1,b2`，即输出流显示为一个4通道音频流，这可能是一个非预期的值。

所有的输入必须有相同的采样率和格式。

如果输入没有相同的持续时间，输出将在最短时间停止。

#### amerge例子 ####
- 合并两个单声道为立体声

	amovie=left.wav [l] ; amovie=right.mp3 [r] ; [l] [r] amerge
- 合并多个到1个视频和6个音频流

    ffmpeg -i input.mkv -filter_complex "[0:1][0:2][0:3][0:4][0:5][0:6] amerge=inputs=6" -c:a pcm_s16le output.mkv

### amix ###
混合多个音频输入到单路音频输出（叠加混合音频，不同于前面的amerge）

**注意**这个滤镜只支持浮动采样（`amerge`和`pan`音频滤镜支持很多格式）。如果`amix`滤镜输入有一个整数采样，则`aresample`滤镜会自动插入转换成浮动采样。

例如：

	ffmpeg -i INPUT1 -i INPUT2 -i INPUT3 -filter_complex amix=inputs=3:duration=first:dropout_transition=3 OUTPUT
会把3个输入音频流混合成一个输出流，持续时间采用第一个输入流的持续时间并且有3秒的结束过渡。

它支持下面的参数：
- inputs

	输入数，如果没有指定则默认为2
- duration

	确定流结束的方法，有：
	longest
		
		按最长持续时间输入（默认）

	shortest

		按最短持续时间输入

	first
		
		按第一个输入持续时间
- dropout_trnsition

	过渡时间，单位秒，指一个输入流结束时音量从正常到无声渐止效果，默认为2秒

### anull ###
输入音频源不变的到输出

### apad ###
在一个音频流的末尾添加无声。

它可以用来同`ffmpeg -shortest`一起把最短的音频流延长到视频相同长度。

选项介绍见下：

- packet_size

    设置垫的包大小字节，默认4096.
- pad_len

    设置要添加到最后的采样点数量（实为时间的一种表达，采样率一定则采样点个数决定了持续时间，这里只设置了差值）。值达到后终止。它与`whole_len`互斥
- whole_len

    设置最小的音频流总输出样本点数（就是直接设置总持续时间的一种方式），如果这个值大于输入音频数，则垫上差值到最后。它与`pad_len`互斥 

如果既不设置`pad_len`也不设置`whole_len`，则接在后面的静音将一直持续。

#### apad例子 ####
- 添加1024个静音样本点到输入末尾

	apad=pad_len=1024
- 让输出至少有10000个样本点，不足就添加静音样本点到末尾

	apad=whole_len=10000
- 利用ffmpeg添加静音样本点，让视频和音频有同样长的持续时间（以视频时间为准的）。

	ffmpeg -i VIDEO -i AUDIO -filter_complex "[1:0]apad" -shortest OUTPUT

### aphaser ###
添加一个移相到输入音频

移相器滤镜创建一系列的波峰和波谷的频谱。波峰和波谷的位置调制,这样他们会随着时间变化,建立一个全面的效果。

可接受参数介绍见下：

- in_gain

    设置输入增益，默认 0.4.
- out_gain

    设置输出增益，默认0.74
- delay

    设置延迟，单位微秒，默认3.0.
- decay

    设置衰减权重，默认0.4.
- speed

    设置调制速度，单位Hz，默认 0.5.
- type

    设置调制类型，默认 triangular.

    它接受下面的值:

    ‘triangular, t’
    ‘sinusoidal, s’

### aresample ###
对输入音频按指定的参数重采样，使用了`libswresample`库，如果没有特殊设定，将自动在输入和输出设置间转换。

这个滤镜还可以用于拉伸/压缩音频数据，使其匹配时间戳，或者通过注入静音/剪切来匹配时间戳

滤镜接受的语法：`[sample_rate:]resampler_options`，这里`sample_rate`是新采样率，`resampler_options`是一个由`：`分隔的`key=value`选项参数值对列表。参考ffmpeg重采样手册完整了解支持的选项。

#### aresample例子 ####
- 重采样为44100Hz:

    aresample=44100
- 拉伸/压缩采样来适应时间戳，最大1000个样本点每秒:

    aresample=async=1000

### asetnsamples ###
设置每个输出音频帧中样本点个数

除了最后一个输出包括有包含不同数量的样本点外，这个滤镜使得持续中的每个数据包中包含一致数量的样本点。

滤镜接受下面的选项：

- nb_out_samples， n

	设置每个输出音频帧中样本点个数，它替代作为每个通道样本点个数，默认1024
- pad， p

	如果设置为1，则滤镜在最后一个音频帧中补0填充，这样所有帧都有一样的样本点个数。默认为1

例如，设置每帧样本点个数为1234，且禁止最后帧补齐

	asetnsamples=n=1234:p=0
	
### asetrate ###
重新设置采样率而不改变PCM数据。这将导致速度和音调的变化。

滤镜接受下面的选项：

- sample_rate， r

	设置输出的采样率，默认为44100Hz

### ashowinfo ###
对每个输入流音频帧显示其所含各种信息。输入音频不被改变

信息由一个序列的键值对构成, 键值对的格式为 `key:value`

下面的信息将被显示（作为键值对的键名）：

- n

    当前的输入（音频）帧序号，从0开始计数
- pts

    输入帧的时间戳，按时基计数，时基依赖于滤镜输入，通常为`1/sample_rate`
- pts_time

    输入帧时间戳按秒的表示
- pos

    输入帧中输入流中的偏移（文件读写指针位置）， -1表示该信息不可用 和/或者 无意义 (例如合成音频)
- fmt

    采样格式
- chlayout

    通道布局
- rate

    音频帧的采样率
- nb_samples

    每帧中（每个通道）采样点数
- checksum

    音频数据Adler-32校验和(以十六进制数据形式输出) ，对于连续音频数据，数据被看作是都连接在一起的。
- plane_checksums

    一个Adler-32校验和列表，对应于每个数据块 

### astats ###
显示音频通道的时域统计信息。统计计算和显示每个音频通道，值适合情况下还提供整体图。

它接受下面的选项：

- length

    按秒给出的小窗口长（指统计动态移动窗），用于RMS测量波峰和波谷。默认为0.05（50微秒），允许值范围为：[0.1-10]

每个显示用选项的介绍见下A：

- DC offset

    从0振幅位移
- Min level

    最小采样点 水平
- Max level

    最大采样点 水平
- Peak level dB
- RMS level dB

    标准的峰值和有效值测量，单位dBFS
- RMS peak dB
- RMS trough dB

    在短窗口中波峰和波谷RMS值水平测量
- Crest factor

    标准比率RMS的峰值水平 (**注意**不是dB值了).
- Flat factor

    平整度(即连续样本具有相同值)信号的峰值水平(即最小水平或最大级别)。
- Peak count

    计数多次(而不是样品的数量),信号达到最小水平或最大水平

### astreamsync ###
将两个音频流控制的发到缓冲区

滤镜接受下面的选项：

- expr, e

    设置一个用于判断哪个流被送出的表达式。如果结果为负则第一个流被转发，否则如果为非负则第二个流被转发。表达式中允许下面的变量:

    b1 b2

        分别指代缓冲区中每个输入音频流到目前为止转发数量
    s1 s2

        到目前为止，已经转发的每个流的采样点数量 
    t1 t2

        当前时间每个流的时间戳 stream 

    默认表达式为`t1-t2`, 它意味着把时间戳小的流进行转发

#### astreamsync例子 ####
压力测试`amerge`通过随机发送给缓冲区作为有错误的输入，同时避免太多同步失锁：
	
	amovie=file.ogg [a] ; amovie=file.mp3 [b] ;
	[a] [b] astreamsync=(2*random(1))-1+tanh(5*(t1-t2)) [a2] [b2] ;
	[a2] [b2] amerge

### asyncts ###
通过需要 压缩/拉伸 和/或 采样点/填补静音 来让音频数据和时间戳同步。

这个滤镜不默认编译，请使用[aresample](#aresample)来压缩/拉伸。

它接受下面的参数：

- compensate

    若允许则通过拉伸/压缩来让数据匹配时间戳。默认禁止。当禁止时数据对时间戳将以静音补齐
- min_delta

    触发数据丢弃/补齐的 时间戳与音频数据最小差异（按秒为单位）默认为0.1。如果默认值还是不完美同步，可以尝试设置为0
- max_comp

    当`compensate=1`时，每秒最大补齐样本点数，默认为500.
- first_pts

    这时一个时基单位（其实设置在前面补齐/去除 时间），第一个PTS为 1/sample_rate, 它允许了在开始补齐/去除 的时间量。默认情况下， 没有假定一个，所有没有补齐/调制。 例如 要让一个音频和另外的视频同步，可能需要在前面加上/或者减去 一些样本点

### atempo ###
调整音频节奏(变奏)

滤镜接受1个参数，表示音频节奏。如果不知道，则默认为1.0，表示不变，参数值范围为`[0.5,2.0]`

#### atempo例子 ####
- 减慢为80%

	atempo=0.8
- 加快为125%

	atempo=1.25

### atrim ###
建设连续输入中的部分作为输出。

接受下面的参数：

- start

    以秒为单位的开始时间戳。即所指时间样本点将作为输出的第一个样本点
- end

    以秒为单位的结束时间戳。即所指时间前最后一个样本点将作为输出的最后样本点，其所指样本点及其后的均被丢弃。
- start_pts

    类似`start`，除了它的不是以秒为单位
- end_pts

    类似`end`, 除了它不是一秒为单位
- duration

    输出的持续时间
- start_sample

    要输出的第一个样本点序号
- end_sample

    要丢弃的第一个样本点序号，（其前的最后一个样本点是输出的最后一个样本点）

其中`start`, `end`和`duration`采样[持续时间](ffmpeg-doc-cn-07.md#持续时间)格式描述，参考相关章节以了解详情。 

**注意**前面的`start`/`end`和`duration`是看帧的时间戳，而有`_sample`的选项则只是简单的对传入数据的样本点计数。所有如`start`/`end_pts`和`start`/`end_sample`会造成不同的结果（当时间戳不准确、或从0开始）。还要注意这个滤镜并不修改时间戳。如果你想让输出时间从0开始，则在其后插入`asetpts`滤镜

如果同时有多个`start`或`end`选项被设置，滤镜尝试（贪婪算法）保留尽量多的样本点作为输出（即`start`和`end`差最大的）。如果想保留多个块，需要连接多个`atrim`滤镜来应用（多个次输入源，接`atrim`后再连接起来）

默认所有输入被保留。所有它可被配置为保留结束前的一切。

例子：

- 丢弃指定时间外的输入：

	ffmpeg -i INPUT -af atrim=60:120
- 仅保留开始的1000帧

	ffmpeg -i INPUT -af atrim=end_sample=1000

### bandpass ###
应用一个通过中心点频率`frequency`定义的两极Butterworth（巴特沃斯）带通滤波器，其有3dB的带宽。`csg`选项指定一个常数作为默认增益（峰值增益 Q ,默认值为0）。滤镜到期后每个8分音度有6dB的衰减。

滤镜接受下面的选项：

- frequency, f

    设置滤镜的中心点频率，默认3000.
- csg

    设置增益常数，若想增益倍值为1（不变化），则值默认为0。
- width_type

    设置指定带通滤波的类型：

    h

        Hz 
    q

        Q-Factor 
    o

        octave-8分音度
    s

        slope-斜率 

- width, w

    设置带通滤波带宽（单位为`width_type`）

### bandreject ###
应用一个通过中心点频率`frequency`定义的两极Butterworth（巴特沃斯）带通滤波器，其有单侧3dB的带宽。滤镜到期后每个8分音度有6dB的衰减。


- frequency, f

    设置滤镜的中心点频率，默认3000.
- width_type

    设置指定带通滤波的类型：

    h

        Hz 
    o

        octave-8分音度
    s

        slope-斜率 

- width, w

    设置带通滤波带宽（单位为`width_type`）

### bass ###
使用双刀搁置滤波器增加或减少低音(低)音频的频率响应，类似于一个标准的高保真的音控。这也被称为搁置平衡(EQ)。

滤镜接受下面选项：

- gain, g

    设置在0Hz的增益，其可用的范围约为-20（大振幅）+20。要合适的值以防被削波（振幅过大超出说了样本格式允许值范围就削波）
- frequency, f

    设置滤镜的中心点频率，默认100 Hz.
- width_type

    设置指定带通滤波的类型：

    h

        Hz 
    o

        octave-8分音度
    s

        slope-斜率 

- width, w

    设置带通滤波带宽（单位为`width_type`）

### biquad ###
应用一个 `biquad` IIR（无限冲激响应）滤镜，它有b0、b1、b2和a0、a1、a3 分别作为分子和分母。

### bs2b ###
Bauer（鲍尔）立体声双声道的转换,使耳机听立体声音频记录

它接受下面的参数：

- profile

    预定义横向进给水平

    default

        默认水平 (fcut=700, feed=50).
    cmoy

        Chu Moy circuit (fcut=700, feed=60).
    jmeier

        Jan Meier circuit (fcut=650, feed=95).
- fcut

    截至频率，单位Hz
- feed

    进给水平，单位Hz

### channelmap ###
重新映射输入通道

它接受下面的参数：

- channel_layout

    输出流通道布局
- map

    从输入到输出的通道映射。参数值是一个由`|`分隔的映射关系描述列表。每个为`in_channel-out_channel`或者`in_channel`格式。`in_channel`可以用于输入通道名称（例如FL表示左前）或者在输入通道中的索引数。`out_channel`可以用输出通道名称或者索引数。如果`out_channel`被省略，则从0开始递增映射每个输出通道

如果没有设置`map`，滤镜将隐式映射，保留所指（通道布局中对应通道）

例如，要把一个5.1 声道的MOV文件下变换

	ffmpeg -i in.mov -filter 'channelmap=map=DL-FL|DR-FR' out.wav
则将创建一个输出WAV文件，其只有2个声道（立体声）。

又如要修复一个5.1声道AAC编码中不当的通道顺序

	ffmpeg -i in.wav -filter 'channelmap=1|2|0|5|3|4:channel_layout=5.1' out.wav

### channelsplit ###
把输入音频流的每个通道分开作为多个输出流

它接受下面的参数：

- channel_layout

	指定输入通道布局，默认为 "stereo"

例如从输入MP3文件中分离立体声

	ffmpeg -i in.mp3 -filter_complex channelsplit out.mkv

会创建一个包含2个音频流的Matroska文件，其中一个对应于原来的左声道，另外一个对应于右声道。

划分5.1声道的WAV文件：

	ffmpeg -i in.wav -filter_complex
	'channelsplit=channel_layout=5.1[FL][FR][FC][LFE][SL][SR]'
	-map '[FL]' front_left.wav -map '[FR]' front_right.wav -map '[FC]'
	front_center.wav -map '[LFE]' lfe.wav -map '[SL]' side_left.wav -map '[SR]'
	side_right.wav

### chorus ###
给声音添加合唱效果

可以让独唱变得像合唱，但也可以用于仪表。

合唱与回声效应都有短延迟,但是回波延迟是常数,合唱则采用不同的正弦或三角调制。调制深度范围定义了调制延迟(播放之前或之后的延迟)。因此延迟的声音听起来较慢或更快,这是原来周围的延迟调整声音,像是有一个与合唱整体略微差异。

它接受下面的参数（每个参数项如果有多个可能值用`|`分隔）：

- in_gain

    设置输入增益，默认0.4.
- out_gain

    设置输出增益，默认0.4.
- delays

    设置延迟，延迟通常在40ms - 60ms
- decays

    设置衰减
- speeds

    设置速度
- depths

    设置深度 
    
#### chorus例子 ####
- 一个延迟（二人合唱效果）:

    chorus=0.7:0.9:55:0.4:0.25:2
- 2个延迟（三人合唱效果）:

    chorus=0.6:0.9:50|60:0.4|0.32:0.25|0.4:2|1.3
- 3个延迟（四人及更多合唱效果）:

    chorus=0.5:0.9:50|60|40:0.4|0.32|0.3:0.25|0.4|0.3:2|2.3|1.3

### compand ###
音频压缩或扩展的动态范围（动态压缩）

它接受下面的参数（参数值有多个时用`|`分隔，各个参数间用`：`分隔）：

- attacks
- decays

    一个以秒计时的单通道输入信号计量瞬时水平平均值的计算窗口宽度列表。`attacks`用于指示增加，`decays`用于指示衰减。对于大多数情况,增强时间(响应音频声)应该比衰减时间短,因为人类的耳朵感觉中，突然大声的音频录音比突然减弱更敏感。典型的对增强采用0.3秒，对衰减采样0.8秒。
- points

    一个要进行传输的点的列表，以dB指定相对于最大可能信号的振幅。每个关键节点由下面的语法描述： `x0/y0|x1/y1|x2/y2|....` 或者 `x0/y0 x1/y1 x2/y2 ....` 即由`|`或者空格分隔的列表。

    所有的输入值（频点）必须按递增排序（传递函数——放大倍数，不需要单调递增），点值`0/0`表示可能覆盖(对应0/输出的按dB值增益)。典型值有`-70/-70|-60/-20`
- soft-knee

    设置对所有关节点的曲线半径，默认为 0.01
- gain

    以dB为单位设置附加增益，对应于所有设置为需要获得传输的点。这允许调整整体增益，默认为0
- volume

    以dB为单位设置初始化音量，其作为开始是每个通道的假设值，即允许用户提供一个名义上的初始值。例如一个非常大的增益并不适用于初始信号在开始运作时有没有压缩。一个典型的表示最初十分安静的值是-90dB，默认为 0.
- delay

    以秒为单位设置延迟。立即输入音频分析,但音频延迟之前美联储音量调节器。指定一个延迟约等于增强/衰减时间允许滤镜有效地预测而不是被动的模式运作。它默认为0。
    
#### compand例子 ####
- 为音乐适合在嘈杂环境中听时让音乐更安静和响亮:

    compand=.3|.3:1|1:-90/-60|-60/-40|-40/-30|-20/-20:6:0:-90:0.2

    另外的例子是耳语和爆炸部分音频:

    compand=0|0:1|1:-90/-900|-70/-70|-30/-9|0/-3:6:0:0:0
- 一个噪声门，对于噪声对于音频有较低水平信号:

    compand=.1|.1:.2|.2:-900/-900|-50.1/-900|-50/-50:.01:0:-90:.1
- 另外一个噪声门, 这次噪声对于音频有更高水平信号。this time for when the noise is at a higher level than the signal (使它在某些方面,类似于压制):

    compand=.1|.1:.1|.1:-45.1/-45.1|-45/-900|0/-900:.01:45:-90:.1

### dcshift ###
将直流转换应用到音频。

这可以有助于消除直流偏置(可能由硬件问题引起的记录链)的音频。直流偏置的影响是减少空间,因此体积。`astats`滤镜可以用来确定一个信号直流偏移。

- shift

    设置直流偏置,允许值为[-1, 1].它叠加到音频上
- limitergain

    可选，它应该有一个远低于1的值(例如 0.05 或0.02)用来防止裁剪。

### earwax ###
让声音更容易在耳机听

这个为44.1kHz立体声（即CD音频格式）添加`cues`（线索）。让声音听起来像离开了耳机，是在扬声器前面（标准应该播放的环境）。

它从SoX移植来。

### equalizer ###
应用一个两极平衡（EQ）峰值滤镜。通过这个滤镜，信号电平值在选定的频率可以增强或者衰减（不想`bandpass`和`bandreject`滤镜），而其它频率不变。

为了产生复杂的平衡曲线,这个过滤器可以被使用几次,每一个都有不同的中心频率。

滤镜接受如下选项：

- frequency, f

    设置中心频点，单位 Hz.
- width_type

    设置带通滤波宽度定义类型

    h

        Hz 
    q

        Q-Factor 
    o

        octave 
    s

        slope 

- width, w

    设置带通滤波宽度，其关联`width_type`
- gain, g

    设置对应增益，单位dB
    
#### equalizer例子 ####
- 1000 Hz 增加为10dB，带宽200HZ

    equalizer=f=1000:width_type=h:width=200:g=-10
- 在1000Hz处增加为2dB,带宽Q 1，在100Hz处增加为5dB，带宽Q2:

    equalizer=f=1000:width_type=q:width=1:g=2,equalizer=f=100:width_type=q:width=2:g=-5

### flanger ###
为音频增加翻边效果

滤镜接受下面的选项：

- delay

    以微秒为单位设置延迟，范围0-30，默认为0
- depth

    以微秒为单位设置swep延迟，范围0-10，默认2
- regen

    设置再生百分比(延迟信号反馈)，范围-95-95，默认0
- width

    设置的延迟信号与原始混合比例。从0到100不等。默认值是71。
- speed

    设置每秒扫描（Hz），范围0.1-10，默认0.5
- shape

    设置波形，可能值为`triangular`或者`sinusoidal`，默认`sinusoidal`
- phase

    对多个通道设置波形转换百分比，范围0-100，默认25
- interp

    设置延迟线内插方法，可能值为`linear`或`quadratic`，默认`linear`
 
### highpass ###
指定频率3dB的高通滤波器。这个滤波器可以是单极或者双极（默认），滤波器每极有6dB倍频（每极10倍频是20dB）

滤镜接受下面选项：

- frequency, f

    设置频点，默认3000.
- poles, p

    设置极数，默认2
- width_type

    设置带宽计算模式.

    h

        Hz 
    q

        Q-Factor 
    o

        octave 
    s

        slope 

- width, w

    设置带宽，其根据`width_type`计数，仅对双极滤镜，有0.707q的.巴特沃斯响应

### join ###
把多个输入流连接成一个多通道流

它接受下面参数：

- inputs

    输入流数，默认2
- channel_layout

    通道布局，默认`stereo`
- map

    从输入流映射通道，参数是'|'分隔的字符串。每个`input_idx.in_channel-out_channel`都是映射自输入流。`input_idx` 是0起始的输入了流索引。`stream._in_chnnel`设置输出流的标识，（例如：FL对应于左前退），其选定需要更多的音视频出入。

这个滤镜可用于未显式映射的一些应用。首先试图找到一个匹配行，如果没有就把第一个未使用的视频的音频信号插入。

- 连接3个输入（正确设置通道布局）

	ffmpeg -i INPUT1 -i INPUT2 -i INPUT3 -filter_complex join=inputs=3 OUTPUT
- 从6路音频合成为5.1输出到

	ffmpeg -i fl -i fr -i fc -i sl -i sr -i lfe -filter_complex
	'join=inputs=6:channel_layout=5.1:map=0.0-FL|1.0-FR|2.0-FC|3.0-SL|4.0-SR|5.0-LFE'
	out

### ladspa ###
加载一个LADSPA(Linux音频开发人员的简和插件API)插件

编译选项：`--enable-ladspa`，接受下面的值。

- file, f

    描述LADSPA库所在文件，如果`LADSPA_PATH`环境变量被定义，`LADSPA`插件将在被逗号分隔的各个路径中查找（LADSPA_PATH中的）。否则采样标准查找顺序查找: `HOME/.ladspa/lib/`, `/usr/local/lib/ladspa/`,` /usr/lib/ladspa/`.
- plugin, p

    指定库中的插件。有些库只有一个插件，而另外一些可能有多个。如果没有设定，则指定库中所有的插件将被列出
- controls, c

    有由’|’分隔的浮点数指列表，以确定加载的行为（例如延迟、阀值或者增益） 。控制参数需要由语法：` c0=value0|c1=value1|c2=value2|...`,这样来设置第i个选项值。如果设置为`help`将输出有效的控制器和可用的打印控制参数
- sample_rate, s

    指定采样率，默认44100，仅用于插件有0号输入。
- nb_samples, n

    设置每个通道每个输出帧中包含的样本点数。默认1024，仅用于插件有0号输入。
- duration, d

    设置最小持续时间。参考[持续时间](#ffmpeg-doc-cn-07.md#持续时间)语法来描述.**注意**实际返回的结果持续时间可能大于指定时间，生成的音频总是少一个完整的帧。如果没有特别指定，或者表示持续时间的值为负数，则表明生成的音频持续不断。仅用于于插件有0号输入。 

#### ladspa例子 ####
- 列出amp（LADSPA e例子插件）库中有效插件:

    ladspa=file=amp

- 列出所有有效的控制项和有效值范围（对vcf_notch插件，其在VC库中）：

    ladspa=f=vcf:p=vcf_notch:c=help

- 利用计算机音乐工具包(CMT)插件库模拟低质量的音频设备:

    ladspa=file=cmt:plugin=lofi:controls=c0=22|c1=12|c2=12

- 使用TAP-plugins添加混响的音频(汤姆-Tom-的音频处理插件):

    ladspa=file=tap_reverb:tap_reverb

- 产生白噪声,有.2振幅:

    ladspa=file=cmt:noise_source_white:c=c0=.2

-  利用Metronome from the C* Audio Plugin Suite (CAPS)库中的C* Click插件，产生20bmp的内容:

    ladspa=file=caps:Click:c=c1=20'
- 应用C* Eq10X2 - Stereo 10段均衡效应:

    ladspa=caps:Eq10X2:c=c0=-48|c9=-24|c3=12|c4=2

### Commands ###
它支持下面的命令：

- cN

	编辑`N-th`控制值

	如果指定的值无效，会忽略它。

### lowpass ###
应用3dB频点倍带宽的低通滤波器。它可以是单极或者双极的（默认）。滤镜每个8度有6dB的衰减（20dB 则是10倍）

滤镜接受下面的选项：

- frequency, f

    设置频点，默认500.
- poles, p

    设置极数，默认2
- width_type

    设置带宽计算模式.

    h

        Hz 
    q

        Q-Factor 
    o

        octave 
    s

        slope 

- width, w

    设置带宽，其根据`width_type`计数，仅对双极滤镜，有0.707q的.巴特沃斯响应 

### pan ###
按指定的增益关系混合。滤镜接受通道布局和一组通道定义

这个滤镜也可以有效的重新映射通道音频流。

滤镜接受的参数格式为："l|outdef|outdef|..." 

- l

    输出通道布局或者通道号
- outdef

    输出通道指定，格式: "out_name=[gain*]in_name[+[gain*]in_name...]"
- out_name

    输出通道名，每个通道 (FL, FR, 等) 或者通道索引数 (c0, c1, etc.)
- gain

    增益倍数，1表示不变
- in_name

    采用的输入通道，参考`out_name`的介绍。它不能是混合了名字和索引号的输入通道（描述） 

如果在通道描述中有‘=’而不是‘<’,则表明对指定通道总是按1倍重整（表明不变），从而避免削波噪音

#### pan的混合例子 ####
例如，如果想下变换立体声为单声道，而且更大的权重是在左声道：

	pan=1c|c0=0.9*c0+0.1*c1
一个定制的下变换工作与 3- 4- 5- 和7- 通道环绕

	pan=stereo| FL < FL + 0.5*FC + 0.6*BL + 0.6*SL | FR < FR + 0.5*FC + 0.6*BR + 0.6*SR
	
#### pan的再映射例子 ####
通道再映射仅在下面的情况起效：

- 增益为0或者1
- 每个输入仅有一个通道输出

如果这些条件都满足,滤镜将通知用户("Pure channel mapping detected"-“纯通道映射发现”),并使用一个优化和无损方法重新映射。

例如：如果有一个5.1声道要映射立体声，去除扩展通道：

	pan="stereo| c0=FL | c1=FR"

对于同一个源，你也可以交换左前和右前，保持输入布局：

	pan="5.1| c0=c1 | c1=c0 | c2=c2 | c3=c3 | c4=c4 | c5=c5"

如果是立体声源，要对左前通道静音（但保持立体声通道布局）：

	pan="stereo|c1=c1"

仍然是立体声，把右前用两次：

	pan="stereo| c0=FR | c1=FR"

### replaygain ###
ReplayGain扫描仪滤镜。这个滤镜以一个音频流作为输入和输出也不改变。在过结束后显示`track_gain`和`track_peak`。

### resample ###
转换音频采样格式，采样率和通道布局，它一般不直接使用。

### silencedetect ###
检测一个音频流中的静音。

这个滤镜是在检测到输入音频小于或等于一个噪音公差值，且持续时间大于或等于最低噪音持续时间时输出日志消息

输出的时间和持续时间以秒为单位

滤镜接受下面的选项：

- duration, d

    设置需通告的静默持续时间(默认为2秒).
- noise, n

    设置噪声限，可以采样为dB描述 (指附加值的dB表示)  或者振幅比，默认为-60dB或者0.001 0.001. 

#### silencedetect例子 ####
- 检测5秒静默，-50dB的噪音限

	silencedetect=n=-50dB:d=5
- 噪音限为0.0001 检测静默（静默时长2秒）

	ffmpeg -i silence.mp3 -af silencedetect=noise=0.0001 -f null -

### silenceremove ###
从音频的开始、中间或者结束删除静默

滤镜接受如下选项：

- start_periods

    这个值用来指定在开始时应该作为静默削减的音频幅度，0表示不削减，指定一个非0值则表示直到找到一个非静默值时的都被削减掉。通常该值为1，而更高的值甚至可以削减掉所有的音频。默认为0
- start_duration

    指定一个非静默持续时间阀值，如果非静默时间超过该阀值则不被削减，否则记为连续静默中的脉冲噪音。通过增加这个值，脉冲噪声可以视为静默被修剪掉，缺省为0
- start_threshold

    这个用于指出该作为静默的样本值。对于数字音频，值为0肯定很合适作为表明是静默的，但对于模拟音频（ADC获取的），你可能希望增加这个值（作为背景噪音），可以指定dB值或者振幅比，默认为0
- stop_periods

    指定为从音频削减沉默数。为了从中间削减沉默，需要为`stop_periods`指定一个负数值。这个值被视为有积极的价值,用于显示效果应该重启start_periods规定的处理,使其适合于去除的沉默的音频。默认值是0。
- stop_duration

    指定一个时间的沉默之前,必须存在音频不是复制，通过指定一个更高的持续时间，沉默会更多的留在音频中，默认为0.
- stop_threshold

    这个值类似`start_threshold`,但是是从音频末尾削减。也可以用dB值或者振幅比指定，默认为0
- leave_silence

    这表明`stop_duration`长度的音频应该原封不动的在每个周期的开始沉默，例如如果你想删除长单词之间的停顿,但不想完全删除停顿。默认值是0。

#### silenceremove例子 ####
- 下面的示例说明了如何使用这个过滤器开始录音,而不包含通常发生的按键后的延迟按，即按下键到开始记录之间的静默：

	silenceremove=1:5:0.02

### treble ###
对频点的3倍（上下）利用双刀搁置（two-pole shelving）滤镜增加或者减少频率响应，类似于高保真的音控，也被称为搁置平衡（EQ）

滤镜接受下面选项：

- gain, g

    Give the gain at whichever is the lower of ~22 kHz and the Nyquist frequency. Its useful range is about -20 (for a large cut) to +20 (for a large boost). Beware of clipping when using a positive gain.
- frequency, f

    设置频点，默认3000Hz.
- width_type

    设置带宽计算模式.

    h

        Hz 
    q

        Q-Factor 
    o

        octave 
    s

        slope 

- width, w

    设置带宽，其根据`width_type`计数，仅对双极滤镜，有0.707q的.巴特沃斯响应 

### volume ###
调整输入音量

接受下面的参数：

- volume

    设置音频音量表达式

    输出值是剪辑的最大值

    输出音频音量有如下关系：:

    output_volume = volume * input_volume

    默认为 "1.0".
precision

    这个参数指出数学（计算）精度

    它决定哪些输入样本格式将被允许,这影响伸缩量的精度.

    fixed

        8-bit整形，它限于输入是 U8, S16, 和 S32. 
    float

        32-bit 浮点，它限于输入格式是FLT，这是默认值
    double

        64-bit 浮点;它限于输入格式是DBL 

replaygain

    选择当输入帧数据遇到`ReplayGain`（重演设置——音量增益）时的处理

    drop

        丢掉ReplayGain，按原始标准 (默认值).
    ignore

        忽略ReplayGain侧数据,但是离开它的框架
    track

        采用轨道设置,如果存在
    album

        采用专辑设置,如果存在 

replaygain_preamp

    在应用`replaygain`的前置放大增益，单位dB

    replaygain_preamp默认值为0.0.
eval

    设置如果计算音量表达式

    它接受以下值:

    ‘once’

        仅在初始化时计算一次，或者`volume`命令被发生时
    ‘frame’

        在每个输入帧都重新计算 

    默认‘once’. 

音量表达式支持下面的参数：.

- n

    帧数 (从0开始计数) 
- nb_channels

    通道数 
- nb_consumed_samples

    总的经过滤镜的样本点数 
- nb_samples

    当前帧中样本点数 
- pos

    在文件中的帧偏移 
- pts

    帧的PTS 
- sample_rate

    采样率 
- startpts

    流开始来的PTS计数 
- startt

    流开始来的时间 
- t

    帧时间 
- tb

    时间戳时基 
- volume

    最近设置的音量值 

**注意**如果计算模式是`once`，则除了`sample_rate`和`tb`有效外其他都无效而等于`NAN`

#### volume命令 ####
滤镜支持下面的命令：

- volume

	编辑音量表达式。这个命令接受相同选项和语法作为命令参数

	如果描述的表达式无效，它将保持当前值
- replaygain_noclip

	通过限制防止剪裁

	默认对于`replaygain_noclip`为1

#### volume例子 ####
- 调整输入音量

	volume=volume=0.5
	volume=volume=1/2
	volume=volume=-6.0206dB

	在上面的例子中所指定的选项名`volume`可以省略，例如：

	volume=0.5
- 输入音频功率增加6dB，使用定点精度

	volume=volume=6dB:precision=fixed
- 一个音频在10秒后5秒内逐渐削弱效果

	volume='if(lt(t,10),1,max(1-(t-10)/5,0))':eval=frame

### volumedetect ###
检测输入音频音量

滤镜没有参数，输入也不会被编辑。统计数据将在输入流结束后输出到日志中。

特别是它显示平均音量（均方根），最大音量（每个样本基础上）和开始来的音量直方图（从最大音量累计1/1000样本）

所有的音量都是相对于最大PCM值的

#### volumedetect例子 ####
这里有一个输出实例：

	[Parsed_volumedetect_0  0xa23120] mean_volume: -27 dB
	[Parsed_volumedetect_0  0xa23120] max_volume: -4 dB
	[Parsed_volumedetect_0  0xa23120] histogram_4db: 6
	[Parsed_volumedetect_0  0xa23120] histogram_5db: 62
	[Parsed_volumedetect_0  0xa23120] histogram_6db: 286
	[Parsed_volumedetect_0  0xa23120] histogram_7db: 1042
	[Parsed_volumedetect_0  0xa23120] histogram_8db: 2551
	[Parsed_volumedetect_0  0xa23120] histogram_9db: 4609
	[Parsed_volumedetect_0  0xa23120] histogram_10db: 8409

它意味着：

- 平均音量是-27dB,或 10……-2.7
- 最大音量点为-4dB或者说介于-4dB 到-5dB
- 有6个样本点是-4dB，62个-5dB，286个-6dB 等等

换句话说，提供+4dB的音量不会引起任何剪裁（削波），而提高+5dB就有6个地方会削波。
