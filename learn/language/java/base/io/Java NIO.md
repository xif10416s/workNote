#JAVA NIO
##[并发编程网 - ifeve.com](http://ifeve.com/java-nio-vs-io/)
##[NIO 入门](http://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)
##[理解Java NIO](https://yq.aliyun.com/articles/2371)

###5中I/O模型
*   阻塞式I/O </br>
![](http://img1.tbcdn.cn/L1/461/1/af9e36726347d0c99fea4c1b33bed19e72268c8d?spm=5176.100239.blogcont2371.6.VAXG9X)
*   非阻塞式I/O </br>
![](http://img1.tbcdn.cn/L1/461/1/3af074b0032d5dbe21418b1c250726b95f495bf7?spm=5176.100239.blogcont2371.7.VAXG9X)
*    I/O复用（Java NIO就是这种模型） </br>
![](http://img3.tbcdn.cn/L1/461/1/826b5df93974b1fc1269e92e815760e3817a2c50?spm=5176.100239.blogcont2371.8.VAXG9X)
*   信号驱动式I/O ,异步I/O </br>
![](http://img2.tbcdn.cn/L1/461/1/2b0758999526065e197f981460f079393e407634?spm=5176.100239.blogcont2371.9.VAXG9X)

##主要组件
*   Channels
    -    连接支持非阻塞IO的读/写的通道
*   Buffers
    -   Channels通道直接用来进行读/写操作的类数组对象
*   Selectors 
    -   能知道哪些Channels通道集合存在IO事件
*   SelectionKeys
    -   提供IO事件状态信息和IO事件绑定功能的类


