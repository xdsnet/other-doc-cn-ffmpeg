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

