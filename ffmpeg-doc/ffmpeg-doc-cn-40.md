## 40 多媒体滤镜 ##
下面介绍当前有效的多媒体滤镜
### avectorscope ###
转换输入音频到视频输出以代表音频矢量范围（一种图形化音频处理）

这个滤镜用来测量立体声音频中两路音频间的区别。如果是单声道信号做成的2个声道（左右耳声道），因为两路完全相同（其实只有1路），所以输出是一个垂直的直线（表示二者无差别）。如果是立体声信号（两路肯定或多或少有差别），则创建一个利萨如（Lissajous）图形，其水平看，线长度与相位等表征了两个声道差异情况。

滤镜接受如下选项：

- mode, m

    设置矢量显示模式，有效值为:

    ‘lissajous’

        利萨如(Lissajous)旋转45度.
    ‘lissajous_xy’

        同上，但不旋转. 

    默认‘lissajous’.
- size, s

    设置输出视频尺寸。语法同于`ffmpeg-utils`手册中的[视频尺寸](ffmpeg-doc-cn-07.md#视频尺寸分辨率)章节。默认400x400.
- rate, r

    设置输出帧率，默认25.
- rc
- gc
- bc

    分别用于指定红、绿、蓝颜色（表征对比图形的颜色）。分别默认为 40, 160 和80. 各个值的范围在 [0, 255].
- rf
- gf
- bf

    分别指定淡入淡出时红、绿、蓝颜色， 分别默认值为15, 10和5.分别值范围为[0, 255].
- zoom

    设置缩放因子，默认为1，允许[1, 10]. 

#### avectorscope例子 ####
- 利用ffplayer播放

	ffplay -f lavfi 'amovie=input.mp3, asplit [a][out1];
             [a] avectorscope=zoom=1.3:rc=2:gc=200:bc=10:rf=1:gf=8:bf=7 [out0]'

### concat ###
连接音频和视频流,把加入的一个接一个的在一起

这个滤镜用于按段同步视频和音频流。所有的段都必须有相同的流（类型和个数），输出也是相同流（类型和个数）

滤镜支持如下选项：

- n

    设置段的数量，默认为2
- v

    设置输出中视频流数量，则每个段中必须有输入视频流的数量。默认为1

- a

    设置输出中音频流数量，则每个段中必须有输入音频流的数量，默认为0
- unsafe

    激活不安全模式，这时如果段中有不同格式不会失败

滤镜有 `v+a`的输出：先是一个视频输出，然后是音频输出。

有`n x (v+a)`：有n段输出，每段都是`v+a`。

相关的流并不总是有相同的时间，由于各种原因还包括不同的编解码帧大小或创作草稿。因此相关同步流（视频和对应音频）要连接，`concat`滤镜将选择持续最长的流（视频的）为基准（除最后段的流），在每个流播放时通过让音频流垫长（重复部分）或者静默（截断）来实现视频流连续。

为了让滤镜工作正常，所有段都必须以0为时间戳开始。

所有应用的流在所有共同的领域必须有相同的参数，滤镜会自动选择一个通用的像素格式（色彩标注、编码颜色的标准和位深等），以及音频采样率和通道布局。但其他设置如视频分辨率必须由用户显式转换。

不同的帧率是可以接受的，但会导致输出帧率的变化，一定要配置输出文件来处理。

#### concat例子 ####
- 连接一个有双语版的多段视频成为整体（视频中0号流，音频在1号和2号流）

	ffmpeg -i opening.mkv -i episode.mkv -i ending.mkv -filter_complex \  
	'[0:0] [0:1] [0:2] [1:0] [1:1] [1:2] [2:0] [2:1] [2:2] concat=n=3:v=1:a=2 [v] [a1] [a2]' \ 
	-map '[v]' -map '[a1]' -map '[a2]' output.mkv

- 连接2个部分，分别处理音频和视频，使用电影来源标准调整分辨率。

	movie=part1.mp4, scale=512:288 [v1] ; amovie=part1.mp4 [a1] ;movie=part2.mp4, scale=512:288 [v2] ;movie=part2.mp4 [a2] ;[v1] [v2] concat [outv] ; [a1] [a2] concat=v=0:a=1 [outa]

	**注意**在开始的不同步现象会发生在音频和视频没有完全相同持续时间情况下。

### ebur128 ###
EBU R128 扫描滤镜，这个滤镜需要一个音频流，但会原样输出。默认情况下，会显示10H更新频率下，的瞬时响度（M）、短期响度（S）、集成响度（I）和响度范围（LRA）

滤镜有一个实时视频输出，展示响度变化。因为上述图像不停更新，所以它不可打印化输出，除非详细日志被设置。主要的绘图区域包含短期响度（3秒分析），以及其后的瞬时响度（400毫秒）

关于EBU R128响度滤镜的更多信息参考[http://tech.ebu.ch/loudness](http://tech.ebu.ch/loudness)

滤镜介绍如下选项：

- video

    激活视频输出。无论怎么设置，音频流在过程中都不会变化（从输入到输出），如果激活，则视频将作为第一个输出流。默认为0
- size

    设置视频尺寸。语法同于`ffmpeg-utils`手册中的[视频尺寸](ffmpeg-doc-cn-07.md#视频尺寸分辨率)章节。默认640x480.
- meter

    设置EBU的规模计。默认为9，通用值是9和18，对应于规模计+9和+18.任何非负整数是可以的。
- metadata

    设置添加的元数据。如果为1，音频输入将被划分为100毫秒的输出帧。他们每个都包含各种响应信息元数据。所有的元数据键前又缀有lavfi.r128..

    默认为0.
- framelog

    强制帧日志层次，允许值:

    ‘info’

        信息日志层次 
    ‘verbose’

        冗长日志层次 

    默认为`info`，如果设置视频或者元数据则选择`verbose`
- peak

    设置峰顶模式

    有效的模式可以累积（可选标志类型），可能的值:

    ‘none’

        没有任何峰顶模式（默认） 
    ‘sample’

        允许sample-peak模式 mode.

        简单的峰值模式，其只寻找高样本值. 在日志中有一个`sample-peak`消息 (标记有SPK). 
    ‘true’

        允许true-peak模式

        如果启用,峰值查找是一个over-sampled版本的输入流，其峰值精度更佳，在日志中有一个`true-peak`消息 (标记有TPK)，且每个帧有一个`true-peak` (标记有FTPK)。这种模式需要与libswresample构建。

#### ebur128例子 ####
- 利用EBU 放缩规模计+18d 的实时图像

	ffplay -f lavfi -i "amovie=input.mp3,ebur128=video=1:meter=18 [out0][out1]"
- 在ffmpeg中分析

	ffmpeg -nostats -i input.mp3 -filter_complex ebur128 -f null -

### interleave和ainterleave ###
从多个输入中暂时交错帧，`interleave`用于视频输入，`ainterleave`用于音频输入。

这个滤镜从多个输入读取帧，然后然后按一定顺序（最老的先发送）发送给输出。

输入流必须有定义良好的、递增的帧时间戳。

为了提交一帧输出,滤镜需至少在一个框架内为每个输入排序,所以不能工作在一个输入还没有终止情况,也不会接收传入帧（持续生存的）

例如考虑一个输入是`select`滤镜其可以按帧丢弃，`interleave`滤镜会保持从输入读取，但不发送任何帧到输出直到输入发送来`end-of-stream`信号。

另外，根据输入同步，滤镜将丢帧，以防止一个输入收到比其他帧(时间戳)偏离太多，和队列满时

滤镜接受下面选项：

- nb_inputs, n

	设置不同数量的输入，默认为2
#### interleave和ainterleave例子 ####
- 使用ffmpeg交错不同的流：

	ffmpeg -i bambi.avi -i pr0n.mkv -filter_complex "[0:v][1:v] interleave" out.avi
- 添加闪烁模糊效果：

	select='if(gt(random(0), 0.2), 1, 2)':n=2 [tmp], boxblur=2:2, [tmp] interleave

### perms和aperms ###
为输出帧设置读/写权限

这些滤镜主要针对开发人员测试下面的滤镜链图的直接路径滤镜

这个滤镜接受下面选项：

- mode
	
	选择权限模式，接受下面的值：
	 
	‘none’
	
	    什么都不做，是默认值 
	‘ro’
	
	    设置所有输出只读 
	‘rw’
	
	    设置所有输出可读写 
	‘toggle’
	
	    翻转，即原来是只读则为可读写，如果原来是可读写则为只读
	‘random’
	
	    随机为只读或者可读写

 
- seed

	设置`mode`为`random`时的随机种子。必须为0-`UINT32_MAX`间的整数。如果不指定，或者指定为-1，则滤镜尝试自动选择一个好的随机种子。

**注意**实际在许可-`permission`滤镜和其跟随的滤镜之间可能会自动插入滤镜，则许可滤镜可能不会有预期的效果给随后的滤镜，要避免这样的问题，可以在许可滤镜之前插入`format`或者`aformat`滤镜（分别针对`perms`/`aperms`）。

### select和aselect ###
选择一些帧来输出

滤镜接受下面的选项：

- expr, e

    设置一个表达式，它拥有对每个输入帧进行评估（计算）

    如果表达式计算结果为0，则该帧被丢弃

    如果计算结果为负或者NaN，则帧被发送给第一个输出，否则发送给指定数 ceil(val)-1 的输出,假设输入索引从0开始。

    例如 值为1.2会被发送给 ceil(1.2)-1 = 2-1 = 1, 即第2路输出.
- outputs, n

    设置输出路数。输出对应于选择帧输出的结果，默认为1

表达式允许下面的内容:

- n

    顺序化的滤镜帧，从0开始
- selected_n

    被选择的（顺序化）帧，从0开始计数
- prev_selected_n

    前一选择（顺序）帧，如果未定义则为`NAN`.
- TB

    输入时间戳的时间基准
- pts

     PTS，滤镜后视频帧（表示时间戳）分，采用TB为单位，如果未定义则为`NAN`
- t

    PTS按秒为单位，滤镜后视频帧（时间戳），如果为定义则为`NAN`
- prev_pts

    前一滤镜后视频帧PTS， 如果为定义则为`NAN`
- prev_selected_pts

    前面最后的滤镜视频帧的PTS， 如果为定义则为`NAN`
- prev_selected_t

    前面最后被选择的滤镜后视频帧的PTS， 如果为定义则为`NAN`
- start_pts

    视频中第一个视频帧的PTS， 如果为定义则为`NAN`
- start_t

    视频的开始时间. 如果为定义则为`NAN`
- pict_type (video only)

    滤镜帧的类型,它可以是下面的值:

    I
    P
    B
    S
    SI
    SP
    BI

- interlace_type (video only)

    帧交错类型，可以是下面的值:

    PROGRESSIVE

        逐行 (非交错). 
    TOPFIRST

        top-field-first类型帧 
    BOTTOMFIRST

        bottom-field-first类型帧 

- consumed_sample_n (audio only)

    当前帧之前选定的样本的数量
- samples_n (audio only)

    样品在当前帧的数量
- sample_rate (audio only)

    输入样本率
- key

    为1则滤镜框架是一个帧，否则为0
- pos

    滤镜帧在文件中的偏移位置，-1为不可用（如合成视频）
- scene (video only)

    值在0-1间，表面一个新场景。较低的值反映当前帧的低概率引入一个新场景，更高的值意味着更可能是同一个（场景，见下面的例子）

表达式的默认值是1 
#### select和aselect例子 ####
- 选择所有的输入

    select

    这个例子等效于:

    select=1
- 跳过所有的帧:

    select=0
- 仅选择I帧:

    select='eq(pict_type\,I)'
- 每100帧选择1帧:

    select='not(mod(n\,100))'
- 只选择10-20时间间隔的帧:

    select=between(t\,10\,20)
- 只选择10-20时间间隔的I帧:

    select=between(t\,10\,20)*eq(pict_type\,I)
- 选择帧的最小距离为10秒:

    select='isnan(prev_selected_t)+gte(t-prev_selected_t\,10)'
- 使用`aselect`选择包含采样大于100的音频帧:

    aselect='gt(samples_n\,100)'
- 创建一个马赛克开始场景:

    ffmpeg -i video.avi -vf select='gt(scene\,0.4)',scale=160:120,tile -frames:v 1 preview.png

    对比场景对一个值在0.3和0.5之间通常是一个合理的选择
- 奇数和偶数帧分别发送给不同的输出，并组合:

    select=n=2:e='mod(n, 2)+1' [odd][even]; [odd] pad=h=2*ih [tmp]; [tmp][even] overlay=y=h

### sendcmd和asendcmd ###
在滤镜链图中向滤镜发送命令

这个滤镜读取命令来发送给滤镜链图中的滤镜

其中`sendcmd`用于两个视频滤镜间，`asendcmd`用于两个音频滤镜间。除此之外它们采用相同方式。

命令可以是规范的滤镜`commands`选项的参数或者指定在`filename`文件选项所指文件中。

它接受如下选项：

- command, c

	设置要发送的命令
- filename, f

	指定一个文件，读取其中命令进行发送

#### sendcmd和asendcmd命令语法 ####
命令是一个序列间隔规则（由`；`分隔），包括一个特定事件时段执行的命令列表。触发事件通常是当前帧的时间达到或者离开一个给定的时间段。

一个时间段命令由以下语法描述：

	START[-END] COMMANDS;
时间段描述由`START`和`END`（可选，默认为最大时间）指定

判断当前帧是否在指定时间段`[START,END)`，既判断当前帧的时间是否大于等于`START`，且小于`END`。

这里`COMMANDS`是对应于时间段的一个或者由`，`分隔的多个命令序列。一个命令的语法规范是：

	[FLAGS] TARGET COMMAND ARG
这里`FLAGS`是可选的，用于指定事件类型与指定时间段关系的（命令发生发送频次），必须由非空标识符，以及`+`或`|`和`[]`封闭的字符，标志有：

- enter 
	
	命令在当前时间戳达到时间段时发送，换句话说命令发送的前一帧时间戳还不满足时间段要求，而当前已经是满足要求的
- leave

	命令在当前时间戳离开时间段时发送，换句话说，命令发送时的前一帧还满足时间段要求，而当前已经不满足了

如果`FLAGS`没有指定，则默认为`[enter]`。

这里`TARGET`是指定命令发送的目标，通常为滤镜类型名字或者滤镜实例名字。而`COMMAND`是发送给目标的命令名字，`ARG`是对应的选项参数。

一个简单的BNF 规格的命令描述语法如下：

	COMMAND_FLAG  ::= "enter" | "leave"
	COMMAND_FLAGS ::= COMMAND_FLAG [(+|"|")COMMAND_FLAG]
	COMMAND       ::= ["[" COMMAND_FLAGS "]"] TARGET COMMAND [ARG]
	COMMANDS      ::= COMMAND [,COMMANDS]
	INTERVAL      ::= START[-END] COMMANDS
	INTERVALS     ::= INTERVAL[;INTERVALS]

#### sendcmd和asendcmd例子 ####

- 指定音频节奏（`tempo`）变化从4秒开始：

	asendcmd=c='4.0 atempo tempo 1.5',atempo
- 帧文件中指定一个`drawtext`和`hue`命令

	\# 在时间5-10间显示一个文字
	5.0-10.0 [enter] drawtext reinit 'fontfile=FreeSerif.ttf:text=hello world',
	         [leave] drawtext reinit 'fontfile=FreeSerif.ttf:text=';
	
	\# 在时间15-20淡化图像
	15.0-20.0 [enter] hue s 0,
	          [enter] drawtext reinit 'fontfile=FreeSerif.ttf:text=nocolor',
	          [leave] hue s 1,
	          [leave] drawtext reinit 'fontfile=FreeSerif.ttf:text=color';
	
	\# 从时间25开始应用指数饱和渐变效果
	25 [enter] hue s exp(25-t)

	滤镜链图中读取然后处理前面的命令列表（存在`test.cmd`文件中），可以是如下处理：

	sendcmd=f=test.cmd,drawtext=fontfile=FreeSerif.ttf:text='',hue

### setpts和asetpts ###
修改发布时间戳（PTS-presentation timestamp），对于输入帧。

其中`setpts`对于视频帧，`asetpts`对于音频帧

滤镜允许如下选项：

- expr

    指定用于计算时间戳的表达式

表达式通过`eval`API和下面一些常量来计算:

- FRAME_RATE

    帧率，仅对于指定了帧率的视频
- PTS

    输入的PTS
- N

    对输入视频帧计数或已经消耗的样本计数，不包括音频的当前帧，从0开始
- NB_CONSUMED_SAMPLES

    采样数（因为音频是按固定频率采样，则一个采样其实就是自然的计时单位——类似帧率），不包括当前帧（仅对音频）
- NB_SAMPLES, S

    当前帧中的采样数 (仅音频)
- SAMPLE_RATE, SR

    音频采样率
- STARTPTS

    第一帧的PTS
- STARTT

    第一帧的按秒时间
- INTERLACED

    指示当前帧是否交错
- T

    当前帧的按秒时间
- POS

    初始位置在文件中的偏移，为当前帧，如果为定义则本值也为定义
- PREV_INPTS

    前一个帧的PTS.
- PREV_INT

    前一帧按秒时间
- PREV_OUTPTS

    前一帧的输出PTS.
- PREV_OUTT

    前一帧按秒输出时间
- RTCTIME

    时间单位为微秒-microseconds. 现在被弃用，使用time(0)时
- RTCSTART

    影片开始时间以微秒为单位
- TB

    输入时间戳时基


#### setpts和asetpts例子 ####
- 从0开始计数PTS

    setpts=PTS-STARTPTS
- 应用快速监看效果

    setpts=0.5*PTS
- 应用慢速监看效果

    setpts=2.0*PTS
- 强制为25的帧率:

    setpts=N/(25*TB)
- 设置有抖动的25帧率:

    setpts='1/(25*TB) * (N + 0.05 * sin(N*2*PI/25))'
- 应用一个10秒的输入偏置:

    setpts=PTS+10/TB
- 从`直播源`和变基生成时间戳转换到当前时基时间戳:

    setpts='(RTCTIME - RTCSTART) / (TB * 1000000)'
- 按当前采样率生成时间戳:

    asetpts=N/SR/TB

### settb和asettb ###
设置输出帧时间戳的时基。它主要用于测试时基配置。

其可以接受如下选项：

- expr， tb

	用于计算输出时基的表达式或者值

这里的`tb`值是一个可得出有理数值的表达式或者值。常数可以包含“AVTB”(默认时基),“intb”(输入时基)和“sr”(采样率,只音频)。默认值是“intb”

#### settb和asettb例子 ####
- 设置时基为1/25

	settb=expr=1/25
- 设置时基为1/10

	settb=expr=0.1
- 设置时基为1001/1000

	settb=1+0.001
- 设置时间为2倍intb

	settb=2*intb
- 设置为默认值

	settb=AVTB

### showcqt ###
转换音频输入为一个频谱视频输出（用恒Q变换Brown-Puckette算法），乐音的规模从`E0`到`D#10`(10个8度)

滤镜接受如下选项：

- volume

    指定转换表达式（对音量的乘数），可含如下变量:

    frequency, freq, f

        转换中评估的频率 
    timeclamp, tc

        timeclamp选项值 

    和下面的函数:

    a_weighting(f)

        一个权重等响度A-weighting
    b_weighting(f)

        一个权重等响度B-weighting 
    c_weighting(f)

        一个权重等响度C-weighting 

    默认值16
- tlength

    指定转换长度表达式，表达式可包含如下变量:

    frequency, freq, f

        转换中评估的频率 
    timeclamp, tc

        timeclamp选项值

    默认值384/f*tc/(384/f+tc).
- timeclamp

    指定转换timeclamp，对于低频，精度在时间域和频率域间权衡，如果timeclamp较低,在时间域表示更准确(如快速低音鼓),否则在频域表示(如低音吉他)更准确。可接受的值是[0.1,1.0]。默认值是0.17。
- coeffclamp

    指定转换coeffclamp. 如果coeffclamp低，转换更准确,否则转换速度更快。可接受的值是[0.1,10.0]。默认值是1.0。
- gamma

    指定gamma。低的`gamma`值有更多比较细节高的则有更大的比较范围，可接受值范围 [1.0, 7.0]默认为3.0.
- gamma2

    指定采用柱状图像gamma，接受值范围 [1.0, 7.0]. 默认为 1.0.
- fontfile

    指定freetype字体文件，如果不知道则采用嵌入的字体
- fontcolor

    指定文字颜色。是一个返回整数的算术表达式值-0xRRGGBB可以包含下面的变量:

    frequency, freq, f

        转换中评估的频率 
    timeclamp, tc

        timeclamp选项值

    和函数:

    midi(f)

        midi数频率，一些midi数为: E0(16), C1(24), C2(36), A4(69) 
    r(x), g(x), b(x)

        红、绿、蓝的值 

    默认值为 st(0, (midi(f)-59.5)/12); st(1, if(between(ld(0),0,1), 0.5-0.5*cos(2*PI*ld(0)), 0)); r(1-ld(1)) + b(ld(1))
- fullhd

    如果为1 (默认值),视频分辨率为1920x1080 (full HD),如果设置为0，则视频分辨率为960x540,使用这个值可以降低CPU使用
- fps

    指定视频帧率——fps，默认25.
- count

    指定每帧中的转换数，fps*count是1秒中的转换数。**注意**音频数据率必须是fps*count的整数倍。默认为6

#### showcqt例子 ####
- 播放音频显示频谱：

    ffplay -f lavfi 'amovie=a.mp3, asplit [a][out1]; [a] showcqt [out0]'
- 同上，但使用30的帧率:

    ffplay -f lavfi 'amovie=a.mp3, asplit [a][out1]; [a] showcqt=fps=30:count=5 [out0]'
- 频谱为960x540和降低CPU使用率:

    ffplay -f lavfi 'amovie=a.mp3, asplit [a][out1]; [a] showcqt=fullhd=0:count=3 [out0]'
- A1 和其谐波: A1, A2, (near)E3, A3:

    ffplay -f lavfi 'aevalsrc=0.1*sin(2*PI*55*t)+0.1*sin(4*PI*55*t)+0.1*sin(6*PI*55*t)+0.1*sin(8*PI*55*t),
                     asplit[a][out1]; [a] showcqt [out0]'
- 类似上面，但帧频域更准确（更慢）:

    ffplay -f lavfi 'aevalsrc=0.1*sin(2*PI*55*t)+0.1*sin(4*PI*55*t)+0.1*sin(6*PI*55*t)+0.1*sin(8*PI*55*t),
                     asplit[a][out1]; [a] showcqt=timeclamp=0.5 [out0]'
- B-weighting等响度

    volume=16*b_weighting(f)
- 低的Q因子

    tlength=100/f*tc/(100/f+tc)
- 定制字体颜色，C-note为green,其他为blue

    fontcolor='if(mod(floor(midi(f)+0.5),12), 0x0000FF, g(1))'
- 定制gamma,现在有光谱线性振幅 

    gamma=2:gamma2=2

### showspectrum ###
转换音频输入为一个视频频谱

滤镜接受如下选项：

- size, s

    指定视频尺寸。语法同于`ffmpeg-utils`手册中的[视频尺寸](ffmpeg-doc-cn-07.md#视频尺寸分辨率)章节。默认为640x512.
- slide

    指定窗口中光谱如何滑动，有如下可能值：

    ‘replace’

        当滑动到最右边后又从左开始 reach the right 
    ‘scroll’

        向左滚动 
    ‘fullframe’

        帧在时间到达时正确生成

    默认为replace.
- mode

    指定显示模式，接受如下值

    ‘combined’

        所有的通道在一个行中显示 
    ‘separate’

        通道显示在各自行中 

    默认为‘combined’.
- color

    指定显示的颜色模式，接受下面的值:

    ‘channel’

        每个通道采样指定的颜色显示 
    ‘intensity’

        每个通道采样相同的配色方案

    默认为‘channel’.
- scale

    指定用于计算强度的颜色值，允许如下的值：

    ‘lin’

        线性 
    ‘sqrt’

        平方根,默认 
    ‘cbrt’

        立方根 
    ‘log’

        对数 

    默认‘sqrt’.
- saturation

    设置显示颜色饱和度修饰符，负值提供可供选择的配色方案，0没有饱和，值范围为[-10.0, 10.0]，默认为1.
- win_func

    设置窗口函数，接受如下值:

    ‘none’

        没有预处理 (这时最快的了) 
    ‘hann’

        Hann窗口 
    ‘hamming’

        Hamming窗口
    ‘blackman’

        Blackman窗口 

    默认为`hann` 

它类似于`showwaves`滤镜，看下面的例子 
#### showspectrum例子 ####
- 采样对数颜色放缩的大窗口

	showspectrum=s=1280x480:scale=log
- 使用ffplay每通道颜色和滑动频谱的完整示例

	ffplay -f lavfi 'amovie=input.mp3, asplit [a][out1];
             [a] showspectrum=mode=separate:color=intensity:slide=1:scale=cbrt [out0]'

### showwaves ###
转换音频输入为视频输出（代表采样波形），接受如下选项：

- size, s

    指定视频尺寸，语法同于`ffmpeg-utils`手册中的[视频尺寸](ffmpeg-doc-cn-07.md#视频尺寸分辨率)章节。默认为600x240.
- mode

    设置显示模式，允许下面的值：

    ‘point’

        每个采样画一个点.
    ‘line’

        每个采样画一条直线
    ‘p2p’

        每个采样画一个点，且用直线连起来。
    ‘cline’

        每个采样画一个中垂直线 

   默认为`point`
- n

    设置一列中采样数量。一个更大的值将降低帧率。必须是正整数。它可以根据帧率计算而不用显式指定
- rate, r

    设置输出帧率(近视)，其可以起到`n`选项左右，默认为25
- split_channels

    是否分别通道或者覆盖，默认为0


#### showwaves例子 ####

- 同时输出音频和其对应视频表示:

    amovie=a.mp3,asplit[out0],showwaves[out1]
- 利用`showwaves`创建一个同步信号并显示，帧率为30:

    aevalsrc=sin(1*2*PI*t)*sin(880*2*PI*t):cos(2*PI*200*t),asplit[out0],showwaves=r=30[out1]

### showwavespic ###
转换音频输入为视频输出（代表采样波形），接受如下选项：

- size, s

    指定视频尺寸，语法同于`ffmpeg-utils`手册中的[视频尺寸](ffmpeg-doc-cn-07.md#视频尺寸分辨率)章节。默认为600x240.
- split_channels

    是否分别通道或者覆盖，默认为0
#### showwavespic例子 ####
- 利用ffmpeg按通道提取波形，并展示在1024x800图片中

	ffmpeg -i audio.flac -lavfi showwavespic=split_channels=1:s=1024x800 waveform.png
### split和asplit ###
从输入中选择一些来输出

其中`asplit`对于音频工作，`split`对于视频工作

这个滤镜接受单个指定输出个数的参数，如果不指定，默认为2
#### split和asplit例子 ####
- 对一个输入创建2给相同的输出:

    [in] split [out0][out1]
- 创建3给或者更多的输出，你需要为每个输出指定标签，如下:

    [in] asplit=3 [out0][out1][out2]
- 创建2个输出，其中一个作为`cropped`的输入，一个作为`pad`的输入:

    [in] split [splitout1][splitout2];
    [splitout1] crop=100:100:0:0    [cropout];
    [splitout2] pad=200:200:100:100 [padout];
- 对输入音频制作5份拷贝:

    ffmpeg -i INPUT -filter_complex asplit=5 OUTPUT

### zmq和azmq ###
接受从`libzmq`客户端发送来的命令，并将它们发送给滤镜链图中的滤镜。

这里`zmq`和`azmq`都是直通滤镜，只是`zmq`工作于两个视频滤镜间，而`azmq`工作于两个音频滤镜间。

编译需要`libzmq`库以及头文件，并采样`--enable-libzmq`配置

关于`libzmq`的更多信息参考[http://www.zeromq.org/](http://www.zeromq.org/)

滤镜`zmq`和`azmq`都需要`libzmq`服务，它通过网络来发送定义于`bind_address`选项的接收消息接口

接收的消息必须有如下格式：

	TARGET COMMAND [ARG]
这里`TARGET`指定命令目标，通常是滤镜类名或者滤镜实例名，`COMMAND`为命令，`ARG`为可选的参数列表（对于`COMMAND`）。

在接收,处理信息和相应的命令，并注入滤镜链图，根据结果，滤镜发送一个返回信息给客户端，采样的格式为：

	ERROR_CODE ERROR_REASON
	MESSAGE
其中`MESSAGE`是可选的。
#### zmq和azmq例子 ####
参考`tools/zmsend`中的一个`zmq`客户端例子，它被用来向这些的滤镜发送命令

考虑以下用于ffplay的滤镜链图：

	ffplay -dumpgraph 1 -f lavfi "
	color=s=100x100:c=red  [l];
	color=s=100x100:c=blue [r];
	nullsrc=s=200x100, zmq [bg];
	[bg][l]   overlay      [bg+l];
	[bg+l][r] overlay=x=100 "

为了改变视频左边的颜色，下面的命令被使用：

	echo Parsed_color_0 c yellow | tools/zmqsend
如果要改变右边：

	echo Parsed_color_1 c pink | tools/zmqsend
