## 例子 ##
### 视频和音频抓取 ###
如果你指定了输入格式和设备，ffmpeg可以直接抓取视频和音频：
> 
    ffmpeg -f oss -i /dev/dsp -f video4linux2 -i /dev/video0 /tmp/out.mpg

或者采用ALSA音频源(单声道，卡的id是1)替代OSS:
> 
	ffmpeg -f alsa -ac 1 -i hw:1 -f video4linux2 -i /dev/video0 /tmp/out.mpg

**注意**对于不同的视频采集卡，你必须正确激活视频源和通道，例如Gerd Knorr的`xawtv`。你还需要设置正确的音频记录层次和混合模式。只有这样你才能采集到想要的视音频。

### X11显示的抓取 ###
可以通过ffmpeg直接抓取X11显示内容：
> 
	ffmpeg -f x11grab -video_size cif -framerate 25 -i :0.0+10，20 /tmp/out.mpg
`0.0`是X11服务的显示屏幕号(display.screen)，定义于`DISPLAY`环境变量。10是水平偏移，20是垂直偏移

### 视频和音频文件格式转换 ###
任何支持的文件格式或者协议都可以作为ffmpeg输入。例如：


- 你可以使用YUV文件作为输入
> 	
	ffmpeg -i /tmp/test%d.Y /tmp/out.mpg
	
	这里可能是这样一些文件
> 
	/tmp/test0.Y, /tmp/test0.U, /tmp/test1.V,
	/tmp/test1.Y, /tmp/test1.U, /tmp/test1.V, etc...

	这里Y还有对应分辨率的2个关联文件U和V。这是一种raw数据文件而没有文件头，它可以被所有的视频解码器生成。你必须利用`-s`对它指定一个尺寸而不是让ffmpeg去猜测。

- 你可以把raw YUV420P文件作为输入：
> 
    ffmpeg -i /tmp/test/yuv /tmp/out.avi

	test.yuv 是一个包含raw YUV通道数据的文件。每个帧先是Y数据，然后是U和V数据。

- 也可以输出YUV420P类型的文件
> 
    ffmpeg -i mydivx.avi hugefile.yuv

- 可以设置一些输入文件和输出文件  
> 
	ffmpeg -i /tmp/a.wav -s 640x480 -i /tmp/a.yuv /tmp/a.mpg
	
	这将转换一个音频和raw的YUV视频到一个MPEG文件中

- 你也可以同时对音频或者视频进行转换
> 
	ffmpeg -i /tmp/a.wav -ar 22050 /tmp/a.mp2

	这里把a.wav转换为MPEG音频，同时转换了采样率为22050HZ

- 你也可以利用映射同时编码多个格式作为输入或者输出：
> 
	ffmpeg -i /tmp/a.wav -map 0:a -b:a 64k /tmp/a.mp2 -map 0:a -b:a 128k /tmp/b.mp2

	这将同时把a.wav以64k码率输出到a.mp2，以128k码率输出到b.mp2。 "-map file:index"指定了对于每个输出是连接到那个输入流的。

- 还可以转换解码VOBs：  
> 
    ffmpeg -i snatch_1.vob -f avi -c:v mpeg4 -b:v 800k -g 300 -bf 2 -c:a libmp3lame -b:a 128k snatch.avi

	这是一个典型的DVD抓取例子。这里的输入是一个VOB文件，输出是MPEG-4编码视频以及MP3编码音频的AVI文件。**注意**在这个命令行里使用了B-frames（B帧）是兼容DivX5的，GOP设置为300则意味着有一个内帧是适合29.97fps的输入视频。此外，音频流采用MP3编码需要运行LAME支持，它需要通过在编译是设置`--enable-libmp3lame`。这种转换设置在多语言DVD抓取转换出所需的语言音频时特别有用。

	**注意**要了解支持那些格式，可以采用`ffmpeg -formats`

- 可以从一个视频扩展生成图片（序列），或者从一些图片生成视频：
	- 导出图片
> 
	ffmpeg -i foo.avi -r 1 -s WxH -f image2 foo-%03d.jpeg

	这将每秒依据foo.avi生成一个图片命名为foo-001.jpeg ,foo-002.jpeg以此类推,图片尺寸是WxH定义的值。

	如果你想只生成有限数量的视频帧，你可以进一步结合`-vframes`或者`-t`或者`-ss`选项实现。
	- 从图片生成视频
> 
	ffmpeg -f image2 -framerate 12 -i foo-%03d.jpeg -s WxH foo.avi

	这里的语法`foo-%03d.jpeg`指明使用3位数字来补充完整文件名，不足3位以0补齐。这类似于C语言的printf函数中的格式，但只接受常规整数作为部分。

	当导入一个图片序列时，`-i`也支持shell的通配符模式(内置的)，这需要同时选择image2的特性选项`-pattern_type glob`：例如下面就利用了所有匹配`foo-*.jpeg`的图片序列创建一个视频：
> 
	ffmpeg -f image2 -pattern_type glob -framerate 12 -i 'foo-*.jpeg' -s WxH foo.avi

-  你可以把很多相同类型的流一起放到一个输出中：
> 
	ffmpeg -i test1.avi -i test2.avi -map 1:1 -map 1:0 -map 0:1 -map 0:0 -c copy -y test12.nut

	这里最后输出文件test12.nut包括了4个流，其中流的顺序完全根据前面`-map`的指定顺序。

- 强制为固定码率编码(CBR)输出视频：
> 
	ffmpeg -i myfile.avi -b 4000k -minrate 4000k -maxrate 4000k -bufsize 1835k out.m2v

- 使用`lambda`工具的4个选项`lmin`，`lmax`，`mblmin`以及`mblmax`使你能更简单的从`q`转换到`QP2LAMBDA`:
> 
	ffmpeg -i src.ext -lmax 21*QP2LAMBDA dst.ext
