# CPP

## 虚函数 多态

+ 一个类函数的调用不是在编译时刻被确定的，而是在运行时刻被确定的。
+ 虚函数只能借助指针或者引用来达到多态效果。

```c++
#include <iostream>
using namespace std;
class A
{
public:
    void foo()
    {
        cout << "A::foo() is called" << endl;
    }
};
class B :public A
{
public:
    void foo()
    {
        cout << "B::foo() is called" << endl;
    }
};
int main(void)
{
    A* a = new B();
    a->foo();   // 在这里，a虽然是指向A的指针，但是被调用的函数(foo)却是B的!
    return 0;
}
```

## 纯虚函数

+ 抽象类

  注意Java和C++的区别
+ **主要区别**

  + abstract修饰JAVA类 C++没有。
  + abstract 修饰Java 方法，C++使用virtual。
  + Java抽象方法没有函数体，纯虚函数定义等于0。
  + 非抽象子类都需要重写抽象方法
+ 实例

  + C++ 纯虚函数

  ```c++
  #include <iostream>

  using namespace std;
  class A
  {
  public:
      virtual void foo()=0;
  };
  class B :public A
  {
  public:
      void foo(){
          cout << "B::foo() is called" << endl;
      }
  };
  class C :public A
  {
  public:
      void foo(){
          cout << "C::foo() is called" << endl;
      }
  };
  int main(void)
  {
      A* b = new B();
      A* c = new C(); 
      b->foo();
      c->foo();
      delete b, c;
      return 0;
  }
  ```

  + Java  抽象类 abstract方法

  ```java
  public class Main {
      public static void main(String[] args) {
          Per s=new Stu();
          Per t=new Tea();
          s.say();
          t.say();
      }
  }
  abstract class Per {
      public abstract void say();//抽象方法不能含有函数体
  }
  class Stu extends Per{
      @Override
      public void say() {
          System.out.println("Stu");
      }
  }
  class Tea extends Per{
      @Override
      public void say() {
          System.out.println("Tea");
      }
  }
  ```

## 多继承

+ C++可以多继承。
+ Java类只能单继承，但接口可以实现多继承。

## 友元

+ 类的友元函数是定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。
+ 友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。

```c++
#include <iostream>

using namespace std;

class Box{
    double width;
public:
    friend void printWidth(Box box);
    friend class BigBox;
    void setWidth(double wid);
};
class BigBox{
public:
    void Print(int width, Box &box)
    {
        // BigBox是Box的友元类，它可以直接访问Box类的任何成员
        box.setWidth(width);
        cout << "Width of box : " << box.width << endl;
    }
};
// 成员函数定义
void Box::setWidth(double wid)
{
    width = wid;
}
// 请注意：printWidth() 不是任何类的成员函数
void printWidth(Box box)
{
    /* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */
    cout << "Width of box : " << box.width << endl;
}

// 程序的主函数
int main()
{
    Box box;
    BigBox big;

    // 使用成员函数设置宽度
    box.setWidth(10.0);

    // 使用友元函数输出宽度
    printWidth(box);

    // 使用友元类中的方法设置宽度
    big.Print(20, box);

    getchar();
    return 0;
}
```
