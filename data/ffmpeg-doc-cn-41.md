## 41 多媒体源
下面是目前可用的多媒体源的描述
### amovie ###
它同于`movie`源，除了它选择一个默认音频流。

### movie ###
从影片内容中读取音频和/或视频流

接受下面的参数：

- filename

    要被读取的资源名（不限于文件，可以是设备或者一些协议下的流).
- format_name, f

    对要读取的影片指定格式，可以是容器或者输入设备，如果没有指定，将从影片名中猜测。
- seek_point, sp

    指定定位点，单位秒。表示输出的开始点。这个参数与`av_strtod`评估，所以数字可能后缀一个`IS`后缀。默认为0
- streams, s

    指定要读取的流。一些流可以被指定，以`+`分隔。这时按顺序源有多个输出。语法同于ffmpeg手册中的[流说明符](ffmpeg-doc-cn-05.md#流说明（限定）符)章节。两个特殊明智`dv`和`da`指定默认的（最合适）的视频和音频流。在滤镜中调用`amovie`时默认为`dv`或者`da`
- stream_index, si

    指定要读取的视频流索引号。如果值为-1，则最适合的视频流被自动选择。默认为`-1`。现在已弃用。如果滤镜中调用`amovie`将自动选择带音频的视频。
- loop

    指定循环次数，如果超过1，则流会重复读取处理指定次数，默认为1.

	**注意**当影片重新读取时内置的时间戳并不改变，所以会产生非递增的时间戳。 

#### movie源例子 ####
- 在文件in.avi中跳过3.2秒，并覆盖输入标签"in":

    movie=in.avi:seek_point=3.2, scale=180:-1, setpts=PTS-STARTPTS [over];
    [in] setpts=PTS-STARTPTS [main];
    [main][over] overlay=16:16 [out]
- 从`video4linux2`设备读取，并覆盖输入标签"in":

    movie=/dev/video0:f=video4linux2, scale=180:-1, setpts=PTS-STARTPTS [over];
    [in] setpts=PTS-STARTPTS [main];
    [main][over] overlay=16:16 [out]
- 从dvd.vob读取0号流（视频流）和id为`0x81`的音频流，视频流连接到标签`video`，音频连接到标签`audio`:

    movie=dvd.vob:s=v:0+#0x81 [video] [audio]

