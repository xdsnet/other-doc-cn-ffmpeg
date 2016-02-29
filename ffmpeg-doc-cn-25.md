## 25 设备选项
`libavdevice`库提供类似`libavformat`的接口，即一个输入设备被认为类似一个分离器活着输出设备类似一个混合器。这些接口也类似`libavformat`一样提供一些常规设备选项。（参考ffmepeg 格式手册）。

当然，一些输入或者输出设备还提供一些私有的选项，它们只在特定的组件中有效。

可以做ffmpeg命令行中采用`－option value`来设定某个选项点，或者通过`libavutil／opt.h`中的API设置，或者通过`AVFormatContext`的显式值来进行设置。
