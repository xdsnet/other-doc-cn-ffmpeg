# ffmpeg 翻译文档 ([ffmpeg-all](http://ffmpeg.org/ffmpeg-all.html "ffmpeg-all") 包含重要组件） #
## 命令语法 ##
    ffmpeg [全局选项] {[输入文件选项] -i 输入文件} ... {[输出文件选项] 输出文件} ...
即
    ffmpeg [global_options] {[input_file_options] -i input_file} ... {[output_file_options] output_file} ...

## 描述/概览 ##
`ffmpeg`是一个非常快的视频/音频转换器，其也可以现场抓取音频/视频源，并在任意采用率之间调整视频，以及提供多种高品质的滤镜系统。

`ffmpeg`从任意数量/形式的输入文件中进行读取（可以是普通文件，管道，网络流，设备源等等），通过输入文件选项对输入文件进行设定，通过`-i`进行标记，并写入到任意数量/形式的输出文件中，任何在命令行中不能被解释为选项的字符串信息（当然也不是被`-i`指定为输入文件的信息）都被作为一个输出文件。

原则上每个输入或输出文件都可包含数量不同的数据流（视频/音频/字幕/附件/数据....）,具体文件中包含的数量和/或数据类型是文件的容器格式限定的，具体选择那些流从输入文件到输出文件则可能是自动或者依据`-map`选项（流选择章中介绍）来指定。

为了明确指定输入文件，你必须采用从0开始的数字索引法，即第1个输入文件由`0`索引，第2个则是`1`。同样的，在一个文件中指定数据流也是通过同样规则的索引法，即`2:3`表示第3个输入文件的第4个数据流。这些内容也可以参考流说明章节。

作为一般规则，选项用于指定紧接着的文件，因此命令中顺序很重要，你可以在命令中多次重复相同的选项，每次都可以应用于紧接着的下一个输入或者输出文件。例外的是全局选项（例如过程信息输出详细程度的选项），这些选项必须首先进行指定，会全局使用。

不要混淆输入和输出文件，要先指定所有的输入文件，然后才是所有的输出文件。也不要混淆选项应用的不同文件，所有的选项**仅仅**作用于紧接着的输入或者输出文件，除非重复指定选项才能作用于其他需要同样设定的文件。
- 为了设定输出视频码率为64kbit/s：

	`ffmpeg -i input.avi -b:v 64k -bufsize 64k output.avi`
- 为了切换帧率到24fps：

	`ffmpeg -i input.avi -r 24 output.avi`

- 为了强制把输入文件帧率设为1fps（仅为了建议raw格式数据）， 并且把输出文件帧率设置为24fps：
	
	`ffmpeg -r 1 -i input.m2v -r 24 output.avi`

	**注意**这里的输入文件必须是raw格式的输入文件。

## 详细说明 ##
`ffmpeg`的每个转换过程像下图描述的程序
> 
     _______              ______________
	|       |            |              |
	| input |  demuxer   | encoded data |   decoder
	| file  | ---------> | packets      | -----+
	|_______|            |______________|      |
    	                                       v
    	                                   _________
    	                                  |         |
    	                                  | decoded |
    	                                  | frames  |
    	                                  |_________|
 	 ________             ______________       |
	|        |           |              |      |
	| output | <-------- | encoded data | <----+
	| file   |   muxer   | packets      |   encoder
	|________|           |______________|

`ffmpeg`调用libavformat库（含分离器）读取输入文件，分离出各类编码的数据包（流），当有多个输入文件时，`ffmpeg`试图跟踪最低时间戳实现任意输入流同步。编码数据包（除非是指定为流式拷贝，相关内容请参考特性描述对流式拷贝的说明）通过解码器解码出非压缩的数据帧（raw视频/PCM格式音频...），这些数据帧可以被滤镜进一步处理（下面会讲到）。经过滤镜处理的数据被重新编码为新的数据包（流），然后经过混合器混合（例如按一定顺序和比例把音频数据包和视频数据包交叉组合），写入到输出文件。

### 滤镜处理/Filtering ###
在编码前，`ffmpeg`可以对raw（真实/原）音频和视频使用libavfilter库中的滤镜进行处理。多个滤镜可以组成滤镜图（滤镜链图filtergraphs ）。在`ffmpeg`看来只有2种滤镜：简单滤镜，复合滤镜。
#### 简单滤镜 ####
简单滤镜就是只有1个输入和输出的滤镜，滤镜两边的数据都是同一类型的，可以理解为在非压缩数据帧到再次编码前简单附加了一步:
> 
	 _________                        ______________
	|         |                      |              |
	| decoded |                      | encoded data |
	| frames  |\                   _ | packets      |
	|_________| \                  /||______________|
	           	 \   __________   /
	  simple     _\||          | /  encoder
	  filtergraph   | filtered |/
	                | frames   |
	                |__________|

简单滤镜一般用于配置每个流 -filter 选项（-vf 和 -af 分别对应视频和音频）。一个最简单的视频滤镜如下:
> 
	 _______        _____________        _______        ________
	|       |      |             |      |       |      |        |
	| input | ---> | deinterlace | ---> | scale | ---> | output |
	|_______|      |_____________|      |_______|      |________|


**注意**一些滤镜改变帧属性而不是帧内容。例如前面提到的fps滤镜就只是引起帧率的变化，但不处理帧内容，另外一个例子是setpts则仅仅设置时间戳，通过滤镜的帧内容完全不变化。
#### 复合滤镜 ####
复合滤镜是那些不能简单描述为一个线性处理过程应用到一个流的情况，例如当过程中有多个输入和/或输出，或者输出流类型不同于输入时，示意图如下：
> 
	 _________
	|         |
	| input 0 |\                    __________
	|_________| \                  |          |
	             \   _________    /| output 0 |
	              \ |         |  / |__________|
	 _________     \| complex | /
	|         |     |         |/
	| input 1 |---->| filter  |\
	|_________|     |         | \   __________
	               /| graph   |  \ |          |
	              / |         |   \| output 1 |
	 _________   /  |_________|    |__________|
	|         | /
	| input 2 |/
	|_________|

复合滤镜由`-filter_complex`选项进行设定。**注意**这是一个全局选项，因为一个复合滤镜必然是不能只关联到一个单一流或者文件的。`-lavfi`选项等效于`-filter_complex`

一个复合滤镜的简单例子就是`overlay`滤镜，它从两路输入中，把一个视频叠加到一个输出上。对应的类似音频滤镜是`amix`。

### 流拷贝 ###
流拷贝（Stream copy）是一种对指定流数据仅仅进行复制的`拷贝（copy）`模式。这种情况下`ffmpeg`不会对指定流进行解码和编码步骤，而仅仅是分离和混合数据包。这种模式常用于文件包装格式的转换或者修改部分元数据信息，这个过程简单图示如下：
> 
	 _______              ______________            ________
	|       |            |              |          |        |
	| input |  demuxer   | encoded data |  muxer   | output |
	| file  | ---------> | packets      | -------> | file   |
	|_______|            |______________|          |________|

因为这种模式下不存在解码和编码过程，所以也特别快，而且不会造成新的质量损失。然而这也使得这样的模式不能适合很多工作需求，例如这个模式下不能使用大量的滤镜了，因为滤镜仅能对未压缩（编码）的数据进行处理。

## 流的选择（指定） ##
默认情况下，`ffmpeg`把输入文件每种类型（视频、音频和字幕）仅仅采用一个流转换输出到输出文件中，就是把*最好*效果的流进行输出：对于视频就是质量最高的流，对于音频就是包含最多声道的，对于字幕则是第一个字幕轨道，如果有多个同型同率（同样类型，码率相同）则选用索引号最小的流。

当然，你可以禁用默认设置，而采用`-vn/-an/-sn`选项进行专门的指定，如果要进行完全的手动控制，则是以`-map`选项，它将禁止默认值而选用指定的配置。

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
