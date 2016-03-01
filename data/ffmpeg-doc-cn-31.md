## 31 graph2dot
FFmpeg工具目录下包含一个`graph2dot`程序可以用来分析滤镜链图描述并产生用`dot`语言描述的对应文本表示。

调用命令：

	graph2dot -h
可以了解如何使用`graph2dot`

你可以把`dot`语言描述用于`dot`程序（`graphviz`程序套件中），并获取到滤镜链图的图形表示。

例如命令序列：

    echo GRAPH_DESCRIPTION | \
    tools/graph2dot -o graph.tmp && \
    dot -Tpng graph.tmp -o graph.png && \
    display graph.png
就用来创建和显示一个由`GRAPH_DESCRIPTION`字符串定义的滤镜链图图示。**注意**这里表示滤镜链图的字符串必须是能完整独立的表达的图，其显示定义了输入和输出。例如对于下面的命令形式：

	ffmpeg -i infile -vf scale=640:360 outfile
你的`GRAPH_DESCRIPTION`字符串需要为：

	nullsrc,scale=640:360,nullsink
你可能需要用`nullsrc`参数以及添加`format filter`来模拟指定一个输入文件。