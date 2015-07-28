## 9 OpenCL选项 ##
当FFmpeg编译时打开了`--enable-opencl`配置，则可以在全局使用OpenCL选项。  
下面是支持的选项：
- `build_options`:设置编译选项，指定编译的注册核心  
	参考"OpenCL Specification Version: 1.2 chapter 5.6.4". 
- `platform_idx`：选定指定平台运行OpenCL的编码（格式），这里指定的索引必须是可以由`ffmpeg -opencl_bench`或者`av_opencl_get_device_list()`查询到的设备列表索引。
- `device_idx`：指定设备运行的OpenCL编码（格式）。这里指定的索引必须是可以由`ffmpeg -opencl_bench`或者`av_opencl_get_device_list()`查询到的设备列表索引。