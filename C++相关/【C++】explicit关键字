&emsp;&emsp;C++ 中 explicit 关键字用于指定类的构造函数或转换函数为<font color=red>**显式**</font>，即它不能用于<font color=red>**隐式转换**</font>和<font color=red>**复制初始化**</font>。
&emsp;&emsp;**C++ 11前**，声明时不带函数说明符 explicit 的拥有**单个无默认值形参的构造函数**被称作**转换构造函数**。就是可以直接转换，比如 A(int) {} 那么 A a = 1; 是可以自动转换的，C++ 11后，多个无默认参数的可以这么转换了，如下：
```cpp
class A {
public:
	A(int) { }
	A(int, int) { }	// 转换构造函数 (C++11)
};

class B {
public:
	explicit B(int) { }
	explicit B(int, int) { }
};

int main()
{
	A a1 = 1;			// OK：A::A(int)
	A a2(2);			// OK：A::A(int)
	A a3{ 4, 5 };		// OK：A::A(int, int)
	A a4 = { 4, 5 };	// OK：A::A(int, int)
	A a5 = static_cast<A>(1);   // OK：显式转型

//  B b1 = 1;			// Error!
	B b2(2);			// OK： B::B(int)
	B b3{ 4, 5 };		// OK： B::B(int, int)
//  B b4 = {4, 5};		// Error!
	B b5 = static_cast<B>(1);   // OK：显式转型
}
```
&emsp;&emsp;可以看到，explicit 的作用就是不让进行隐式转换来进行初始化，C++ 11中很典型的例子就是智能指针的初始化，如下：
```cpp
shared_ptr<string> p_nico(new string("nico"));	// OK
shared_ptr<string> p_bob = new string("bob"));	// Error!!
```

<div align=center><img src="https://img-blog.csdnimg.cn/20190826151702861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "80%"></div>

&emsp;&emsp;跳到 shared_ptr 定义，可以看到单参的构造函数是 explicit 的，所以不能用 = 进行复制初始化。

&emsp;&emsp;google 的 c++ 规范中提到 explicit 的优点是可以**避免不合时宜的类型变换**，**缺点无**。所以 google 约定所有单参数的构造函数都必须是显示的，只有极少数情况下拷贝构造函数可以不声明称 explicit。

&emsp;&emsp;effective c++ 中提到：被声明为 explicit 的构造函数通常比其 non-explicit 兄弟更受欢迎。因为它们**禁止编译器执行非预期（往往也不被期望）的类型转换**。除非我有一个好理由允许构造函数被用于隐式类型转换，否则我会把它声明为explicit，鼓励大家遵循相同的政策。
