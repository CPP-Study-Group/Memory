# Task003: STL内存管理

STL内存由空间适配器(alloc)进行控制管理。
	
包含以下两个：
1. SGI标准空间适配器
	对C++的new和delete进行了简单的封装

2. SGI特殊空间适配器

总得来说，SGI以malloc()和free()完成内存分配与释放。

## C++内存操作的基本流程

对于一个C++的伪类Foo：

```C++
class Foo {...}
Foo* pf = new Foo;
delete pf;
```

存在new和delete两个基本流程。

### new的基本流程

1. 分配内存空间
2. 调用Foo构造函数构造内容

### delete的基本流程

1. 调用析构函数
2. 释放空间

可以看到，内存空间控制与内容控制是分开进行的。

## STL内存控制流程

STL内存控制流程参考了C++的内存操作基本流程，将内存空间分配释放与内容的构造和析构分开在不同的函数中实现。

### 分配内存函数

```C++
alloc::allocate() 
```

### 释放内存函数

```C++
alloc::deallocate() 
```

### 对象构造函数

```C++
alloc::construct()
```

construct()本质上是对构造函数T()的封装，源码如下：


```C++
template <class T1, class T2>
inline void construct(T1* p, const T2& value){
  new (p) T1(value); // placement new，即operator new 的重载版本，只调用对象的构造函数，并不进行内存的分配.
}
```

### 对象析构函数

```C++
alloc::destroy()
```

destroy()本质上是对析构函数~T()的封装，源码如下：

```C++
template <class T>
inline void destroy(T* pointer){
    pointer->~T(); // 调用析构函数
}
```

## vector的内存管理机制

C++中的vector模板可以看成是一个动态数组。

在添加元素的过程中，其预留空间变化规律如下：

* size() 元素个数
* capacity() 预留空间

1. 初始化时size()恒等于capacity()
2. 首次调用push_back()后，因为没有多余的空间存放添加的元素，此时将内存加倍。具体来说，若添加之前的size()等于n，则调用push_back()后，capacity()变成2n。
3. 以后再调用push_back()，若空间够用，则直接添加，当不够用时，再次加倍空间。
4. 调用insert()情况类似，但注意确保迭代器有效。

测试代码：

```C++
#include 
#include 
using namespace std;
using std::vector;

int main()
{
    vector ivec;
    vector ivec1(3);
    cout << "空vector" << endl;
    cout << "size:" << ivec.size() << endl;
    cout << "capacity:"<< ivec.capacity() << endl;
    
    cout << "非空vector" << endl;
    cout << "size:" << ivec1.size() << endl;
    cout << "capacity:" << ivec1.capacity() << endl;
  
    cout << "****push_back()*****" << endl;
    for (int i = 1; i != 20; i++)
  	{
  		ivec.push_back(i);
  		cout << i << "\t" << "size:" << ivec.size() << "\t"
            << "capacity:" << ivec.capacity() << endl;
  	} 
  	
  	
    cout << "****insert()*****" << endl;
    vector::iterator iter = ivec1.begin();
    for(vector::size_type j = 0; j != 20; j++)
    {
    	ivec1.insert(iter, j);
    	cout << j << "\t" << "size:" << ivec1.size() << "\t"
            << "capacity:" << ivec1.capacity() << endl;
    	iter = ivec1.begin();
    }
    return 0;
}
```

## 注意事项

如果容器中保存了对象指针，则要在清除容器前手动删除指针，否则就会内存泄露。

```C++
class Unit {

public:
	Unit();
	Unit(int id);
	~Unit();

private:
	int id = -1;

};

Unit::Unit() {
}

Unit::Unit(int _id) : id(_id) {
	printf("Unit construction. id=%d\n", id);
}

Unit::~Unit() {
	printf("Unit destruction. id=%d\n", id);
}

std::vector<Unit*> units;

units.reserve(100);

for (int i = 0; i < 3; ++i) {
	units.push_back(new Unit(i));
}

for (Unit* ptr:units) {
	delete ptr;
	ptr = nullptr;
}
```

## 个人经验谈
因为STL对内存管理做了很好的封装，所以对于搞不定的复杂内存问题可以借用STL来解决。
例如，将对象压进vector模板，通过逻辑控制使vector数组中有且只有一个元素。
该处理方法虽然有些dirty，但是简单方便，安全稳定，百试不爽。

## 参考资料
www.cnblogs.com/zpjjtzzq/p/4540545.html
https://blog.csdn.net/w_guichao1/article/details/16807673
作者：澄澈水光 来源：CSDN 
https://blog.csdn.net/how0723/article/details/82959484
