## 24 协议 ##
FFmpeg协议配置元素,用于访问资源时要求特定的协议。

默认编译时会自动支持所有可用协议。你可以在编译脚本中添加 "–list-protocols"选项来了解有哪些协议被支持。

你也可以在编译时通过 "–disable-protocols"禁止所有的协议支持，然后通过 "–enable-protocol=PROTOCOL"来启用个别协议，或者在默认基础上通过 "–disable-protocol=PROTOCOL"关闭个别协议支持。

在ff*工具集中，"-protocols"选项可以了解编译支持了的协议

当前有效协议介绍如下。

### bluray ###
读取BluRay（蓝光）播放列表，其接受如下选项：

- angle

	BluRay 角度/方向
- chapter

	开始章(1...N)
- playlist

	在(BDMV/PLAYLIST/?????.mpls) 读取播放列表 

例如：

从加载映射目录/mnt/bluray 读取很长的蓝光播放列表

	bluray:/mnt/bluray

从加载映射目录/mnt/bluray 读取蓝光播放列表，从章2（`chapter 2`）开始，读取`playlist 4`的`angle 2`

	-playlist 4 -angle 2 -chapter 2 bluray:/mnt/bluray

### cache ###
对输入流的缓冲封装

缓存输入流到临时文件，以提供对在线流的搜寻/定位支持。

	cache:URL

### concat ###
物理连接协议

从许多资源（资源序列，把它们当作一种独特的资源）中读取和搜索定位

这个协议按如下语法接受URL：

	concat:URL1|URL2|...|URLN

这里的URL1、URL2...URLN都是需要连接起来的资源url，每个可能是一个不同的协议

例如，需要利用`ffplay`播放一个序列的资源(包含split1.mpeg,split2.mpeg,split3.mpeg):

	ffplay concat:split1.mpeg\|split2.mpeg\|split3.mpeg

**注意**在大多数`shell`中可能你需要对"|"进行转义

### crypto ###
AES加密(AES-encrypted)流读取协议，其接受如下选项：

- key：设置AES解密密钥，是用16进制表示的二进制数据块
- iv:设置AES解密初始化向量二进制块，也是用16进制表示。

允许如下的URL格式：

- crypto:URL
- crypto+URL

### data ###
在URI中的行内数据，参考[http://en.wikipedia.org/wiki/Data_URI_scheme](http://en.wikipedia.org/wiki/Data_URI_scheme)

例如可以利用ffmpeg 转换行内的GIF文件数据
> 
   ffmpeg -i "data:image/gif;base64,R0lGODdhCAAIAMIEAAAAAAAA//8AAP//AP///////////////ywAAAAACAAIAAADF0gEDLojDgdGiJdJqUX02iB4E8Q9jUMkADs=" smiley.png

### file ###
文件访问协议，实现对文件的读写。

文件协议的URL格式为：

    file:filename
这里的`filename`是文件的路径

如果在URL中没有前导协议头则会作为一个文件URL。根据编译的版本，URL可以是Windows下的包含盘符的文件URL（通常在类Unix系统上不支持）

例如利用`ffmpeg`读取`input.mpeg`文件：

    ffmpeg -i file:input.mpeg output.mpeg

这个协议支持下面的选项：
- truncate

    如果设置为1，则如果文件存在，则截断文件的写入（进行覆盖），如果设置为0则文件存在时不会被删除（覆盖掉），默认为1
- blocksize

    设置可选的I/O最大块尺寸，单位bytes。默认值是`INT_MAX`，它让请求的块没有尺寸限制。合理设置这个值可以提高用户请求反应时间，这对低速媒体上的文件读取有益。

### ftp ###
FTP（File Transfer Protocol——文件传输）协议

通过FTP协议进行远程读写

请求语法是：

    ftp://[user[:password]@]server[:port]/path/to/remote/resource.mpeg
这个协议接受如下选项：
- timeout

	以毫秒（microseconds）为单位，设置socket I/O操作超时时间。默认值为`-1`表示没有指定
- ftp-anonymous-password

    当作为`anonymous`用户登录时的密码，通常为一个`e-mail`地址（`anonymous@FTP-address`）
- ftp-write-seekable

    控制持续编码时连接的可搜索性。如果设置为1表示可以搜索，如果设置为0则不可搜索，默认为0

**注意**协议可以用于输出，但通常不建议这样使用，除非特殊任务要求（测试时、定制的服务器配置等等）。不同的FTP服务提供不同的持续定位处理。ff*工具对它们的支持是不完整的。

### hls ###
把Apple的HTTP直播分段流作为一个统一的进行读取。描述分段的`M3U8`播放列表文件可以是远程HTTP资源或者本地文件（通过标准文件协议）。通过在`hls`后附加`+proto`的方式嵌套指定`hls`的URI，这里`proto`可以是`file`或者`http`

    hls+http://host/path/to/remote/resource.m3u8
	hls+file://path/to/local/resource.m3u8

使用这个协议将自动采用`hls`分离器（如果不是请报告问）。为了使用`hls`分离器，可以把URL简单的指向`m3u8`文件。

### http ###
HTTP (Hyper Text Transfer Protocol)协议

协议支持如下选项：

- seekable

    控制连接是否可搜索，如果设置为1表示资源支持搜索，否则为0.如果设置为-1则尝试自动检测是否支持搜索。默认为-1
- chunked_post

    如果设置为1则启用分块传输编码，默认为1
- content_type

    设置一个特定内容类型的消息。
- headers

    设置定制的`HTTp`头，它可以覆盖掉默认头。这些值必须是可在头中编码的字符串。
- multiple_requests

    为1表示使用持久连接，默认为0
- post_data

    设置定制的`HTTP`数据
- user-agent
- user_agent

    覆盖掉用户代理（客户端类型）头。如果没有指定，则采用在`libavformat`编译中的字符串 ("Lavf/<version>")
- timeout

    以毫秒（microseconds）为单位设置I/O操作超时时间。默认为-1，表示没有指定。
- mime_type

    导出MIME类型
- icy

    如果设置为1将从服务器请求ICY (SHOUTcast)元数据。如果服务器支持，将通过`icy_metadata_headers`和 `icy_metadata_packet`传递具体的请求，默认值为1。
- icy_metadata_headers

    如果服务器支持ICY元数据，它将包含由换行符分隔（即每行1个）的ICY-specific HTTP 应答头
- icy_metadata_packet

    如果服务器支持ICY元数据，且`icy`被设置为1，这将包含服务器发送的最后非空元数据包, 它用于定期处理程序执行期间中间流元数据更新
- cookies

    设置未来请求的cookie。 格式为`Set-Cookie HTTP`响应一样，多个cookie值可以由换行符隔开。
- offset

    设置初始字节偏移量
- end_offset

    尝试偏移限定，单位字节
- method

    当用在客户端选项时，它设置请求的HTTP方法。

	当用在服务器端选项时，它设定（预计）从客户机获要用的方法。如果预期和收到的HTTP方法都不能与客户端匹配将是一个很糟糕的问题。当前对是否设置本方式不检测，以后会自动检测。
- listen

    如果设置为1，将使用实验性质的HTTP服务。它可以用于在输出中指定数据（例如）——对输出文件，或者通过`HTTP POST`读取输入端数据
```    
    # 服务器段（发送）:
    ffmpeg -i somefile.ogg -c copy -listen 1 -f ogg http://server:port

    # 客户端 (接收):
    ffmpeg -i http://server:port -c copy somefile.ogg

    # 客户端可以像下面这样工作
    wget http://server:port -O somefile.ogg

    # 服务器端接收

    # 客户端 (发送):
    ffmpeg -i somefile.ogg -chunked_post 0 -c copy -f ogg http://server:port

    # 客户端也可以联用在`wget`中
    wget --post-file=somefile.ogg http://server:port
```
### HTTP Cookie ###
一些HTTP请求将被拒绝,除非cookie值按要求进行传递。`cookies`选项允许对cookie进行指定。至少cookie需要指定一个路径和域。HTTP请求将自动匹配域和路径，并把包含的cookie值放置HTTP Cookie头中。多个cookie可以由换行分隔。

下面的请求语法就是播放一个流时指定了cookie：

    ffplay -cookies "nlqptid=nltid=tsn; path=/; domain=somedomain.com;" http://somedomain.com/somestream.m3u8

### Icecast ###
Icecast (stream to Icecast servers) 协议

协议支持如下的选项：

- ice_genre

    设置流样式
- ice_name

    设置流名字
- ice_description

    设置流说明
- ice_url

    设置流web URL
- ice_public

    为1表示流可以被公开，否则为0表示不能公开，默认为0（不公开）
- user_agent

    覆盖用户代理头。如果没有指定则 "Lavf/<version>" 被使用
- password

    设置Icecast挂载password.
- content_type

    设置内容类型。它对audio/mpeg 必须设置为不同
- legacy_icecast

    这将使得支持 Icecast的版本< 2.4.0,不过源方法将不再支持HTTP PUT方法。

协议URL指定方法：

	icecast://[username[:password]@]server:port/mountpoint

### mmst ###
基于TCP的MMS (Microsoft Media Server)协议.

### mmsh ###

基于HTTP的MMS (Microsoft Media Server)协议

请求语法:

	mmsh://server[:port][/app][/playpath]

### md5 ###
MD5输出协议

计算要写数据的MD5,它可以写入文件或者标准输出设备上（如果没有指定输出文件），可以用来测试混合器的输出而无需真实写入文件。

下面是一些例子：

- 把编码成avi文件对应的md5写入output.avi.md5

	ffmpeg -i input.flv -f avi -y md5:output.avi.md5
- 把编码成avi文件对应的md5输出到标准输出设备上

    ffmpeg -i input.flv -f avi -y md5:

**注意**一些格式（例如MOV）要求输出协议是可以搜索/检索（seekable）的，这时MD5输出协议将不被支持（对于这样的MD5计算要求，只能是先生成文件，再利用其它工具对文件计算MD5）。

### pipe ###
UNIX先入先出访问协议（管道）

通过UNIX pipe读写

语法为：

    pipe:[number]

这里`number`是对应管道文件描述符（例如0对应stdin,1对应stdout,2对应stderr）。如果`number`没有指定，默认为`stdout`被用于输出写，`stdin`被用于输入读

例如 ffmpeg从stdin读取

    cat test.wav | ffmpeg -i pipe: 

又如写入标准输出：

    ffmpeg -i test.wav -f avi pipe:1 | cat > test.avi
它等效于

	ffmpeg -i test.wav -f avi pipe: | cat > test.avi

协议支持如下选项：
- blocksize

	设置I/O操作最大块尺寸，单位byte（字节）。默认是`INT_MAX`，表示没有限制。如果设置了这个值将改善对数据传输缓慢的情况下用户终止请求反应的时间。

**注意**一些格式（例如MOV）要求输出协议是可以搜索/检索（seekable）的，这时pipe输出协议将不被支持

### rtmp ###
rtmp（Real-Time Messaging Protocol.）

RTMP被用通过TCP/IP网络流式处理多媒体内容

请求语法：

    rtmp://[username:password@]server[:port][/app][/instance][/playpath]

其中各个参数为：
- username

    可选的用户名，一般在公开发布时
- passwrod

    可选的密码，一般在公开发布时
- server

	RTMP服务器地址
- port

    TCP端口号，默认1935
- app

    它是根据应用程序的名称来访问，它通常对应于应用程序在RTMP服务器上的安装路径（例如`/ondemand/`, `/flash/live/`等等），你可以通过`rtmp_app`选项值来覆盖URI
- playpath

    是通过`app`指定的要播放资源的路径或者名称，可能有`mp4:`前缀，你可以通过`rtmp_playpath`选项值来覆盖
- listen

    作为服务器，侦听传入的连接
- timeout

    设置传入连接最大等待时间，其间一直侦听

额外的，下面的参数可以通过命令行（或者在代码中通过`AVOptionS`）设置


- rtmp_app

    连接到RTMP服务的程序名。它可以用于覆盖URI中对应部分
- rtmp_buffer

    设置客户端缓冲时间，单位milliseconds,默认3000.
- rtmp_conn

    额外的AMF连接参数，参数值是字符串，从中解析出AMF信息。例如： B:1 S:authMe O:1 NN:code:1.23 NS:flag:ok O:0 .每个值有一个单字符前导（B表示布尔值，N表示数字值，S表示字符串，O表示对象，Z表示null），后面加冒号":"，对于布尔值，范围为0或者1分别对应`FALSE`和`TRUE`。同样对于对象值，必须由0或者1开始或者结束. 数据子项可以被命名，如果数据前面指定了N并且在值前面命名了数据（例如NB:myFlag:1），这个选项可以被多次在AMF序列中使用。
- rtmp_flashver

    SWF播放器插件版本，默认 LNX 9,0,124,2. (当发布，默认为FMLE/3.0 (兼容与 <libavformat version>).)
- rtmp_flush_interval

    在同一请求的数据包刷新数量(只RTMPT)。默认值是10。
- rtmp_live

    指示是一个直播流。直播流不能重放和搜索。默认值为`any`,它意味着首先尝试按直播流去搜索`playpath`，如果没有找到再以记录流去搜索。其他参数值为`live`和`recorded`
- rtmp_pageurl

    嵌入在URL中的媒体页面。默认没有值被发送
- rtmp_playpath

    要播放/发布的流的ID。这个值将覆盖URI中的对应部分
- rtmp_subscribe

    直接订阅的直播流名字，默认没有值被发送。这个值仅在被指定，而且`rtmp_live`被设置为`live`时发送
- rtmp_swfhash

     解压SWF文件的SHA256(32 bytes).
- rtmp_swfsize

    解压SWF文件的尺寸，被`SWFVerification`要求
- rtmp_swfurl

    对应媒体SWF播放器的URL。默认没有值被发送
- rtmp_swfverify

    播放对应swf文件URL，会自动计算hash/size
- rtmp_tcurl

    目标流URL，默认为`proto://host[:port]/app`
例如利用ffplay播放`myserver`服务器上`vod`应用下的`sample`资源：

    ffplay rtmp://myserver/vod/sample
发布一个资源到密码保护的服务器上

    ffmpeg -re -i <input> -f flv -rtmp_playpath some/long/path -rtmp_app long/app/name rtmp://username:password@myserver/
**注**这里最终发布的URL为`rtmp://username:password@myserver/long/app/name/some/long/path`

### rtmpe ###
加密的Real-Time Messaging Protocol. 

通过提供的标准加密原语加密流式多媒体内容的RTMPE。由Diffie-Hellman 和 HMACSHA256作为密钥交换，产生一对RC4密钥

### rtmps ###
基于安全SSL的rtmp

通过SSL通道传输的rtmp

### rtmpt ###
基于HTTP的Rtmp

由HTTP通道承载的RTMP，这样可以突破很多防火墙限制。

### rtmpte ###
加密RTMP，且通过http传输

在HTTP通道承载下，进行加密rtmp的传输。

### rtmpts ###
通过HTTPS传输的rtmp。

通过HTTPS进行传输的rtmp，它可以突破很多防火墙并提供安全的连接

### libsmbclient ###
libsmbclient提供对CIFS/SMB网络资源访问支持

支持一下语法：

    smb://[[domain:]user[:password@]]server[/share[/path[/file]]]
支持下面选项：

- timeout

    以miliseconds为单位设置I/O操作时限。默认设置为-1，表示没有限制
- truncate

    如果设置为1，表示文件存在则覆盖，否则为0，默认为1
- workgroup

    设置工作组属性覆盖参数，默认未设置

更多信息参考[http://www.samba.org/](http://www.samba.org/)

### libssh ###
通过libssh提供安全文件传输协议

可以通过SFTP协议读写远程资源

请求按下面语法：

    sftp://[user[:password]@]server[:port]/path/to/remote/resource.mpeg

支持下面选项：

- timeout

    以miliseconds为单位设置I/O操作时限。默认设置为-1，表示没有限制
- truncate

    如果设置为1，表示文件存在则覆盖，否则为0，默认为1
- private_key

    指定一个文件路径用于持续期认证，默认在`~/.ssh/`搜索密钥。

例如：播放一个远程服务器上资源

    ffplay sftp://user:password@server_address:22/home/user/resource.mpeg
### librtmp rtmp, rtmpe, rtmps, rtmpt, rtmpte ###
通过librtmp提供的rtmp（以及rtmpe, rtmps, rtmpt, rtmpte）

在编译时需要明确指定相应的头和库，并且在配置中通过"–enable-librtmp"予以编译允许，它将替代原生的RTMP协议族支持

它提供了关于（rtmp, rtmpe, rtmps, rtmpt, rtmpte）大部分客户端功能和一些服务器功能。

请求语法为：

    rtmp_proto://server[:port][/app][/playpath] options

这里`rtmp_proto`可以是（rtmp, rtmpe, rtmps, rtmpt, rtmpte）中的一个，此外诸如`server`、`port`、`app`、`playpath`都和在原生RTMP中一样。`options`是由空格分隔的`key=val`值对列表

例如，在ffmpeg中流式输出到RTMP协议：

    ffmpeg -re -i myfile -f flv rtmp://myserver/live/mystream
用ffplay播放这个流：

    ffplay "rtmp://myserver/live/mystream live=1"

### rtp ###
Real-time Transport Protocol——RTP，实时传输协议

请求语法：`rtp://hostname[:port][?option=val...]`

- port ：用于指定RTP端口

下面是被支持的URL选项：


ttl=n

    设置TTL(Time-To-Live)值，仅多播
rtcpport=n

    设置远端RTCP端口为n.
localrtpport=n

    设置本地RTP端口为n.
localrtcpport=n'

    设置本地RTCP端口为n.
pkt_size=n

    设置最大包尺寸为n，单位 bytes.
connect=0|1

    为1则在UDP socket上连接（connect()），否则（为0）不连接
sources=ip[,ip]

    列出允许的源IP地址
block=ip[,ip]

    列出禁止的源IP地址
write_to_source=0|1

    如果为1，将数据包发送到最新收到的源地址，否则发送到默认远程地址（为0）
localport=n

    设置本地RTP端口为n

    这时一个废弃选项，用`localrtpport`替代

**重要提示：**

1. 如果`rtcpport`没有被设置，则RTCP端口将是RTP端口加1
2. 如果`localrtpport`（本地RTP端口）没有设置，则任何有效端口可能被用作本地RTP/RTCP端口
3. 如果`localrtcpport`（本地RTCP）没有设置，则默认为本地RTP端口加1.

### rtsp ###
Real-Time Streaming Protocol——RTSP，实时流协议

RTSP在libavformat中不是作为技术协议处理，而是作为分离器和混合器。分离器支持两种常见的RTSP（基于RTP的数据传输，被用在Apple 和Microsoft）和Real-RTSP（基于RDT的数据传输）

混合器可以用来发送流（使用`RTSP ANNOUNCE`）给支持它的服务器（当前有`Darwin`流服务器和`Mischa Spiegelmock`的[`RTSP server`](https://github.com/revmischa/rtsp-server)）

请求的语法为：

    rtsp://hostname[:port]/path
一些选项可以用于ffmpeg/ffplay命令行，或者通过`AVOptionS`进行编码设置，或者在`avformat_open_input`进行编码。

下面的选项被支持：

- initial_pause

    为1表示不立即开始，否则为0为立即开始，默认为0
- rtsp_transport

    设置RTSP传输承载协议，有如下值：

    ‘udp’

        采用UDP作为底层传输协议
    ‘tcp’

        采用TCP作为底层传输协议(RTSP控制通道也交叉混合在一起)
    ‘udp_multicast’

        采用UDP多播为底层传输协议
    ‘http’

        采用HTTP隧道为底层传输协议，它通常用于代理

    多个底层传输协议可以被指定，这时它们按顺序依次被尝试（一个失败就试下一个）。对应混合器，只支持‘tcp’和‘udp’
- rtsp_flags

    设置RTSP标志，下面的值被支持：

    ‘filter_src’

        仅从对等协商的地址和端口接收数据包 
    ‘listen’

        当作服务器，侦听连接 
    ‘prefer_tcp’

        首先尝试采用TCP作为传输，如果TCP可用，就使用RTSP RTP 传输

    默认为 ‘none’.
- allowed_media_types

    设置从服务器端接收数据类型，下面标志被允许：

    ‘video’
    ‘audio’
    ‘data’

    默认接受所有类型
- min_port

    设置最小的UDP端口，默认为 5000.
- max_port

    设置最大的UDP端口，默认为 65000.
- timeout

    设置等待数据连接的最大时间，单位秒

    默认值为-1，表无限。这个选项蕴含着`rtsp_flags`选项被设置为‘listen’.
- reorder_queue_size

    设置在缓冲区处理重排序的包数量
- stimeout

    设置socket TCP I/O超时时间，单位microseconds.
- user-agent

    覆盖用户代理头。如果不指定，将默认为libavformat标识字符串
#### rtsp例子 ####
下面是使用ffplay和ffmpeg工具的例子
- 使用UDP最大延迟0.5秒的播放

    ffplay -max_delay 500000 -rtsp_transport udp rtsp://server/video.mp4
- 观看HTTP隧道流播放

    ffplay -rtsp_transport http rtsp://server/video.mp4
- 发送流到RTSP服务器

    ffmpeg -re -i input -f rtsp -muxdelay 0.1 rtsp://server/live.sdp
- 获取实时流

    ffmpeg -rtsp_flags listen -i rtsp://ownaddress/live.sdp output

### sap ###
Session Announcement Protocol(RFC 2974),会话通告协议。它在libavformat中不作为技术协议处理，而是作为混合器和分离器。它被用于RTP流信号，以宣布SDP流经常在一个单独的端口。

#### sap作为混合器 ####
作为分离器时的SAP URL语法

    sap://destination[:port][?options]
RTP数据包被送往`destination`的`port`端口，如果`port`没有指定则默认为5004.`options`是一个“&”分隔的列表，允许下面的值：

- announce_addr=address

    指定目的地IP地址发送公告地址，如果省略，则采用SAP多播地址`224.2.127.254` (sap.mcast.net),或者`ff0e::2:7ffe`(IPv6 )
- announce_port=port

    指定公告发送端口，默认9875
- ttl=ttl

    指定公告和RTP包的TTL（time to live）值，默认255.
- same_port=0|1

    如果设置为1，则所有的RTP流在同一个端口发送，如果设置为0（默认），则所有流在独立的端口发送，每个流在比前一个流高2号的端口进行发送。 VLC/Live555 要求设置为1，以可以获取流。`libavformat`下的RTP则要求独立端口，即需要设置为0

#### sap作为分离器 ####
作为分离器的语法：

    sap://[address][:port]
这里`address`是侦听的多播地址，如果省略则为`224.2.127.254` (sap.mcast.net)。`port`是侦听端口，默认9875

demuxers在给定的地址和端口监听公告。一旦收到公告,它试图接受特定流

命令行例子如下：

播放常规的SAP多播地址的第一个流：

    ffplay sap://
为了播放IPV6 SAP多播地址的第一个流

    ffplay sap://[ff0e::2:7ffe]

### sctp ###
Stream Control Transmission Protocol——SCTP，流控制传输协议

允许的URL语法：

    sctp://host:port[?options]
协议接收如下选项：
- listen

    可以设置为任何值，用于侦听传入连接。所有外向/传出连接都被作为默认值
- max_streams

    设置最大数量的流，默认没有限制

### srtp ###
Secure Real-time Transport Protocol——SRTP，安全实时传输协议

接收如下选项：

- srtp_in_suite
- srtp_out_suite

    选择输入和输出编码套件

    支持的值:

    ‘AES_CM_128_HMAC_SHA1_80’
    ‘SRTP_AES128_CM_HMAC_SHA1_80’
    ‘AES_CM_128_HMAC_SHA1_32’
    ‘SRTP_AES128_CM_HMAC_SHA1_32’

- srtp_in_params
- srtp_out_params

    设置输入和输出编码参数，采用base64编码表示的二进制块。第一个16字节的块用作主要密钥，随后的14字节作为密钥的“盐” 

### subfile ###
提取一段文件或者流的模拟。必须是可以定位/搜索的底层流。

接收的选项：
- start

	提取段的开始位置偏移，单位byte（字节）
- end

	提取段的结束位置偏移，单位byte（字节）

例子：

- 从DVD的VOB文件中提取章节（节的开始和结束区块数乘以2048）

    subfile,,start,153391104,end,268142592,,:/media/dvd/VIDEO_TS/VTS_08_1.VOB

- 播放直接从TAR打包中播放AVI文件：

    subfile,,start,183241728,end,366490624,,:archive.tar

### tcp ###
Transmission Control Protocol——TCP，传输控制协议

请求的TCP url语法为：

    tcp://hostname:port[?options]
其中`options`是由“&”分隔的`key=val`对选项列表

允许下面一些选项：

- listen=1|0

	是否对传入连接侦听，默认为0，表示不侦听
- timeout=microsenconds

	设置超时错误抛出时间，单位毫秒
	这个选项仅工作在读模式：如果超过设置时间周期没有获取到数据则抛出错误
- listen_timeout=milliseconds

	设置侦听超时，单位毫秒
下面的例子展示了如何在ffmpeg中应用TCP连接，以及如何用ffplay播放一个TCP内容

	ffmpeg -i input -f format tcp://hostname:port?listen
	ffplay tcp://hostname:port	
### tls ###
Transport Layer Security (TLS) / Secure Sockets Layer (SSL) 

对于TLS/SSL的 URL语法是：

    tls://hostname:port[?options]
下面是可以在命令行中设置的选项（或者通过`AVOptionS`在编程时设置）

- ca_file, cafile=filename

    设置含证书颁发机构(CA)作为受信任的根证书的文件。如果连接的TLS库包含一个默认而不需要指定即可工作的值，但不是所有库和配置都有默认值。文件必须是OpenSSL PEM格式
- tls_verify=1|0

    如果允许，则试图对等验证。**注意**如果使用OpenSSL，这是目前唯一支持通过的CA根证书签署数据库对等验证，但它在我们试图连接到一个主机时并不验证根证书匹配（GnuTLS验证了主机名）。
    
    默认禁用，因为大多时候它需要调用者提供一个CA数据库
- cert_file, cert=filename

   设置包含一个用于处理同行连接的证书。(当操作服务器,在侦听模式中,这是经常需要证书的,而客户端证书仅在某些设置中授权。)
- key_file, key=filename

    设置包含证书的私钥文件
- listen=1|0

    如果允许，则在提供的端口上监听连接，并假设当前为服务器角色，而不是客户端角色

命令行中的例子:

- 根据输入流创建一个TLS/SSL服务

	ffmpeg -i input -f format tls://hostname:port?listen&cert=server.crt&key=server.key

- 播放一个TLS/SSL:

	ffplay tls://hostname:port
### UDP ###
User Datagram Protocol——UDP，用户数据报协议

请求UDP URL的语法：

	udp://hostname:port[?options]

这里`options`是由“&”分隔的`key=val`选项列表

在启用线程模式的系统，一个循环缓存被用于存储传入的数据，它可以减少数据由于UDP套接字（socket）缓冲区溢出的损失。`fifo_size` 和 `overrun_nonfatal`选项就是关于这个缓冲区设置的。

下面列出支持的选项:

- buffer_size=size

    设置UDP 最大socket 缓冲区大小，单位bytes,它用于设置接收或者发生的缓冲区大小，其取决于套接字的需求，默认为64KB。也可以参看`fifo_size`
- localport=port

    覆盖本地UDP端口来绑定
- localaddr=addr

    选择本地IP地址，这是对需要设置多播或者主机有多个IP时是必要的。通过设置用户可以选择通过那个IP地址作为发送接口
- pkt_size=size

    以字节为单位设置UDP包大小
- reuse=1|0

    显式设置允许或者不允许UDP套接字(socket)
- ttl=ttl

    设置time to live（TTL）值 (仅对多播).
- connect=1|0

    如果设置为1则利用 connect()初始化UDP套接字。在这种情况下目的地址不能由后面的`ff_udp_set_remote_url`改变，如果目的地址在开始时是未知的，这个选项可以允许指定`ff_udp_set_remote_url`，它允许根据`getsockname`为包找到源地址，如果目的地址不可到达则通过` AVERROR(ECONNREFUSED) `返回。它可以保证只从指定的地址/端口进行数据获取
- sources=address[,address]

    只从多播组的一个指定发送IP地址获取数据包
- block=address[,address]

    忽略多播组中指定IP地址的包
- fifo_size=units

    设置UDP获取循环缓冲区大小，单位为188字节的包个数。如果没有指定，默认7*4096.
- overrun_nonfatal=1|0

    生存的UDP接收循环缓冲区溢出，默认为0.
- timeout=microseconds

    设置抛出超时错误的时限，单位毫秒

    这个选项仅与读模式相关：如果超过设置的时间没有获取到数据则抛出超时错误。
- broadcast=1|0

    显式地允许或不允许UDP广播

    **注意**,在网络广播风暴的保护环境可能无法正常工作
#### UDP例子 ####

- 使用ffmpeg输出流到远程UDP端点

    ffmpeg -i input -f format udp://hostname:port

- 使用ffmpeg，流式输出到UDP端点，UDP包大小是188字节，使用一个大的输入缓冲区:

    ffmpeg -i input -f mpegts udp://hostname:port?pkt_size=188&buffer_size=65535

- 使用FFmepg获取基于UDP传来的远程端点数据:

    ffmpeg -i udp://[multicast-address]:port ...

### UNIX ###
Unix本地socket（套接字）

请求Unix socket的URL语法为：

	unix://filepath

下面的参数允许在命令行设置（通过`AVOptions`在编码时设置）:

- timeout

    以ms设置超时. 
- listen

    以侦听模式建立一个Unix socket

  
