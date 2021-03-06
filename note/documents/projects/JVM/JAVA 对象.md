## Java对象相关

#### 1 对象的创建流程

![20220428191621-image.png](https://raw.githubusercontent.com/1635269861/note-image/main/image/20220428191621-image.png)

#### 2 对象的内存布局

    在`HotSpot`虚拟机中， 对象在堆内存中的存储布局可以划分成三部分：`对象头（Header）`、`实例数据`、`对齐填充`。

> **疑问1：一个空对象在内存中占用多少字节？**
> 
> java的一个空对象在内存中占用的内存是8字节。一个空对象是只包含对象头的，如果一个对象中没有任何的引用和基本数据类型那么这个对象的大小就只有`MarkWord`是占用内存的，该位置所占用的内存跟虚拟机的位数是密切相关的，`MarkWord`在64位虚拟机中占用的大小是64bit，在32位虚拟机中占用32bit，但是由于对齐填充部分的存在，java对象在虚拟机中的大小必须是8字节的整数倍。所以java一个空对象的大小是8字节。
> 
> **疑问2：java中一个引用指针占用的内存大小是多大？**
> 
> java中一个指针的大小是4字节。
> 
> **疑问3：举例来说明？**
> 
> ![20220429103720-image.png](https://raw.githubusercontent.com/1635269861/note-image/main/image/20220429103720-image.png)
> 
> 上述对象的大小为：8(空对象)+int(4)+byte(1)+String引用(4)=17byte，但是由于对齐填充部分的存在，java对象的大小必须是8字节的整数倍，所以这个java对象的大小是24字节。
