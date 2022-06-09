## JAVA NIO

### 1 IO类型

#### 1.1 阻塞IO

    通常在进行同步IO操作的时候，如果读取数据，代码会阻塞直至有可供读取的数据。

#### 1.2 非阻塞IO

    NIO中非阻塞IO采用了基于Reactor模式的工作方式，I/O调用不会被阻塞，相反的是注册感兴趣的待定IO时间，如数据可读数据到达，新的套接字链接等等，在发生特定的事件的时候，系统再通知我们。NIO中实现非阻塞IO的核心对象就是`Selector`。`Selector`会帮助我们监听多个`Channel`通道，当有数据可读或者可写的时候，`Selector`就会通知我们，我们就可以遍历`SelectKey`获取到相应的事件。

| IO                | NIO           |
| ----------------- | ------------- |
| 面向流(Stram)        | 面向缓冲区（Buffer） |
| 阻塞IO（Blocking IO） | 非阻塞IO         |
| 无                 | 选择器           |

#### 1.3 NIO概述

JAVA NIO由以下几个核心部分组成

- Channel

- Buffers

- Selectors

##### 1.3.1 Channel 通道

     `Channel `可以翻译成通道。`Channel`和`IO`中的`Stream（流）`是差不多的概念。只不过`Stream`是单向的，既可以用来读操作，又可以用来进行写操作。

    `NIO`中的Channel的主要实现有`FileChannel`、`DatagramChannel`、`SocketChannel`和`ServerSocketChannel`，分别对应文件IO、UDP和TCP（CLient和Server）。

##### 1.3.2 Buffer

    `NIO`中的关键`Buffer`实现有`ByteBuffer`，`CharBuffer`,`DoubleBuffer`，`FloatBuffer`,`IntBuffer`,`LongBuffer`,`ShortBuffer`，分别对应相应的基本数据类型。

##### 1.3.3 Selector

    `Selector`运行单线程处理多个`Channel`，如果你的应用打开了多个通道，但是每个连接的流量都很低，使用Selector就会很好的监视这些`Channel`。

### 2 java NIO（Channel）




