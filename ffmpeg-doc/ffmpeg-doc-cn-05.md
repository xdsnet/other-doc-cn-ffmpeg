## 选项 ##
所有的数值选项，如果没有特殊定义，则需要一个接受一个字符串代表一个数作为输入，这可能跟着一个单位量词首字母，例如`"k"`,`"m"`或`"G"`

如果`i`是附加到SI单位的首字母，完整的字母将被解释为一个2的幂数单位，这是基于1024而不是1000的，添加`B`的SI单位则是再将此值乘以8。例如`KB`，`MiB`，`G`和`B`

对于选项中不带参数的布尔选项，即把相应的值设置为`true`，它们可以添加`no`设置为false，例如`nofoo`就相当于`foo false` 。

### 流说明（限定）符 ###
很多选项是作用于单独的流的，例如码率（bitrate）或者编码（codec），流说明符就是精确的为每个流指定相应的选项。

一个流说明符是一个以冒号分隔的字符串，其中分隔出的部分是附加选项，例如`-codec:a:1 ac3`表示编码器是对第2音频流以ac3编码。

一个流说明符可能匹配多个流，则该选项是所有匹配项的选项，例如`-b:a 128k`表示所有的音频流都是128k的码率。

一个空的流说明符匹配所有的流，例如`-codec copy`或者`-codec: copy`表示所有的流都不进行再次编码（包括视频和音频）

可能的流说明符有：
- **`stream_index`**:匹配流的索引，例如`-threads:1 4`表示对2号流采用4个线程处理
- **`stream_type[:stream_index]`**:`stream_type`有`v`表示视频，`a`表示音频，`s`表示字幕，`d`表示数据和`t`表示附加/附件等可能，如果`stream_index`同时被指定，则匹配该索引对于的该类型的流。例如`-codec:v:0 h264`表示第1视频流是h.264编码。
- **`p:program_id[:stream_index]`**:如果`stream_index`被指定，则表示被`program_id`指定的程序仅作用于`stream_index`所指流，否则将作用于所有流。
- **`#stream_id`或者`i:stream_id`**：匹配`stream_id`所指流（MPEG-TS中的PID）
- **`m:key[:value]`**:匹配在元数据中以标签`key`=`value`值的流，如果`value`没有设置，则匹配所有。
- **`u`**：匹配不能被配置的流，这时编码器必须被定义且有必要的视频维度或者音频采样率之类的信息。**注意**，`ffmpeg`匹配由元数据标识的状态仅对于输入文件有效。

### 常规选项 ###
这些常规选项也可以用在`ffmpeg`项目中其他`ff*`工具，例如`ffplayer`

- `-L`：显示授权协议
- `-h，-？，-help，--help[arg]`:显示帮助，一个附加选项可以指定帮助显示的模式，如果没有参数，则是基本选项（没有特别声明）说明被显示，下面是参数定义
	- `long`：在基本选项说明基础上增加高级选项说明
	- `full`：输出完整的选项列表，包括编（解）码器，分离器混合器以及滤镜等等的共享和私有选项
	- `decoder=decoder_name`：输出指定解码器名的详细信息。可以使用`-decoders`来获取当前支持的所有解码器名
	- `encoder=encoder_name`：输出指定编码器名的详细信息。可以使用`-encoders`来获取当前支持的所有编码器名
	- `demuxer=demuxer_name`：输出指定分离器名详细信息。可以使用`-formats`来获取当前支持的所有分离器和混合器
	- `muxer=muxer_name`：输出指定混合器名详细信息。可以使用`-formats`来获取当前支持的所有分离器和混合器
	- `filter=filter_name`：输出指定滤镜名的详细信息。可以使用`-filters`来获取当前支持的所有滤镜
- `-version`：显示版本信息
- `-formats`：显示所有有效的格式（包括设备）
- `-devices`：显示有效设备
- `-codecs`：显示所有已支持的编码（libavcodec中的）格式**注意**`编码/codec`仅仅是文档中一个用于指示媒体数据流格式的术语。
- `-decoders`：显示所有有效解码器
- `-encoders`：显示所有有效的编码器
- `-bsfs`：显示有效的数据流（bitstream）滤镜
- `-protocols`：显示支持的协议
- `-filters`：显示libavfilter中的滤镜
- `-pix_fmts`：显示有效的像素（pixel）格式
- `-sample_fmts`：显示有效的实例格式
- `-layouts`：显示信道名字和信道布局
- `-colors`：显示注册的颜色名
- `-sources device[,opt1=val1[,opt2=val]...]`：显示自动识别的输入设备源。一些设备可能需要提供一些系统指派的源名字而不能自动识别。返回的列表不能认为一定是完整的（即有可能还有设备没有列出来）
	
	`ffmpeg -sources pulse,server=192.168.0.4`
- `-sinks device[,opt1=val1[,opt2=val]...]`:显示自动识别的输出设备。一些设备可能需要提供一些系统指派的源名字而不能自动识别。返回的列表不能认为一定是完整的（即有可能还有设备没有列出来）
	
	`ffmpeg -sinks pulse,server=192.168.0.4`
- `-loglevel [repeat+]loglevel 或者 -v [repeat+]loglevel`：设置日志层次。如果附加有`repeat+`则表示从第一条非压缩行到达到最后消息n次之间的行将被忽略。`"repeat"`也可以一直使用，如果没有现有日志层级设置，则采用默认日志层级。如果有多个日志层级参数被获取，使用`"repeat"`不改变当前日志层级。日志层级是一个字符串或数值，有以下可能值：
	- `quiet,-8`，什么都不输出，是无声的
	- `panic,0`，仅显示造成进程失败的致命错误，它当前不能使用
	- `fatal,8`仅仅显示致命错误，这些错误使得处理不能继续
	- `error,16`显示所有的错误，包括可以回收的错误（进程还可以继续的）
	- `warning,24`显示所有警告和错误，任何错误或者意外事件相关信息均被显示
	- `info,32`显示过程中的信息，还包括警告和错误，则是默认值
	- `verbose,40`类似`info`，但更冗长
	- `debug,48`显示所有，包括调试信息
	- `trace,56`
	
	默认的日志输出是stderr设备，如果在控制台支持颜色，则错误和警告标记的颜色将被显示处理，默认日志的颜色设置可以由环境变量的`AV_LOG_FORCE_NOCOLOR`或者`NO_COLOR`或者环境变量`AV_LOG_RORCE_COLOR`覆盖。环境变量`NO_COLOR`不推荐使用，因为其已经不被新版本支持。

- `-report`：复制所有命令行和控制台输出到当前目录下名为`program-YYYMMDD-HHMMSS.log`文件中。这常用于报告bug，所以一般会同时设置`-loglevel verbose`
   
	设置环境变量`FFREPORT`可以起到相同的效果。如果值是一个以`：`分隔的关键值对，则将影响到报告效果。值中的特殊符号或者分隔符`：`必须被转义（参考ffmepg-utils手册中"引用逃逸"(“Quoting and escaping”)章节）。以下是选项值范围：
	
	- file：设置报告文件名字，`%p`被扩展为程序名字，`%t`是时间码，`%%`表示一个字符`%`
	- level：用数字设定日志信息详略程度（参考`-longlevel`）
	
		例如：
		
			`FFREPORT=file=ffreport.log:level=32 ffmpeg -i input output`
		
		会把日志信息输出到环境变量定义的文件中，	内容包括简要过程信息，警告和错误。

- `-hide_banner`：禁止打印输出banner。所有FFmpeg工具使用中常规都会在前面显示一些版权通知、编译选项和库版本等，这个选项可以禁止这部分的显示。
- `cpuflags flags(global)`：允许设置或者清除cpu标志性和。当前这个选项主要还是测试特性，不要使用，除非你明确需要：
> 
	ffmpeg -cpuflags -sse+mmx ...
	ffmpeg -cpuflags mmx ...
	ffmpeg -cpuflags 0 ...

	可能的选项参数有：
	
	- `x86`
		- mmx
		- mmxext
		- sse
		- sse2
		- sse2slow
		- sse3
		- atom
		- sse4.1
		- sse4.2
		- avx
		- avx2
		- xop
		- fma3
		- fma4
		- 3dnow
		- 3dnowext
		- bmi1
		- bmi2
		- cmov
	- `ARM`
	 	- armv5te
	 	- armv6
	 	- armv6t2
	 	- vfp
	 	- vfpv3
	 	- neon
	 	- setend
	- `AArch64`
		- armv8
		- vfp
		- neon
	- `PowerPC`
		- altivec
	- `Specific Processors`
		- pentium2
		- pentium3
		- pentium4
		- k6
		- athlon
		- athlonxp
		- k8
		
- `-opencl_bench`：输出所有效OpenCL设备的基准测试情况。当前选项仅在编译FFmepg中打开了`--enable-opencl`才有效。

	当FFmpeg指定了`--enable-opencl`编译后，这个选项还可以通过全局参数`-opencl_options`进行设定，参考OpenCL选项，在ffmpeg-utils手册中对于选项的支持情况，这包括在特定的平台设备上支持OpenCL的能力。默认，FFmpeg会运行在首选平台的首选设备上，通过设置全局的OpenCL则可以实现在选定的OpenCL设备上运行，这样就可以在更快的OpenCL设备上运行（平时节点，需要时才选用性能高但耗电的设备）
	
	这个选项有助于帮助用户了解信息以进行有效配置。它将在每个设备上运行基准测试，并以性能排序所有设备，用户可以在随后调用`ffmpeg`时使用`-opencl_options`配置合适的OpenCL加速特性。

	一般以下面的步骤使用这个参数：		
	> 
		ffmpeg -opencl_bench		
	                     
	**注意**输出中第一行的平台ID（*pidx*）和设备ID（*didx*），然后在选择平台和设备用于命令行：
    > 
		ffmpeg -opencl_options platform_idx=pidx:device_idx=didx ...

- `opencl_options options(global)`:设置OpenCL环境选项，这个选项仅仅在FFmpeg编译选项中打开了`--enable-opencl`才有效。

	*options*必须是一个由`:`分隔的`key=value`键值对列表。参考OpenCL选项，在ffmpeg-utils手册中对于选项的支持情况

### AV选项 ###
这些选项由特定的库提供（如libavformat，libavdevice以及libavcodec）。为了更多的了解AV选项，使用`-help`进行进一步了解。它们可以指定下面2个分类：
- generic（常规）：这类选项可以用于设置 容器、设备、编码器、解码器等。通用选项对列在`AVFormatContext`中的容器/设备以及`AVCodecContext`中的编码有效。
- private（私有）：这类仅对特定的容器、设备或者编码有效。私有选项由相应的 容器/设备/编码 指定（确定）。

例如要在一个默认为ID3v2.4为头的MP3文件中写入ID3v2.3头，需要使用id3v2_version 私有选项来对MP3混流：
> 
	ffmpeg -i input.flac -id3v2_version 3 out.mp3

所有编码AV选项是针对单独流的，所以必须详细指定。

**注意**`-nooption`语法不能被用于AV选项中的布尔值项目，而必须使用`-option 0/-option 1`

**注意**以往使用`v/a/s`命名指定每个流的AV选项语法已经不建议使用，它们很快就会失效移除。

### 主要选项 ###
- `-f fmt (input/output)` :指定输入或者输出文件格式。常规可省略而使用依据扩展名的自动指定，但一些选项需要强制明确设定。
- `-i filename （input）`：指定输入文件
- `-y （global）`：默认自动覆盖输出文件，而不再询问确认。
- `-n (global)`:不覆盖输出文件，如果输出文件已经存在则立即退出
- `c[:stream_specifier] codec (input/output,per-stream) `
- `-codec[:stream_specifier] codec (input/output,per-stream)`
	为特定的文件选择编/解码模式，对于输出文件就是编码器，对于输入或者某个流就是解码器。选项参数中`codec`是编解码器的名字，或者是`copy`（仅对输出文件）则意味着流数据直接复制而不再编码。例如：
>
	ffmpeg -i INPUT -map 0 -c:v libx264 -c:a copy OUTPUT 
    
	是使用libx264编码所有的视频流，然后复制所有的音频流。

	再如除了特殊设置外所有的流都由`c`匹配指定：
>
	ffmpeg -i INPUT -map 0 -c copy -c:v:1 libx264 -c:a:137 libvorbis OUTPUT
	
	这将在输出文件中第2视频流按libx264编码，第138音频流按libvorbis编码，其余都直接复制输出。

- `-t duration (input/output)`:限制输入/输出的时间。如果是在`-i`前面，就是限定从输入中读取多少时间的数据；如果是用于限定输出文件，则表示写入多少时间数据后就停止。`duration`可以是以秒为单位的数值或者 `hh:mm:ss[.xxx]`格式的时间值。 **注意**`-to`和`-t`是互斥的，`-t`有更高优先级。
- `-to position (output)`:只写入`position`时间后就停止，`position`可以是以秒为单位的数值或者 `hh:mm:ss[.xxx]`格式的时间值。 **注意**`-to`和`-t`是互斥的，`-t`有更高优先级。
- `-fs limit_size (output)`:设置输出文件大小限制，单位是字节(bytes)。
- `-ss position (input/output)`:
	- 当在`-i`前，表示定位输入文件到`position`指定的位置。**注意**可能一些格式是不支持精确定位的，所以`ffmpeg`可能是定位到最接近`position`（在之前）的可定位点。当有转码发生且`-accurate_seek`被设置为启用（默认），则实际定位点到`position`间的数据被解码出来但丢弃掉。如果是复制模式或者`-noaccurate_seek`被使用，则这之间的数据会被保留。
	
	- 当用于输出文件时，会解码丢弃`position`对应时间码前的输入文件数据。
	- `position`可以是以秒为单位的数值或者 `hh:mm:ss[.xxx]`格式的时间值
- `-itsoffset offset (input)`:设置输入文件的时间偏移。`offset`必须采用时间持续的方式指定，即可以有`-`号的时间值（以秒为单位的数值或者 `hh:mm:ss[.xxx]`格式的时间值）。偏移会附加到输入文件的时间码上，意味着所指定的流会以时间码+偏移量作为最终输出时间码。
- `-timestamp date (output)`:设置在容器中记录时间戳。`date`必须是一个时间持续描述格式，即
> 
	[(YYYY-MM-DD|YYYYMMDD)[T|t| ]]((HH:MM:SS[.m...]]])|(HHMMSS[.m...]]]))[Z] 格式。
- `-metadata[:metadata_specifier] key=value (output,per-metadata)`：指定元数据中的键值对。
	
	流或者章的`metadata_specifier`可能值是可以参考文档中`-map_metadata`部分了解。
	
	简单的覆盖`-map_metadata`可以通过一个为空的选项实现，例如：
	> 
		ffmpeg -i in.avi -metadata title="my title" out.flv
	设置第1声道语言:
	> 
		ffmpeg -i INPUT -metadata:s:a:0 language=eng OUTPUT
- `-taget type （output）`：指定目标文件类型(vcd,svcd,dvd,dv,dv50)，类型还可以前缀一个`pal-`,`ntsc-`或者`film-`来设定更具体的标准。所有的格式选项(码率、编码、缓冲尺寸)都会自动设置，而你仅仅只需要设置目标类型：
	> 
		ffmpeg -i myfile.avi -taget vcd /tmp/vcd.mpg

	当然，你也可以指定一些额外的选项，只要你知道这些不会与标准冲突，如：
	> 
		ffmpeg -i myfile.avi -target vcd -bf 2 /tmp/vcd.mpg
- `-dframes number (output)`:设定指定`number`数据帧到输出文件，这是`-frames:d`的别名。
- `frames[:stream_specifier] framecount (output,per-stream)`:在指定计数帧后停止写入数据。
- `-q[:stream_specifier] q (output,per-stream)`
- `-qscale[:stream_specifier] q (output,per-stream)`
	
	使用固定的质量品质(VBR)。用于指定`q|qscale`编码依赖。如果`qscale`没有跟`stream_specifier`则只适用于视频。其中值`q`取值在0.01-255,越小质量越好。
-  `-filter[:stream_specifier] filtergraph (output,per-stream)`:创建一个由`filtergraph`指定的滤镜，并应用于指定流。
	
	`filtergraph`是应用于流的滤镜链图，它必须有一个输入和输出，而且流的类型需要相同。在滤镜链图中，从`in`标签指定出输入，从`out`标签出输出。要了解更多语法，请参考`ffmpeg－filters`手册。
	
	参考`－filter_complex`选项以了解如何建立多个输入／输出的滤镜链图。
	
- `－filter_script［：stream_specifier］ filename （output，per－stream）`：这个选项类似于`－filter`，只是这里的参数是一个文件名，它的内容将被读取用于构建滤镜链图。

- `－pre［：stream_specifier］ preset_name （output，per－stream）`：指定预设名字的流（单个或者多个）。

- `－stats （global）`：输出编码过程／统计，这是系统默认值，如果你想禁止，则需要采用`－nostats`。
- `－progress url （global）`：发送友好的处理过程信息到`url`。处理过程信息是一种键值对（key=value）序列信息，它每秒都输出，或者在一次编码结束时输出。信息中最后的一个键值对表明了当前处理进度。
- `-stdin`:允许标准输入作为交互。在默认情况下除非标准输入作为真正的输入。要禁用标准输入交互，则你需要显式的使用`-nostdin`进行设置。禁用标准输入作为交互作用是有用的，例如FFmpeg是后台进程组，它需要一些相同的从shell开始的调用（`ffmpeg ... </dev/null`）。
- `-debug_ts (global)`：打印时间码信息，默认是禁止的。这个选项对于测试或者调试是非常有用的特性，或者用于从一种格式切换到另外的格式（包括特性）的时间成本分析，所以不用于脚本处理中。还可以参考`-fdebug ts`选项。
- `-attach filename (output)`：把一个文件附加到输出文件中。这里只有很少文件类型能被支持，例如使用Matroska技术为了渲染字幕的字体文件。附件作为一种特殊的流类型，所以这个选项会添加一个流到文件中，然后你就可以像操作其他流一样使用每种流选项。在应用本选项时，附件流须作为最后一个流(例如根据`-map`映射流或者自动映射时需要注意)。**注意**对于`Matroska`你也可以在元数据标签中进行类型设定：
>
	ffmpeg -i INPUT -attach DejaVuSans.ttf -metadata:s:2 mimetype=application/x-truetype-font out.mkv

(这时要访问到附件流，则就是访问输出文件中的第3个流)

- `-dump_attachment[:stream_specifier] filename （input,per-stream）`：从输入文件中解出指定的附件流到文件filename：
>
	ffmpeg -dump_attachment:t:0 out.ttf -i INPUT

	如果想一次性把所有附件都解出来，则
>
	ffmpeg -dump_attachment:t "" -i INPUT

	技术说明：附件流是作为编码扩展数据来工作的，所以其他流数据也能展开，而不仅仅是这个附件属性。
- `-noautorotate`：禁止自动依据文件元数据旋转视频。

### 视频（video）选项 ###
- `-vframes number (output)`：设置输出文件的帧数，是`-frames:v`的别名。
- `-r[:stream_specifier] fps (input/output,per-stream)`：设置帧率（一种Hz值，缩写或者分数值）。
	 
	在作为输入选项时，会忽略文件中存储的时间戳和时间戳而产生的假设恒定帧率`fps`，即强制按设定帧率处理视频产生（快进/减缓效果）。这不像`-framerate`选项是用来让一些输入文件格式如image2或者v412(兼容旧版本的FFmpeg)等，要注意这一点区别，而不要造成混淆。
	
	作为输出选项时，会复制或者丢弃输入中个别的帧以满足设定达到`fps`要求的帧率。
- `-s[:stream_specifier] size (input/output,per-stream)`：设置帧的尺寸。

	当作为输入选项时，是私有选项`video_size`的缩写，一些文件没有把帧尺寸进行存储，或者设备对帧尺寸是可以设置的，例如一些采集卡或者raw视频数据。

	当作为输出选项是，则相当于`scale`滤镜作用在滤镜链图的最后。请使用`scale`滤镜插入到开始或者其他地方。

	数据的格式是`wxh`，即`宽度值X高度值`，例如`320x240`，（默认同源尺寸）
- `aspect[:stream_specifier] aspect (output,per-stream)`：指定视频的纵横比（长宽显示比例）。`aspect`是一个浮点数字符串或者`num:den`格式字符串(其值就是num/den)，例如"4:3","16:9","1.3333"以及"1.7777"都是常用参数值。

	如果还同时使用了`-vcodec copy`选项，它将只影响容器级的长宽比，而不是存储在编码中的帧纵横比。

- `-vn (output)`：禁止输出视频
- `-vcodec codec (output)`：设置视频编码器，这是`-codec:v`的一个别名。
- `-pass[:stream_specifier] n (output,per-stream)`:选择当前编码数(1或者2)，它通常用于2次视频编码的场景。第一次编码通常把分析统计数据记录到1个日志文件中（参考`-passlogfile`选项），然后在第二次编码时读取分析以精确要求码率。在第一次编码时通常可以禁止音频，并且把输出文件设置为`null`，在windows和类unix分别是:
> 
	ffmpeg -i foo.mov -c:v libxvid -pass 1 -an -f rawvideo -y NUL
	ffmpeg -i foo.mov -c:v libxvid -pass 1 -an -f rawvideo -y /dev/null

- `-passlogfile[:stream_specifier] prefix (output,per-stream)`：设置2次编码模式下日志文件存储文件前导，默认是"ffmepg2pass"，则完整的文件名就是"PREFIX-N.log"，其中的N是指定的输出流序号（对多流输出情况）
- `-vf filtergraph (output)`：创建一个`filtergraph`的滤镜链并作用在流上。它实为`-filter:v`的别名，详细参考`-filter`选项。

### 高级视频选项 ###
- `-pix_fmt[:stream_specifier] format (input/output,per-stream)`：设置像素格式。使用`-pix_fmts`可以显示所有支持的像素格式。如果设置的像素格式不能被选中（启用），则ffmpeg会输出一个警告和并选择这个编码最好（兼容）的像素格式。如果`pix_fmt`前面前导了一个`+`字符，ffmepg会在要求的像素格式不被支持时退出，这也意味着滤镜中的自动转换也会被禁止。如果`pix_fmt`是单独的`+`，则ffmpeg选择和输入（或者滤镜通道）一样的像素格式作为输出，这时自动转换也会被禁止。
- `-sws_flags flags (input/output)`:选择`SwScaler`放缩标志量。
- `-vdt n`：丢弃的门限设置。
- `-rc_override[:stream_specifier] override (output,per-stream)`:在特定时间范围内的间隔覆盖率，`override`的格式是"int\int\int"。其中前两个数字是开始帧和结束帧，最后一个数字如果为正则是量化模式，如果为负则是品质因素。
- `-ilme`：支持交错编码（仅MPEG-2和MPEG-4）。如果你的输入是交错的，而且你想保持交错格式，又想减少质量损失，则选此项。另一种方法是采用`-deinterlace`对输入流进行分离，但会引入更多的质量损失。
- `-psnr`：计算压缩帧的`PSNR`
- `-vstats`：复制视频编码统计分析到日志文件`vstats_HHMMSS.log`
- `-vstats_file file`:复制视频编码统计分析到`file`所指的日志文件中。
- `-top[:stream_specifier] n (output,per-stream)`: 指明视频帧数据描述的起点。`顶部=1/底部=0/自动=-1`（以往CRT电视扫描线模式）
- `-dc precision`：Intra_dc_precision值。
- `-vtag fourcc/tag (output)`:是`-tag:v`的别名，强制指定视频标签/fourCC （FourCC全称Four-Character Codes，代表四字符代码 (four character code), 它是一个32位的标示符，其实就是typedef unsigned int FOURCC;是一种独立标示视频数据流格式的四字符代码。）
- `-qphist (global)`：显示`QP`直方图。
- `-vbsf bitstream_filter`：参考`-bsf`以进一步了解。
- `-force_key_frames[:stream_specifier] time[,time...] (output,per-stream)` ：（见下）
- `-force_key_frames[:stream_specifier] expr:expr (output,per-stream)`：强制时间戳位置帧为关键帧，更确切说是从第一帧起每设置时间都是关键帧（即强制关键帧率）。

	如果参数值是以`expr:`前导的，则字符串`expr`为一个表达式用于计算关键帧间隔数。关键帧间隔值必须是一个非零数值。

	如果一个时间值是"`chapters` [delta]"则表示文件中从`delta`章开始的所有章节点计算以秒为单位的时间，并把该时间所指帧强制为关键帧。这个选项常用于确保输出文件中所有章标记点或者其他点所指帧都是关键帧（这样可以方便定位）。例如下面的选项代码就可以使“第5分钟以及章节chapters-0.1开始的所有标记点都成为关键帧”：	
> 
	-force_key_frames 0:05:00,chapters-0.1

	其中表达式`expr`接受如下的内容：
	- `n`：当前帧序数，从0开始计数
	- `n_forced`：强制关键帧数
	- `prev_forced_n`：之前强制关键帧数，如果之前还没有强制关键帧，则其值为`NAN`
	- `prev_forced_t`：之前强制关键帧时间，如果之前还没有强制关键帧则为`NAN`
	- `t`：当前处理到的帧对应时间。
	
	例如要强制每5秒一个关键帧：
> 
	-force_key_frames expr:gte(t,n_forced*5)
	
	从13秒后每5秒一个关键帧：
> 
	-force_key_frames expr:if(isnan(prev_forced_t),gte(t,13),gte(t,prev_forced_t+5))

	**注意**设置太多强制关键帧会损害编码器前瞻算法效率，采用固定`GOP`选项或采用一些近似设置可能更高效。

- `-copyinkf[:stream_specifier] (output,per-stream)`:流复制时同时复制非关键帧。
- `-hwaccel[:stream_specifier] hwaccel (input,per-stream)`：使用硬件加速解码匹配的流。允许的`hwaccel`值为：
	- `none`：没有硬件加速（默认值）
	- `auto`：自动选择硬件加速
	- `vda`：使用Apple的VDA硬件加速
	- `vdpau`：使用VDPAU（Video Decode and Presentation API for Unix，类unix下的技术标准）硬件加速
	- `dxva2`：使用DXVA2 (DirectX Video Acceleration，windows下的技术标准) 硬件加速。

	这个选项可能并不能起效果（它依赖于硬件设备支持和选择的解码器支持）

	**注意**：很多加速方法（设备）现在并不比现代CPU快了，而且额外的`ffmpeg`需要拷贝解码的帧（从GPU内存到系统内存）完成后续处理（例如写入文件），从而造成进一步的性能损失。所以当前这个选项更多的用于测试。

- `-hwaccel_device:[:stream_specifier] hwaccel_device (input,per-stream)`：选择一个设备用于硬件解码加速。这个选项必须同时指定了`-hwaccel`才可能生效。它也依赖于指定的设备对于特定编码的解码加速支持性能。
	- `vdpau`：对应于`VDPAU`，在`X11`（类Unix）显示/屏幕 上的，如果这个选项值没有选中，则必须在`DISPLAY`环境变量中有设置。
	- `dxva2`：对应于`DXVA2`，这个是显示硬件（卡）的设备号，如果没有指明，则采用默认设备（对于多个卡时）。

### 音频选项 ###
- `-aframes number (output)`：设置`number`音频帧输出，是`-frames:a`的别名
- `-ar[:stream_specifier] freq (input/output,per-stream)`:设置音频采样率。默认是输出同于输入。对于输入进行设置，仅仅通道是真实的设备或者raw数据分离出并映射的通道才有效。对于输出则可以强制设置音频量化的采用率。
- `-aq q (output)`：设置音频品质(编码指定为VBR)，它是`-q:a`的别名。
- `-ac[:stream_specifier] channels (input/output,per-stream)`：设置音频通道数。默认输出会有输入相同的音频通道。对于输入进行设置，仅仅通道是真实的设备或者raw数据分离出并映射的通道才有效。
- `-an (output)`：禁止输出音频
- `-acode codec (input/output)`：设置音频解码/编码的编/解码器，是`-codec:a`的别名
- `-sample_fmt[:stream_specifier] sample_fmt (output,per-stream)`:设置音频样例格式。使用`-sample_fmts`可以获取所有支持的样例格式。
- `-af filtergraph (output)`：对音频使用`filtergraph`滤镜效果，其是`-filter:a`的别名，参考`-filter`选项。

### 高级音频选项 ###
- `-atag fourcc/tag (output)`：强制音频标签/fourcc。这个是`-tag:a`的别名。
- `-absf bitstream_filter`：要深入了解参考`-bsf`
- `-guess_layout_max channels (input,per-stream)`:如果音频输入通道的布局不确定，则尝试猜测选择一个能包括所有指定通道的布局。例如：通道数是2，则`ffmpeg`可以认为是2个单声道，或者1个立体声声道而不会认为是6通道或者5.1通道模式。默认值是总是试图猜测一个包含所有通道的布局，用0来禁用。

### 字幕选项 ###
- `-scodec codec （input/output）`：设置字幕解码器，是`-codec:s`的别名。
- `-sn (output)`：禁止输出字幕
- `-sbsf bitstream_filter`：深入了解请参考`-bsf`

### 高级字幕选项 ###
- `-fix_sub_duration`：修正字幕持续时间。对每个字幕根据接下来的数据包调整字幕流的时间常数以防止相互覆盖（第一个没有完下一个就出来了）。这对很多字幕解码来说是必须的，特别是DVB字幕，因为它在原始数据包中只记录了一个粗略的估计值，最后还以一个空的字幕帧结束。  
	
	这个选项可能失败，或者出现夸张的持续时间或者合成失败，这是因为数据中有非单调递增的时间戳。

	**注意**此选项将导致所有数据延迟输出到字幕解码器，它会增加内存消耗，并引起大量延迟。

- `-canvas_size size`：设置字幕渲染区域的尺寸（位置）

### 高级选项 ###
- `-map [-]input_file_id[:stream_specifier][,sync_file_id[:stream_specifier]] | [linklabel] (output)`：设定一个或者多个输入流作为输出流的源。每个输入流是以`input_file_id`序数标记的输入文件和`input_stream_id`标记的流序号共同作用指明，它们都以0起始计数。如果设置了`sync_file_id:stream_specifier`，则把这个输入流作为同步信号参考。

	命令行中的第一个`-map`选项指定了输出文件中第一个流的映射规则（编号为0的流，0号流），第二个则指定1号流的，以此类推。

	如果在流限定符前面有一个`-`标记则表明创建一个“负”映射，这意味着禁止该流输出，及排除该流。

	一种替代的形式是在复合滤镜中利用`[linklabel]`来进行映射（参看`-filter_complex`选项）。其中的`linklabel`必须是输出滤镜链图中已命名的标签。

	例子：映射第一个输入文件的所有流到输出文件：
> 
	ffmpeg -i INPUT -map 0 output

	又如，如果在输入文件中有两路音频流，则这些流的标签就是"0:0"和"0:1"，你可以使用`-map`来选择某个输出，例如：
>
	ffmpeg -i INPUT -map 0:1 out.wav

	这将只把输入文件中流标签为"0:1"的音频流单独输出到out.wav中。

	再如，从文件a.mov中选择序号为2的流（流标签0:2），以及从b.mov中选择序号为6的流(流标签1:6)，然后共同复制输出到out.mov需要如下写:
>
	ffmpeg -i a.mov -i b.mov -c copy -map 0:2 -map 1:6 out.mov

	选择所有的视频和第三个音频流则是:
>
	ffmpeg -i INPUT -map 0:v -map:a:2 OUTPUT

	选择所有的流除了第二音频流外的流进行输出是：
>
	ffmpeg -i INPUT -map 0 -map -0:a:1 OUTPUT
	
	选择输出英语音频流:
>
	ffmpeg -i INPUT -map 0:m:language:eng OUTPUT

	**注意**应用了该选项将自动禁用默认的映射。
- `-ignore_unknown`：如果流的类型未知则忽略，而不进行复制。
- `-copy_unknown`：复制类型未知的流。
- `-map_channel [input_file_id.stream_specifier.channel_id|-1][:output_file_id.stream_specifier]`:从输入文件中指定映射一个通道的音频到输出文件指定流。如果`output_file_id.stream_specifier`没有设置，则音频通道将映射到输出文件的所有音频流中。

	使用`-1`插入到`input_file_id.stream_specifier.chnnel_id`会映射一个静音通道

	例如`INPUT`是一个立体声音频文件，你可以分别选择两个音频通道(下面实际上对于输入是交换了2个音频通道顺序进行输出)：
>
	ffmpeg -i INPUT -map_channel 0.0.1 -map_channel 0.0.0 OUTPUT

	如果你想静音第一个通道，而只保留第二通道，则可使用:
>
	ffmpeg -i INPUT -map_channel -1 -map_channel 0.0.1 OUTPUT

	以`-map_channel`选项指定的顺序在输出文件中输出音频流通道布局，即第一个`-map_channel`对应输出中第一个音频流通道，第二个对应第二个音频流通道，以此类推（只有一个则是单声道，2个是立体声）。联合使用`-ac`与`-map_channel`，而且在输入的`-map_channel`与`-ac`不匹配（例如只有2个`-map_channel`，又设置了`-ac 6`）时将使指定音频流通道提高增益。

	你可以详细的对每个输入通道指派输出以分离整个输入文件，例如下面就把有`INPUT`文件中的两个音频分别输出到两个输出文件中(OUTPUT_CH0 和 OUTPUT_CH1 )：
>
	ffmpeg -i INPUT -map_channel 0.0.0 OUTPUT_CH0 -map_channel 0.0.1 OUTPUT_CH1

	下面的例子则把一个立体声音频的两个音频通道分离输出到两个相互独立的流（相当于两个单声道了）中（但还是放置在同一个输出文件中）:
>
	ffmpeg -i stereo.wav -map 0:0 -map 0:0 -map_channel 0.0.0:0.0 -map_channel 0.0.1:0.1 -y out.ogg

	**注意**当前一个输出流仅能与一个输入通道连接，既你不能实现利用`-map_channel`把多个输入的音频通道整合到不同的流中（从同一个文件或者不同文件）或者是混合它们成为单独的流，例如整合2个单声道形成立体声是不可能的。但是分离一个立体声成为2个独立的单声道是可行的。

	如果你需要类似的应用，你需要使用`amerge`滤镜，例如你需要整合一个媒体（这里是input.mkv）中的2个单声道成为一个立体声通道(保持视频流不变)，你需要采用下面的命令:
>
	ffmpeg -i input.mkv -filter_complex "[0:1] [0:2] amerge" -c:a pcm_s16le -c:v copy output.mkv

- `-map_metadata[:metadata_spec_out] infile[:metadata_spec_in] (output,per-metadata)`：在下一个输出文件中从`infile`读取输出元数据信息。**注意**这里的文件索引也是以0开始计数的，而不是文件名。参数`metadata_spec_in/out`指定的元数据将被复制，一个元数据描述可以有如下的信息块:
	- `g`:全局元数据，这些元数据将作用于整个文件
	- `s[:stream_spec]`:每个流的元数据，`steam_spec`的介绍在`流指定`章节。如果是描述输入流，则第一个被匹配的流相关内容被复制，如果是输出元数据指定，则所有匹配的流相关信息被复制到该处。
	- `c:chapter_index`:每个章节的元数据，`chapter_index`也是以0开始的章节索引。
	- `p:program_index`：每个节目元数据，`program_index`是以0开始的节目索引
	
	如果元数据指定被省略，则默认是全局的。

	默认全局元数据会从第一个输入文件每个流每个章节依次复制（流/章节），这种默认映射会因为显式创建了任意的映射而失效。一个负的文件索引就可以禁用默认的自动复制。
	
	例如从输入文件的第一个流复制一些元数据作为输出的全局元数据
>
	ffmpeg -i in.ogg -map_metadata 0:s:0 out.mp3

	与上相反的操作，例如复制全局元数据给所有的音频流
>
	ffmpeg -i in.mkv -map_metadata:s:a 0:g out.mkv

	**注意**这里简单的`0`在这里能正常工作是因为全局元数据是默认访问的。
- `-map_chapters input_file_index (output)`:从输入文件中复制由`input_file_index`指定的章节的内容到输出。如果没有匹配的章节，则复制第一个输入文件至少一章内容（第一章）。使用负数索引则禁用所有的复制。
- `-benchmark (global)`：在编码结束后显示基准信息。则包括CPU使用时间和最大内存消耗，最大内存消耗是不一定在所有的系统中被支持，它通常以显示为0表示不支持。
- `-benchmark_all (global)`:在编码过程中持续显示基准信息，则包括CPU使用时间（音频/视频 的 编/解码）
- `-timelimit duration (global)`:ffmpeg在编码处理了`duration`秒后退出。
- `-dump (global)`：复制每个输入包到标准输出设备
- `-hex (global)`:复制包时也复制荷载信息
- `-re (input)`：以指定帧率读取输入。通常用于模拟一个硬件设备，例如在直播输入流（这时是读取一个文件）。不应该在实际设备或者在直播输入中使用（因为这将导致数据包的丢弃）。默认`ffmpeg`会尽量以最高可能的帧率读取。这个选项可以降低从输入读取的帧率，这常用于实时输出（例如直播流）。
- `-loop_input`：循环输入流。当前它仅作用于图片流。这个选项主要用于FFserver自动化测试。这个选项现在过时了，应该使用`-loop 1`。
- `-loop_output number_of_times`：重复播放`number_of_times`次。这是对于GIF类型的动画（0表示持续重复而不停止）。这是一个过时的选项，用`-loop`替代。
- `-vsync parameter`：视频同步方式。为了兼容旧，常被设置为一个数字值。也可以接受字符串来作为描述参数值，其中可能的值是:
	- `0,passthrough`:每个帧都通过时间戳来同步（从解复用到混合）。
	- `1，cfr`：帧将复制或者降速以精准达到所要求的恒定帧速率。
	- `2，vfr`：个别帧通过他们的时间戳或者降速以防止2帧具有相同的时间戳
	- `drop`：直接丢弃所有的时间戳，而是在混合器中基于设定的帧率产生新的时间戳。
	- `-1，auto`：根据混合器功能在1或者2中选择，这是默认值。
	
	**注意**时间戳可以通过混合器进一步修改。例如`avoid_negative_ts`被设置时。

	利用`-map`你可以选择一个流的时间戳作为凭据，它可以对任何视频或者音频 不改变或者重新同步持续流到这个凭据。
- `-frame_drop_threshold parameter`：丢帧的阀值，它指定后面多少帧内可能有丢帧。在帧率计数时1.0是1帧，默认值是1.1。一个可能的用例是避免在混杂的时间戳或者需要增加精准时间戳的情况下确立丢帧率。
- `-async samples_per_second`：音频同步方式。"拉伸/压缩"音频以匹配时间戳。参数是每秒最大可能的音频改变样本。`-async 1`是一种特殊情况指只有开始时校正，后续不再校正。
	
	**注意**时间戳还可以进一步被混合器修改。例如`avoid_negative_ts`选项被指定时

	已不推荐这个选项，而是用`aresample`音频滤波器代替。
- `-copyts`：不处理输入的时间戳，保持它们而不是尝试审核。特别是不会消除启动时间偏移值。

	**注意**根据`vsync`同步选项或者特定的混合器处理流程（例如格式选项`avoid_negative_ts`被设置）输出时间戳会忽略匹配输入时间戳（即使这个选项被设置）
- `-start_at_zero`：当使用`-copyts`,位移输入时间戳作为开始时间0.这意味着使用该选项，同时又设置了`-ss`，例如`-ss 50`则输出中会从50秒开始加入输入文件时间戳。
- `-copytb mode`：指定当流复制时如何设置编码时间基准。`mode`参数是一个整数值，可以有如下可能：
	- `1`表示使用分离器时间基准，从分离器中复制时间戳到编码中。复制可变帧率视频流时需要避免非单调递增的时间戳。
	- `0`表示使用解码器时间基准，使用解码器中获取的时间戳作为输出编码基准。
	- `-1`尝试自动选择，只要能产生一个正常的输出，这是默认值。
- `-shortest (output)`：完成编码时最短输入端。
- `-dts_delta_threshold`：时间不连续增量阀值。
- `-muxdelay seconds (input)`：设置最大 解复用-解码 延迟。参数是秒数值。
- `-maxpreload seconds (input)`：设置初始的 解复用-解码延迟，参数是秒数值。
- `-streamid output-stream-index:new-value (output)`:强制把输出文件中序号为`output-stream-id`的流命名为`new-value`
的值。这对应于这样的场景：在存在了多输出文件时需要把一个流分配给不同的值。例如设置0号流为33号流，1号流为36号流到一个mpegts格式输出文件中（这相当于对流建立链接/别名）：
> 
    ffmpeg -i infile -streamid 0:33 -streamid 1:36 out.ts
- `-bsf[:stream_specifier] bitstream_filters (output,per-stream)`：为每个匹配流设置bit流滤镜。`bitstream_filters`是一个逗号分隔的bit流滤镜列表。可以使用`-bsfs`来获得当前可用的bit流滤镜。
> 
    ffmpeg -i h264.mp4 -c:v copy -bsf:v h264_mp4toannexb -an out.h264
	ffmpeg -i file.mov -an -vn -bsf:s mov2textsub -c:s copy -f rawvideo sub.txt

- `-tag[:stream_specifier codec_tag (input/output,per-stream`：为匹配的流设置标签/fourcc。

- `-timecode hh:mm:ssSEDff`:指定时间码，这里`SEP`如果是`:`则不减少时间码，如果是`;`或者`.`则可减少。
> 
	ffmpeg -i input.mpg -timecode 01:02:03.04 -r 30000/1001 -s ntsc output.mpg
- `-filter_complex filtergraph (global)`：定义一个复合滤镜，可以有任意数量的输入/输出。最简单的滤镜链图至少有一个输入和一个输出，且需要相同类型。参考`-filter`以获取更多信息（更有价值）。`filtergraph`用来指定一个滤镜链图。关于`滤镜链图的语法`可以参考`ffmpeg-filters`相关章节。

	其中输入链标签必须对应于一个输入流。filtergraph的具体描述可以使用`file_index:stream_specifier`语法（事实上这同于`-map`）。如果`stream_specifier`匹配到了一个多输出流，则第一个被使用。滤镜链图中一个未命名输入将匹配链接到的输入中第一个未使用且类型匹配的流。

	使用`-map`来把输出链接到指定位置上。未标记的输出会添加到第一个输出文件。

	**注意**这个选项参数在用于`-lavfi`源时不是普通的输入文件。
>
	ffmpeg -i video.mkv -i image.png -filter_complex '[0:v][1:v]overlay[out]' -map '[out]' out.mkv

	这里`[0:v]`是第一个输入文件的第一个视频流，它作为滤镜的第一个（主要的）输入，同样，第二个输入文件的第一个视频流作为滤镜的第二个输入。

	假如每个输入文件只有一个视频流，则我们可以省略流选择标签，所以上面的内容在这时等价于:
> 
	ffmpeg -i video.mkv -i image.png -filter_complex 'overlay[out]' -map '[out]' out.mkv

	此外，在滤镜是单输出时我们还可以进一步省略输出标签，它会自动添加到输出文件，所以进一步简写为:
> 
	ffmpeg -i video.mkv -i image.png -filter_complex 'overlay' out.mkv
	
	利用`lavfi`生成5秒的 红`color`（色块）:
> 
    ffmpeg -filter_complex 'color=c=red' -t 5 out.mkv
- `-lavfi filtergraph (global)`：定义一个复合滤镜，至少有一个输入和/或输出，等效于`-filter_complex`。
- `-filter_complex_script filename (global)`：这个选项类似于`-filter_complex`，唯一不同就是它的参数是文件名，会从这个文件中读取复合滤镜的定义。
- `-accurate_seek (input)`：这个选项会启用/禁止输入文件的精确定位（配合`-ss`)，它默认是启用的，即可以精确定位。需要时可以使用`-noaccurate_seek`来禁用，例如在复制一些流而转码另一些的场景下。
- `-seek_timestamp (input)`：这个选项配合`-ss`参数可以在输入文件上启用或者禁止利用时间戳的定位。默认是禁止的，如果启用，则认为`-ss`选项参数是正式的时间戳，而不是由文件开始计算出来的偏移。这一般用于具有不是从0开始时间戳的文件，例如一些传输流（直播下）。
- `-thread_queue_size size (input)`：这个选项设置可以从文件或者设备读取的最大排队数据包数量。对于低延迟高速率的直播流，如果不能及时读取，则出现丢包，所以提高这个值可以避免出现大量丢包现象。
- `-override_ffserver (global)`:对`ffserver`的输入进行指定。使用这个选项`ffmpeg`可以把任意输入映射给`ffserver`并且同时控制很多编码可能。如果没有这个选项，则`ffmpeg`仅能根据`ffserver`所要求的数据进行传输。

	这个选项应用场景是`ffserver`需要一些特性，但文件/设备不提供，这时可以利用`ffmpeg`作为中间处理环节控制后输出到`ffserver`到达所需要求。
- `-sdp_file file (global)`：输出`sdp`信息到文件`file`。它会在至少一个输出不是`rtp`流时同时输出`sdp`信息。
- `-discard (input)`：允许丢弃特定的流或者分离出的流上的部分帧，但不是所有的分离器都支持这个特性。
	- `none`：不丢帧
	- `default`：丢弃无效帧
	- `noref`：丢弃所有非参考帧
	- `bidir`：丢弃所有双向帧
	- `nokey`：丢弃所有非关键帧
	- `all`：丢弃所有帧
- `-xerror (global)`:在出错时停止并退出


作为一个特殊的例外，你可以把一个位图字幕（bitmap subtitle）流作为输入，它将转换作为同于文件最大尺寸的视频（如果没有视频则是720x576分辨率）。**注意**这仅仅是一个特殊的例外的临时解决方案，如果在`libavfilter`中字幕处理方案成熟后这样的处理方案将被移除。

例如需要为一个储存在DVB-T上的MPEG-TS格式硬编码字幕，而且字幕延迟1秒：
>
    ffmpeg -i input.ts -filter_complex \
	 '[#0x2ef] setpts=PTS+1/TB [sub] ; [#0x2d0] [sub] overlay' \
 	 -sn -map '#0x2dc' output.mkv

(0x2d0, 0x2dc 以及 0x2ef 是MPEG-TS 的PIDs，分别指向视频、音频和字幕流，一般作为MPEG-TS中的0:0,0:3和0：7是实际流标签)

### 预设文件 ###
一个预设文件是选项/值对的序列(option=value)，每行都是一个选项/值对， 用于指定一系列的选项，而这些一般很难在命令行中指定（限于命令行的一些限制，例如长度限制）。以`#`开始的行是注释，会被忽略。一般`ffmpeg`会在目录树中检查`presets`子目录以获取预设文件。

有两种类型的预设文件:ffpreset 和 avpreset。

#### ffpreset类型预设文件 ####
采用`ffpreset`类型预设文件主要包含`vpre`、`apre`、`spre`和`fpre`选项。其中`fpre`选项的参数可以代替预设的名称作为输入预设文件名，以用于任何一种编码格式。对于`vpre`、`apre`和`spre`选项参数会指定一个预设定文件用于当前编码格式以替代（作为）同类项的预订选项。

选用预设文件传递`vpre`、`apre`和`spre`的参数`arg`有下面一些搜索应用规则：

- 将在目录`$FFMPEG_DATADIR`(如果设置了)和`$HOME/.ffmpeg`目录和配置文件中定义的数据目录（一般是`PREFIX/share/ffmpeg`），以及`ffpresets`所在的执行文件目录下ffmpeg搜索对应的预定义文件`arg.ffpreset`，例如参数是`libvpx-1080p`,则对应于文件`libvpx-1080p.ffpreset`
- 如果没有该文件，则进一步在前述目录下搜索`codec_name-arg.ffpreset`文件，如果找到即应用。例如选择了视频编码器`-vcodec libvpx`和`-vpre 1080p`则对应的预设文件名是`libvpx-1080p.ffpreset`

#### avpreset类型预设文件 ####
`avprest`类型预设文件以`pre`选项引入。他们工作方式类似于`ffpreset`类型预设文件（即也是选项值对序列），但只对于特定编码器选项，因此一些 选项值 对于不适合的编码器是无效的。根据`pre`的参数`arg`查找预设文件基于如下规则：
- 首先搜索`$AVCONV_DATADIR`所指目录(如果定义了)，其次搜索`$HOME/.avconv`目录，然后搜索执行文件所在目录(通常是`PREFIX/share/ffmpeg`)，在其下查找`arg.avpreset`文件。第一个匹配的文件被应用。
- 如果查找不到，如果还同步还指定了编码（如`-vcodec libvpx`）再以前面目录顺序，以`codec_name-arg.avpreset`再次查找文件。例如对于有选项`-vcodec libvpx`和`-pre 1080p`将搜索`libvpx-1080p.avpreset`
- 如果还没有找到，将在当前目录下搜索`arg.avpreset`文件。

