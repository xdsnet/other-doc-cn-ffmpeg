## 13 音频解码器
介绍一些有效的音频解码器

### ac3 ###
AC-3 音频解码器，该解码器实现在ATSC A/52:2010 和 ETSI TS 102 366部分，以及RealAudio 3（又名DNET）中。

#### ac3解码器选项 ####
- `-drc_scale value`

    动态范围因子。该因子适合应用于从AC-3流中获取的动态值范围。这个值是指数值。有3个显著效果的典型值(范围)：

    drc_scale == 0

        DRC禁用，会产生原始全范围音频
    0 < drc_scale <= 1

        DRC启用，在一部分DRC中应用，音频表现为全范围与全压缩之间。 
    drc_scale > 1

        DRC启用。适用于drc_scale不对称场景，表现为响亮的全压缩，轻柔的声音增强。 

### flac ###
FLAC音频解码器，它由Xiph实现了对FLAC的完整规格

#### FLAC解码器选项 ####
- `-use_buggy_lpc`
	
	FLAC编码器用于产生高`lpc`值的数据包流（例如在默认设置下），本选项允许正确解码采用老版本`buggy lpc`逻辑编码的FLAC音频。

### ffwavesynth ###
内部波表合成装置。

该解码器根据预定义的序列产生音频波。其使用纯粹内部不公开的数据格式表征特定的序列（集）

### libcelt ###
libcelt解码器再封装

libcelt允许libavcodec解码Xiph CELT超低延迟音频编码。在需要libcelt头和库存在配置（能进一步搜索到库的配置）。在创建ffmpeg工具集时需要显式采用`--enable-libcelt`以打开支持

### libgsm ###
libgsm解码器再封装

libgsm允许libavcodec解码GSM全速率语音。需要libgsm头和库存在配置（能进一步搜索到库的配置）。需要在创建ffmpeg时显式设置`--enable-libgsm`以启用

该解码器支持普通GSM和微软修订

### libilbc ###
libilbc解码器再封装

libilbc允许libavcodec解码网络低码率编码（iLBC）音频。需要libilbc头和库存在配置。需要在创建ffmpeg时显式设置`--enable-libilbc`以启用

#### libilbc选项 ####
下面的选项被libilbc封装支持
- `enhance`
	
	设置为1，则解码时使音频增强，默认是0（禁用）

### libopencore-amrnb ###
libopencore-amrnb解码器再封装

libopencore-amrnb允许libavcodec解码 自适应多速率窄带（Adaptive Multi-Rate Narrowband）音频。需要libopencore-amrnb头和库存在配置才能使用。需要在创建ffmpeg时显式设置`--enable-libopencore-amrnb`以启用

现在FFmpeg已经直接支持解码AMR-NB，所以不需要该库了。

### libopencore-amrwb ###
libopencore-amrwb解码器再封装

libopencore-amrwb允许libavcodec解码 自适应多速率宽带（Adaptive Multi-Rate Wideband）音频。需要libopencore-amrwb头和库存在配置才能使用。需要在创建ffmpeg时显式设置`--enable-libopencore-amrwb`以启用

现在FFmpeg已经直接支持解码AMR-WB，所以不需要该库了。

### libopus ###
libopus解码器再封装

libopus允许libavcodec解码Opus互动音频。需要libopus头和库存在配置才能使用。需要在创建ffmpeg时显式设置`--enable-libopus`以启用。

现在FFmpeg已经直接支持解码Opus，所以不需要该库了。