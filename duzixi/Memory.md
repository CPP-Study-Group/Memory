# Task02 C++与内存

## C++ 内存区域划分

C++对程序员是完全信任和尊重的，肯让你碰内存的编程语言得有多大方。
现代计算机经过操作系统的处理，物理内存已经被抽象成了逻辑内存，每一个单位的内存都有自身的地址。

内存地址从小到大主要划分区域：

* [stack]栈区：
	控制主体：操作系统（程序员控制不了）
	分配速度：快
	分配大小：小（Windows总共2M）
	分配顺序：连续、地址从大到小
	备注：包含const局部变量

* [heap]堆区/自由存储区：
	控制主体：程序员（向操作系统申请）
	分配速度：慢
	分配大小：大
	分配顺序：遍历空闲内存地址链表，分配不连续且无序（容易碎片化）
	备注：需要程序员自行负责销毁，否则容易内存泄漏

* [static]可读写区/全局区/静态区：存全局变量和静态变量
	控制主体：操作系统

* [comment]文字常量区：常量字符串
	控制主体：操作系统
	
* [code]程序代码区：函数体二进制代码
	控制主体：操作系统

小结：由此可见，C++内存管理通常指堆内存的管理。

## C++ 与内存打交道的方式

1. 获取/查看变量地址：

变量前加&

代码示例：
```C++
    int i1 = 10;
    const int i2 = 20;
    int i3 = 30;
    std::cout << &i1 << " " << &i2 << " " << &i3 << std::endl;

    ClassName objectName;
    std::cout << &objectName << endl;
```

2. 申请堆内存：

new运算符

代码示例：
```C++
	int* pi = new int(100);
	vector<int>* p = new vector<int>();
	ClassName* objectPointer = new ClassName();
```

3. 释放堆内存：

delete运算符

代码示例：
```C++
	delete pi;
```


## 参考资料：

https://www.cnblogs.com/ChenZhongzhou/p/5685537.html

https://www.cnblogs.com/yyxt/p/4268304.html

https://blog.csdn.net/baidu_37964071/article/details/81428139
