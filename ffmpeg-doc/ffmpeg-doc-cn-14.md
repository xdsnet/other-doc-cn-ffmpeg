## 字幕解码器 ##

### dvdsub ###
解码用于dvd的bitmap类型字幕解码。该类型也用于vobsub文件和一些Matroska文件。

#### dvdsub解码选项 ####
- `palette`

    指定位图的全局调色板。当存储在VobSub中时，调色板可以依据索引表示颜色。在Matroska文件中，调色板以同于VobSub的格式存储在扩展数据区。在DVD中，存储在IFO文件中，因此在只读取VOB文件时不可用则。

    选项的格式是24位（bit）数，用16进制表示字符串（没有前导的0x）例如 0d00ee, ee450d, 101010, eaeaea, 0ce60b, ec14ed, ebff0b, 0d617a, 7b7b7b, d1d1d1, 7b2a0e, 0d950c, 0f007b, cf0dec, cfa80c, 7c127b.

- `ifo_palette`

    指定IFO文件中得到全局调色板（实验性质）=

- `forced_subs_only`

    只在解码强制字幕。有些字幕在同一轨道中保留了强制字幕和非强制字幕，选用这个选项将只解码强制字幕，即强制字幕标志设为1的字幕。默认是0 

### libzvbi-teletext ###
libzvbi允许libavcodec解码DVB的teletext页面和DVB的teletext字幕。需要libzvbi头和库存在配置。在编译是需配置` --enable-libzvbi`以启用

#### libzvbi-teletext选项 ####
- `txt_page`

    列出需要解码的teletext页编号。你可以用`*`表示所有页，没有匹配的页都被丢弃。默认是`*` 
- `txt_chop_top`

    取消顶部teletext行。默认是 1. 
- `txt_format`

    指定解码字幕的格式。这个teletext解码器可解码位图或简单文本，如果页面中有特定的图形和颜色则可能需要设置为"bitmap"，因为简单文本不能包含这些内容。否则可以选用"text"以简化处理。默认是bitmap
- `txt_left`

    位图X偏移量, 默认是 0. 
- `txt_top`

    位图Y偏移量, 默认是 0. 
- `txt_chop_spaces`

    在生成的文本中保留前导和尾随的空格去除空行。该选项常用于字幕需要显示为多行（双行）使得字幕放置的更好看。 默认是1. 
- `txt_duration`

    teletex页面或者字幕显示的时间，单位是ms，默认值是3000，即30秒
- `txt_transparent`

    让生成的teletext位图透明，默认是0，表示有一个不透明（黑）的背景色。 
