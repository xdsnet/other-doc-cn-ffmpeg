## 27 输出设备 ##
输出设备是可配置用于ffmpeg写入多媒体数据的元素，其附加到系统的输出设备。

在编译配置ffmepg时，所有支持的输出设备都被默认允许。你可以使用配置选项`–list-outdevs`了解有哪些设备。

你可以通过`–disable-outdevs`禁止编译所有输出设备，然后再通过`–enable-outdev=OUTDEV`以支持个别的设备，也可以通过默认配置，再添加`–disable-outdev=OUTDEV`来禁用个别设备。

在ff*工具集中，`-devices`可以显示当前允许的输出设备。

当前有效的输出设备介绍见下。

### alsa ###
ALSA(Advanced Linux Sound Architecture) 音频输出设备
#### alsa例子 ####
- 在默认ALSA设备播放:

    ffmpeg -i INPUT -f alsa default
- 在声卡1的7音频设备播放:

    ffmpeg -i INPUT -f alsa hw:1,7
### caca ###
CACA输出设备

这个输出设备允许在CACA窗口显示视频流。每个程序仅有一个CACA窗口。所以在一个实例中你只能有一个CACA输出。

要允许这个输出设备，需要编译时配置`--enable-libcaca`，`libcaca`是一个输出文本而不是像素的图形库。

关于libcaca的更多信息参考[http://caca.zoy.org/wiki/libcaca](http://caca.zoy.org/wiki/libcaca)

#### caca选项 ####
- window_title

    设置CACA窗口标题,如果没有设置，文件名作为默认值指定到输出设备
- window_size

    设置CACA窗口尺寸, 参数可以是`widthxheight`或者一个视频大小缩写。如果没有指定，则以输入视频为默认
- driver

    设置显示驱动
- algorithm

    设置抖动算法。有时抖动是必要的，其可以让感觉到的颜色大于调色板。其接受`-list_dither`列出的算法
- antialias

    设置抗锯齿算法。抗锯齿可柔滑渲染后的图像,并避免常见的楼梯效果。其接受`-list_dither`列出的抗锯齿算法
- charset

    设置哪些字符将显示文本时使用。它接受`-list_dither`列出的字符集
- color

    设置将用于渲染的颜色，接受`-list_dither`列出的颜色
- list_drivers

    如果设置为真，输出可用的设备并退出
- list_dither

    抖动可用选项列表相关的参数。参数为`algorithms`, `antialiases`, `charsets`, `colors`

#### caca例子 ####
- 下面的命令将强制一个80x25的CACA窗口输出:

    ffmpeg -i INPUT -vcodec rawvideo -pix_fmt rgb24 -window_size 80x25 -f caca -
- 列出有效设备并退出:

    ffmpeg -i INPUT -pix_fmt rgb24 -f caca -list_drivers true -
- 列出有效抖动颜色并退出:

    ffmpeg -i INPUT -pix_fmt rgb24 -f caca -list_dither colors -

### decklink ###
在Blackmagic DeckLink设备输出回放。

编译需要Blackmagic DeckLink SDK，以及配置`--extra-cflags`和`--extra-ldflags`以允许。在Windows下，你需要通过`widl`运行`IDL`文件。

DEckLink非常挑剔所支持的格式。像素格式必须是uyvy422，帧率和视频分辨率必须是设备`-list_formats 1`列出的值，音频采样频率必须是48kHz。

#### decklink选项 ####
- list_devices

    如果设置为true，将列出设备并退出。默认为false.
- list_formats

    如果设置为true,将列出支持的格式并退出，默认为false.
- preroll

    以秒为单位设置预卷视频的时间，默认为0.5
#### decklink例子 ####
- 列出输出设备:

    ffmpeg -i test.avi -f decklink -list_devices 1 dummy
- 列出支持的格式:

    ffmpeg -i test.avi -f decklink -list_formats 1 'DeckLink Mini Monitor'
- 播放视频片段:

    ffmpeg -i test.avi -f decklink -pix_fmt uyvy422 'DeckLink Mini Monitor'
- 按非标准的帧率和视频大小播放视频:

    ffmpeg -i test.avi -f decklink -pix_fmt uyvy422 -s 720x486 -r 24000/1001 'DeckLink Mini Monitor'

### fbdev ###
Linux framebuffer（linux帧缓冲）输出设备

Linux framebuffer是独立于硬件的计算机屏幕显示图形的抽象层，通常用于控制台，它通过文件设备点访问，通常为`/dev/fb0`

更多信息阅读Linux源码树中的 Documentation/fb/framebuffer.txt文档。
#### fbdev选项 ####
- xoffset
- yoffset

    设置左上角的x/y坐标，默认为0 

#### fbdev例子 ####
在设备/dev/fb0播放文件，需要的像素格式依赖于当前framebuffer设置

    ffmpeg -re -i INPUT -vcodec rawvideo -pix_fmt bgra -f fbdev /dev/fb0
更多请参考[http://linux-fbdev.sourceforge.net/](http://linux-fbdev.sourceforge.net/)的`fbset(1)`

### opengl ###
OpenGL输出设备

编译允许配置选项` --enable-opengl`

这个输出设备允许渲染输出OpenGL内容。内容可以是由程序提供或者默认创建的的SDL窗口。

当设备呈现到外部环境时，程序必须实现处理如下的消息：
- AV_DEV_TO_APP_CREATE_WINDOW_BUFFER - 在当前线程创建OpenGL 环境
- AV_DEV_TO_APP_PREPARE_WINDOW_BUFFER - OpenGL当前上下文（环境） 
- AV_DEV_TO_APP_DISPLAY_WINDOW_BUFFER - 交换缓冲区
- AV_DEV_TO_APP_DESTROY_WINDOW_BUFFER - 分解/摧毁OpenGL环境
- AV_APP_TO_DEV_WINDOW_SIZE - 告知相关设备（更新信息，程序向设备的） 

#### opengl选项 ####

- background

    设置背景颜色，默认为`Black` (黑色) 
- no_window

    非0表示禁止默认的SDL窗口。程序必须提供OpenGL环境（上下文）同时设置 `window_size_cb`与`window_swap_buffers_cb`两个回调
- window_title

    设置SDL窗口标题,如果没有指定将以指代输出设备的文件名作为默认。当`no_window`设置时会被忽略。 
- window_size

    设置首选窗口尺寸，可以是形如`widthxheight`的字符串参数或者视频尺寸短语。如果不指定则默认以输入视频尺寸进行等比例缩放（让高或者宽恰好等于窗口最大可能且完全展示的尺寸）。如果`no_window`没有设置可用

#### opengl例子 ####
使用OpenGL渲染播放到SDL窗口

    ffmpeg  -i INPUT -f opengl "window title"
### oss ###
OSS（open Sound System）输出设备

### pulse ###
PulseAudio输出设备

编译选项开关`--enable-libpulse`

更多关于PulseAudio信息参考[http://www.pulseaudio.org](http://www.pulseaudio.org)

#### pulse选项 ####
- server

    以IP地址描述的用于连接到的PulseAudio服务器，如果不提供则为默认服务器（其他地方进行配置）
- name

    将显示在活动客户端的应用名，默认为`LIBAVFORMAT_IDENT`
- stream_name

    显示为活动流的流名称，默认为指定的输出名
- device

    指定设备。如果不指定则采用默认设备。可以用命令`pactl`列出输出设备
- buffer_size
- buffer_duration

    控制缓冲尺寸和持续时间。一个小的缓冲区提供了更多的控制，当需要更频繁的更新。

    buffer_size是按字节指定  
    buffer_duration以milliseconds为单位指定

    当两个都被指定时，使用更大的值(持续时间将按使用流的参数重新计算——即按流参数可以把`buffer_size`转换成`buffer_duration`来比较). 如果设置为0 (默认值), 将采用默认的PulseAudio持续时间，即2秒.
- prebuf

    指定pre-buffering尺寸，单位字节。它指定服务器开始播放前至少缓冲的量。默认会同于`buffer_size` 或`buffer_duration`（中大的一个）
- minreq

    指定最小请求尺寸，单位字节。即如果数据量达到这里指定的值，就可向服务器发送请求，而不是达到缓冲区填满。它不被建议设置，由服务器来初始化这个值更明智
#### pulse例子 ####
在默认服务器的默认设备上播放文件

    ffmpeg  -i INPUT -f pulse "stream name"

### sdl ###
SDL（Simple DirectMedia Layer）输出设备

其可以允许在SDL窗口上显示视频流。每个进程仅能创建一个SDL窗口所以你的程序实例只有一个SDL设备输出。

编译需要`libsdl`库。

关于SDL的更多信息参考`http://www.libsdl.org/`

#### sdl选项 ####
- window_title

    设置SDL窗口标题，如果没有指定，则用输出文件名
- icon_title

    置图标化SDL窗口的名称，如果没有指定则采用和`window_title`
- window_size

    设置SDL窗口尺寸，可以是`widthxheight`格式，也可以是视频尺寸短语。如果没有指定则以输入文件的等比例填充放大最大可能值（某边和屏幕窗口边重合）
- window_fullscreen

    非0则设置全屏模式，默认为0

#### sdl交换命令 ####
- 窗口创建的设备可以通过下面的交互式控制命令；

    Quit the device immediately.

#### sdl例子 ####
下面强制以`qcif`尺寸标准中SDL窗口上显示图像

### andio ###
sndio 音频输出设备
### xv ### 
XV（XVideo）输出设备

这个X环境设备允许在Xwindow系统的一个窗口上显示视频流

#### xv选项 ####
- display_name

    指定用在显示的硬件名，它决定了显示和通信

    显示名或者`DISPLAY`环境变量值是一个格式字符串`hostname[:number[.screen_number]]`

    `hostname`是指定了主机的物理连接，`number`指明了在主机上显示服务索引号，`screen_number`指定了服务上的那个屏幕

    如果不指定，则采用`DISPLAY`环境变量值

    例如：`dual-headed:0.1`指定了是`dual-headed`主机上的0号显示服务的1号屏幕

    通过X11介绍了解更多关于显示名的格式信息
- window_id

    为非0值表示不创建新窗口而是使用已有的`window_id`窗口（如果该`window_id`窗口已经存在）。默认为0表示创建自己的窗口。
- window_size

    设置窗口尺寸，参数可以是`widthxheight`或者视频尺寸短语。如果不指定，则默认以输入视频尺寸，当`window_id`被设置时忽略
- window_x
- window_y

    设置创建窗口的坐标偏移。默认都为0.它可能被窗口管理器忽略。当`window_id`被设置后被忽略。
- window_title

    设置窗口标题，如果不设置默认以输出文件名作为值，当`window_id`被设置后被忽略

#### xv例子 ####
- 同时解码、显示和编码输入

    ffmpeg -i INPUT OUTPUT -f xv display
- 解码显示输入视频到多个X11窗口:

    ffmpeg -i INPUT -f xv normal -vf negate -f xv negated













 


