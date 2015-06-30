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



