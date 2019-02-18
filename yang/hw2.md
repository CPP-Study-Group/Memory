# Memory 内存

C++与内存

* C++中的栈内存和堆内存的区别  

  数据结构中的堆与栈：    
  栈：是一种连续储存的数据结构，具有先进后出的性质。通常的操作有入栈（圧栈）、出栈和栈顶元素。想要读取栈中的某个元素，就要将其之前的所有元素出栈才能完成。类比现实中的箱子一样。  
  堆：是一种非连续的树形储存数据结构，每个节点有一个值，整棵树是经过排序的。特点是根结点的值最小（或最大），且根结点的两个子树也是一个堆。常用来实现优先队列，存取随意。  

  内存中的栈区与堆区：  
  一般说到内存，指的是计算机的随机储存器（RAM），程序都在这里面运行。计算机内存的大致划分如下图所示：  
  ![Image text](https://images2015.cnblogs.com/blog/928019/201607/928019-20160719170222107-1820485296.png)  
  栈内存：由程序自动向操作系统申请分配以及回收，速度快，使用方便，但程序员无法控制。若分配失败，则提示栈溢出错误。注意，const局部变量也储存在栈区内，栈区向地址减小的方向增长。  
  ```C  
  //测试堆内存和栈内存的区别
  #include <iostream>
  int main()
  {
      int i = 10; //变量i储存在栈区中
      char pc[] = "hello!"; //储存在栈区
      const double cd = 99.2; //储存在栈区
      static long si = 99; //si储存在可读写区，专门用来储存全局变量和静态变量的内存
      int* pi = new int(100); //指针pi指向的内存是在 堆区，专门储存程序运行时分配的内存
      std::cout << &i << " " << &pc << " " << &cd << " " << &si << " " << pi << std::endl;
      delete pi; //需程序员自己释放
      return 0;
  }
  ```   
  测试输出为：  
  ![Image text](https://images2015.cnblogs.com/blog/928019/201607/928019-20160719170537091-233465737.png)  
  运行多次后会发现pi所指向的地址并不连续，是跳跃式的；而&si是一致的，储存在可读写区；前三个变量都储存在栈区，由程序自动分配和销毁。 
  
* 内存的申请与释放  
  申请和释放某一个类型的内存方法：
  ```C
  int *p = new int;  
  delete p;  
  ```
  申请块内存的方法
  ```C
  int *arr = new int[10];
  delete []arr;
  ```
  判断内存是否申请成功
  ```C
  int *p =new int [100000];
  if(NULL == p)
  {
      cout<<"内存申请失败"<<endl;
  }  
  ```
  
* C++中内存泄漏的几种情况  
  
  * 在类的构造函数和析构函数中没有匹配的调用new和delete函数
    两种情况下会出现这种内存泄露：
      一是在堆里创建了对象占用了内存，但是没有显示地释放对象占用的内存；二是在类的构造函数中动态的分配了内存，但是在析构函数中没有释放内存或者没有正确的释放内存。  

  * 没有正确地清除嵌套的对象指针。

  * 在释放对象数组时在delete中没有使用方括号
    方括号是告诉编译器这个指针指向的是一个对象数组，同时也告诉编译器正确的对象地址值并调用对象的析构函数，如果没有方括号，那么这个指针就被默认为只指向一个对象，对象数组中的其他对象的析构函数就不会被调用，结果造成了内存泄露。如果在方括号中间放了一个比对象数组大小还大的数字，那么编译器就会调用无效对象（内存溢出）的析构函数，会造成堆的奔溃。如果方括号中间的数字值比对象数组的大小小的话，编译器就不能调用足够多个析构函数，结果会造成内存泄露。
    释放单个对象、单个基本数据类型的变量或者是基本数据类型的数组不需要大小参数，释放定义了析构函数的对象数组才需要大小参数。

  * 指向对象的指针数组不等同于对象数组
    对象数组是指：数组中存放的是对象，只需要delete []p，即可调用对象数组中的每个对象的析构函数释放空间。  
    指向对象的指针数组是指：数组中存放的是指向对象的指针，不仅要释放每个对象的空间，还要释放每个指针的空间，delete []p只是释放了每个指针，但是并没有 释放对象的空间，正确的做法，是通过一个循环，将每个对象释放了，然后再把指针释放了。
  
* 深拷贝与浅拷贝  

  浅拷贝是增加了一个指针，指向原来已经存在的内存，在多个对象指向一块空间的时候，释放一个空间会导致其他对象所使用的空间也被释放了，再次释放便会出现错误。而深拷贝是增加了一个指针，并新开辟了一块空间让指针指向这块新开辟的空间。
  
* STL模板类

  * 序列式容器（Sequence containers），每个元素都有固定位置-取决于插入时机和地点，和元素值无关。  
  
      Vectors：将元素置于一个动态数组中加以管理，可以随机存取元素（用索引直接存取），数组尾部添加或移除元素非常快速。但是在中部或头部安插元素比较费时。为了支持快速随机访问，vector只能将元素连续存储——每个元素紧挨着前一个元素存储。然而当我们向vector 或者string添加元素的时候；如果没有空间容纳新元素，容器不可能简单地把它添加在内存的其他位置——因为元素必须连续存储。容器就必须重新分配新的内存空间来保存已有的元素和新元素，将旧元素移动到新空间，然后添加新的元素。  
      
      Deques：是“double-ended queue”的缩写，可以随机存取元素（用索引直接存取），数组头部和尾部添加或移除元素都非常快速。但是在中部或头部安插元素比较费时；  
      
      Lists：双向链表，不提供随机存取（按顺序走到需存取的元素，O(n)），在任何位置上执行插入或删除动作都非常迅速，内部只需调整一下指针； 
      
  * 关联式容器（Associated containers），元素位置取决于特定的排序准则，和插入顺序无关。  
  
      Sets/Multisets：内部的元素依据其值自动排序，Set内的相同数值的元素只能出现一次，Multisets内可包含多个数值相同的元素，内部由二叉树实现（实际上基于红黑树(RB-tree）实现），便于查找；  
      
      Maps/Multimaps：map是一类关联式容器，它是模板类。关联的本质在于元素的值与某个特定的键相关联，而并非通过元素在数组中的位置类获取。它的特点是增加和删除节点对迭代器的影响很小，除了操作节点，对其他的节点都没有什么影响。对于迭代器来说，不可以修改键值，只能修改其对应的实值。Map的元素是成对的键值/实值，内部的元素依据其值自动排序，Map内的相同数值的元素只能出现一次，Multimaps内可包含多个数值相同的元素，内部由二叉树实现（实际上基于红黑树(RB-tree）实现），便于查找； 
      
  另外有其他容器hash_map,hash_set,hash_multiset,hash_multimap。  
    
* 类成员的内存  

  1.空类所占字节数为1，可见代码如下

    ```C
    #include <iostream>
    using namespace std;

    class Parent
    {

    };

    class Child:public Parent
    {
      public:
          int b ;
    };

    int main(int argc, char* argv[])
    {
        Child b;
        Parent a;

        cout << "a.sizeof = " << sizeof(a) << endl;
        cout << "b.sizeof = " << sizeof(b) << endl;

        system("pause");
        return 0;
    }
    ```
    
    打印结果为：  
    
    ![Image text](https://img2018.cnblogs.com/blog/1203656/201809/1203656-20180913201140902-1758645312.png)  

    分析：

      为了能够区分不同的对象，一个空类在内存中只占一个字节；

      在子类继承父类后，如果子类仍然是空类，则子类也在内存中指针一个字节；

      如果子类不是空类，则按照成员变量所占字节大小计算。



  2.类中的成员函数不占内存空间，虚函数除外；

    ```C
    #include <iostream>
    using namespace std;

    class Parent
    {
      public:
          void func() {};
          void func1() { int a; };
          void func2() { int b; };
    };

    class Child:public Parent
    {
      public:
          int b ;
    };

    int main(int argc, char* argv[])
    {
        Child b;
        Parent a;

        cout << "a.sizeof = " << sizeof(a) << endl;
        cout << "b.sizeof = " << sizeof(b) << endl;

        system("pause");
        return 0;
    }
    ```
    
    输出结果如下： 
    
    ![Image text](https://img2018.cnblogs.com/blog/1203656/201809/1203656-20180913201915748-713319535.png)  

    分析：上述代码中父类，在内存中仍然只占有一个字节；原因就是因为函数在内存中不占字节；但是，如果父类中如果有一个虚函数，则类所占字节发生变化，如果是32位编译器，则占内存四个字节；  

    ```C
    #include <iostream>
    using namespace std;

    class Parent
    {
      public:
          virtual void func() {};
          virtual void func1() { int a; };
          void func2() { int b; };
    };

    class Child:public Parent
    {
      public:
          int b ;
      };

    int main(int argc, char* argv[])
    {
        Child b;
        Parent a;

        cout << "a.sizeof = " << sizeof(a) << endl;
        cout << "b.sizeof = " << sizeof(b) << endl;

        system("pause");
        return 0;
    }
    ```
    输出结果：  
    
    ![Image text](https://img2018.cnblogs.com/blog/1203656/201809/1203656-20180913202223640-787940856.png)  


    分析：

      通过上述代码可见，编译器为32时，无论几个虚函数所占的字节数都为4；

      而子类在内存中占的字节数为父类所占字节数+自身成员所占的字节数；



  3.和结构体一样，类中自身带有四字节对齐功能

    ```C
    #include <iostream>
    using namespace std;

    class Parent
    {
      public:
          char a;
          virtual void func() {};
          virtual void func1() { int a; };
          void func2() { int b; };
    };

    class Child:public Parent
    {
      public:
          char c;
          int b ;
    };

    int main(int argc, char* argv[])
    {
        Child b;
        Parent a;

        cout << "a.sizeof = " << sizeof(a) << endl;
        cout << "b.sizeof = " << sizeof(b) << endl;

        system("pause");
        return 0;
    }
    ```
    输出结果：  
    
    ![Image text](https://img2018.cnblogs.com/blog/1203656/201809/1203656-20180913203112039-489083589.png)


    分析：

      Parent类中，char a;占一个字节，虚函数占有四个字节，由于类的字节对齐，所以总共父类占有8个字节；

      子类中，char c 占有一个字节，int 占四个字节，由于字节对齐，本身共占有8字节，再加上父类的8字节，共占有16字节；

  4.类中的static静态成员变量不占内存，静态成员变量存储在静态区

    ```C
    #include <iostream>
    using namespace std;

    class G
    {
      public:
          static int a;
    };

    int main(int argc, char * argv[]) 
    {
        cout << sizeof(G)<<endl;

        system("pause");
        return 0;
    }
    ```
    结果输出：   
    
    ![Image text](https://img2018.cnblogs.com/blog/1203656/201809/1203656-20180914184955507-133793502.png)  


  总结：

    1.空类必须占一个字节；

    2.函数指针不占字节；

    3.虚函数根据编译器位数，占相应字节；

    4.类具有4字节对齐功能；

    5.类中的静态成员变量不占类的内存；并且静态成员变量的初始化必须在类外初始化；
  


