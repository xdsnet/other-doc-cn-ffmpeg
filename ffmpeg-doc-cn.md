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