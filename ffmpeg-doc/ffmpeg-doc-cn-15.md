## 15 编码器 ##
编码器是ffmpeg用来编码多媒体流的配置单元。

当编译生成ffmpeg时，所有内置编码器默认被支持。可以通过手动设置`--enable-lib`选项以支持外部（扩展）库。可以在配置选项中利用` --list-encoders`了解所有可能的编码器

可以利用`--disable-encoders`禁用所有编码器，也可以单独的利用 `--enable-encoder=ENCODER / --disable-encoder=ENCODER`启用/禁用个别的编码器。

在ffmpeg工具集中利用选项` -encoders`可以了解支持的编码器。