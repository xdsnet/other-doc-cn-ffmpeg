## 语法 ##
这个章节介绍采用ffmpeg库和工具时的一些语法和格式要求。

### 引用与转义(Quoting and escaping) ###
ffmpeg采用如下的引用和转义机制，除非明确规定，以下规则都适用：
- `"'"`和`"\"`分别用于（引用和转义）特殊字符。除了它们可能还有其它特殊字符，但这只在特定的语法中有效。
- 一个特殊字符必须有转义前缀`"\"`
- 所有的引用字符串都由`"'"`封闭包含。引号`"'"`本身不能被引用，所以你可能需要关闭引用或者转义。
- 前导和尾随的空格字符除非专门由引号引用或者转义，否则都会在解析字符串时移除。

**注意**在使用命令行或者脚本时，你可能需要2级转义，这取决于你shell环境支持的语法。

声明在`libavutil/avstring.h`中的函数`av_get_token`被用于任务分析中的引用和转义处理。

ffmpeg源码中的工具`tools/ffescape`被用于自动处理引用和转义。

#### 语法例子 ####
- 转义`Crime d'Amour`中的`'`这个特殊字符:
> 	
	Crime d\'Amour	
- 下面的字符串因为是引用，所有其中的`'`需要特殊转义处理
> 
	'Crime d'\''Amour'
- 包括前导和后随的空格必须要引用模式:
> 
	'  this string starts and ends with whitespaces  '
- 转义和引用可以接续在一起：
> 
	' The string '\'string\'' is a string '
- 为了包括一个`\`需要引用或者转义
> 
	'c:\foo' can be written as c:\\foo

### 日期 ###
接受如下的语法:
> 
	[(YYYY-MM-DD|YYYYMMDD)[T|t| ]]((HH:MM:SS[.m...]]])|(HHMMSS[.m...]]]))[Z]
	now
如果值为`now`表示当前时间

时间是本地时间，除非`Z`被附加到最后，它表示采用`UTC`时间。如果年-月-日没有指定就以当前的年-月-日。

### 持续时间 ###
它有两种表示方式：
- `[-][HH:]MM:SS[.m...]` 
	
	`HH`表示小时数，`MM`表示分钟数（最多2位数字）`SS`表示秒数（也最多2位数字），`m`是`SS`的小数位值
- `[-]S+[.m...]`

	`S`是秒的数值，`m`是`S`的小数位值。

两种语法前面都可选`'-'`号，表示负数持续时间。

#### 持续时间例子 ####
下面均是有效持续时间:
- '55' 表示55秒
- '12:03:45' 表示12小时3分钟45秒
- '23.189' 表示23.189秒

### 视频尺寸（分辨率） ###
指定视频源的尺寸大小，它可以是一些表示特定（预设）尺寸的字符串名或者`widthxheight`（其中width和height都是数字值）的字符串
下面是一些预设的表示尺寸的字符串名及其对应分辨率:
- ‘ntsc’    720x480 
- ‘pal’    720x576 
- ‘qntsc’    352x240 
- ‘qpal’    352x288 
- ‘sntsc’    640x480 
- ‘spal’    768x576 
- ‘film’    352x240 
- ‘ntsc-film’    352x240 
- ‘sqcif’    128x96 
- ‘qcif’    176x144 
- ‘cif’    352x288 
- ‘4cif’    704x576 
- ‘16cif’    1408x1152 
- ‘qqvga’    160x120 
- ‘qvga’    320x240 
- ‘vga’    640x480 
- ‘svga’    800x600 
- ‘xga’    1024x768 
- ‘uxga’    1600x1200 
- ‘qxga’    2048x1536 
- ‘sxga’    1280x1024 
- ‘qsxga’    2560x2048 
- ‘hsxga’    5120x4096 
- ‘wvga’    852x480 
- ‘wxga’    1366x768 
- ‘wsxga’    1600x1024 
- ‘wuxga’    1920x1200 
- ‘woxga’    2560x1600 
- ‘wqsxga’    3200x2048 
- ‘wquxga’    3840x2400 
- ‘whsxga’    6400x4096 
- ‘whuxga’    7680x4800 
- ‘cga’    320x200 
- ‘ega’    640x350 
- ‘hd480’    852x480 
- ‘hd720’    1280x720 
- ‘hd1080’    1920x1080 
- ‘2k’    2048x1080 
- ‘2kflat’    1998x1080 
- ‘2kscope’    2048x858 
- ‘4k’    4096x2160 
- ‘4kflat’    3996x2160 
- ‘4kscope’    4096x1716 
- ‘nhd’    640x360 
- ‘hqvga’    240x160 
- ‘wqvga’    400x240 
- ‘fwqvga’    432x240 
- ‘hvga’    480x320 
- ‘qhd’    960x540 

### 视频帧率 ###
指定视频的帧速率，除了用每秒帧数表示外，还可以用`frame_rate_num/frame_rate_den`这样的格式字符串表示，此外还有一些预定义的帧率名字符串。
下面就是一些预定义的帧率名及对应的帧率:
- 'ntsc' 30000/1001
- 'pal' 25/1
- 'qpal'	 25/1
- 'sntsc' 30000/1001
- 'spal' 25/1
- 'film' 24/1
- 'ntsc-film' 24000/1001
### 比率 ###
一个比率值可以是一个表达式或者`numerator.denominator`一样的含小数值。

**注意**比率无限值（1/0)或者负数值被认为是有效的，这里你需要摒弃以往的一些看法。

未定义的值可以用"0:0"字符串表示。

### 颜色/Color ###
允许采用下面预定义的颜色名或者一个`[0x|#]RRGGBB[AA]`这样序列的16进制数字值，可以通过`@`来附加透明度表示，透明度分量（alpha）可以是"0x"后面跟一个16进制数或者0到1之间的十进制字符串，它代表不透明度值（'0x00'或者'0'表示完全透明，'0xFF'或者'1'表示完全不透明），如果没有专门指定透明分量，则默认为'0XFF'。

'random'字符串会随机一个颜色。

下面是预定义的颜色名以及对应的颜色值:

- ‘AliceBlue’    0xF0F8FF 
- ‘AntiqueWhite’   0xFAEBD7 
- ‘Aqua’    0x00FFFF 
- ‘Aquamarine’    0x7FFFD4 
- ‘Azure’    0xF0FFFF 
- ‘Beige’    0xF5F5DC 
- ‘Bisque’    0xFFE4C4 
- ‘Black’    0x000000 
- ‘BlanchedAlmond’    0xFFEBCD 
- ‘Blue’    0x0000FF 
- ‘BlueViolet’    0x8A2BE2 
- ‘Brown’    0xA52A2A 
- ‘BurlyWood’    0xDEB887 
- ‘CadetBlue’    0x5F9EA0 
- ‘Chartreuse’    0x7FFF00 
- ‘Chocolate’    0xD2691E 
- ‘Coral’    0xFF7F50 
- ‘CornflowerBlue’    0x6495ED 
- ‘Cornsilk’    0xFFF8DC 
- ‘Crimson’    0xDC143C 
- ‘Cyan’    0x00FFFF 
- ‘DarkBlue’    0x00008B 
- ‘DarkCyan’    0x008B8B 
- ‘DarkGoldenRod’    0xB8860B 
- ‘DarkGray’    0xA9A9A9 
- ‘DarkGreen’    0x006400 
- ‘DarkKhaki’    0xBDB76B 
- ‘DarkMagenta’    0x8B008B 
- ‘DarkOliveGreen’    0x556B2F 
- ‘Darkorange’    0xFF8C00 
- ‘DarkOrchid’    0x9932CC 
- ‘DarkRed’    0x8B0000 
- ‘DarkSalmon’    0xE9967A 
- ‘DarkSeaGreen’    0x8FBC8F 
- ‘DarkSlateBlue’    0x483D8B 
- ‘DarkSlateGray’    0x2F4F4F 
- ‘DarkTurquoise’    0x00CED1 
- ‘DarkViolet’    0x9400D3 
- ‘DeepPink’    0xFF1493 
- ‘DeepSkyBlue’    0x00BFFF 
- ‘DimGray’    0x696969 
- ‘DodgerBlue’    0x1E90FF 
- ‘FireBrick’    0xB22222 
- ‘FloralWhite’    0xFFFAF0 
- ‘ForestGreen’    0x228B22 
- ‘Fuchsia’    0xFF00FF 
- ‘Gainsboro’    0xDCDCDC 
- ‘GhostWhite’    0xF8F8FF 
- ‘Gold’    0xFFD700 
- ‘GoldenRod’    0xDAA520 
- ‘Gray’    0x808080 
- ‘Green’    0x008000 
- ‘GreenYellow’    0xADFF2F 
- ‘HoneyDew’    0xF0FFF0 
- ‘HotPink’    0xFF69B4 
- ‘IndianRed’    0xCD5C5C 
- ‘Indigo’    0x4B0082 
- ‘Ivory’    0xFFFFF0 
- ‘Khaki’    0xF0E68C 
- ‘Lavender’    0xE6E6FA 
- ‘LavenderBlush’    0xFFF0F5 
- ‘LawnGreen’    0x7CFC00 
- ‘LemonChiffon’    0xFFFACD 
- ‘LightBlue’    0xADD8E6 
- ‘LightCoral’    0xF08080 
- ‘LightCyan’    0xE0FFFF 
- ‘LightGoldenRodYellow’    0xFAFAD2 
- ‘LightGreen’    0x90EE90 
- ‘LightGrey’    0xD3D3D3 
- ‘LightPink’    0xFFB6C1 
- ‘LightSalmon’    0xFFA07A 
- ‘LightSeaGreen’    0x20B2AA 
- ‘LightSkyBlue’    0x87CEFA 
- ‘LightSlateGray’    0x778899 
- ‘LightSteelBlue’    0xB0C4DE 
- ‘LightYellow’    0xFFFFE0 
- ‘Lime’    0x00FF00 
- ‘LimeGreen’    0x32CD32 
- ‘Linen’    0xFAF0E6 
- ‘Magenta’    0xFF00FF 
- ‘Maroon’    0x800000 
- ‘MediumAquaMarine’    0x66CDAA 
- ‘MediumBlue’    0x0000CD 
- ‘MediumOrchid’    0xBA55D3 
- ‘MediumPurple’    0x9370D8 
- ‘MediumSeaGreen’    0x3CB371 
- ‘MediumSlateBlue’    0x7B68EE 
- ‘MediumSpringGreen’    0x00FA9A 
- ‘MediumTurquoise’    0x48D1CC 
- ‘MediumVioletRed’    0xC71585 
- ‘MidnightBlue’    0x191970 
- ‘MintCream’    0xF5FFFA 
- ‘MistyRose’    0xFFE4E1 
- ‘Moccasin’    0xFFE4B5 
- ‘NavajoWhite’    0xFFDEAD 
- ‘Navy’    0x000080 
- ‘OldLace’    0xFDF5E6 
- ‘Olive’    0x808000 
- ‘OliveDrab’    0x6B8E23 
- ‘Orange’    0xFFA500 
- ‘OrangeRed’    0xFF4500 
- ‘Orchid’    0xDA70D6 
- ‘PaleGoldenRod’    0xEEE8AA 
- ‘PaleGreen’    0x98FB98 
- ‘PaleTurquoise’    0xAFEEEE 
- ‘PaleVioletRed’    0xD87093 
- ‘PapayaWhip’    0xFFEFD5 
- ‘PeachPuff’    0xFFDAB9 
- ‘Peru’    0xCD853F 
- ‘Pink’    0xFFC0CB 
- ‘Plum’    0xDDA0DD 
- ‘PowderBlue’    0xB0E0E6 
- ‘Purple’    0x800080 
- ‘Red’    0xFF0000 
- ‘RosyBrown’    0xBC8F8F 
- ‘RoyalBlue’    0x4169E1 
- ‘SaddleBrown’	0x8B4513 
- ‘Salmon’    0xFA8072 
- ‘SandyBrown’    0xF4A460 
- ‘SeaGreen’    0x2E8B57 
- ‘SeaShell’    0xFFF5EE 
- ‘Sienna’    0xA0522D 
- ‘Silver’    0xC0C0C0 
- ‘SkyBlue’    0x87CEEB 
- ‘SlateBlue’    0x6A5ACD 
- ‘SlateGray’    0x708090 
- ‘Snow’    0xFFFAFA 
- ‘SpringGreen’    0x00FF7F 
- ‘SteelBlue’    0x4682B4 
- ‘Tan’    0xD2B48C 
- ‘Teal’    0x008080 
- ‘Thistle’    0xD8BFD8 
- ‘Tomato’    0xFF6347 
- ‘Turquoise’    0x40E0D0 
- ‘Violet’    0xEE82EE 
- ‘Wheat’    0xF5DEB3 
- ‘White’    0xFFFFFF 
- ‘WhiteSmoke’    0xF5F5F5 
- ‘Yellow’    0xFFFF00 
- ‘YellowGreen’    0x9ACD32 

### 通道布局 ###
对于多音频通道的流，一个通道布局可以具体描述其配置情况。为了描述通道布局，ffmpeg采用了一些特殊的语法。

除了可以采用ID标识外，也可以采用下表的预定义:
- ‘FL’    front left 左前
- ‘FR’    front right 右前
- ‘FC’    front center 前中
- ‘LFE’    low frequency 重低音
- ‘BL’    back left 左后
- ‘BR’    back right 右后
- ‘FLC’    front left-of-center 前左中 
- ‘FRC’    front right-of-center 前右中
- ‘BC’    back center 后中
- ‘SL’    side left 左侧
- ‘SR’    side right 右侧
- ‘TC’    top center 顶中
- ‘TFL’    top front left 顶左前 
- ‘TFC’    top front center 顶前中
- ‘TFR’    top front right 顶右前
- ‘TBL’    top back left 顶左后
- ‘TBC’    top back center 顶后中
- ‘TBR’    top back right 顶右后
- ‘DL’    downmix left 左缩混
- ‘DR’    downmix right 右缩混
- ‘WL’    wide left 左边
- ‘WR’    wide right 右边
- ‘SDL’    surround direct left  左直通
- ‘SDR’    surround direct right 右直通
- ‘LFE2’    low frequency 2 超重低音

标准的通道布局可以采用如下的定义:
- ‘mono’    FC 
- ‘stereo’    FL+FR 
- ‘2.1’    FL+FR+LFE 
- ‘3.0’    FL+FR+FC 
- ‘3.0(back)’    FL+FR+BC 
- ‘4.0’    FL+FR+FC+BC 
- ‘quad’    FL+FR+BL+BR 
- ‘quad(side)’    FL+FR+SL+SR 
- ‘3.1’    FL+FR+FC+LFE 
- ‘5.0’    FL+FR+FC+BL+BR 
- ‘5.0(side)’    FL+FR+FC+SL+SR 
- ‘4.1’    FL+FR+FC+LFE+BC 
- ‘5.1’    FL+FR+FC+LFE+BL+BR 
- ‘5.1(side)’    FL+FR+FC+LFE+SL+SR 
- ‘6.0’    FL+FR+FC+BC+SL+SR 
- ‘6.0(front)’    FL+FR+FLC+FRC+SL+SR 
- ‘hexagonal’    FL+FR+FC+BL+BR+BC 
- ‘6.1’    FL+FR+FC+LFE+BC+SL+SR 
- ‘6.1’    FL+FR+FC+LFE+BL+BR+BC 
- ‘6.1(front)’    FL+FR+LFE+FLC+FRC+SL+SR 
- ‘7.0’    FL+FR+FC+BL+BR+SL+SR 
- ‘7.0(front)’    FL+FR+FC+FLC+FRC+SL+SR 
- ‘7.1’    FL+FR+FC+LFE+BL+BR+SL+SR 
- ‘7.1(wide)’    FL+FR+FC+LFE+BL+BR+FLC+FRC 
- ‘7.1(wide-side)’    FL+FR+FC+LFE+FLC+FRC+SL+SR 
- ‘octagonal’    FL+FR+FC+BL+BR+BC+SL+SR 
- ‘downmix’    DL+DR 

一个特定的通道布局可以是一组由`+`或者`|`连接起来多个值，其中每个值可以是：
- 前面的标准通道布局名，例如'mono','stereo','4.0','quad','5.0'等等
- 或者单个命名通道如'FL','FR','FC','LFE'等等
- 多个通道数字序号，使用后缀'c'的十进制，对于利用数字表示的默认通道布局，参考`av-get_default_channel_layout`说明。
- 通道布局蒙版(mask)，是'0x'起始的16进制，参考在`libavutil/channel_layout.h`中声明的`AV_CH_*`宏。

从libavutil版本53开始，尾随'c'的十进制数表示通道变成必须（除非采用16进制的蒙版来表示通道）

参考声明于`libavutil/channel_layout.h`中的`av_get_channel_layout`进行深入了解。
