## 35 音频源
下面介绍当前可用的音频源

### abuffer ###
缓冲音频帧，作为滤镜链图中有效的组成（起点）

它主要编程使用，特别是通过`libavfilter/asrc_abuffer.h`中的接口进行调用。

接受如下参数：

- time_base

    用于提交帧的时间戳时基。是浮点数或者分数形式。
- sample_rate

    进入音频缓冲的采样率。
- sample_fmt

    进入音频缓冲的采样格式。`libavutil/samplefmt.h`下`AVSampleFormat`枚举值中的一个格式名称或者对应的整数
- channel_layout

    进入音频缓冲的通道布局。为`libavutil/channel_layout.c`中的`channel_layout_map`定义的布局名称或者`libavutil/channel_layout.h`中 `AV_CH_LAYOUT_*`类宏（对应的整数表示）
- channels

    进入缓冲的通道数。如果`channels` 和 `channel_layout`同时被设置，则二者必须一致。
#### abuffer例子 ####

	abuffer=sample_rate=44100:sample_fmt=s16p:channel_layout=stereo
会用来指出源接受16位（信号）立体声（采样率44100Hz）。`sample_fmt`为`s16p`即`6`，`channel_layout`为`stereo`，即`0x3`，`sample_rate`为`44100`，它等效于

	abuffer=sample_rate=44100:sample_fmt=6:channel_layout=0x3

### aevalsrc ###
按表达式生成一个音频信号（信号发生器）

它接受一个或者多个表达式（每个对应一个通道），根据表达式计算产生相应的音频信号。

接受如下的选项：

- exprs

    由’|’分隔的表达式列表，每个表达式对应一个通道。以防`channel_layout`没有指定选项，选中的通道布局取决于提供的数量表达式。否则最后指定表达式应用于剩下的输出通道。
- channel_layout, c

    设置通道布局。这里的通道数必须等于表达式数量。
- duration, d

    设置源音频持续时间。参考`ffmpeg-utils`工具集手册（ffmpeg-utils(1)）中的[持续时间](ffmpeg-doc-cn-07.md#持续时间)章节内容以了解语法。**注意**由此生成的音频持续时间可能会超过这里指定的时间，因为生成的音频最少是一个完整的帧内容。

    如果不指定，或者指定一个非负数，表面会持续生成音频信号。
- nb_samples, n

    设置每个输出帧中每个通道的样例数量，默认1024。
- sample_rate, s

    指定采样频率，默认44100. 

每个表达式可以包含下面的常量:

- n

    评估样本的数量，从0开始计数
- t

    样本时间表示，从0开始计时
- s

    样本采样率
#### aevalsrc例子 ####
- 生成静音（无声）:

    aevalsrc=0
- 生成频率为440Hz的正弦波，采样频率8000Hz:

    aevalsrc="sin(440*2*PI*t):s=8000"
- 生成双路信号，这里指定为（中前和中后），表达式为:

    aevalsrc="sin(420*2*PI*t)|cos(430*2*PI*t):c=FC|BC"
- 生成白噪声:

    aevalsrc="-2+random(0)"
- 生成一个振幅调制信号:

    aevalsrc="sin(10*2*PI*t)*sin(880*2*PI*t)"
- 生成2.5赫兹双耳节拍在360赫兹的载体:

    aevalsrc="0.1*sin(2*PI*(360-2.5/2)*t) | 0.1*sin(2*PI*(360+2.5/2)*t)"

### anullsrc ###
null（空）音频源会产生未处理的音频帧。它一般用于分析/调试，或作为滤镜可忽略的输入源（例如`sox`合成滤镜）

这个源接受下面选项：

- channel_layout, cl

    指定通道布局，可以是整数或对应的短语，默认为"stereo".

    检查定义在`libavutil/channel_layout.c`了解短语和数字值对应关系。
- sample_rate, r

    指定采样率,默认 44100.
- nb_samples, n

    设置每帧中样例数量

#### anullsrc例子 ####
- 以采样率48000 Hz ，单声道(`AV_CH_LAYOUT_MONO`).

    anullsrc=r=48000:cl=4
- 同上的效果（采样短语定义布局）:

    anullsrc=r=48000:cl=mono

所有选项参数都必须明确定义。

### flite ###
使用libflite库合成声音话语

编译选项是`--enable-libflite`

**注意**`flite`库不是线程安全的。

接受如下选项：

- list_voices

    如果为1，列出有效的语音并退出，默认0.
- nb_samples, n

    设置每个帧最大样例数量，默认512.
- textfile

    设置要朗读的文件名
- text

    设置要朗读的文本
- voice, v

    设置语音合成的声音，默认`kal`. 参考`list_voices`选项

#### flite例子 ####
- 从文件speech.txt读，使用标准声音合成:

    flite=textfile=speech.txt
- 读取指定文本，并用`slt`语音合成:

    flite=text='So fare thee well, poor devil of a Sub-Sub, whose commentator I am':voice=slt
- 作为ffmpeg输入:

    ffmpeg -f lavfi -i flite=text='So fare thee well, poor devil of a Sub-Sub, whose commentator I am':voice=slt
- 播放合成语音:

    ffplay -f lavfi flite=text='No more be grieved for which that thou hast done.'

关于`libflite`库的更多信息，确认[http://www.speech.cs.cmu.edu/flite/](http://www.speech.cs.cmu.edu/flite/) 

### sine ###
生成一个音频信号的振幅的正弦波1/8

是一个bit-exact音频信号（脉冲？）

接受如下选项：
- frequency, f

    设置载波频率，默认 440 Hz.
- beep_factor, b

    每个`beep_factor`倍载波频率周期产生一个`beep`，默认为0，表示`beep`被禁止
- sample_rate, r

    指定采样率，默认44100.
- duration, d

    指定产生音频持续时间
- samples_per_frame

    设置每帧样例数，默认1024

#### sine例子 ####
- 产生440Hz的`sine`波Generate a simple 440 Hz sine wave:

    sine
- 产生220Hz`sine`波，且880Hz产生一个`beep`，持续5秒:

    sine=220:4:d=5
    sine=f=220:b=4:d=5
    sine=frequency=220:beep_factor=4:duration=5

