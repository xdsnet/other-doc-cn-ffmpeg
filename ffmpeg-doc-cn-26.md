## 26 输入设备 ##
FFmpeg中的输入设备配置元素用来启用对附加到您的系统一个多媒体设备访问数据。

当编译时，默认会支持所有的输入设备。你可以通过在配置脚本执行时附加`–list-indevs`了解到支持的设备。

可以通过`–disable-indevs`在编译时禁用所有输入设备，也可以在此基础上通过`–enable-indev=INDEV`允许个别设备，或者在默认支持基础上通过`–disable-indev=INDEV`禁用个别设备支持达到类似的目的。

在ff*工具集中，使用`-devices`可以获取当前支持的设备信息。

下面是当前可用的输入设备介绍。

### alsa ###
ALSA (Advanced Linux Sound Architecture——高级Linux音频架构) 输入设备

为了能够使用这个设备，在你的系统上必须安装有`libasound`库。

这个设备允许从ALSA设备采集，设备通过名称来作为ALSA卡标识符，以进行采集。

ALSA标识语法为：

    hw:CARD[,DEV[,SUBDEV]]
这里`DEV`和`SUBDEV`是可选的。通过这3个参数（`CARD`、`DEV`和`SUBDEV`）可以指定一个卡的序号或者标识、设备序号和子设备序号（-1意味着任何一个）

在你的系统上要列出当前可用的卡，可以通过文件：`/proc/asound/cards and /proc/asound/devices`

例如要利用FFmpeg采集ALSA设备（卡ID为0），你可以如下：

    ffmpeg -f alsa -i hw:0 alsaout.wav

更多信息参考[http://www.alsa-project.org/alsa-doc/alsa-lib/pcm.html](http://www.alsa-project.org/alsa-doc/alsa-lib/pcm.html)

### avfoundation ###
AVFoundation 输入设备

AVFoundation是当前Apple OSX（>=10.7）下建议的流采集框架，它在IOS上也是可用的。而老的`QTKit`框架从OSX10.7开始已经废弃。

这个设备作为输入文件名的语法为：

    -i "[[VIDEO]:[AUDIO]]"
第一部分选择视频输入，然后选择音频输入。流必须通过设备列表中的设备名或者设备索引号来指定。或者视频和/或音频输入设备可以通过使用` -video_device_index <INDEX>`和/或`-audio_device_index <INDEX>`语法指定，它将覆盖设备名或者索引来作为输入文件名。

所有有效的设备都可以通过使用`-list_devices true`枚举出来，它会列出所有设备的名称以及对应的索引号。

下面是两个设备的别名：

- default：选择AVFoundation默认设备（类型）。
- none：不记录相应的媒体类型，这相当于指定一个空的设备名或者索引

**译者补注**：none可以用来在进行指定时明确表示没有某种类型，比如

    -i "none:[AUDIO]"
表示没有视频只有音频

#### avfoundation选项 ####
avfoundation支持如下的选项：

- -list_devices <TRUE|FALSE>

    如果设置为`true`则列出所有有效输入设备，显示设备名和对应的索引
- -video_device_index <INDEX>

    通过索引指定视频设备，它将覆盖作为输入文件名
- -audio_device_index <INDEX>

    通过索引指定音频设备，它将覆盖作为输入文件名
- -pixel_format <FORMAT>

    描述视频设备采用的像素格式，如果不知道，将列出可用设备中第一个有效的支持格式。像素格式是： monob, rgb555be, rgb555le, rgb565be, rgb565le, rgb24, bgr24, 0rgb, bgr0, 0bgr, rgb0, bgr48be, uyvy422, yuva444p, yuva444p16le, yuv444p, yuv422p16, yuv422p10, yuv444p10, yuv420p, nv12, yuyv422, gray

#### avfoundation例子 ####

- 输出AVFoundation支持的设备

    $ ffmpeg -f avfoundation -list_devices true -i ""

- 从视频设备0和音频设备0 采集输出到into out.avi:

    $ ffmpeg -f avfoundation -i "0:0" out.avi

- 从视频输入设备2和音频输入设备1采集输出到 out.avi:

    $ ffmpeg -f avfoundation -video_device_index 2 -i ":1" out.avi

- 从系统默认视频设备以bgr0像素格式采集，而不采集音频到out.avi:

    $ ffmpeg -f avfoundation -pixel_format bgr0 -i "default:none" out.avi

### bktr ###
BSD 视频输入设备

### decklink ###
decklink输入设备提供从Blackmagic DeckLink 采集的能力

要支持这个设备，编译时需要Blackmagic DeckLink SDK ，且需要采用`--extra-cflags`和`--extra-ldflags`编译选项。在Windows，你可能需要通过`widl`运行IDL。

DeckLink非常挑剔支持输入格式。像素格式万恶有`uyvy422/210`.对于视频你必须利用`-list_formats 1`指定一个视频画面尺寸（-list_formats 1.）和帧率。音频采样率被设置为48KHz。音频数可能是2、8或16

#### decklink选项 ####

- list_devices

    如果设置为`true`，输出设备列表然后退出，默认为false.
- list_formats

    如果设置为`true`,输出支持的格式然后退出，默认为`false.
- bm_v210

    如果设置为1，则视频采集采用10bit量化的uyvy422 v210标准。不是所有的Blackmagic设备都支持这个选项

#### decklink例子 ####
- 列出所有输入设备:

    ffmpeg -f decklink -list_devices 1 -i dummy
- 列出支持的格式:

    ffmpeg -f decklink -list_formats 1 -i 'Intensity Pro'
- 采集1080i50视频格式 (format 11):

    ffmpeg -f decklink -i 'Intensity Pro@11' -acodec copy -vcodec copy output.avi
- 以10bit采集1080i50视频格式:

    ffmpeg -bm_v210 1 -f decklink -i 'UltraStudio Mini Recorder@11' -acodec copy -vcodec copy output.avi
- 采集720p50格式，同时采集32bit音频:

    ffmpeg -bm_audiodepth 32 -f decklink -i 'UltraStudio Mini Recorder@14' -acodec copy -vcodec copy output.avi
- 采集576i50采集视频，同时采集8路音频:

    ffmpeg -bm_channels 8 -f decklink -i 'UltraStudio Mini Recorder@3' -acodec copy -vcodec copy output.avi

### dshow ###
Windows DirectShow 输入设备。

DirectShow在ffmpeg中由`mingw-w64`项目提供支持。当前只有音频和视频设备能够使用。

多个单独输入的设备可能被打开,但它们也可能打开相同的输入,这将改善他们之间的同步

输入名可以按格式（语法）：

    TYPE=NAME[:TYPE=NAME]
这里`TYPE`可以是`audio`或者`video`，`NAME`是设备名或者别名。

#### dshow选项 ####
如果没有特别指定，将采用设备的默认值。如果设备不支持要求的选项，则会打开失败。

- video_size

    设置采集视频的尺寸
- framerate

    设置采集视频的帧率
- sample_rate

    设置采集音频的采样率（单位Hz）
- sample_size

    设置采集音频的采样位深（单位bits）
- channels

    选择采集音频的通道
- list_devices

    如果为真，输出设备列表并退出
- list_options

    如果为真，输出选择设备的选项列表并退出
- video_device_number

    对视频设备名设置索引编号(从0开始，默认0).
- audio_device_number

    对音频设备名设置索引编号(从0开始，默认0).
- pixel_format

    选择用于DirectShow的像素格式。当视频编码没有设置或者设置为`rawvideo`时需要设置
- audio_buffer_size

    以milliseconds为单位设置音频设备缓冲大小（它可以直接影响延迟，则取决于依赖的设备）。默认使用设备默认缓冲（通常为500ms的倍数）。这个值设置过低会降低性能。参考[http://msdn.microsoft.com/en-us/library/windows/desktop/dd377582(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/dd377582(v=vs.85).aspx)
- video_pin_name

    通过pin名称（或者别名）选择视频捕获源
- audio_pin_name

    通过pin名称（或者别名）选择音频捕获源
- crossbar_video_input_pin_number

    从交错/交叉设备（音视频交错编码）中选择视频输入端口。可以选择交错设备的视频解码输出端。**注意**改变这个值将影响未来的调用（设置了一个新的默认值），直到发生系统重启
- crossbar_audio_input_pin_number

    从交错/交叉设备（音视频交错编码）中选择音频输入端口。可以选择交错设备的音频解码输出端。**注意**改变这个值将影响未来的调用（设置了一个新的默认值），直到发生系统重启
- show_video_device_dialog

    如果设为真，在开始采集前会弹出一个面向用户的对话框，以允许他们改变视频滤镜属性和一些手动配置。**注意**对于交错设备，可能需要同时在PAL (25 fps)和NTSC (29.97)输入帧率、尺寸、隔行等等属性。改变这些值以不同的扫描率/帧率和避免底部绿色、闪烁的扫描行等等。**注意**这些改变将影响未来的调用（作为新的默认值）直到系统被重启
- show_audio_device_dialog

    如果为真，将在开始采集前弹出一个面向用户的对话框，以允许他们改变音频滤镜属性和一些手动配置
- show_video_crossbar_connection_dialog

    如果为真，如果打开视频设备将在开始采集前弹出一个面向用户的对话框，以允许手动编辑交错设备路由
- show_audio_crossbar_connection_dialog

    如果为真，如果打开音频设备将在开始采集前弹出一个面向用户的对话框，以允许手动编辑交错设备路由
- show_analog_tv_tuner_dialog

    如果为真，将在开始采集前弹出一个面向用户的对话框，以允许手动调整电视频道/频率
- show_analog_tv_tuner_audio_dialog

    如果为真，将在开始采集前弹出一个面向用户的对话框，以允许手动调整电视音频设置 (例如 mono与stereo, 语言 A,B 或者 C)
- audio_device_load

    从文件加载一个音频捕获设备而不是根据名字搜索。它可以同时加载附加参数（如果滤镜支持）。它用于音频源必须指定为一个值，但可以是任何虚拟的
- audio_device_save

    存储当前音频采集滤镜设备和他们的参数（如果滤镜支持）到一个文件。如果文件存在则被覆盖（这个文件可以被`audio_device_load`加载）
- video_device_load

    从文件加载一个视频捕获设备而不是根据名字搜索。它可以同时加载附加参数（如果滤镜支持）。它用于视频源必须指定为一个值，但可以是任何虚拟的
- video_device_save

    存储当前视频采集滤镜设备和他们的参数（如果滤镜支持）到一个文件。如果文件存在则被覆盖（这个文件可以被`video_device_load`加载）


#### dshow例子 ####
- 输出DirectShow支持的设备列表并退出:

    $ ffmpeg -list_devices true -f dshow -i dummy

- 打开摄像头:

    $ ffmpeg -f dshow -i video="Camera"

- 打开名为`Camera`的第二个视频设备:

    $ ffmpeg -f dshow -video_device_number 1 -i video="Camera"

- 打开摄像头和话筒:

    $ ffmpeg -f dshow -i video="Camera":audio="Microphone"

- 输出选择设备支持的选项列表并退出:

    $ ffmpeg -list_options true -f dshow -i video="Camera"

- 通过名字/别名指定pin名来采集，指定别名设备名:

    $ ffmpeg -f dshow -audio_pin_name "Audio Out" -video_pin_name 2 -i video=video="@device_pnp_\\?\pci#ven_1a0a&dev_6200&subsys_62021461&rev_01#4&e2c7dd6&0&00e1#{65e8773d-8f56-11d0-a3b9-00a0c9223196}\{ca465100-deb0-4d59-818f-8c477184adf6}":audio="Microphone"

- 配置交错设备，指定交错pin，允许在开始时进行视频采集属性调整:

    $ ffmpeg -f dshow -show_video_device_dialog true -crossbar_video_input_pin_number 0
         -crossbar_audio_input_pin_number 3 -i video="AVerMedia BDA Analog Capture":audio="AVerMedia BDA Analog Capture"

### dv1394 ###
Linux DV1394输入设备

### fbdev ###
Linux framebuffer（Linux帧缓冲）输入设备

Linux framebuffer是一种独立于硬件的图像抽象层，它用于在计算机屏幕上显示图像,通常是在控制台（环境）。它可以通过一个文件设备节点访问，通常为：`/dev/fb0`

要了解更多详细信息请阅读Linux源码文件树下文档：`Documentation/fb/framebuffer.txt`

为了从`/dev/fb0`读取：

    ffmpeg -f fbdev -r 10 -i /dev/fb0 out.avi
你可以通过下面的命令截屏：

    ffmpeg -f fbdev -frames:v 1 -r 1 -i /dev/fb0 screenshot.jpeg
此外还可以在[http://linux-fbdev.sourceforge.net/](http://linux-fbdev.sourceforge.net/)了解fbset(1)。

### gdigrab ###
Win32 GDI 屏幕截取设备

这个设备允许你截取显示在Windows（系统）上的屏幕区域。

它有两个可选的输入文件名（形式）：`desktop`或者`title=window_title`

第一个可选名（`desktop`）会截取整个桌面或者桌面的指定区域，第二个可选名（根据窗口标题）会截取单独的窗口，而无论在屏幕上的位置（即即使根据某些操作，该窗口已经移除屏幕可见区域，或者被其他窗口覆盖了也可以截取到）

下面是截取整个桌面的例子：

    ffmpeg -f gdigrab -framerate 6 -i desktop out.mpg
截取桌面上从点（10,20)开始的640x480大小区域

    ffmpeg -f gdigrab -framerate 6 -offset_x 10 -offset_y 20 -video_size vga -i desktop out.mpg
截取名为 "Calculator"的窗口

    ffmpeg -f gdigrab -framerate 6 -i title=Calculator out.mpg

#### gdigrab选项 ####
- draw_mouse

    为1指定是截取鼠标，0表示不截取，默认为1
- framerate

    设置帧率，默认为ntsc,相应帧率为30000/1001.
- show_region

    在屏幕上显示截取区域。Show grabbed region on screen.

    如果指定为1，则指定的截取范围会显示在屏幕上，通过这个选项，可以很容易的知道要截取的范围，这在只截取屏幕的一部分时很有用。

    **注意**`show_region`在截取单独窗口时无效（即不可用）

    例如:

    	ffmpeg -f gdigrab -show_region 1 -framerate 6 -video_size cif -offset_x 10 -offset_y 20 -i desktop out.mpg

- video_size

    设置视频帧尺寸，默认为屏幕（以`desktop`为源）或者窗口（以`title=window_title`为源）尺寸
- offset_x

    当区域截取时,起点的x轴的偏移（左边距屏幕左边距离）

    **注意**坐标体系是以可见屏幕左上为原点的。如果你有一个监看对象从左边超出了屏幕可见范围，则`offset_x`的值为负数值。
- offset_y

    当区域截取时,起点的y轴的偏移（上边距屏幕上边距离）

    **注意**坐标体系是以可见屏幕左上为原点的。如果你有一个监看对象从上边超出了屏幕可见范围，则`offset_y`的值为负数值

### iec61883 ###
使用`iec61883`的FireWire（火线） DV/HDV输入设备。

要允许这个输入设备，需要`libiec61883`, `libraw1394` 和 `libavc1394`被安装到系统中。此外还要在编译时配置`--enable-libiec61883`以支持。

`iec61883`支持通过 IEEE1394 (FireWire)接口连接设备获取视频(使用`libiec61883` 和新的Linux FireWire stack (火线堆栈juju)）。从Linux Kernel 2.6.37开始它是默认的DV/HDV输入方法了，而老的`FireWire stack`已经被移除。

#### iec61883的选项 ####
- dvtype

    覆盖自动检测的DV/HDV类型。它仅用于自动检测类型失败的情况，或需要禁止者需要的格式被禁止的条件。错误的指定将使设备不能正常工作。选项支持`auto`，`dv`和`hdv`为参数
- dvbuffer

    对传入的数据设置缓冲（单位帧）。对于DV，它是一个精确的帧数，对于HDV，它不是精确的帧数，因为HDV没有一个固定的帧大小。
- dvguid

    通过GUID来指定截取设备ID。这样捕获将仅从指定的设备，或者失败（没有指定的设备）。对于系统连接了多个可用设备的情况它非常有用。在系统的`/sys/bus/firewire/devices`可以找到连接设备的GUID。

#### iec61883的例子 ####
- 获取播放FireWire DV/HDV

	ffplay -f iec61883 -i auto
- 获取记录FireWire DV/HDV，包缓冲大小为100000个包

	ffmpeg -f iec61883 -i auto -hdvbuffer 100000 out.mpg

### jack ###
JACK输入设备。

为了能使用JACK设备，需要系统上存在`libjack`

一个JACK输入设备创建1个或者多个JACK可写客户端，每一个对应于一个音频通道，命名（指定）为`client_name:input_N`，这里`client_name`由程序提供，`N`是通道id号。每个可写客户端作为ffmpeg的输入设备发送数据。

你一次可以创建1个或者多个JACK可读客户端，来连接到1个或者多个JACK可写客户端。

可以使用`jack_connect`和`jack_disconnect`连接或者断开（不连接）JACK客户端，或者通过图形化接口实现：例如通过`qjackctl`，

可以通过`jack_lsp`来列出JACK客户端和它们的属性列表。

下面的例子展示ffmpeg如何从JACK可读客户端采集数据：
> 
	# Create a JACK writable client with name "ffmpeg".
	$ ffmpeg -f jack -i ffmpeg -y out.wav
	
	# Start the sample jack_metro readable client.
	$ jack_metro -b 120 -d 0.2 -f 4000
	
	# List the current JACK clients.
	$ jack_lsp -c
	system:capture_1
	system:capture_2
	system:playback_1
	system:playback_2
	ffmpeg:input_1
	metro:120_bpm
	
	# Connect metro to the ffmpeg writable client.
	$ jack_connect metro:120_bpm ffmpeg:input_1

更多信息参考[http://jackaudio.org/](http://jackaudio.org/)

### lavfi ###
`Libavfilter`输入虚拟设备

这个输入设备可以从`libavfilter`滤镜链图的一个开放输出端口读取数据。

对于每个滤镜链图开放输出端口，这个输入设备将创建一个对应的流映射到这个端口进行输出。当前只支持视频数据。滤镜链图是通过选项`graph`描述的。

#### lavfi选项 ####
- graph

	描述用作输入的滤镜链图。每个视频开放输出必须由一个形如`outN`的独立标签命名，这里`N`是从0开始的数字，以指代要映射作为设备的输入流（序号）。第一个没有标签命名的输出自动被作为`out0`，但所有其他的必须明确指定。

	通过附加后缀“+subcc”可以向输出标签创建一个额外的封闭包装字幕（实验性质：现只对EIA-608 / CEA-708）。这个`subcc`流在所有其它常规流创建后才附加，并按对应流顺序。例如有 "out19+subcc", "out7+subcc" 以及最高普通流"out42"，则43号流是`subcc`对应于`out7`，44号流也是`subcc`流对应`out19`

	如果没有指定（选项）默认值为输入设备指定的文件名（这里文件名其实是滤镜链图描述）
- graph_file

    设置通过文件读取/发送（给其他滤镜）滤镜链图的文件名。在文件中的语法与通过`graph`选项描述滤镜链图的语法相同。

#### lavi例子 ####
- 创建一个颜色流并播放：

    ffplay -f lavfi -graph "color=c=pink [out0]" dummy
- 类似前面的例子，但是有文件名来指定滤镜链图描述，并且省略了"out0"标签:

    ffplay -f lavfi color=c=pink
- 创建3个不同的视频测试滤镜源并播放:

    ffplay -f lavfi -graph "testsrc [out0]; testsrc,hflip [out1]; testsrc,negate [out2]" test3
- 从文件中使用`amovie`读取音频流来播放:

    ffplay -f lavfi "amovie=test.wav"
- 读取音频和视频流来播放:

    ffplay -f lavfi "movie=test.avi[out0];amovie=test.wav[out1]"
- 复制解码出来的帧和对应字幕到图片（实验）:

    ffmpeg -f lavfi -i "movie=test.ts[out0+subcc]" -map v frame%08d.png -map s -c copy -f rawvideo subcc.bin

### libcdio ###
基于`libcdio`的音乐CD输入设备。

需要系统中有`libcdio`才能启用，且编译时需要用`--enable-libcdio`配置选项允许。

设备允许从音频CD播放和获取

例如利用ffmpeg在`/dev/sr0`获取整个音频CD内容:

    ffmpeg -f libcdio -i /dev/sr0 cd.wav
#### libcdio选项 ####
- speed

    设置读取速度，默认为0

	这个速度指定了CD-ROM速度，它通过`libcdio`的`cdio_cddap_speed_set`函数设置。很多CD-ROM驱动器如果设置更大的值将获得更快的速度。
- paranoia_mod

    设置纠偏恢复模式的标志，它接受下面的值：
	
	‘disable’  
	‘verify’  
	‘overlap’  
	‘neverskip’  
	‘full’  

	默认值是‘disable’  
	
	关于可用纠偏模式的更多信息，请咨询纠偏项目文档
### libdc1394 ###
IIDC1394输入设备，其基于`libdc1394`和`libraw1394`

编译允许需要配置`--enable-libdc1394`

### openal ###
这个OpenAL输入设备支持在所有实现了`OpenAL 1.1`的系统上进行音频捕获。

要编译使用它需要系统包含`OpenAL`头和`libraries`库，并且设置编译选项`--enable-openal`

`OpenAL`头和`libraries`库可以是你OpenAL实现的部分，或者作为附件下载（SDK）。根据你的安装方式，你可能需要通过`--extra-cflags`和`--extra-ldflags`为编译指定本地的头文件和库文件

兼容OpenAL的实现有：

- **Creative**

	官方的Windows实现,提供后备支持硬件加速的设备和软件，参考[http://openal.org/](http://openal.org/)
- **OpenAL Soft**

	便携式,开源(LGPL)软件实现。包括在Windows,Linux、Solaris、BSD操作系统上提供最常见的后端声音api。参考[http://kcat.strangesoft.net/openal.html](http://kcat.strangesoft.net/openal.html)
- **Apple**
	
	OpenAL是核心音频的一部分,官方的Mac OSX音频接口。参考[ http://developer.apple.com/technologies/mac/audio-and-video.html]( http://developer.apple.com/technologies/mac/audio-and-video.html)

这个设备允许通过OpenAL处理来捕获音频输入。

你需要通过提供文件名来指定捕获设备的名称。如果为空字符串（''），则会自动选择默认设备。你可以通过`list_devices`获取到支持设备列表。

#### openal选项 ####
- channels

    设置捕获音频的通道。只有1（单声道）和2（立体声）被支持，默认为2
- sample_size

    设置音频采样位宽（单位bit——位）。当前只支持8和16，默认16
- sample_rate

    设置音频采样频率（单位Hz），默认44.1k.
- list_devices

    如果为真(true)，则列出系统上支持的设备并退出，默认为false.

#### openal例子 ####
- 输出OpenAL支持设备列表并退出

	$ ffmpeg -list_devices true -f openal -i dummy out.ogg
- 从设备`DR-BT101 via PulseAudio`捕获音频:
	
	$ ffmpeg -f openal -i 'DR-BT101 via PulseAudio' out.ogg
- 从默认设备捕获音频 (**注意**文件名字符串为空):
	
	$ ffmpeg -f openal -i '' out.ogg
- 同时从两个设备捕获，写入不同的文件:
	
	$ ffmpeg -f openal -i 'DR-BT101 via PulseAudio' out1.ogg -f openal -i 'ALSA Default' out2.ogg
	
	**注意**不是所有的OpenAL实现设备都支持多路同时捕获。如果上面不工作，则在最新版OpenAL软件上尝试（测试）

### pulse ###
PulseAudio（脉冲音频）输入设备

要使用须编译配置设置`--enable-libpulse`

需要提供文件名或者"default"来指定输入源设备

通过`pactl list sources` 可以列出所有PulseAudio设备以及属性。

更多信息参考[http://www.pulseaudio.org](http://www.pulseaudio.org)

#### pulse选项 ####
- server

    连接到指定PulseAudio服务器，指定是用IP地址。如果没有设置就用默认服务器
- name

    指定用作显示活动客户端的程序名，默认为`LIBAVFORMAT_IDENT`
- stream_name

    指定流名称Specify the stream name PulseAudio will use when showing active streams, 默认为"record"
- sample_rate

    Specify the samplerate in Hz, by default 48kHz is used.
- channels

    Specify the channels in use, by default 2 (stereo) is set.
- frame_size

    Specify the number of bytes per frame, by default it is set to 1024.
- fragment_size

    Specify the minimal buffering fragment in PulseAudio, it will affect the audio latency. By default it is unset. 

#### pulse例子 ####
从默认设备捕获来记录：

    ffmpeg -f pulse -i default /tmp/pulse.wav

### qtkit ###
QTKit输入设备

文件名作为设备名或者索引序号参数被传递。设备索引也可以使用` -video_device_index` 选项来设定。一个获取的设备索引可以覆盖任何获取的设备名。如果所需的设备仅包含数字，则使用`-video_device_index`来识别。如果文件名为空字符串或者设备名为"default"都会选择默认设备。有效设备可以由`-list_devices`枚举。
	
	ffmpeg -f qtkit -i "0" out.mpg
	
	ffmpeg -f qtkit -video_device_index 0 -i "" out.mpg
	
	ffmpeg -f qtkit -i "default" out.mpg
	
	ffmpeg -f qtkit -list_devices true -i ""

### sndio ###
sndio输入设备。

要使用它需要系统安装并配置有`libsndio`库

文件名作为输入设备节点，通常为`/dev/audio0`

例如从`/dev/audio0`捕获音频：

    ffmpeg -f sndio -i /dev/audio0 /tmp/oss.wav

### video4linux2 ,v4l2 ###
Video4Linux2 输入视频设备

"v4l2"是"video4linux2"的别名

编译需要`v4l-utiles`支持（`--enable-libv4l2`编译选项被配置），也可用于`-use_libv4l2`输入设备选项。

捕获的设备名是一个文件设备节点，通常Linux系统在设备（例如USB摄像头）插入到系统时自动创建这样的节点，会被命名为`/dev/videoN`，`N`是设备索引序号

Video4Linux设备通常只支持有限的分辨率（`width x height`）和帧率,通过`-list_formats all`选项来获取支持情况。一些设备，例如电视卡可以支持1个或者多个标准，它支持的标准可以通过`-list_standards all`来了解。

时间戳时基单位为1microsecond。根据内核版本和配置，时间戳可以基于实时间（real time clock——绝对时间，一种起源于Unix的表示方式）或者单调时钟（monotonic clock——通常源自启动时间，不受NTP或者手动改变）。`-timestamps abs`或者`-ts abs`选择启用实时间。

在ffmpeg或ffplay使用的例子如下：
- 列出支持的设备（video4linux2）

    ffplay -f video4linux2 -list_formats all /dev/video0
- 捕获并显示（对video4linux2设备）

    ffplay -f video4linux2 -framerate 30 -video_size hd720 /dev/video0
- 捕获并记录输入的Grab and record the input of a video4linux2 device, leave the frame rate and size as previously set:

    ffmpeg -f video4linux2 -input_format mjpeg -i /dev/video0 out.mpeg

更多关于video4linux的信息参考[http://linuxtv.org/](http://linuxtv.org/)

#### video4linux,vl4的选项 ####
- standard

    设置采用的标准，必须是被支持的标准。为了获取当前支持的标准，需要使用 `list_standards` 选项
- channel

    设置采用的输入通道索引，默认值为-1，表示采用前面选择的通道。
- video_size

    设置视频帧尺寸`frame size`. 参数是格式为`WIDTHxHEIGHT`的字符串或者有效的索引
- pixel_format

    选择像素格式（仅对raw视频输入有效）
- input_format

    设置欲采用的像素格式(仅对raw视频格式)或者编码名。这个选项允许选择一个输入格式（当有多个有效值时）
- framerate

    设置首选帧率
- list_formats

    列出有效的格式 (支持像素格式、编码和帧尺寸)然后退出

    有效值:

    ‘all’

        显示有效可能 (压缩和未压缩的) 格式
    ‘raw’

        仅显示raw video (非压缩)格式
    ‘compressed’

        仅显示压缩格式 

- list_standards

    列出支持的标准然后退出

    有效值:

    ‘all’

        显示所有支持的标准 

- timestamps, ts

    设置捕获帧的时间戳标准

    有效值:

    ‘default’

        根据核心的默认值
    ‘abs’

        使用绝对时间戳(时间时钟).
    ‘mono2abs’

        强制从单调时间转换为绝对时间戳 

    默认值是 default


### vfwcap ###
vfw（Video for Windows）捕获输入端

文件名必须是捕获设备索引，范围0-9可以用`list`作为文件名，将输出一个设备列表。任何数字外其它文件名被视作设备索引0。

### x11grab ###
X11 视频输入设备

使用需要`libxcb`库，它会在编译时自动检测。

另外，配置`--enable-x11grab`以对应遗留的Xlib用户。

这个设备允许捕获X11显示区域。

作为输入的文件名语法为：

    [hostname]:display_number.screen_number[+x_offset,y_offset]
`hostname:display_number.screen_number`指定了要捕获的X11显示屏幕名，`hostname`可以省略则默认为"localhost"。环境变量`DISPLAY`可以指定默认显示名。`x_offset`,`y_offset`指定捕获偏移，是对于左上建立的X11屏幕，默认为0.

通过X11文档（`man X`）来了解更详细信息。

使用`xdpyinfo`程序来获得关于你X11显示的基本属性信息（配合 `grep` "name" 或者 "dimensions"）

例如使用ffmpeg捕获 `:0.0` ：

    ffmpeg -f x11grab -framerate 25 -video_size cif -i :0.0 out.mpg
捕获坐标 10，20

    ffmpeg -f x11grab -framerate 25 -video_size cif -i :0.0+10,20 out.mpg

#### X11grab选项 ####
- draw_mouse

    指定是否捕获鼠标，0表示不，1为默认表示要
- follow_mouse

    随鼠标定义捕获区域。参数可以是`centered`或者像素值`PIXELS`

    当设定为"centered", 捕获区域跟随鼠标指针保持指针所指在区域中，否则捕获区仅当鼠标指向达到距边缘 PIXELS (大于0)像素值离区域内边缘时

    例如:

    ffmpeg -f x11grab -follow_mouse centered -framerate 25 -video_size cif -i :0.0 out.mpg

    遵循只有当鼠标指针达到100像素内边缘:

    ffmpeg -f x11grab -follow_mouse 100 -framerate 25 -video_size cif -i :0.0 out.mpg

- framerate

    设置捕获帧率。默认ntsc,为30000/1001.
- show_region

    显示捕获区域

    如果`show_region`设置为1，则区域将显示在屏幕上，通过这个选项可以很容易的判断哪些内容将被捕获
- region_border

    设置`-show_region`设置为1时使用的区域边框线粗细 ，值范围1 -128,默认为3 (仅XCB-based x11grab ).

    例如:

    ffmpeg -f x11grab -show_region 1 -framerate 25 -video_size cif -i :0.0+10,20 out.mpg

    设置了`follow_mouse`:

    ffmpeg -f x11grab -follow_mouse centered -show_region 1 -framerate 25 -video_size cif -i :0.0 out.mpg

- video_size

    设置帧尺寸，默认vga.
- use_shm

    对共享内存使用MIT-SHM扩展，默认为1，可能需要禁用远程显示器(仅legacy x11grab). 

#### x11grab 的grab_x,grab_y AV选项 ####
语法：

    -grab_x x_offset -grab_y y_offset
设置区域坐标。它们表示抵消X11左上角。默认值为0。



