#Tensorflow 深度学习框架
##[中文文档](http://wiki.jikexueyuan.com/project/tensorflow-zh/tutorials/mnist_beginners.html)

##使用 TensorFlow, 你必须明白 TensorFlow:
*   使用图 (graph) 来表示计算任务.
*   在被称之为 会话 (Session) 的上下文 (context) 中执行图.
*   使用 tensor 表示数据.
*   通过 变量 (Variable) 维护状态.
*   使用 feed 和 fetch 可以为任意的操作(arbitrary operation) 赋值或者从其中获取数据.

##Tensor
*   TensorFlow 程序使用 tensor 数据结构来代表所有的数据, 计算图中, 操作间传递的数据都是 tensor. 你可以把 TensorFlow tensor 看作是一个 n 维的数组或列表. 一个 tensor 包含一个静态类型 rank, 和 一个 shape. 想了解 TensorFlow 是如何处理这些概念的, 参见 Rank, Shape, 和 Type.

##变量
*Variables for more details. 变量维护图执行过程中的状态信息


##名词术语
*   MNIST
    -   MNIST是一个入门级的计算机视觉数据集，它包含各种手写数字图片


##安装
*   sudo pip install --upgrade https://storage.googleapis.com/tensorflow/mac/tensorflow-0.8.0-py2-none-any.whl  --user -U


##例子
http://blog.jobbole.com/110558/
