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
对于一个类的构造函数，如果没有显式的声明构造函数的话，那么编译器会自动的生成一个参数列表为空的构造函数。
```
    class classA
    {
    public :
    private:
        int m_var1 = 0;
    };
    int main()
    {
        classA a = classA();
    }
    //这段代码编译通过
```
但如果我们定义了一个带有参数的，或者拷贝赋值的构造函数的话，编译器就不会再帮我们自动生成无参的构造函数，这个时候下面这段代码就会报错：
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