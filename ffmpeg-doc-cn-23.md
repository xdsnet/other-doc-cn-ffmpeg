## 23 元数据（metadata） ##
FFmpeg能够提取媒体文件元数据，并转储到一个简单的utf-8编码的类INI文本文件中,然后在分离器/混合器中再次使用

转储的文件格式为：

1. 文件包含一个头，以及一些元数据标签，元数据放置在各自子节的行中
2. 文件头有一个 ‘;FFMETADATA’ 字符串，紧接着版本号（目前为1）
3. 元数据标签以‘key=value’ 形式给出
4. 头紧跟着是全局元数据
5. 在全局元数据后可能有分部的元数据（每个流/每个章）
6. 分节元数据从分节名，由(‘[’, ‘]’)括起的大写字符串（STREAM 或者 CHAPTER），直至下一节或者文件结束
7. 在一章的开始部分可能有一个可选的时基用于开始/结束值(start/end)，其形如`TIMEBASE=num/den`，这里`num`和`den`是整数。如果没有设置，则开始/结束 时间以milliseconds为单位

	下一章（节）的元数据描述包含了开始结束时间的（形如 ‘START=num’, ‘END=num’）则时间值（这里的`num`）必须是正整数
8. 空行（无效）,开始字符是";"或者"#"的行被忽略
9. 如果元数据标签或者值中包含特殊字符(‘=’, ‘;’, ‘#’, ‘\’和 回车/换行)，必须由'\'进行转义
10. **注意**空格在元数据中（例如‘foo = bar’）会被认为是标签的一部分（前面的标签关键字是 ‘foo ’——注意有一个空格的，值是 ‘ bar’——也有一个空格的）

一个ffmetadata文件大致像：

> 
	;FFMETADATA1
	title=bike\\shed
	;this is a comment
	artist=FFmpeg troll team  
	[CHAPTER]
	TIMEBASE=1/1000
	START=0
	#chapter ends at 0:01:00
	END=60000
	title=chapter \#1
	[STREAM]
	title=multi\
	line

通过使用ffmetadata，混合器和分离器可以从输入的ffmetadata文件中导出元数据，也可以编辑ffmetadata文件以转换输出到输出文件中

利用ffmetadata导出元数据：

    ffmpeg -i INPUT -f ffmetadata FFMETADATAFILE
从FFMETADATAFILE 文件中加载元数据信息输出到输出文件中：

    ffmpeg -i INPUT -i FFMETADATAFILE -map_metadata 1 -codec copy OUTPUT