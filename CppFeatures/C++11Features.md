# C++11 Features

## 01 auto & decltype
auto 和 decltype用于自动推导类型,区别在于auto只能用于对变量的类型推导，而decltype用于实体或者表达式的类型推导，例如
```#
    auto var1 = 10;     //int 
    auto var2 = 10.0;   //double
    auto var3 = &var1;  //int *

    decltype(var1) var4 = 1;    // var4 is int
    decltype(10.8) var5 = 5.5;  // var5 is double
    decltype(var5 + 100) var6;  // var6 is double

```
某些情况下，我们不需要知道某个函数的返回值，或者返回值有可能发生变动的情况下，可以使用decltype来表示，如
````
    decltype(f()) a = f();
````
auto可以实现类似迭代器的功能，如
````
    map<int,string> map1;
    for(auto &item:map1 )
    {
        .....
    }
````
首先注意应当写为 auto &item 而不是 auto item，auto item值传递取到的是map1中元素的拷贝，如果需要对元素进行修改的话，应当获取引用，否则这样获取的item是没有意义的。
另外相对于迭代器，这种方式只能实现正向的遍历。
## 02 default & deleted
对于一个类，有这样几种特殊成员函数：默认构造函数、析构函数、复制构造函数和复制赋值运算符 ，以及移动构造函数和移动赋值运算符。在大部分情况下，编译器会自动的生成这几种函数，这时可以称为默认特殊成员函数：
```
    class classA
    {
    public :
    private:
        int m_var1 = 0;
    };
    int main()
    {
        classA a = classA();//调用默认的构造函数和复制赋值操作。
    }
    //这段代码编译通过
```
对于默认构造函数，只有在没有声明其他任何构造函数的情况下，才会自动的生成，否则就需要开发者去手工定义无参的构造函数，这个时候下面这段代码就会报错：
```
    class classB
    {
    public :
        classB(int input)
        {
            m_var1 = input;
        }
    private:
        int m_var1 = 0;
    };
    int main()
    {
        classB a = classB();
    }
    //这段代码会提示无法找到函数classB::classB()
```
但是自动生成的默认构造函数通常效率是高过手写的，所以我们又希望编译器能自动生成默认的构造函数，就可以使用default关键字，显式的定义默认的无参构造函数，编译器会按照默认方式生成这个函数：
```
    class classB
    {
    public :
        classB() = default;
        classB(const classB& b) = default;
        classB(int input)
        {
            m_var1 = input;
        }
    private:
        int m_var1 = 0;
    };
    int main()
    {
        classB a;
    }
```
另一种情况下，我们可能会希望定义的类，不允许通过拷贝或者赋值的方式初始化，只允许默认的构造函数，那么可以通过使用delete和default来实现对默认函数的控制：
```
    class classB
    {
    public :
        classB() = default;
        classB(const classB& b) = delete;
    private:
        int m_var1 = 0;
    };
    int main()
    {
        classB a ;
        //classB b = classB(); 这里会编译错误 无法引用 函数 "classB::classB(const classB &b)" 

    }
```
## 03 final & override
override是C++ 11中新增加的标识符，用来明确该函数是派生类中重写的，继承自基类的虚函数。
```
    class Base {
        virtual void func() {
            cout << "base" << endl;
        }
    };

    class Derived : public Base {
        void func() override { // 确保func被重写
            cout << "derived" << endl;
        }

        void fu() override { // error，基类没有fu()，不可以被重写

        }
    };
```
final关键字可以用于修饰类，也可以用于修饰类中的虚函数,表示这个类或者函数不能再被继承或者重写，甚至可以用于修饰纯虚函数，只是这样做是没有任何意义的。
```
    class Base {
        virtual void func() final{
            cout << "base" << endl;
        }
    };

    class Derived : public Base {
        void func() override { // func 不可被重写
            cout << "derived" << endl;
        }
    };
```

```
    class Base final {
        virtual void func()  {
            cout << "base" << endl;
        }
    };

    class Derived : public Base {// final 类不可被重写
        void func() override { 
            cout << "derived" << endl;
        }
    };
```
## 04  右值引用
### 04.01 什么是右值
首先要明确以下概念：
a. c++11之前的左值与右值：
最初左值与右值的概念就是来自于赋值的表达式：
```
    a = 1;
```
以赋值符号划分左右，a为左值，1为右值，可以理解为a是在内存中存储的变量，可以取地址，有命名，即为左值；反之不符合该条件的就是右值。
b.C++11中的定义：
C++11中对于右值有了更明确的定义，一个表达式中，可以分为如下几部分：

左值(lvalue):不仅变量为左值，变量组成的表达式(如a+b)同样为左值，并且左值可以作为右值使用。所以对于a=b，a与b都是左值，这个操作是将b的值取出，作为右值赋给a;

纯右值(pure rvalue):可以等同于C++11之前的右值。

将亡右值(xrvalue/expiring rvalue):当我们需要用去初始化或者赋值一个对象A时,被用于初始化的B，如果在初始化或者赋值后就消失，那么就可以将这个B视作将亡值。这种情况通常发生在移动构造或者移动赋值的时候，如:
```
    classA var1 = classA();
    classA var2 = std::move(var1);
```
在上面这段代码中，本身为左值，但 std::move(var1)这个操作会将var1转换为右值。

需要注意的是，如果这个classA没有针对移动构造函数或者移动赋值的操作特殊处理的话。那么这两句代码在执行结束后，我们会看到var1和var2的成员变量都指向了相同的地址，这样的话就引入了一个问题，var1和var2在生命周期结束时，都会对同一个地址进行销毁(BOOM!),所以对于这个容易引起错误理解的函数名，必须要清楚：
```
std::move()只是将左值转换为右值，并不移动任何东西。
```



