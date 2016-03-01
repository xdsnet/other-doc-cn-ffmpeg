## 11 解码器
解码器是让FFmpeg能对多媒体流进行解码的配置元素。

默认在编译FFmpeg时所有（内置）有效的解码器都会自动支持。如果解码器需要特别扩展库，则需要手动通过`--enable-lib`选项来进行支持。可以在配置编译项目中通过`--list-decoders`了解所有有效解码器（包括需要扩展库的）。

也可以通过在配置中采用`--disable-decoders`选项单独禁用某个解码器。`--enable-decoder=DECODER / --disable-decoder=DECODER`分别是启用/禁用`DECODER`解码器。

在`ff**`工具中（ffmpeg/ffplayer 等等）选项`-decoders` 可以显示当前有效的解码器。