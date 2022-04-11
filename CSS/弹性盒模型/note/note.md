一.布局（Layout）
布局的传统解决方案，基于盒模型，并依赖display、position和float属性实现。

二.Flex布局
可以简便、完整，响应式地实现各种页面布局。
1.采用Flex布局的元素，称为Flex容器（flex container），简称“容器”。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称“项目”。
2.容器默认存在两根轴，即主轴（main axis）和交叉轴（cross axis）。默认情况下，主轴是水平方向，交叉轴是垂直方向。主轴开始位置（与边框的交叉点）叫做main start，结束位置叫做main end。交叉轴开始位置叫做cross start，结束位置叫做cross end。
3.默认情况下，项目沿主轴排列，单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

三.Flex容器属性
1.flex-direction：定义Flex容器的主轴方向，即Flex项目的排列方向。
（1）row：默认值，主轴为水平方向，起点在左端，即主轴方向从左至右。
（2）row-reverse：主轴为水平方向，起点在右端，即主轴方向从右至左。
（3）column：主轴为垂直方向，起点在上沿，即主轴方向从上至下。
（4）column-reverse：主轴为垂直方向，起点在下沿，即主轴方向从下至上。
2.flex-wrap：定义Flex容器中的Flex项目的换行规则。
（1）nowrap：默认值，不换行。
（2）wrap：换行，第一行在上方。
（3）wrap-reverse：换行，第一行在下方。
3.flex-flow：flex-direction和flex-wrap的简写形式。
默认值，row nowrap。
4.justify-content：定义Flex容器中的Flex项目在主轴方向的对齐方式。
（1）flex-start：默认值，与主轴起点对齐。
（2）flex-end：与主轴终点对齐。
（3）center：与主轴中心对齐。
（4）space-between：两端对齐，项目之间的间隔都相等。
（5）space-around：每个项目两侧的间隔相等，项目之间的间隔是项目与边框间隔的两倍。
5.align-items：定义Flex容器中的Flex项目在交叉轴方向的对齐方式。
（1）flex-start：与交叉轴的起点对齐。
（2）flex-end：与交叉轴的终点对齐。
（3）center：与交叉轴的中心对齐。
（4）baseline：与项目第一行文字的基线对齐。
（5）stretch：默认值，如果Flex项目未设置高度或者设置为auto，将占满整个容器的高度。
6.align-content：定义Flex容器中的Flex项目在多根轴方向的对齐方式，如果只有一根轴线，该属性不起作用。
（1）flex-start：与交叉轴的起点对齐。
（2）flex-end：与交叉轴的终点对齐。
（3）center：与交叉轴的中心对齐。
（4）space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
（5）space-around：每个轴线两侧的间隔都相等，所以轴线之间的间隔是轴线与边框之间的间隔的两倍。
（6）stretch：默认值，轴线占满整个交叉轴。

四.Flex项目属性
1.order：定义Flex项目的排列顺序，数值越小，排列越靠前，默认值为0，接受负数。
2.flex-grow：定义Flex项目的放大比例，默认值为0，即如果存在剩余空间，也不放大。
如果所有Flex项目的flex-grow都设置为1，则所有Flex项目将等分剩余空间（如果有的话）。
如果一个Flex项目的flex-grow值为2，其他Flex项目的flex-grow值为1，则前者占据剩余空间将比其他Flex项目多一倍。
3.flex-shrink：定义Flex项目的缩小比例，默认为1，如果空间不足，该项目将缩小。
如果所有Flex项目的flex-shrink属性都为1，当空间不足时，都将等比缩小。
如果一个Flex项目的flex-shrink属性为0，其他Flex项目都为1，则空间不足时，前者不缩小。
4.flex-basis：定义在分配多余空间之前，Flex项目占据的主轴空间。浏览器根据这个属性，计算主轴是否有多余空间。默认值为auto，即Flex项目本来的大小。
它也可以设置为固定大小。
5.flex：是flex-grow、flex-shrink，flex-basis的简写，默认值为0 1 auto。
该属性有两个快捷值：
（1）auto：1 1 auto。
（2）none：0 0 auto。
建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。
6.align-self：允许单个Flex项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承Flex容器的align-items属性，如果没有Flex容器，则等同于stretch。该属性可取6个值，除了auto，其他都与align-items属性完全一致。