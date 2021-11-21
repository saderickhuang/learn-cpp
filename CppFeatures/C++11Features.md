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

#### a. c++11之前的左值与右值：

最初左值与右值的概念就是来自于赋值的表达式：
```
    a = 1;
```
以赋值符号划分左右，a为左值，1为右值，可以理解为a是在内存中存储的变量，可以取地址，有命名，即为左值；反之不符合该条件的就是右值。

#### b.C++11中的定义：

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
std::move()只是将左值转换为右值，并不移动任何东西。本质上move = static_cast,相当于一个静态转换
```
那么，引入右值引用的目的是什么呢？主要是两点：移动语义的实现和完美转发

### 04.02 右值引用的用途：移动语义

首先，需要理解一个概念：深拷贝与浅拷贝。例如我们有一个string A，string在实际的实现中，大致都可以看作是一个指向堆内存的char*。深拷贝即我们在把A拷贝到string B时，开辟相同大小的一段空间，同时将A的内容完全复制一份到B的内存空间中。而浅拷贝顾名思义，就是直接将B的char*指向A的位置，两者指向同一个地址。

很容易想到两者在实际执行时的区别，深拷贝会占用更多的资源，而浅拷贝则很有可能会导致异常：例如A在生命周期结束时销毁，而这时如果不是手动的去销毁并同时销毁B，则会导致B在再次使用或者析构时发生异常，所以这种情况下浅拷贝毫无意义。

那么如果A是一个右值呢？因为我们确定A在这次操作后不再被使用，我们可以直接用B接管A的变量(或者称之为资源)，这样便解决了深拷贝所消耗的开辟内存和复制所消耗的资源。这种操作，便成为移动语义的实现。
在C++11后的STL容器，全部都实现了移动语义：
```
#include <vector>

// 返回一个可能很大的vector
std::vector<int> get_vector(int sz)
{
    std::vector<int> vec(sz, 0); // 全部填0
    return vec;
}

int main()
{
    std::vector<int> vec_red;
    vec_red = get_vector(999); // 返回含有999个0的vector
}
```
上面的代码在C++11之前，会先在getVector中生成一个999长度的vector，然后赋值给vec_red时，会再执行一次vector的拷贝构造函数,显然get_vector返回的vector是右值，这句赋值结束后，这个值也就消失了。

而在C++11之后，这段代码就只会在get_vector中构造一次999长度的vector，然后由vec_red"接管"这个返回值。

对于我们自己定义的类，又如何实现移动语义呢？下面的代码中定义了移动构造函数的实现方式。注意当类中同时包含拷贝构造函数和移动构造函数时，如果使用临时对象初始化当前类的对象，编译器会优先调用移动构造函数来完成此操作。只有当类中没有合适的移动构造函数时，编译器才会退而求其次，调用拷贝构造函数。

```
#include <iostream>
#include <memory>
using namespace std;

class Array_custom
{
public:
	Array_custom() :_element(nullptr), _length(0)
	{
		cout << "start default deconstructor" << endl;
	}
	Array_custom(size_t len)
	{
		cout << "start deconstructor with argument" << endl;
		_length = len;
		_element = new int[len];
	}
	Array_custom(Array_custom&& item)//移动构造函数
	{
		cout << "start move deconstructor" << endl;
		_length = item._length;
		_element = item._element;
		if (item._element != nullptr)
		{
			item._element = nullptr;
			item._length = 0;
		}
	}
	~Array_custom()
	{
		cout << "start deconstructor" << endl;
		cout << "length:"<< _length << endl;
		if (_element != nullptr)
		{
			delete[]_element;
			_length = 0;
		}
		cout << "deconstructor end" << endl;
	}

private:
	int* _element;
	size_t _length;
};

int main()
{
	Array_custom a(10);
	Array_custom b(std::move(a));
}
```
以上代码的执行结果为:
```
start deconstructor with argument
start move deconstructor
start deconstructor
length:10
deconstructor end
start deconstructor
length:0
deconstructor end
```

### 04.03 右值引用的用途：完美转发

所谓的完美转发是指，在模板函数中使用输入的函数传递给函数中调用的其他函数时，能够正确的区分左右值，如模板函数传入的是右值，那么调用的子函数输入的也应是右值。
C++11之前并非不能实现这一点，我们知道定义模板时，const修饰的参数只能接收右值，或者说常量，非const修饰的参数只能接收左值。
所以如果我们有个模板函数TemplateFunc，在C++11之前如果要实现完美转发需要写出如下代码。
```
void func_Called(int &a)
{

}
void func_Called(const int &a)
{

}
template <Typename T>
void TemplateFunc(T& t)
{
    func_Called(t);
}
template <Typename T>
void TemplateFunc(Const T& t)
{
    func_Called(t);
}
```
利用重载，对参数为左值和右值的情况分别实现一次函数，显然这种情况只适用于参数数量极少的情况，否则就需要实现大量的模板函数。
而在C++11中，模板函数允许使用右值引用的类型作为参数：
```

template <Typename T>
void TemplateFunc(T&& t)
{

}

int main()
{
	int a = 10;
	TemplateFunc(a);

    int &b = a;
    TemplateFunc(b);

	TemplateFunc(int(10));
}

```
以上代码中，三种调用方式均可编译通过，那么这几种情况下，参数具体是如何传递的呢？这就涉及另一个概念，叫做引用折叠，根据TemplateFunc(T&& t)传入参数类型的不同，实际情况可能是：

TemplateFunc(a);    ->  T=int&   (左值引用)   ->TemplateFunc(int& && t) ->TemplateFunc(int&t)

TemplateFunc(b);    ->  T=int&  (左值引用)  ->TemplateFunc(int& && t)   ->TemplateFunc(int&t)

TemplateFunc(int(10));    ->  T=int  (右值)  ->TemplateFunc(int&& && t) ->TemplateFunc(int&& t)

对于这个模板来说，参数类型转换后，形成了引用的引用，我们知道在正常的代码里，是不允许这么做的，那么为什么模板函数可以编译通过呢，是因为编译器在处理时引入了一个叫做引用折叠的概念进行转换，来判断传入的到底是左值还是右值并转换为正确的类型，简单来说就是把模板参数的引用类型和T的引用类型结合来看，如果均为右值引用，则转换后的参数为右值，否则为左值。

到这一步，我们解决了模板函数的参数接收左值和右值的问题，现在回到完美转发这个概念上，
```
template<typename T>
void print(T & t){
    std::cout << "Lvalue ref" << std::endl;
}

template<typename T>
void print(T && t){
    std::cout << "Rvalue ref" << std::endl;
}
template <Typename T>
void TemplateFunc(T&& t)
{
    print(t);
    print(std::forward<T>(t))
}
int main()
{
    int a = 10;
    TemplateFunc(a);//左值调用
    cout << "----------------------------" << endl;
    TemplateFunc(std::move(a));//右值调用

}
```
对于上面这段代码，执行结果为：
```
Lvalue ref
Lvalue ref
----------------------------
Lvalue ref
Rvalue ref

```
可以看到在右值调用时，print(t)时，t被看作左值，这时因为在调用时，在函数TemplateFunc中来看，传入的参数t都已经转换为左值了。因此需要使用print(std::forward<T>(t))来继续调用，这样才能够保证调用时，保持参数t的左值右值属性。以上就是完美转发的使用。
所谓右值的概念的引入和在C++11中的细化，主要还是为了提升在拷贝时的效率问题，通过对临时对象的再利用，节省资源提高效率。


## 05 nullptr
nullptr是为了解决二义性的问题引入的，用来表示空指针，因为原先的NULL实际是int类型的0，那么在如下代码编译时，编译器便无法区分实际调用的函数应该是哪个了:
```
void function(void * ptr)
{}
void function(int val)
{}
int main()
{
    function(NULL);
}
```
## 06 enum class
传统的枚举类型，有这些问题：
a.在作用域内均可见，由此带来一个问题就是代码时需要去避免命名的冲突：
```
enum COLOR{RED,BLUE}
enum EMO{HAPPY,BLUE}
```
b.不能指定枚举变量转换的类型，而是会强制转换为int，即使我们不希望这么做

因此C++11引入了枚举类
```
enum class Color:char{RED,BLUE}
//usage:
Color color = Color::RED;
```
## 07 constexpr
使用constexpr修饰的表达式或者函数，编译器会在编译期尝试编译为常量值.
```
constexpr int a = 1 + 2 + 3;
char arr[a]; // 合法，a 是编译期常量

constexpr int foo() 
{
  return 5;
}
```
注意：对于无法在编译器确定返回结果的函数，会退化为普通的运行期函数，而不是报错.

## 08 初始化列表
C++11之前，数组和POD类型的变量可以使用初始化列表的方式来精简代码：
```
int arry[] = {1,2,3,4}
struct LOCATION{
    int x;
    int y;
}
LOCATION loc = {1,2};
```
C++11扩大了初始化列表可以使用的范围：
```
class person{
    person(string name,int height)
    {
        ......
    }
}

person p1 = {"WU DALANG",150};//用于初始化

person getAperson()
{
    return {"WU SONG",188};//用于返回值
}

void viewPerson(person p);//用于普通函数的参数
//calling:
viewPerson({"WU SONG",188});
```

## 09 委托构造函数和继承构造函数
在同一个类中，可能同时存在多个构造函数，那么我们可以在某个构造函数的基础上再去实现另一个构造函数,称为委托构造函数：
```
class PERSON
{
public:
    PERSON() = default;
    PERSON(int age){m_age=age;}
    PERSON(string name,int age):PERSON(age)
    {
        m_name = name;
    }
private:
    string m_name;
    int    m_age;

}
```
C++11之前，在类B需要使用基类A的构造函数的话，必须在B里显式的声明，如果有多个版本的构造函数都想继承下来的话，就需要相应的全部声明一边。

因此在C++11中，允许使用using关键字来继承基类的构造函数：
```
class PERSON
{
public:
    PERSON() = default;
    PERSON(int age){m_age=age;}
    PERSON(string name,int age):PERSON(age)
    {
        m_name = name;
    }
private:
    string m_name;
    int    m_age;

}
class STUDENT:public PERSON
{
public:
using PERSON::PERSON;//一句话即可完成全部构造函数的继承
private:
}
```
但需要注意的是，这种方式使得派生类新添加的变量无法初始化，可以考虑下面两种方式:
```
class STUDENT:public PERSON
{
public:
    using PERSON::PERSON;
private:
    double m_double{0.0};//C++11允许就地初始化
}

class STUDENT:public PERSON
{
public:
    using PERSON::PERSON;
    Derived(string name,int age,double b):PERSON(name,age),m_double(b){}//为派生类定义新的构造函数
private:
    double m_double{0.0};//C++11允许就地初始化
}
```
## 10 long long
其实这个应该视作一个历史遗留问题，即在C++的标准里，long 和int的长度定义目前实际是一样的。因为int在早期的16位系统下，长度为16位，而long则定义为32位，而在目前32和64位的系统下，int类型也成了32位，这就导致long实际并无作用。参考CPPREFERENCE的定义:
```
int - basic integer type. The keyword int may be omitted if any of the modifiers listed below are used. If no length modifiers are present, it’s guaranteed to have a width of at least 16 bits. However, on 32/64 bit systems it is almost exclusively guaranteed to have width of at least 32 bits.
long - target type will have width of at least 32 bits.
```
换句话说，按照标准，long至少要保证32bit长度，而int至少要保证有16位长度，并未要求long的长度一定要大于int，而现在的编译器至少都会给int4个字节(16位的嵌入式系统可能会有例外)，因此我们目前基本可以将两者视作相同的长度。

C++11 为了兼容之前的规定，便引入了long long类型，规定这一类型至少有64 bit的长度:
```
long long - target type will have width of at least 64 bits.(since C++11)
```

因为C/C++代码级跨平台的特性，这一设计引入正式标准也就意味着不会因为编译器或者开发平台的不同而有差异，那么相应的类库也就会逐步的引入这一类型了。

## 11 typdef & using
C++11引入了 using产生类型别名的方法，比如下面的代码就是等价的.
```
typedef long long verylong;
using verylong = long long;
```
那么，为什么还要引入using呢？因为当我们想为模板定义别名时就会发现：
```
template <typename T>
typedef std::vector<T> v;//编译器不允许使用typedef

template <typename T>
using v =  std::vector<T> ;//允许使用using
```

## 12 变长参数模板
在传统 C++ 中，类模板或函数模板只能接受固定数量的模板参数。

⽽ C++11 允许任意多个、任意类别的模板参数，同时在定义时⽆需固定参数个数。如下所⽰：
```
template<typename... T> class DogT;

// 传⼊多个不同类型的模板参数
class DogT<int, 
            std::vector<int>,
            std::map<std::string,
            std::vector<int>>> dogT;

// 不传⼊参数（ 0 个参数）
class DogT<> nothing;

// 第⼀个参数必传，之后为变⻓参数
template<typename require, typename... Args> class CatT;

//同样适用于模板函数
template<typename... Args>
void my_print(const std::string& str, Args... args) {
  // 使⽤ sizeof... 计算参数个数
  std::cout << sizeof...(args) << std::endl;
}
```
## 13 智能指针
### 13.01 auto_ptr
传统的C++(其实是C++03),提供了auto pointer,思路很简单，就是将资源保存在对象里，这样就可以让资源的销毁动作在对象析构中完成，避免开发者人为的去判段和销毁资源。

但同时auto pointer的设计又被认为是失败的，因为在当时还没有拷贝语义的出现，而语言设计者们希望实现资源由单个智能指针独占的设计，因此在实现auto pointer的拷贝构造时，采用了类似我们现在概念中移动构造的理念，即当auto_ptrA 拷贝或者赋值给auto_ptr B时，会同时将A的指针release掉，因此下面这段代码是无法编译通过的：

``` 
int main(int argc, char const *argv[]) {
    std::auto_ptr<int> iptr(new int(0));
    std::vector<std::auto_ptr<int> > integer_vec;

    integer_vec.push_back(iptr);//问题在这里
    return 0;
}
```
因为对于auto_ptr类型的vector进行拷贝构造时，通过push_back传入的iptr会作为常量传入,即 const auto_ptr &iptr，而auto_ptr的拷贝构造不支持常量参数，为什么不支持呢？原因正是上面提到的,auto_ptr在拷贝构造时，需要同时释放掉构造函数传入的参数。用C++11的概念来说，就是auto_ptr只能使用左值进行拷贝构造。

换句话说，auto_ptr使用拷贝的方式实现了移动的操作，既带来的迷惑性，在应用中也有很大的局限。

### 13.02 unique_ptr
unique_ptr是C++11中对auto_ptr原本意图的真正实现，即内存所有权在同⼀个时间内是唯⼀的，但不同于auto_ptr对拷贝语义的魔改，借助 ##04 中提到的移动语义和move操作，来实现unique_ptr之间所有权的转移，并且禁用了常量引用作为参数的构造函数和复制操作。

例如下面的代码：
```
    std::unique_ptr<int> iptr(new int(1));
    std::vector<std::unique_ptr<int> > integer_vec;

    integer_vec.push_back(iptr); //拷贝构造函数已禁用。
    integer_vec.push_back(std::move(iptr));//pass
```
## 13.03 shared_ptr / weak_ptr
shared_ptr允许多个shared_ptr指向同一个资源。指针内部使用引用计数器来判断是否还有指针在引用资源，计数器归零时，资源将被回收。推荐使用make_shared 来创建资源，并返回一个share_ptr指针。

weak_ptr可以指向share_ptr指向的内存，weak_ptr不能够操作*或者->,即不能操作资源本身，只是允许通过lock(),返回一个指向资源的shared_ptr，如果指向的资源已释放，则返回一个nullptr；或者通过expired()返回bool值来判断资源是否已经释放。

weak_ptr的引入一方面为我们提供了非常方便的判断资源是否释放的操作；更重要的是为了解决shared_ptr的引用计数器机制，在互相引用时导致的循环引用的问题:
```
    auto a = std::make_shared<A>();
	auto b = std::make_shared<B>();
	a->pointer = b;
	b->pointer = a;
	std::weak_ptr<A> wp = a;
	a.reset();
	b.reset();
	cout << boolalpha << wp.expired() << endl;//false
	return 0;
```
上面的代码最终会输出false,这是因为a b之间互相用share_ptr指向，导致即使 ab均reset后，引用计数器仍为1，同时又无法获取到该资源，导致内存邪路，解决方法就是将a或者b替换为weak_ptr，这样在离开作用域时，最终只有a或者b还未析构，并且未被智能指针指向，最终被释放掉。
