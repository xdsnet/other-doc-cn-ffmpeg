## 分离器 ##
分离器是使得ffmpeg能从特定类型文件中读取多媒体流的组件元素。

当编译ffmepg时，所有支持的分离器都默认被包含，你可以通过编译配置脚本中的`--list-demuxers`列出所有支持的分离器。

你也可以通过配置`--disable-demuxers`禁用所有的分离器，如果要在此基础上允许单独的分离器可以选用`--enable-demuxer=DEMUXER`形式配置，也可以在默认情况下通过`--disable-demuxer=DEMUXER`禁用个别分离器。

ff*工具集中`-formats`选项可以列出所有编译支持了的分离器（对多媒体容器的支持情况）

下面介绍当前有效的分离器

### applehttp ###
Apple HTTP Live Streaming（苹果http直播流）分离器

这个分离器会从所有变化流中提取出AVStreams，id字段是码率变化（流）索引数。通过在`AVStreams`设置丢弃标志（`a`或者`v`），调用者可以获取所有变化的流。总的码率变化被放置在元数据关键字段"variant_bitrate"里。

### apng ###
Animated Portable Network Graphics（便携网络图形动画）分离器

这个分离器用于APNG文件。所有的头，从PNG标识到（不包括）第一个作为extradata传输的fcTL块。帧被分割到两个fcTL间的块中，或者是最后一个fcTL到iEND块间。

- -ignore_loop bool

    即使文件设置了循环标志也忽略
- -max_fps int

    每秒最大帧率，0表示不限制 
- -default_fps int

    定义默认帧率（文件中没有特别指定时），0表示最大可能值

### asf ###
Advanced Systems Format（高级系统格式）分离器

这个分离器用于ASF文件和MMS网络流

- -no_resync_search bool
	
	通过寻找一定的可选启动代码而不试图重新同步

### concat ###
Virtual concatenation script（虚拟级联脚本）分离器

分离器从一个文本文件中读取一个文件列表和其他指令，然后分离它们，就像文件和指令是被混合在一起的（文件对应特定类型流，指令也对应特殊流）

时间戳是以第一个文件开始作为0，然后任何以后的文件都以前一个文件播放完成为开始时间。**注意**这是全局的，即使所有的流并不是相同长度。

文件中必须有相同的流（相同编码、相同时间基准等等）

每个文件的持续时间用来调整下一个文件的时间戳:如果持续时间不正确（因为这是通过比特率计算出的时间，而文件可能是被窃取出来的），这将导致伪影（这里指代后续文件不正确的时间戳），这时时间指令就可以用来覆盖调整存储在文件中的（计算出）的持续时间，而把整个时间戳调整正确。

#### concat语法 ####
这个脚本文件是extended-ASCII(扩展ASCII)文本文件，每个指令一行。空行和前导空格和由“#”开始的行被忽略。有下面的指令:

- file path

    一个文件的路径:如果路径中包含特殊字符和空格，必须通过“\”转义或者由单引号括起来

    采用相对于这个文件的相对路径指示包含的文件
- ffconcat version 1.0

    标识脚本类型和版本，也可以设置为安全选项（如果设置为1），默认是-1

    为了让ffmpeg自动识别这个格式，这个指令必须是（看起来像）脚本的第一行（没有额外的空间或字节顺序标记）
- duration dur

    指示文件持续时间。这个信息可以用于指定文件，以用于有效的帮助处理可能从文件信息中获取（计算）的不可用或不准确持续时间

    如果是所有文件的持续时间，则可在整个文件中进行定位
- stream

    指定虚拟文件流。随后所有关于流的指令适用于这个指定的流。一些流必须设置为允许识别子文件匹配的流，如果没有指定流则从第一个文件流进行复制
- exact_stream_id id

    设置流的id。如果设置了，将作为子文件的标识id来用，这通常用于MPEG-PS(VOB)文件，这里数据流的顺序是不可靠（用）的

#### concat选项 ####
分离器接受如下选项：

- safe

    如果设置为1，将调整不安全的文件路径。所谓安全文件路径指不包括协议规范头，以相对路径指出，且只包括便携字符集（字母、数字、点号、下划线和连字符）且不以点号开始的（即只接受安全路径）

    如果设置为0，则任何文件路径都可以被接受

    默认值是-1，表示自动探测，如果可能则为0，否则为1
- auto_convert

    如果为1，则尝试对分组流进行自动转换，默认为1

    当前它仅支持在MP4格式中的H.264流添加`h264_mp4toannexb`比特流滤镜，这是必要的，尤其有分辨率变化时。

### flv ###
Adobe Flash Video Format 分离器

它用于分离FLV文件和RTMP网络流

- -flv_metadata bool
	
	根据元数据的数组内容分配流

### libgme ###

Game Music Emu库是一个汇集视频游戏音乐文件模拟器

参考[http://code.google.com/p/game-music-emu/](http://code.google.com/p/game-music-emu/)获得更多信息

一些文件有很多音轨，这个分离器会把第一个作为默认。`track_index`选项可以用来指定不同的音轨。音轨索引从0开始。分离器可以从音轨元数据中导出音轨索引

对于巨大的文件，`max_size`选项可以用来调整

### libquvi ###
从使用quvi项目的因特网服务器播放媒体

这个分离器要求`format`选项被指定一个品质，默认是`best`

参考[http://quvi.sourceforge.net/](http://quvi.sourceforge.net/)了解更多信息

编译时需要`--enable-libquvi`以获取支持

### gif ###
Animated GIF（动画GIF）分离器

接受如下选项：

- min_delay

    设置帧间最小间隔，单位百分之一秒，范围0-6000，默认2
- max_gif_delay

    设置帧间最大间隔，单位百分之一秒，范围0-65535（大约11分钟），这个值只能被指定
- default_delay

    设置帧间默认间隔，单位百分之一秒，范围0-6000，默认10
- ignore_loop

    GIF文件可以包含一个用于指定循环次数的信息，如果这个选项被设置为1，则文件中的指定将被忽略。设置为0则启用文件中的设定，默认值是1 

	例如，下面将一个gif文件按循环次数接到mp4文件后，一起输出:

	ffmpeg -i input.mp4 -ignore_loop 0 -i input.gif -filter_complex overlay=shortest=1 out.mkv

	**注意**前面例子中以简写参数形式采用了个滤镜。

### image2 ###
图像文件分离器

这个分离器按指定格式规范（模式）读取一个图像文件列表，语法由选项`pattern_type`指定。

格式规范可以包含一个后缀扩展名来自动确定包含该图像格式文件。

在列表（序列）中图像文件的尺寸、像素格式和图像格式必须是一样的。

分离器接受下面的选项：

- framerate

    设置对应视频流帧率，默认25 
- loop

    如果为1，则循环设置覆盖输入，否则为0（默认） 
- pattern_type

    选择格式规范（模式）类型

    pattern_type 接受下面值：

    - none

        禁用模式匹配（规则匹配），即只包括指定的图像，如果你不想创建包含多个图像文件的序列和文件名中可能包含特殊字符（会和模式冲突）则采用这个选项参数
    - sequence

        指定序列模式，通过特定的文件索引指定一个序列文件

        序列模式可以包括"%d" 或 "%0Nd"字符串。这将指定所有匹配字符串的文件。如果是"%0Nd"则表示N位数字，如果不足前面用0补足。如果文件名中本身含有“%”，则要用“%%”转义代替

        在序列模式中包含了 "%d"或者"%0Nd",则第一个文件是匹配模式的文件，整个序列是start_number到start_number+start_number_range-1，且所有文件成为连续序列

        例如匹配设置 "img-%03d.bmp"可匹配img-001.bmp, img-002.bmp, ..., img-010.bmp 等等;而匹配设置 "i%%m%%g-%d.jpg" 将匹配 i%m%g-1.jpg, i%m%g-2.jpg, ..., i%m%g-10.jpg 等到

        **注意**该模式不一定包含"%d"或者 "%0Nd",例如转换单个图像img.jpeg 你可以采用:

        ffmpeg -i img.jpeg img.png

    - glob

        选择一个全局通配符模式

        这种模式下类似glob()模板，它仅在libavformat编译支持了globbing时有效.
    - glob_sequence (deprecated, will be removed)

        混合通配符和序列的模式

        如果libavformat编译支持了globbing，且在匹配字符串中包含了至少1个通配符，普通的`%*?[]{}`等字符需要加上`%`进行转义。这将类似glob()进行通配符匹配，然后再附加序列模板

        所有特殊字符`%*?[]{}`等字符需要加上`%`进行转义，例如对于字符"%"需要表示为"%%"

        例如foo-%*.jpeg将匹配所有`foo-`且扩展名为".jpeg"的文件。foo-%?%?%?.jpeg将匹配`foo-`前缀，后面接3个字符的序列，扩展名为 ".jpeg".

        模式类型在favor of glob 以及 sequence中描述 

    默认值是`glob_sequence` 
- pixel_format

    设置像素格式，如果没有指定则猜测一个（从第一个图像获取）
- start_number

    设置索引开始计数的数字值，默认是0
- start_number_range

    设置找到序列中第一个文件，以此设置全文件时间检测范围。起始时间基准由`start_number`指定，默认为5
- ts_from_file

    如果为1，则图像文件帧时间码可被编辑。**注意**不是由时间戳提供：如果没有这个选项，图像将按相同的顺序.默认值为0，如果设置为2，将在纳秒级精确的对图像时间进行修正
- video_size

    设置图像尺寸，如果没有指定，将以序列第一个文件进行探测

#### image2例子 ####
- 从一个文件序列 img-001.jpeg, img-002.jpeg, ...,创建视频，帧率为10:

    ffmpeg -framerate 10 -i 'img-%03d.jpeg' out.mkv

- 类似上例，但开始的数字是100，即索引是从100开始计数:

    ffmpeg -framerate 10 -start_number 100 -i 'img-%03d.jpeg' out.mkv

- 读取"*.png" 以通配符模式处理，这将包含所有".png"结尾的文件:

    ffmpeg -framerate 10 -pattern_type glob -i "*.png" out.mkv

### mpegts ###
MPEG-2传输流分离器


- fix_teletext_pts

    依据从第一个程序PCR计算出的时间戳覆盖PTS和DTS，图文信息流部分不丢弃。默认值是1.如果你想保持PTS和DTS不变则设置为0

### rawvideo ###

原始视频（raw video）分离器

这个分离器允许读取原始视频数据。因为没有报头指定数据参数，为了能正确解码用户必须人为设定一些参数。

接受如下的选项:

- framerate

    甚至视频帧率，默认25
- pixel_format

    设置输入视频的像素格式，默认yuv420p.
- video_size

    设置输入视频的分辨率，这个必须进行指定 

例如要用ffplay播放input.raw中的原视频，像素格式是rgb24，分辨率320x240，帧率10则相应命令为：

	ffplay -f rawvideo -pixel_format rgb24 -video_size 320x240 -framerate 10 input.raw

### sbg ###

SBaGen 脚本分离器

这个分离器读取SBaGen脚本， [http://uazu.net/sbagen/](http://uazu.net/sbagen/)可以相关处理细节，一个SBG脚本看起来像：
> 
	-SE
	a: 300-2.5/3 440+4.5/0
	b: 300-2.5/0 440+4.5/3
	off: -
	NOW      == a
	+0:07:00 == b
	+0:14:00 == a
	+0:21:00 == b
	+0:30:00    off

SBG脚本可以混合绝对和相对时间戳。如果脚本只使用绝对时间戳（包括脚本启动时间）或者只使用相对时间戳，那么它的布局是古代的，转换十分简单。如果脚本是混合时间戳，相对时间将从脚本读取执行剧本开始计算，而一些绝对时间戳指定的内容可能被冻结，这意味着如果脚本是直播播放的形式，实际时间与绝对时间戳影响的声音控制器时间精度会一直，但如果其间用户进行了暂停等操作，则相应的时间戳会方式变化

### tedcaptions ###

 用于TED任务的JSON

TED并不提供这样的内容主题，但可以从页面猜测。ffmpeg源代码库中间的文件 `tools/bookmarklets.html`

允许如下选项:

- start_time

    设置TED任务开始时间。单位milliseconds,默认值15000 (15s).它可以用来同步标题（对于下载视频），因为它包括了15s的输入引导
  
例子： 转换一个主题:
	
	ffmpeg -i http://www.ted.com/talks/subtitles/id/1/lang/en talk1-en.srt

