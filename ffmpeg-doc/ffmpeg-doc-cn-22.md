## 22 混合器（复用器） ##
复用器是ffmpeg中负责写入多媒体流到文件中分区的可配置组件。

默认编译时自动允许被支持的混合器。你可以使用`--list-muxers`作为参数运行编译配置脚本以了解当前支持的所有混合器。

编译也可以同`--disable-muxers`禁用所有的混合器，或者通过`--enable-muxer=MUXER` / `--disable-muxer=MUXER`打开/关闭指定的混合器 

在ff*工具集中附加`-formats`也可以了解到混合器列表。

下面将详细描述有效的混合器直播：

### aiff ###
Audio Interchange File Format（aif）密码器

#### aiff选项 ####
接受下面选项：
- write_id3v2

    如果设为1则允许ID3v2标签，否则0禁止（默认）
- id3v2_version

    选择id3v2版本

### crc值 ###
CRC (Cyclic Redundancy Check)测试格式

这个混合器通过所有输入的音频和视频帧计算（混合）Adler-32 CRC。默认音频会被转换为16bit符号原始音频，视频被解压为原始视频再进行这个计算。

输出会有一个形如`CRC=0xCRC`的一行，其中CRC的值是由16进制以0补足的8位数字，它由所有帧解码计算的。

参考[framecrc] 混合器

#### crc值计算例子 ####
计算一个crc放置到out.crc:

	ffmpeg -i INPUT -f crc out.crc

计算crc并直接输出到标准输出设备:

	ffmpeg -i INPUT -f crc -

还可以选择对特定音频、视频编码数据计算crc，例如计算输入文件音频转换成PCM 8bit无符号数据格式，视频转换成MPEG-2 的CRC:

	ffmpeg -i INPUT -c:a pcm_u8 -c:v mpeg2video -f crc -

### framecrc ###
每个数据包的CRC（循环冗余校验）测试格式。

它将对每个数据包做Adler-32 CRC计算并输出。默认音频被转换成16bit符号原始音频，视频被转换成原始视频再进行CRC计算。

输出是针对每个音频/视频数据包都有一行如下格式的信息：

	stream_index, packet_dts, packet_pts, packet_duration, packet_size, 0xCRC

其中CRC值是16进制，以0补足的8位数字值。

#### framecrc例子 ####
对INPUT输入作每数据包CRC计算，输出到out.crc:

	ffmpeg -i INPUT -f framecrc out.crc

直接把计算结果输出到标准输出设备:

	ffmpeg -i INPUT -f framecrc -

通过ffmpeg，还可以选择输出特定音频和视频格式对应的帧CRC值，例如音频转换成PCM8bit无符号编码，视频为mpeg2计算帧CRC校验值:

	ffmpeg -i INPUT -c:a pcm_u8 -c:v mpeg2video -f framecrc -

参看[crc]混合器 

### framemd5 ###
每个数据包MD5校验值

计算输出每个数据包MD5校验值，默认音频被转换成16bit符号原始音频，视频被转换成原始视频再进行MD5计算

每个数据包计算对应输出一行如下格式数据:

	stream_index, packet_dts, packet_pts, packet_duration, packet_size, MD5

其中MD5就是计算出的MD5 哈希值

#### framemd5例子 ####
计算INPUT输入的帧md5值，其中音频被转换成16bit符号原始音频数据，视频被转换成原始视频数据，输出到out.md5

	ffmpeg -i INPUT -f framemd5 out.md5

直接输出到标准输出设备：

	ffmpeg -i INPUT -f framemd5 -

参考[md5]混合器部分

### gif ###
Animated GIF（动画GIF）混合器

它接受如下选项：

- loop

    设置循环次数，-1表示不循环，0表示一直循环（默认值）
- final_delay

    强制最后一帧延迟 (以厘秒为单位——centiseconds) ，默认为1，这是一个对于循环gif的特殊设定，它为最后一帧播放到新开始播放设置一个特殊的值，比如你可能希望有一个停顿的感觉。

	例如像循环10次，每次重新播放前停顿5秒，则：

	ffmpeg -i INPUT -loop 10 -final_delay 500 out.gif

**注意1**如果你想提取帧到指定的GIF文件序列，你可能需要`image2`混合器

	ffmpeg -i INPUT -c:v gif -f image2 "out%d.gif"
**注意2**Gif格式有一个非常小的时基：两帧之间的间隔不可小于百分之一秒。

### hls ###
Apple HTTP 直播流混合器，它根据HTTP直播流（HLS）规范进行MPEG-TS分割

它创建一个播放列表文件，包括1个或者多个分段文件，输出文件为指定的播放列表文件。

默认混合器对每段创建一个文件，这些文件有相同的基于播放列表的文件名，段索引数和.tx扩展名

例如，转一个输入文件：

	ffmpeg -i in.nut out.m3u8

这将根据产品播放列表文件out.m3u8产生分段文件:out0.ts out1.ts out2.ts 等等

参考[segment]混合器，它提供了更多可用于HTL分割的常规处理和修正介绍

#### hls选项 ####
这个混合器支持如下选项


- hls_time seconds

    设置段长度，单位秒，默认为2
- hls_list_size size

    设置播放列表中字段最大数。如果为0，则包含所有分段。默认为5
- hls_ts_options options_list

    设置输出格式选项，使用'-'分割的`key=value`参数对，如果包括特殊字符需要被转义处理
- hls_wrap wrap

    一种循环机制，设置数量后以0-设定数形成一个环依次循环使用作为输出段号.为0表示不限制， 默认为0

    选项可避免磁盘被多个段文件填满，并限制写入磁盘的最大文件数
- start_number number

    设置播放列表中最先播放的索引号，默认 0.
- hls_allow_cache allowcache

    设置客户端是否：可能(1) 或 必须不 (0) 缓冲媒体段 
- hls_base_url baseurl

    对每个列表中的记录添加一个基本的URL，一般用于采用相对路径描述的列表

	**注意**列表序号必须是每段独特的，不可分割的文件名和序列号，序列号是可循环的，则可能会引起困惑，例如hls_wrap选项设置了
- hls_segment_filename filename

    设置段文件名。除非`hls_flags` `single_file`被设置，设置这个文件名可以用于段命名格式化（依据段序数）:

    ffmpeg in.nut -hls_segment_filename 'file%03d.ts' out.m3u8

    这个例子中，段文件会输出为: file000.ts, file001.ts, file002.ts, 等等，而不是默认的out0.ts out1.ts out2.ts 等等
- hls_key_info_file key_info_file

    使用key_info_file对段进行加密。`key_info_file` 中的第一行指定一个URI，是写入播放列表的，这个key URL被用于存放播放期访问的加密密钥。第二行指定用于加密过程中的key文件路径。key文件作为一个单一排列的16进制数组以二进制格式数据读入。可选的第三行则指定初始化向量（IV，一个十六进制字符串用于代替部分序列（默认）进行加密）。改变key_info_file将导致段加密采用新的key/IV 以及播放列表中任意条目采用新的 URI/IV

    key_info_file 格式:
> 
    key URI
    key file path
    IV (optional)

    key URIs 例子:
> 
    http://server/file.key
    /path/to/file.key
    file.key

    key文件路径例子:
> 
    file.key
    /path/to/file.key

    IV例子:
> 
    0123456789ABCDEF0123456789ABCDEF

    完整key_info_file 示例:
> 
	http://server/file.key 
    /path/to/file.key 
    0123456789ABCDEF0123456789ABCDEF 

    shell脚本例子:
> 
	    #!/bin/sh
	    BASE_URL=${1:-'.'}
	    openssl rand 16 > file.key
	    echo $BASE_URL/file.key > file.keyinfo
	    echo file.key >> file.keyinfo
	    echo $(openssl rand -hex 16) >> file.keyinfo
	    ffmpeg -f lavfi -re -i testsrc -c:v h264 -hls_flags delete_segments \
	      -hls_key_info_file file.keyinfo out.m3u8

- hls_flags single_file

    如果这个标记被设置，则会把所有段存储到一个MPEG-TS文件中，且在播放列表中使用字节范围。 HLS播放列表在版本4中支持这种方法：

    ffmpeg -i in.nut -hls_flags single_file out.m3u8

    这里所有的输出都放置在out.ts中了
- hls_flags delete_segments

    在播放的段已经过了持续时间后就删除掉对应的文件。

### ico ###
ICO文件混合器

微软ICON（ICO）文件格式有一些限制需要注意：

- 每个方向不超过256像素
- 仅BMP和PNG图像可以被存储
- 如果是BMP图像，必须有如下像素格式：
> 
	bmp位深度	ffmpeg像素格式  
	1bit		pal8  
	4bit		pal8  
	8bit		pal8  
	16bit		rgb555le  
	24bit 		bgr24  
	32bit		bgra  
- 如果是BMP图像，必须有 BITMAPINFOHEADER DIB 头
- 如果是PNG图像，必须是rgba像素格式

### image2 ###
图像文件混合器

它可以把视频帧重新混合为图像文件

输出文件按模板指定，可以设置成为一个序列数文件。模板中的"%d" 或者 "%0Nd"用于指定序列，其中"%0Nd"表示N位数字，以0补齐。如果文件名中有“%”需要以“%%”转义的形式指定。

如果模板中包含了"%d"或者"%0Nd"则文件名从1计数输出序列

模板可以包含一个后缀用来自动确定图像文件格式

例如模板"img-%03d.bmp"将使得输出为img-001.bmp, img-002.bmp, ...,img-010.bm 等等。而模板"img%%-%d.jpg"则生成img%-1.jpg, img%-2.jpg, ..., img%-10.jpg,等等

#### image2例子 ####
把输入视频图像帧生成为img-001.jpeg, img-002.jpeg, ...,

	ffmpeg -i in.avi -vsync 1 -r 1 -f image2 'img-%03d.jpeg'

**注意**ffmpeg如果没有通过`-f`指定输出文件格式，image2混合器将自动被选择，所以前面的等效于

	ffmpeg -i in.avi -vsync 1 -r 1 'img-%03d.jpeg'

如果`strfime`选项允许你导出按时间/日期信息命名的文件 "%Y-%m-%d_%H-%M-%S" 模板，在` strftime() `的文档中了解相关语法

例如：
	
	ffmpeg -f v4l2 -r 1 -i /dev/video0 -f image2 -strftime 1 "%Y-%m-%d_%H-%M-%S.jpg"

#### image2选项 ####

- start_number

    设置开始序列的数字，默认为0
- update

    如果设置为1，文件名直接作为唯一文件名，而没有模板。即相应的文件被不断改写为新的图像。默认为0
- strftime

    如果设置为1，可以让输出文件支持strftime()提供的日期格式，默认为0 

这个图像混合器支持.Y.U.V图像文件格式，这种格式将根据每帧输出3个文件，对于每个YUV420P压缩，对于读或者写这种文件格式，只需要指定.Y文件即可，混合器会自动打开需要的.U和.V文件

### matroska ###
Matroska内容混合器

混合输出matroska和webm内容

#### matroska元数据 ####
混合器需要指定一些必要元数据

- title

    设置单个轨道的标题名
language

    以Matroska语言字段指定语言

    语言可是3个字符（依ISO-639-2 (ISO 639-2/B)） (例如 "fre" 表示法语——French),或者语言混合国家/地区代码，(like "fre-ca" 表示加拿大法语——Canadian French).
stereo_mode

    设置3D视频两个视图在单个视频轨道播放时的布局规则

    允许如下值:

    ‘mono’

        video不是双路的 
    ‘left_right’

        两路分别一端，即左眼看左视图，右眼看右视图
    ‘bottom_top’

        上下布局，左眼看下视图，右眼看上视图
    ‘top_bottom’

        与上一个相反，左眼看上，右眼看下
    ‘checkerboard_rl’

        根据序列确认，左眼看第一个
    ‘checkerboard_lr’

        根据序列确认，右眼看第一个
    ‘row_interleaved_rl’

        根据行序列确认，右眼看第一行
    ‘row_interleaved_lr’

        根据行序列确认，左眼看第一行，
    ‘col_interleaved_rl’

        列序列确认，右眼第一列
    ‘col_interleaved_lr’

        列序列确认，左眼第一列 
    ‘anaglyph_cyan_red’

        都在一副图中，通过颜色滤镜过滤red-cyan filters 
    ‘right_left’

        右眼看左图 
    ‘anaglyph_green_magenta’

        混合图，通过green-magenta滤镜看 
    ‘block_lr’

        间隔块，左眼先看 
    ‘block_rl’

        间隔块，右眼先看 

例如，对于3DWebM影片，可以由下面命令建立：

	ffmpeg -i sample_left_right_clip.mpg -an -c:v libvpx -metadata stereo_mode=left_right -y stereo_clip.webm

#### matroska选项 ####
支持如下选项：

- reserve_index_space

	默认对于定位索引（可以被Matoska调用）将写到文件的末尾部分，因为一开始不知道需要多少空间放置索引。但这将导致流式播放时定位特别慢（因为不知道定位索引），这个选项将把索引放置到文件的开始。

	如果这个选项设置为非0值，混合器将预先在头部放置一个用于写入索引的空间，但如果空间无效则将混合失败。一个较安全的值是大约1小时50KB。

	**注意**这些寻址线索仅当输出文件是可寻址且选项设置了有效值时写入。

### md5 ###
MD5检测格式

将计算输出一个MD5值，对于所有的音视频帧。默认音频帧转换为有符号16bit原始音频，视频转换为原始视频来计算。

输出是一个MD5=`MD5` 格式，其中`MD5`就是计算出的值。

例如：

	ffmpeg -i INPUT -f md5 out.md5
也可以输出到标准输出设备

	ffmpeg -i INPUT -f md5 -

参考[framemd5]混合器

### mov,mp4,ismv ###
MOV/MP4/ISMV (Smooth Streaming——平滑流)混合器

MOV/MP4/ISMV混合器支持零碎文件（指数据的组织形式）。通常MOV/MP4文件把所有的元数据存储在文件的一个位置中（这是不零碎的数据组织形式，通常在末尾，也可以移动到起始以更好的支持随机定位播放，比如使用`qt-faststart`工具，并添加`movflags`快速启动标志）。这样一个零碎文件包含了很多片段，其中数据包和元数据是存储在一起的。这样零碎数据组织的文件在解码到写中断（普通的MOV/MP4则不能解码了，因为可能缺少元数据）时也能正常解码，而且这种方式要求更少的内存就可以写很大的文件（因为普通形式的MOV/MP4需要收集所有的信息才能最终完成元数据集中存储，则这一过程中这些数据一直需要缓存在内存中，直到编码完成，元数据完成存储），这是一个优势。缺点是这种组织数据的格式不太通用（很多程序不支持）

#### mov,mp4,ismv选项 ####
零碎形式也支持AVOtions，它可以定义如何切分文件到零碎片段中：

- -moov_size bytes

    在文件开头设置预留空间用于存储moov原子数据（一些元数据），而不是把这些数据存储在文件尾部。如果预设的空间不够，将导致混合失败 
- -movflags frag_keyframe

    在每个关键帧都开始一个新的碎片
- -frag_duration duration

    每duration microseconds时长就创建一个碎片
- -frag_size size

    碎片按size字节（这是一个上限）进行划分 
- -movflags frag_custom

    允许调用者手动切片，通过调用av_write_frame(ctx, NULL) 在当前位置写入一个片段(它仅能与libavformat库集成，在ffmpeg中不支持) 
- -min_frag_duration duration

    如果少于duration microseconds就不单独创建片段

	如果指定了多个条件，当一个条件满足是，片段被切分出来。例外的是`-min_frag_duration`, 它在任何其它条件满足时都使用来进行判断

此外，输出还可以通过一些其他选项进行调整：

- -movflags empty_moov

    写入一个空的moov atom到文件开始,而没有任何样品描述。一般来说，一个mdat/moov在普通MOV/MP4文件开始时写入，只包括了很少的内容，设置了这个选项将没有初始的moov atom，而仅是一个描述了轨道，但没有持续时间的moov atom。

    这个选项在ismv文件中隐式设定 
- -movflags separate_moof

    为每个轨道写独立的moof（电影片段）atom。通常，追踪所有分组是写在一个moof atom中，而通过这个选项，混合器将对每个轨道单独写moof/MDAT，以方便轨道间隔离

    这个选项在ismv文件中隐式设定 
- -movflags faststart

    再次移动index（moov atom）到文件开始位置。这个选项可以与其他选项一起工作，除了碎片化输出模式。默认情况是不允许 
- -movflags rtphint

    添加RTP打标轨道到输出文件中
- -movflags disable_chpl

    禁止Nero章标签(chpl atom)。通常，Nero章标签和QuickTime章标签都被写入到文件中，通过这个选项，可以强制只输出QuickTime标签。Nero章标签可能导致文件在某些程序处理标签时失败，例如 mp3Tag 2.61a 和 iTunes 11.3，可能其他版本也会受到影响
- -movflags omit_tfhd_offset

    在thfd atom（原子数据）中不写入任何绝对`base_data_offset`。这将避免片段文件/流中的绝对定位绑定
- -movflags default_base_moof

    类似`omit_tfhd_offset`，这个标志避免在tfhd atom中写绝对`base_data_offset`，而是用新的`default-base-is-moof`，这个标志定义在14496-12:2012 。它会使片段在某些情况下更容易被解析（避免通过在前一轨道片段基础上隐式进行追踪计算碎片位置）

#### mov,mp4,ismv例子 ####
平滑流内容可以通过IIS进行发布，例如：
	
	ffmpeg -re <normal input/transcoding options> -movflags isml+frag_keyframe -f ismv http://server/publishingpoint.isml/Streams(Encoder1)

### mp3 ###
MP3混合器通过下面选项写原始的MP3流：

- 一个ID3v2元数据头会写在开始处（默认），支持版本2.3和2.4， `id3v2_version`私有选项可以使用(3或4)，如果设置`id3v2_version`为0表示禁用ID3v2头

    混合器还支持附加图片（APIC帧）到ID3v2头。这个图片以单一分组视频流的形式提供给混合器。可以有任意数量的这种流，每个都是单独的APIC帧。对于APIC帧的描述和图片类型要求，以及流元数据标题及内容提交者 等参考[http://id3.org/id3v2.4.0-frames](http://id3.org/id3v2.4.0-frames "http://id3.org/id3v2.4.0-frames")。

    **注意**APIC帧必须写在开始的地方，所以混合器会缓冲音频帧直到所有的图片已经获取完成。因此建议尽快提供图片，以避免过度缓冲。

- Xing/LAME帧正确放置在ID3v2头之后(如果提供)。它也是默认的，但仅仅在输出是可定位情况下写入。`write_xing`私有选项可以用来禁用它。这些帧中包括的变量信息通常用于解码器，例如音频持续时间或者编码延迟
   
- 一个遗留的ID3v1标签放置在文件的末尾（默认禁止），它可以通过`write_id3v1`私有选项来启用，但其意义非常有限，所以不建议采用 

一些例子：
- 写一个mp3，有ID3v2.3 头和ID3v1的末尾标签

	ffmpeg -i INPUT -id3v2_version 3 -write_id3v1 1 out.mp3
- 通过`map`附加图片到音频：

	ffmpeg -i input.mp3 -i cover.png -c copy -map 0 -map 1 -metadata:s:v title="Album cover" -metadata:s:v comment="Cover (Front)" out.mp3

- 写入一个"干净"的MP3，而没有额外特性

	ffmpeg -i input.wav -write_xing 0 -id3v2_version 0 out.mp3

### mpegts ###
MPEG传输流混合器

这个混合器声明在 ISO 13818-1 和ETSI EN 300 468的部分内容中.

对于通用的元数据设置`service_provider`和`service_name`，如果没有特别指明，则默认`service_provider`为"FFmpeg",`service_name`为"Service01"

#### mpegts 选项 ####
mpegts混合器选项有：

- -mpegts_original_network_id number

    设置`original_network_id` (默认0x0001). 在DVB是一个唯一的网络标识，它用于标识特殊的服务（通过Original_Network_ID和Transport_Stream_ID）
- -mpegts_transport_stream_id number

    设置`transport_stream_id` (默认0x0001).在DVB是一个传输的标识
- -mpegts_service_id number

    设置`service_id` (默认0x0001),在DVB作为程序标识 DVB. 
- -mpegts_service_type number

    设置程序`service_type` (默认digital_tv), 参考下面预设值
- -mpegts_pmt_start_pid number

    对PMT设置第一个PID (默认 0x1000,最大0x1f00). 
- -mpegts_start_pid number

    对数据包设置第一个PID(默认0x0100,最大0x0f00). 
- -mpegts_m2ts_mode number

    如果设置为1则允许`m2ts`模式，默认为-1，表示禁止 value is -1 which disables m2ts mode. 
- -muxrate number

    设置内容为混合码率（默认VBR） 
- -pcr_period numer

    覆盖默认的PCR重传时间（默认20ms），如果`muxrate`被设置将会被忽略
- -pes_payload_size number

    以单位字节设置最小PES播放加载包大小 
- -mpegts_flags flags

    设置一个标志(后面介绍). 
- -mpegts_copyts number

    如果设置为1则保留原始时间戳。默认为-1，将从0开始更新时间戳
- -tables_version number

    设置PAT, PMT 和SDT版本 (默认0,范围0-31) 。这个选项允许更新流结构， 以便用户可以检测到更改。比如在打开AVFormatContext （API使用时）或重启FFMPEG来周期性改变`tables_version`时:

    ffmpeg -i source1.ts -codec copy -f mpegts -tables_version 0 udp://1.1.1.1:1111  
    ffmpeg -i source2.ts -codec copy -f mpegts -tables_version 1 udp://1.1.1.1:1111  
    ...
    ffmpeg -i source3.ts -codec copy -f mpegts -tables_version 31 udp://1.1.1.1:1111  
    ffmpeg -i source1.ts -codec copy -f mpegts -tables_version 0 udp://1.1.1.1:1111  
    ffmpeg -i source2.ts -codec copy -f mpegts -tables_version 1 udp://1.1.1.1:1111  
    ...

选项`mpegts_service_type`接受如下值:

- hex_value

    一个16进制值，范围0x01到0xff，定义在 ETSI 300 468. 
- digital_tv

    数字TV服务 
- digital_radio

    数字广播服务 
- teletext

    图文电视服务 
- advanced_codec_digital_radio

    高级编码数字广播服务 
- mpeg2_digital_hdtv

    MPEG2数字HDTV服务 
- advanced_codec_digital_sdtv

    高级编码数字SDTV服务 
- advanced_codec_digital_hdtv

    高级编码数字HDTV服务 

选项`mpegts_flags`可以设置如下标志:

- resend_headers

    写下一个包前反弹PAT/PMT 
- latm

    对AAC编码使用LATM打包 

#### mpegts例子 ####

	ffmpeg -i file.mpg -c copy \  
     -mpegts_original_network_id 0x1122 \  
     -mpegts_transport_stream_id 0x3344 \  
     -mpegts_service_id 0x5566 \  
     -mpegts_pmt_start_pid 0x1500 \  
     -mpegts_start_pid 0x150 \  
     -metadata service_provider="Some provider" \  
     -metadata service_name="Some Channel" \  
     -y out.ts

### null ###
Null混合器

这个混合器将不产生任何输出文件，通常用于测试和基准检测

例如要检测一个解码器，你可以使用：

	ffmpeg -benchmark -i INPUT -f null out.null
**注意**前面的命令行并不读写out.null，仅仅是因为ffmpeg语法要求必须有个输出

等效的，你可以采用：

	ffmpeg -benchmark -i INPUT -f null -

### nut ###

- -syncpoints flags

    利用nut改变同步点：

    - default:默认采用低开销的定位模式。没有不使用同步点的，但可减少开销，只是流是不可定位的。

	- none:一般不建议采用这个选项，因为它导致文件是损坏敏感的（稍微破坏就不能正常解码了），且不可定位。一般同步点开销是很小以至于可以忽略的。**注意**`-write_index 0`可用于禁止所有增长的数据表，允许重复使用有效的内存，而没有这些缺点。
	
	- timestamped:时间戳字段扩展来与时钟同步。
	
	`none`和`timestamped`还处于试验阶段 
- -write_index bool

    在最后写索引，这是写索引的默认值

	ffmpeg -i INPUT -f_strict experimental -syncpoints none - | processor

### ogg ###
Ogg内容混合器

- -page_duration duration

    首选页面持续时间（其实是定位点间隔），单位microseconds。混合器将尝试按设定时间创建页面。这允许用户在定位和容器粒度开销间进行平衡。默认1秒。如果设为0， 将填充所有字段，使索引数据很大。在大多数情况下，设为1将使得每个页面1个数据包，且可以有一个很小的定位粒度，但将产生额外的容器开销（文件变大）
- -serial_offset value

    用于设置流序号的一些值。设置来不同且足够大，可以保证产生的ogg文件可以安全的被锁住

### segment, stream_segment, ssegment ###
基本流分段

混合器将输出流到指定的文件（根据最接近的持续时间分段）。输出文件名模板可以采用类似与[image2]的方式，或者使用`strftime`模板（如果`strftime`选项被允许）

`stream_segment`是用于流式输出格式的混合器变种，例如不需要全局头，并要求诸如MPEG传输流分段输出的情况。`ssegment`是`stream_segment`的别名。

每个片段都开始于所选流的关键帧，这是通过`reference_stream`选项设置的

**注意**如果你想精确分割视频文件，你需要准确输入按关键帧整数倍对应的预期分割器，或者指定混合器按新片段必须是关键帧开始。

分段混合器对于固定帧率的视频有更好的工作表现

或者它可以生成一个创建段的列表，这需要通过`segment_list`选项设置，列表的类型由`segment_list_type`选项指定。在段列表输入一个文件名被默认为相应段文件的基本名称。

参看[hls]混合器，其提供更多关于`HLS`分段的特定实现

#### segment, stream_segment, ssegment选项 ####
segment混合器器支持如下选项：


- reference_stream specifier

    Set the reference stream, as specified by the string specifier. If specifier is set to auto, the reference is chosen automatically. Otherwise it must be a stream specifier (see the “Stream specifiers” chapter in the ffmpeg manual) which specifies the reference stream. The default value is auto.
- segment_format format

    Override the inner container format, by default it is guessed by the filename extension.
- segment_format_options options_list

    Set output format options using a :-separated list of key=value parameters. Values containing the : special character must be escaped.
- segment_list name

    Generate also a listfile named name. If not specified no listfile is generated.
- segment_list_flags flags

    Set flags affecting the segment list generation.

    It currently supports the following flags:

    ‘cache’

        Allow caching (only affects M3U8 list files).
    ‘live’

        Allow live-friendly file generation. 

- segment_list_type type

    Select the listing format.

    flat use a simple flat list of entries.
    hls use a m3u8-like structure.

- segment_list_size size

    Update the list file so that it contains at most size segments. If 0 the list file will contain all the segments. Default value is 0.
- segment_list_entry_prefix prefix

    Prepend prefix to each entry. Useful to generate absolute paths. By default no prefix is applied.

    The following values are recognized:

    ‘flat’

        Generate a flat list for the created segments, one segment per line.
    ‘csv, ext’

        Generate a list for the created segments, one segment per line, each line matching the format (comma-separated values):

        segment_filename,segment_start_time,segment_end_time

        segment_filename is the name of the output file generated by the muxer according to the provided pattern. CSV escaping (according to RFC4180) is applied if required.

        segment_start_time and segment_end_time specify the segment start and end time expressed in seconds.

        A list file with the suffix ".csv" or ".ext" will auto-select this format.

        ‘ext’ is deprecated in favor or ‘csv’.
    ‘ffconcat’

        Generate an ffconcat file for the created segments. The resulting file can be read using the FFmpeg concat demuxer.

        A list file with the suffix ".ffcat" or ".ffconcat" will auto-select this format.
    ‘m3u8’

        Generate an extended M3U8 file, version 3, compliant with http://tools.ietf.org/id/draft-pantos-http-live-streaming.

        A list file with the suffix ".m3u8" will auto-select this format. 

    If not specified the type is guessed from the list file name suffix.
- segment_time time

    Set segment duration to time, the value must be a duration specification. Default value is "2". See also the segment_times option.

    Note that splitting may not be accurate, unless you force the reference stream key-frames at the given time. See the introductory notice and the examples below.
- segment_atclocktime 1|0

    If set to "1" split at regular clock time intervals starting from 00:00 o’clock. The time value specified in segment_time is used for setting the length of the splitting interval.

    For example with segment_time set to "900" this makes it possible to create files at 12:00 o’clock, 12:15, 12:30, etc.

    Default value is "0".
- segment_time_delta delta

    Specify the accuracy time when selecting the start time for a segment, expressed as a duration specification. Default value is "0".

    When delta is specified a key-frame will start a new segment if its PTS satisfies the relation:

    PTS >= start_time - time_delta

    This option is useful when splitting video content, which is always split at GOP boundaries, in case a key frame is found just before the specified split time.

    In particular may be used in combination with the ffmpeg option force_key_frames. The key frame times specified by force_key_frames may not be set accurately because of rounding issues, with the consequence that a key frame time may result set just before the specified time. For constant frame rate videos a value of 1/(2*frame_rate) should address the worst case mismatch between the specified time and the time set by force_key_frames.
- segment_times times

    Specify a list of split points. times contains a list of comma separated duration specifications, in increasing order. See also the segment_time option.
- segment_frames frames

    Specify a list of split video frame numbers. frames contains a list of comma separated integer numbers, in increasing order.

    This option specifies to start a new segment whenever a reference stream key frame is found and the sequential number (starting from 0) of the frame is greater or equal to the next value in the list.
- segment_wrap limit

    Wrap around segment index once it reaches limit.
- segment_start_number number

    Set the sequence number of the first segment. Defaults to 0.
- strftime 1|0

    Use the strftime function to define the name of the new segments to write. If this is selected, the output segment name must contain a strftime function template. Default value is 0.
- break_non_keyframes 1|0

    If enabled, allow segments to start on frames other than keyframes. This improves behavior on some players when the time between keyframes is inconsistent, but may make things worse on others, and can cause some oddities during seeking. Defaults to 0.
- reset_timestamps 1|0

    Reset timestamps at the begin of each segment, so that each segment will start with near-zero timestamps. It is meant to ease the playback of the generated segments. May not work with some combinations of muxers/codecs. It is set to 0 by default.
- initial_offset offset

    Specify timestamp offset to apply to the output packet timestamps. The argument must be a time duration specification, and defaults to 0. 

#### segment, stream_segment, ssegment例子 ####

- Remux the content of file in.mkv to a list of segments out-000.nut, out-001.nut, etc., and write the list of generated segments to out.list:

    ffmpeg -i in.mkv -codec copy -map 0 -f segment -segment_list out.list out%03d.nut

- Segment input and set output format options for the output segments:

    ffmpeg -i in.mkv -f segment -segment_time 10 -segment_format_options movflags=+faststart out%03d.mp4

- Segment the input file according to the split points specified by the segment_times option:

    ffmpeg -i in.mkv -codec copy -map 0 -f segment -segment_list out.csv -segment_times 1,2,3,5,8,13,21 out%03d.nut

- Use the ffmpeg force_key_frames option to force key frames in the input at the specified location, together with the segment option segment_time_delta to account for possible roundings operated when setting key frame times.

    ffmpeg -i in.mkv -force_key_frames 1,2,3,5,8,13,21 -codec:v mpeg4 -codec:a pcm_s16le -map 0 \  
    -f segment -segment_list out.csv -segment_times 1,2,3,5,8,13,21 -segment_time_delta 0.05 out%03d.nut

	In order to force key frames on the input file, transcoding is required.
- Segment the input file by splitting the input file according to the frame numbers sequence specified with the segment_frames option:

    ffmpeg -i in.mkv -codec copy -map 0 -f segment -segment_list out.csv -segment_frames 100,200,300,500,800 out%03d.nut

- Convert the in.mkv to TS segments using the libx264 and libfaac encoders:

    ffmpeg -i in.mkv -map 0 -codec:v libx264 -codec:a libfaac -f ssegment -segment_list out.list out%03d.ts

- Segment the input file, and create an M3U8 live playlist (can be used as live HLS source):

    ffmpeg -re -i in.mkv -codec copy -map 0 -f segment -segment_list playlist.m3u8 \  
    -segment_list_flags +live -segment_time 10 out%03d.mkv

### smoothstreaming ###
平滑流混合器生成一组文件（清单、块），适用于传统web服务器

- window_size

    指定清单中保留的片段数。默认是0，表示保留所有的
- extra_window_size

    从磁盘移除前，保留清单外片段数，默认5
- lookahead_count

    指定先行片段数，默认2
- min_frag_duration

    指定最小片段持续时间（单位microseconds），默认5000000.
- remove_at_exit

    指定完成后是否移除所有片段，默认0，表示不移除

### tee ###
tee混合器可以用于同时把相同数据写入多个文件，或者任何其他类型的混合器。例如使用它可以同时把视频发布到网络上以及保存到磁盘上。

它不同于在命令行指定多个输出，因为利用tee混合器，音频和视频数据只被编码了一次，而编码是一个非常昂贵的行为。它是很有效的，当利用libavformat的API直接可以把相同的数据包用于多个混合器输出（多种封装格式或者场景）

多个输出文件由’|’分隔，如果参数中包含任意前导或尾随的空格，任何特殊字符都必须经过转义(参考 ffmpeg-utils(1)手册中的中 "Quoting and escaping" 章节).

Muxer options can be specified for each slave by prepending them as a list of key=value pairs separated by ’:’, between square brackets. If the options values contain a special character or the ’:’ separator, they must be escaped; note that this is a second level escaping.

下列选项被要求:

- f

    指定格式名，通常用于不能由输出名后缀推测格式的情况
- bsfs[/spec]

    指定一个比特流滤镜应用到指定的输出

    它可以为每个流指定一个比特流滤镜，通过"/"添加一个流选择（说明符），有些流必须由说明符进行指定(格式规范见流说明符)。 如果流说明符没有指定，则比特流滤镜适用于所有输出流。

    可以同时指定多个比特流滤镜，用","分隔。
- select

    Select the streams that should be mapped to the slave output, specified by a stream specifier. If not specified, this defaults to all the input streams. 

#### tee例子 ####

- Encode something and both archive it in a WebM file and stream it as MPEG-TS over UDP (the streams need to be explicitly mapped):

    ffmpeg -i ... -c:v libx264 -c:a mp2 -f tee -map 0:v -map 0:a
      "archive-20121107.mkv|[f=mpegts]udp://10.0.1.255:1234/"

- Use ffmpeg to encode the input, and send the output to three different destinations. The dump_extra bitstream filter is used to add extradata information to all the output video keyframes packets, as requested by the MPEG-TS format. The select option is applied to out.aac in order to make it contain only audio packets.

    ffmpeg -i ... -map 0 -flags +global_header -c:v libx264 -c:a aac -strict experimental
           -f tee "[bsfs/v=dump_extra]out.ts|[movflags=+faststart]out.mp4|[select=a]out.aac"

-  As below, but select only stream a:1 for the audio output. Note that a second level escaping must be performed, as ":" is a special character used to separate options.

    ffmpeg -i ... -map 0 -flags +global_header -c:v libx264 -c:a aac -strict experimental
           -f tee "[bsfs/v=dump_extra]out.ts|[movflags=+faststart]out.mp4|[select=\'a:1\']out.aac"

	Note: some codecs may need different options depending on the output format; the auto-detection of this can not work with the tee muxer. The main example is the global_header flag. 

### webm_dash_manifest ###
WebM DASH 清单混合器.

这个混合器实现了按WebM DASH清单规范生成DASH清单XML文件。它还支持生成DASH直播流

更多参考:
- WebM DASH Specification: [https://sites.google.com/a/webmproject.org/wiki/adaptive-streaming/webm-dash-specification](https://sites.google.com/a/webmproject.org/wiki/adaptive-streaming/webm-dash-specification "https://sites.google.com/a/webmproject.org/wiki/adaptive-streaming/webm-dash-specification")
- ISO DASH Specification: [http://standards.iso.org/ittf/PubliclyAvailableStandards/c065274_ISO_IEC_23009-1_2014.zip](http://standards.iso.org/ittf/PubliclyAvailableStandards/c065274_ISO_IEC_23009-1_2014.zip "http://standards.iso.org/ittf/PubliclyAvailableStandards/c065274_ISO_IEC_23009-1_2014.zip")

#### webm_dash_manifest选项 #### 
支持如下选项:

- adaptation_sets

    这个选项参数有如下语法： "id=x,streams=a,b,c id=y,streams=d,e" 这里的x，y都是唯一合适设置的标识符，a,b,c,d和e是相应的音频和视频流的指代。任何合适的数字可以被用于这个选项。
- live

    如果为1表示创建一个直播流DASH，默认为0
- chunk_start_index

    第一个块的索引号，默认为0，它将作为清单中‘SegmentTemplate’元素的 ‘startNumber’ 属性值
- chunk_duration_ms

    每个块的持续时间，单位milliseconds,默认1000，将作为清单中‘SegmentTemplate’元素的‘duration’属性值
- utc_timing_url

    URL将指示从何处获取UTC时间戳（ISO格式的），它作为清单中 ‘UTCTiming’元素的‘value’ 属性值，默认： None.
- time_shift_buffer_depth

    最小时间（单位秒）的移动缓冲区，为保障可用的任意值，作为清单中‘MPD’元素的‘timeShiftBufferDepth’属性值，默认： 60.
- minimum_update_period

    清单最小更新时间 (单位秒)， 清单中‘MPD’元素的 ‘minimumUpdatePeriod’ 属性值，默认: 0.

#### webm_dash_manifest例子 ####
    ffmpeg -f webm_dash_manifest -i video1.webm \
       -f webm_dash_manifest -i video2.webm \
       -f webm_dash_manifest -i audio1.webm \
       -f webm_dash_manifest -i audio2.webm \
       -map 0 -map 1 -map 2 -map 3 \
       -c copy \
       -f webm_dash_manifest \
       -adaptation_sets "id=0,streams=0,1 id=1,streams=2,3" \
       manifest.xml

### webm_chunk ###
WebM直播块混合器

这个混合器输出WebM头和块分离文件，通过DASH它可以被支持WebM直播流的客户端处理。

#### webm_chunk选项 ####
支持如下选项:

- chunk_start_index

    第一个块的序号，默认0
- header

    文件名将写入初始化数据的头
- audio_chunk_duration

    每个音频块时间，单位milliseconds (默认5000). 

#### webm_chunk例子 ####
    ffmpeg -f v4l2 -i /dev/video0 \
       -f alsa -i hw:0 \
       -map 0:0 \
       -c:v libvpx-vp9 \
       -s 640x360 -keyint_min 30 -g 30 \
       -f webm_chunk \
       -header webm_live_video_360.hdr \
       -chunk_start_index 1 \
       webm_live_video_360_%d.chk \
       -map 1:0 \
       -c:a libvorbis \
       -b:a 128k \
       -f webm_chunk \
       -header webm_live_audio_128.hdr \
       -chunk_start_index 1 \
       -audio_chunk_duration 1000 \
       webm_live_audio_128_%d.chk