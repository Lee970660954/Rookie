问题：用户从在浏览器地址栏中输入url，到页面显示的过程，浏览器内部都发生了什么？

一.进程和线程
1.概念
（1）进程：process，是程序在执行过程中分配和管理资源的基本单位。
（2）线程：thread，是CPU资源调度和任务分派的基本单位，它可与同属一个进程的其他线程共享进程所拥有的全部资源。
简单说，进程可以理解为正在执行的应用程序，线程可以理解为应用程序中的代码的执行器。它们之间的关系是，线程是跑在进程里，一个进程可能拥有一个或多个线程，而一个线程只能隶属于一个进程。

2.进程间通信
在应用程序中，为了满足功能的需要，启动的进程会创建另外的新的进程来处理其他任务，这些创建出来的新进程拥有全新的独立内存空间，不能与原来的进程共享内存，如果这些进程需要通信，可以通过IPC（Inter Process Communication）机制进行。
很多应用程序都会采用这种多进程的方式来设计，因为进程和进程之间是互相独立的，它们互不影响，也就是说，当其中一个进程挂掉了之后，不会影响到其他进程的执行，只需要重启挂掉的进程就可以恢复运行。


二.浏览器架构
浏览器的架构可以是单进程多线程，也可以是使用IPC机制通信的多进程。
以Chrome为例，介绍浏览器多进程架构。
1.主要进程及功能
（1）浏览器进程（Browser Process）：负责浏览器的Tab的前进、后退、地址栏，书签栏的工作，处理浏览器的一些不可见的底层操作，比如网络请求和文件访问等。
（2）渲染进程（Renderer Process）：负责一个Tab内显示相关的工作，也被称为渲染引擎。
（3）插件进程（Plugin Process）：负责控制网页使用到的插件。
（4）GPU进程（GPU Process）：负责处理整个应用程序的GPU任务。

2.四个进程间的关系
以开头的问题为例说明，当我们要浏览一个网页时，我们会在浏览器地址栏中输入url，这个时候会经历以下阶段：
（1）Browser Process会获取到地址栏中的url并发送请求，获取url对应的html内容。
（2）然后将html交给Renderer Process，Renderer Process解析html内容，解析过程中遇到需要请求的资源又会通知Browser Process进行加载，同时通知Browser Process，需要Plugin Process加载插件资源，执行插件代码。
（3）解析完成后，Renderer Process计算得到图像帧，并将这些图像帧交给GPU Process，GPU Process将其转化为图像显示在屏幕上。

3.多进程架构的好处
（1）更高的容错性。
当今Web应用中，Html，Javascript和Css日益复杂，这些跑在渲染引擎中的代码，频繁的出现Bug，而有些Bug会直接导致渲染引擎崩溃，多进程架构使得每一个渲染引擎运行在各自的进程中，相互之间不受影响，也就是说，当其中一个页面崩溃之后，其他页面还可以正常运行而不受影响。
（2）更高的安全性和沙盒性。
浏览器对不同进程限制了不同的权限，并为其提供沙盒运行环境，使其更安全可靠。
（3）更高的响应速度。
单进程架构中，各任务相互竞争抢夺CPU资源和进程的内存，使得浏览器响应速度变慢，而多进程架构正好规避了这个问题。

4.浏览器的进程模式
为了节省内存，Chrome浏览器提供了四种进程模式（Process Models），不同的进程模式会对tab进程做不同的处理。
（1）Process-per-site-instance：默认模式，同一个site-instance使用一个进程。
（2）Process-per-site：同一个site使用一个进程。
（3）Process-per-tab：每个tab使用一个进程。
（4）Single process：所有tab共用一个进程。

site和site-instance区别？
site：主域名和协议相同。
site-instance：打开的新页面和旧页面满足site，并且是通过a标签或JS代码打开的新tab页面，即同属一个site-instance。

为什么选择Process-per-site-instance作为默认模式？
因为它兼容了性能和安全性，是一个比较中庸的模式。
一方面，相较于Process-per-tab，能够少开很多进程，意味着更少的内存占用，即降低性能损耗。
另一方面，相较于Process-per-site，能够更好的隔离相同主域名下毫无关联的tab，更加安全。


三.核心线程
1.Browser Process针对工作的不同，划分出不同的线程：
（1）UI thread：UI线程，控制浏览器上的按钮、输入框和标签栏等。
（2）Network thread：网络线程，处理网络请求。
（3）Storage thread：控制文件访问。
2.Renderer Process包含的线程：
（1）Main thread：一个主线程。
（2）Worker thread：多个工作线程。
（3）Compositor thread：一个合成器线程。
（4）Raster thread：多个光栅化线程。


四.导航过程都发生了什么
从用户浏览网页的场景，深入了解浏览器进程间通信、进程如何调配线程、渲染进程是如何呈现页面的？
1.处理输入
当用户在浏览器的地址栏中输入内容并按下回车时，Browser Process的UI thread会判断输入的内容是搜索关键词还是url，如果是搜索关键词，使用默认搜索引擎进行搜索，否则开始请求url。

2.网络请求
UI thread将关键词搜索对应的url或输入的url交给Network thread，此时UI thread使Tab图标变为加载中状态，Network thread通过DNS解析、建立TCP连接，发送http请求并接收响应等过程，获取到html内容。

3.读取响应
Network thread接收到服务器的响应后，解析http响应报文，然后根据响应头中的Content-Type字段来确定响应主体的媒体类型（MIME Type），如果媒体类型是一个html文件，则将响应数据通过IPC通信机制交给Renderer Process来进行下一步操作，如果是zip文件或者其他文件，会把相关数据传输给下载管理器。

解析响应的同时，Network thread会做CORB检查，来确定跨站请求是否安全，如果安全才能进行下一步操作，否则会将响应拦截，不会交给Renderer Process。

4.查找渲染进程
在Network thread做好各种检查，确定将响应通过IPC通信机制交给Renderer Process时，会通知UI thread数据已加载完毕，UI thread会查找到一个Renderer Process进行网页渲染。

浏览器为了对查找Renderer Process这一步骤进行优化，考虑到网络请求获取响应需要时间，所以在网络请求阶段开始，浏览器已经预先启动了一个Renderer Process，当Network thread接收到数据时，Renderer Process就已经准备好了，但是如果遇到重定向，这个预先准备的Renderer Process就不可用了，需要重新启动一个Renderer Process。

5.初始化加载完成
Renderer Process开始解析资源并渲染页面，当页面渲染完成后，会向Browser Process发送IPC消息，告知Browser Process，这个时候UI thread会停止Tab中的加载中图标。


五.浏览器渲染原理
浏览器进程通过IPC通信机制把数据交给渲染进程，渲染进程负责Tab内的所有事情，核心工作是将Html、Css和Js代码，转化为用户可进行交互的Web页面，那么渲染进程是如何工作的呢？

1.构建Dom
Main thread解析Html并转化为Dom。

2.样式计算（Style calculation）
Main thread在解析Html过程中，遇到style标签或者link标签的Css资源，会并行加载Css代码并解析转化为Cssom，即根据Css代码确定每个Dom节点的计算样式（computed style）。

3.JS的加载与执行
Main thread在解析构建Dom过程中，如果遇到script标签，渲染进程会停止对Html的解析，而去加载并执行JS代码。
原因在于JS代码可能会改变Dom的结构。
不过开发者可以通过在script标签中添加async或defer等属性，来让浏览器异步加载和执行JS代码，而不会阻塞渲染。

4.布局（Layout）
Dom Tree和计算样式完成后，我们还需要知道每一个节点在页面上的位置，布局其实就是确认所有节点的几何关系的过程。
Main thread会遍历Dom及相关元素的计算样式，构建出每个节点的页面坐标信息和盒子模型大小的布局树（Render tree），遍历过程中，会跳过隐藏的元素，另外，伪元素虽然在Dom Tree上不可见，但是在Render Tree上是可见的。

5.绘制（Paint）
经过Layout之后，我们目前知道了所有元素的结构、样式和几何关系，我们要绘制出一个页面，我们还需要知道每个元素的绘制的先后顺序。
在绘制阶段，Main thread会遍历Render Tree，生成一系列的绘画记录（paint records）。绘画记录可以看做是记录各元素绘制先后顺序的笔记。

6.合成（Compositing）
经过Paint之后，我们目前有了元素结构、元素样式、元素的几何关系和绘画顺序信息。这个时候要绘制一个页面，需要把上述信息转化为显示器中的像素，这个转化过程叫做光栅化（rasterizing）。

我们要绘制一个页面，最简单的做法是只光栅化视口（viewport）内的网页内容，如果用户进行了页面滚动，就移动光栅帧（rastered frame）并且光栅化更多的内容以补上页面缺失的部分。Chrome的第一个版本就是采用这种简单的绘制方式，这一方式的唯一缺点就是每当页面滚动，光栅线程需要对新移进视口的内容进行光栅化，这是一定的性能损耗，为了优化这种情况，Chrome采取一种更加复杂的做法，即合成。

合成，是一种将页面分成若干层，然后分别对它们进行光栅化，最后在一个单独的线程，即合成线程（Compositor thread）里面合并成一个页面的技术。当用户滚动页面时，由于页面各个层都已经被光栅化了，浏览器需要做的只是合成一个新的帧来展示滚动后的效果罢了。

为了实现合成技术，浏览器需要对元素进行分层，确定哪些元素放置在哪一层，Main thread需要遍历Render Tree来创建一个层次树Layer Tree，对于添加了will-change Css属性的元素，会被看做单独的一层，没有will-change Css属性的元素，浏览器会根据情况决定是否要把该元素放置在单独的层。

Layer Tree创建完成，渲染顺序被确定，Main thread会把这些信息通知给Compositor thread，Compositor thread开始对Layer Tree的每一层进行光栅化。有的层可以达到整个页面的大小，所以Compositor thread需要将它们切分为一块又一块的小图块（tiles），之后将这些小图块分别进行发送给Raster thread进行光栅化，结束后Raster thread会将每个图块的光栅结果存在GPU Process的内存中。

为了优化显示体验，Compositor thread可以给不同的Raster thread赋予不同的优先级，将那些在视口中的或者视口附近的层先被光栅化。

当图层上面的图块都被光栅化后，Compositor thread会收集图块上叫做绘画四边形的信息来构建一个合成帧。
绘画四边形：包含图块在内存的位置和涂层合成后该图块在页面的位置之类的信息。
合成帧：代表页面一个帧的内容的绘制四边形的集合。

上述步骤完成后，Compositor thread会通过IPC机制向Browser Process提交一个渲染请求。这些合成帧都会被发送给GPU Process从而展示在屏幕上。如果Compositor thread收到页面滚动的事件，Compositor thread会构建另一个合成帧发送给GPU Process来更新页面。

合成的好处在于这个过程没有涉及到Main thread，所以Compositor thread不需要等待样式的计算以及JS完成执行。

六.浏览器对事件的处理

