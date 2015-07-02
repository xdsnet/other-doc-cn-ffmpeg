## 36 音频槽 ##
下面介绍当前有效的音频皮肤。
### abuffersink ###
缓冲音频帧，并可作为滤镜的结束。

这个槽主要用于编程使用，特别是通过`libavfilter/buffersink.h`的接口或选择操作系统

它接受指向`AVABufferSinkContext`结构的指针，用于定义传入缓冲区的格式，作为不透明参数传递给`avfilter_init_filter`以初始化。
### anullsink ###
Null（空）音频槽，绝对没有输入的音频。它主要用作模板以分析/调试工具。