
```C++
//  Created by 石迎 on 2019/2/14.
//  Copyright © 2019年 石迎. All rights reserved.
//

#include <iostream>
using namespace std;

class person{
public:
    person(){
        cout<<"基类构造函数运行中.....\n";
    }
    ~person(){
        cout<<"基类析构函数运行中.....\n";
    }
};
class DS:public person{
public:
    DS(){
        cout<<"派生类构造函数运行中.....\n";
    }
    ~DS(){
        cout<<"派生类析构函数运行中.....\n";
    }
};

int main(int argc, const char * argv[]) {
    DS p;
    return 0;
}
输出：
基类构造函数运行中.....
派生类构造函数运行中.....
派生类析构函数运行中.....
基类析构函数运行中.....
实例化派生类DS一个对象p 先调用基类构造-派生类构造-派生类析构-基类析构。没有问题 

*********假设new了。却没delete。******
int main(int argc, const char * argv[]) {
    DS *p = new DS();
    return 0;
}
输出：
基类构造函数运行中.....
派生类构造函数运行中.....
此时会造成内存泄漏，因为new了  但是没有delete 程序在执行的过程中不会调用析构函数，直到程序结束才会释放，所以造成了泄漏。
int main(int argc, const char * argv[]) {
    DS *p = new DS();
    delete p;
    return 0;
}
输出：
基类构造函数运行中.....
派生类构造函数运行中.....
派生类析构函数运行中.....
基类析构函数运行中.....

此时正常。

***********基类指针仅仅指向了派生类继承自己的部分，只释放了基类，派生类析构未调用，此时同样会造成内存泄漏************
int main(int argc, const char * argv[]) {
    person *p = new DS();
    delete p;
    return 0;
}
输出：
基类构造函数运行中.....
派生类构造函数运行中.....
基类析构函数运行中.....
解决方案：
在基类的析构函数增加virtual,此时在基类指针调用析构时候，因为析构函数是虚函数，所以会先调用派生类的析构，再去调用基类析构。
class person{
public:
    person(){
        cout<<"基类构造函数运行中.....\n";
    }
    virtual ~person(){
        cout<<"基类析构函数运行中.....\n";
    }
};
输出：
基类构造函数运行中.....
派生类构造函数运行中.....
派生类析构函数运行中.....
基类析构函数运行中.....

****************深拷贝  浅拷贝**************
//  Created by 石迎 on 2019/2/15.
//  Copyright © 2019年 石迎. All rights reserved.
//


#include <iostream>
using namespace std;

class Student
{
private:
    int num;
    char *name;
public:
    Student();
    ~Student();
};

Student::Student()
{
    name = new char(20);
    cout << "Student"<< (long)name << endl;
    
}
Student::~Student()
{
    cout << "~Student " << (long)name << endl;
    delete name;
    name = NULL;
}

int main()
{
    {// 花括号让s1和s2变成局部对象，方便测试
        Student s1;
        Student s2(s1);// 复制对象
    }
    system("pause");
    return 0;
}
输出：
Student4300223056
~Student 4300223056
~Student 4300223056
testcpp(2956,0x10039b380) malloc: *** error for object 0x100503250: pointer being freed was not allocated
*** set a breakpoint in malloc_error_break to debug
报错了，由于未定义拷贝构造函数，所以系统默认进行了浅拷贝。所以 s1和S2指向了同一个内存空间，导致同一个内存空间释放了两次。

解决方案：
#include <iostream>
using namespace std;

class Student
{
private:
    int num;
    char *name;
public:
    Student();
    ~Student();
    Student(const Student &s);//拷贝构造函数
};

Student::Student()
{
    name = new char(20);
    cout << "Student"<< (long)name << endl;
    
}
Student::~Student()
{
    cout << "~Student " << (long)name << endl;
    delete name;
    name = NULL;
}
Student::Student(const Student &s)
{
    name = new char(strlen(s.name));
    memcpy(name, s.name, strlen(s.name));
    cout << "copy " << endl;
}

int main()
{
    {// 花括号让s1和s2变成局部对象，方便测试
        Student s1;
        Student s2(s1);// 复制对象
    }
    system("pause");
    return 0;
}
输出：
Student4299172368
copy 
~Student 4299172384
~Student 4299172368
执行结果调用了一次构造函数调用了一次拷贝函数，两次析构函数 name的内存地址发生了变化，没有指向同一个地址。

所以浅拷贝只是对指针进行拷贝。拷贝后两个指针指向同一个内存地址。而深拷贝不但对指针进行了拷贝，同时也对指针指向的内容进行了拷贝。所以指针指向的地址也不同。
1.当函数的参数为对象时，实参传递给形参的实际上是实参的一个拷贝对象，系统自动通过拷贝构造函数实现；
2.当函数的返回值为一个对象时，该对象实际上是函数内对象的一个拷贝，用于返回函数调用处。
