## 比特流滤镜 ##
默认编译时所有的比特流滤镜都被支持，你可以在配置脚本中以`--list-bsfs`获取有效的滤镜列表

可以利用`--disable-bsfs`禁用所有的比特流滤镜。要指定个别的滤镜可用，则在此基础上`--enable-bsf=BSF`，或者在默认（没有指定`--disable-bsfs`）下禁用个别的滤镜`--disable-bsf=BSF`，这里`BSF`是个别滤镜名称。

在ff*工具集中，`-bsfs`可以列出（编译允许了的）支持的比特流滤镜。

在ff*工具集中，`-bsf`选项可以指定滤镜应用到每个流，具体滤镜由`滤镜名1=选项名1=选项值1/选项名2=选项值2...'给出，例如
    
	ffmpeg -i INPUT -c:v copy -bsf:v filter1[=opt1=str1/opt2=str2][,filter2] OUTPUT

下面介绍了当前有效的一些比特流滤镜

### aac adtstoasc ###
转换MPEG-2/4 的AAC ADTS 到 MPEG-4 音频的专用配置比特流滤镜。

这个滤镜从MPEG-2/4 的ADTS头创建一个移除了ADTS头的MPEG-4音频专用配置流

这是十分必要的，例如把由raw ADTS转换的AAC音频内容复制到FLV/MOV/MP4文件时就需要进行这样的处理

### chomp ###
移除每个包后面附加的0

### dump extra ###
对过滤包的开始添加extradata 

附加参数指定了如何处理，有如下可能值：

- ‘a’

    对每个包添加extradata,除了flags2 codec context field中local_header被设置的
- ‘k’

    每个关键包添加extradata
- ‘e’

    每个包添加extradata

如果没有指定则为`k`

例如下面ffmpeg强制在编码H.264中采用全局头（禁用单独数据包头），即将全局头添加作为extradata添加修正每个数据包

    ffmpeg -i INPUT -map 0 -flags:v +global_header -c:v libx264 -bsf:v dump_extra out.ts

### h264_mp4toannexb ###
转换H.264编码比特流，从长前导模式为开始码前导模式（定义在ITU-T H.264 的附录B）

它是一些流格式要求的，通常如MPEG-2传输流格式("mpegts")

例如采用ffmpeg重新混编一个H.264的MP4文件，把流转换成`mpegts`格式，可以使用：

    ffmpeg -i INPUT.mp4 -codec copy -bsf:v h264_mp4toannexb OUTPUT.ts

### imxdump ###
修正为可编辑的比特流MOV格式，以用于Final Cut Pro解码。这个滤镜仅仅用于mpeg2video编码，对于需适用于Final Cut Pro 7以下版本的 ，同时使用`-tag：v`选项

例如，重新混编 30MB/sec 的NTSC IMX 到MOV:

    ffmpeg -i input.mxf -c copy -bsf:v imxdump -tag:v mx3n output.mov

### mjpeg2jpeg ###
转换MJPEG/AVI1 包为全 JPEG/JFIF包

MJPEG是一种视频编码，它每帧基本都是一个JPEG图像，可以直接无损提取，如：

    ffmpeg -i ../some_mjpeg.avi -c:v copy frames_%d.jpg

不幸的是，这些块是不完整的JPEG图像，因为他们缺乏解码所需的DHT段。
[http://www.digitalpreservation.gov/formats/fdd/fdd000063.shtml](http://www.digitalpreservation.gov/formats/fdd/fdd000063.shtml):
> 
	Avery Lee, writing in the rec.video.desktop newsgroup in 2001, commented that "MJPEG, or at least the MJPEG in AVIs having the MJPG fourcc, is restricted JPEG with a fixed – and *omitted* – Huffman table. The JPEG must be YCbCr colorspace, it must be 4:2:2, and it must use basic Huffman encoding, not arithmetic or progressive. . . . You can indeed extract the MJPEG frames and decode them with a regular JPEG decoder, but you have to prepend the DHT segment to them, or else the decoder won’t have any idea how to decompress the data. The exact table necessary is given in the OpenDML spec." 

这个比特流滤镜可以扩展修复MJPEG流（携带AVI1头ID，但缺失DHT段）为完整的JPEG图像

	ffmpeg -i mjpeg-movie.avi -c:v copy -bsf:v mjpeg2jpeg frame_%d.jpg
	exiftran -i -9 frame*.jpg
	ffmpeg -i frame_%d.jpg -c:v copy rotated.avi

### mjpega dump header ###
### movsub ###
### mp3头解压 ###
### mpeg4 unpack bframes ###
解包DivX包装的B帧

DivX风格的B帧是无效的MPEG-4，且仅用于windows系统的零散解决方案，它使用更多的空间，而能看导致轻微的AV同步问题，需要更多的CPU资源用于解码（除非播放采用一些2,0,2,0帧分组方式），如果直接复制到一个MP4标准封装或MPEG PS/TS流中将引起问题（MPEG-4不能正确解码，因为它不是有效的MPEG-4编码数据）

例如修正一个包含DivX风格B帧数据的AVI文件到MPEG-4流，可以使用:

    ffmpeg -i INPUT.avi -codec copy -bsf:v mpeg4_unpack_bframes OUTPUT.avi

### noise ###
不破坏容器，仅处理内容包。一般用于冻结或测试 错误恢复/隐藏

参数:一个数字字符串，其值通常由输出而被修改，因此值小于等于0被禁止。低于更新频率的字节被修改，1表示每个字节被修改。

    ffmpeg -i INPUT -c copy -bsf noise[=1] output.mkv
表示每个字节都被修改。
### remove extra ###
