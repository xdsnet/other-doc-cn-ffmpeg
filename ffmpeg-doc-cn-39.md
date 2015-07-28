## 39 视频槽 ##
下面是当前有效的视频槽（池）介绍。
### buffersink ###
缓冲视频帧，可作为滤镜链图中有效的结束点。

这个槽主要用于编程使用，特别是通过`libavfilter/buffersink.h`的接口或选择操作系统

它接受指向`AVABufferSinkContext`结构的指针，用于定义传入缓冲区的格式，作为不透明参数传递给`avfilter_init_filter`以初始化。
### nullsink ###
Null（空）视频槽，绝对没有输入的视频。它主要用作模板以分析/调试工具。
