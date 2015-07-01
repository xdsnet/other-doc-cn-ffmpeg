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





 


