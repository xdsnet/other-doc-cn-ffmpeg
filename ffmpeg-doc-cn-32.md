## 滤镜链图介绍 ##
一个滤镜链图（filtergraph）是连接滤镜的有向图。它可以包含循环动作，也可以在多个滤镜间形成链路，每个链接都有一个连接到滤镜的输入和一个连接到滤镜的输出。

滤镜链图中的每个滤镜都是一个滤镜注册类应用程序的实例，它定义了滤镜的功能、输入接口和输出接口。

如果滤镜没有输入端（接口），则被称作“源”，如果滤镜没有输出端则被称作“槽”（这样的滤镜用于描述/测试等场景，而不用于实际处理）

### 滤镜链图语法 ###
滤镜链图采用文本表示，其有由一些ffmpeg和ffplay通用的选项`-filter/-vf/-af`和`-filter_complex`（ffmpeg）以及`-vf/-af`（ffplay）外加定义与`libavfilter/avfilter.h`的`avfilter_graph_parse_ptr()`等来描述。

一个滤镜链包含序列链接起来的滤镜，这个序列由“，”分隔各个滤镜描述

一个滤镜链图包含序列滤镜链，这个序列有“；”分隔各个滤镜链描述

一个滤镜由一个字符串表单表示：` [in_link_1]...[in_link_N]filter_name=arguments[out_link_1]...[out_link_M] `

这里`filter_name`是滤镜类名字，用于指定滤镜实例（它注册于程序中）。其后的`=arguments`用于指定滤镜选项。`arguments`是用于初始化滤镜的参数，它可能有下面两类表单中的一个：

- 一个“：”分隔的`key=value`列表
- 一个“：”分隔的列表`value`值，在这种情况下，键（key）假定为选项名声明顺序，如`fade`滤镜按顺序声明了3个选项`type`、`start_frame`和`nb_frames`，则参数`in:0:30`意味着`type`为`in`，`start_frame`为`0`，`nb_frames`为`30`。
- 前面二者的混合。这种情况下，键值对必须在前，然后接遵循相同约束的若干值。在键值对中可以按任意顺序设置优先顺序。（后接值按最后一个键值对顺序延续设置）。

如果选项的值本身就是一个列表（例如`format`滤镜有一个像素格式列表选项），则这种列表通常用“|”分隔。

列表参数可以被`'`（单引号）包含起来。字符`\`作为转义字符用于引号包含的文本。否则参数字符串在遇到特殊字符（例如'[]=;,'）处被看作终止。

在滤镜名和参数前 和 后 有一个连接标签列表。一个连接标签允许命名1个名字的连接，其作为滤镜的输入或者输出端口。以下预订标签`in_link_1 ... in_link_N`作为滤镜的输入端，`out_link_1 ... out_link_M`作为滤镜的输出端。

当中一个滤镜链图中找到相同名字的连接标签时，一个在相应输入端和输出端之间的连接被创建（即认为它们是连接在一起的，如果用做一个滤镜的输出，又用着一个滤镜的输入，则表示从前一个滤镜输出到后一个滤镜）

如果一个输出端没有命名标签，它默认连接到滤镜链上后面滤镜中第一个没有命名标签的输入端。例如：

	nullsrc, split[L1], [L2]overlay, nullsink
这里`split`有两路输出，`overlay`有两路输入，`split`的第一路输出被命名为标签"L1",`overlay`的第一路输入被命名为标签"L2"。则`split`的第二路输出链接到`overlay`的第二路输入（它们都没有用标签命名）。

在一个滤镜描述中，如果第一个滤镜的输入没有指定，则假定为"in"，如果最后一个滤镜输出没有指定，则假定为"out"

在一个完整的滤镜链上，所有未标签命名的输入和输出必须被连接（匹配）。滤镜链图中如果所有滤镜的输入和输出都被连接则被认为是有效的。

如果格式要求，则`Libavfilter`会自动插入`scale`滤镜。对于滤镜链图描述，可以通过`sws_flags=flags`来指定`swscale`标志实现自动插入放缩。

这里有一个`BNF`描述的滤镜链图语法：

    NAME ::= sequence of alphanumeric characters and '_'
    LINKLABEL::= "[" NAME "]"
    LINKLABELS   ::= LINKLABEL [LINKLABELS]
    FILTER_ARGUMENTS ::= sequence of chars (possibly quoted)
    FILTER   ::= [LINKLABELS] NAME ["=" FILTER_ARGUMENTS] [LINKLABELS]
    FILTERCHAIN  ::= FILTER [,FILTERCHAIN]
    FILTERGRAPH  ::= [sws_flags=flags;] FILTERCHAIN [;FILTERGRAPH]

### 注意滤镜链图中的转义 ###
滤镜链图成分需要包含多层的转义。参考ffmpeg-utils(1) 手册中的“引用与转义”章节（"Quoting and escaping" ）了解更多关于转义过程的信息。

第一层的转义效果在每个滤镜选项值中，其可能包含特殊字符":"来分隔值，或者一个转义符"\"

第二层的转义在整个滤镜描述，其可能包含转义符"\"或者特殊字符"[],;" ，它们被用于滤镜链图的描述

实际上，当你在shell命令行上描述滤镜链图时还可能遇见第三层的转义（处理shell中需要转义的字符）

例如下面的字符需要嵌入`drawtext`滤镜描述的`text`值

    this is a 'string': may contain one, or more, special characters
这个字符串包含`'`字符需要被转义，还有`:`这里也需要被转义，方法是：

    text=this is a \'string\'\: may contain one, or more, special characters

当嵌入滤镜描述时发生第二层的转义，为了转义所有的滤镜链图特殊字符，需要按下例处理：

    drawtext=text=this is a \\\'string\\\'\\: may contain one\, or more\, special characters

（**注意**对于\'转义中的每个特殊字符都需要再次转义，就成了\\\')

最终当在shell命令行中写这个滤镜链图时再次转义。例如需要对每个特殊字符和转义处理的`\`等进行转义，最终为：
    
    -vf "drawtext=text=this is a \\\\\\'string\\\\\\'\\\\: may contain one\\, or more\\, special characters"