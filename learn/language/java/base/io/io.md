#I/O
*   提供数据的数据源  =》 接受数据的地方
*   在 Java 编程中，直到最近一直使用 流 的方式完成 I/O。所有 I/O 都被视为单个的字节的移动，通过一个称为 Stream 的对象一次移动一个字节byte。流 I/O 用于与外部世界接触。它也在内部使用，用于将对象转换为字节，然后再转换回对象。
*   流与块的比较
原来的 I/O 库(在 java.io.*中) 与 NIO 最重要的区别是数据打包和传输的方式。正如前面提到的，原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。


#5中I/O模型
*   阻塞式I/O </br>
![](http://img1.tbcdn.cn/L1/461/1/af9e36726347d0c99fea4c1b33bed19e72268c8d?spm=5176.100239.blogcont2371.6.VAXG9X)
*   非阻塞式I/O </br>
![](http://img1.tbcdn.cn/L1/461/1/3af074b0032d5dbe21418b1c250726b95f495bf7?spm=5176.100239.blogcont2371.7.VAXG9X)
*    I/O复用（Java NIO就是这种模型） </br>
![](http://img3.tbcdn.cn/L1/461/1/826b5df93974b1fc1269e92e815760e3817a2c50?spm=5176.100239.blogcont2371.8.VAXG9X)
*   信号驱动式I/O ,异步I/O </br>
![](http://img2.tbcdn.cn/L1/461/1/2b0758999526065e197f981460f079393e407634?spm=5176.100239.blogcont2371.9.VAXG9X)