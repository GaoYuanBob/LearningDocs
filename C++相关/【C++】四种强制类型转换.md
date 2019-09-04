## C++ 四种强制类型转换
&emsp;&emsp;C语言中的强制类型转换（Type Cast）有<font color=red>**显式**</font>和<font color=red>**隐式**</font>两种，显式一般就是直接用小括号强制转换，TYPE b = (TYPE)a; 隐式就是直接 float b = 0.5; int a = b; 这样隐式截断（by the way 这样隐式的截断是==向 0 取整的==，我喜欢这么叫因为 0.9 会变成 0，1.9 变成 1，-0.9 变成 0，-1.9 变成 -1）。\
&emsp;&emsp;C++对C兼容，所以上述方式的类型转换是可以的，但是有时候会有问题，所以推荐使用C++中的四个强制类型转换的关键字：\
&emsp;&emsp;1、**<font color=#FF0000 >static_cast</font>**\
&emsp;&emsp;2、**<font color=#FF0000 >const_cast</font>**\
&emsp;&emsp;3、**<font color=#FF0000 >reinterpret_cast</font>**\
&emsp;&emsp;4、**<font color=#FF0000 >dynamic_cast</font>**
### 1）static_cast
&emsp;&emsp;这应该四种中是最常见的。用法为 **<font color=#FF0000 > static_cast < type-id > (expression)</font>**。\
&emsp;&emsp;该运算符把 expression 转换为 type-id 类型，但<font color=red>**没有运行时类型检查来保证转换的安全性**</font>。\
&emsp;&emsp;主要用法如下：\
&emsp;&emsp;&emsp;&emsp;（1）用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换。\
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；\
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;进行下行转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是不安全的。\
&emsp;&emsp;&emsp;&emsp;（2）用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。\
&emsp;&emsp;&emsp;&emsp;（3）把空指针转换成目标类型的空指针。\
&emsp;&emsp;&emsp;&emsp;（4）把任何类型的表达式转换成void类型。\
&emsp;&emsp;最常用的应该还是基本数据类型之间的转换，如下：

```cpp
	const auto a1 = 11;	// int
	const auto a2 = 4;	// int

// C style
	double res1 = (double)(a1) / (double)(a2);	// 其实写一个 (double) 就行
	cout << "res1 = " << res1 << endl;			// res1 = 2.75

// C++ style
	auto res2 = static_cast<double>(a1) / static_cast<double>(a2);
	cout << "res2 = " << res2 << endl;			// res2 = 2.75
	cout << typeid(res2).name() << endl;		// double
```
&emsp;&emsp;如果在 Visual Studio 中编写，且安装了 **Resharper C++** 的话，如果按照 C 风格写的话，会提示你，并帮助你自动修改为 C++ 风格，如下图所示。
<div align=center><img  src="https://img-blog.csdnimg.cn/20190228133229878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70"/>
</div>
&emsp;&emsp;需要注意的是，static_cast 不能转换掉 expression 的 const、volitale 或者 __unaligned 属性，如下图所示。
<div align=center><img src="https://img-blog.csdnimg.cn/20190228182706804.png"/>
</div>

### 2）const_cast
&emsp;&emsp;上边的 static_cast 不能将 const int* 转成 int*，const_cast 就可以，用法为 **<font color=#FF0000 > const_cast < type-id > (expression)</font>**。如下面代码
```cpp
	const int a = 10;
	const int * p = &a;
	*p = 20;						 // Compile error: Cannot assign readonly type 'int const'
	int res1 = const_cast<int>(a);   // Compile error: Cannot cast from 'int' to 'int' via const_cast
									 // only conversions to reference or pointer types are allowed
	int* res2 = const_cast<int*>(p); // ok
```
&emsp;&emsp;也就是说，==const_cast <>里边的内容必须是引用或者指针==，就连把 int 转成 int 都不行。\
&emsp;&emsp;对于 const_cast 的使用，其实还有很多不明白的（C++未定义的行为），比如下面三张图。
<div align=center><img src="https://img-blog.csdnimg.cn/20190228194454195.png"/>
</div>
<div align=center><img src="https://img-blog.csdnimg.cn/20190228194503518.png"/>
</div>
<div align=center><img src="https://img-blog.csdnimg.cn/20190228194512888.png"/>
</div>

&emsp;&emsp;更神奇的是，==在 VS2017 中，debug 模式下，执行完 *p = 20; 这一步后，a 的值就变成了 20，但是输出 a 和 *p 还都是 11==。我理解的就是 const int 类型的变量，在编译器就优化了，里边所有单独的 a 都已经变成了 11，所以怎么修改都不影响了，这种情况包括：1、直接用 11 这种常量来初始化 a，2、用同样为 const int 类型的 c 来初始化 a。如果是用 int c = 11; 这样的 c 来初始化 a，那 a 就是可变的？？
<div align=center><img src="https://img-blog.csdnimg.cn/20190228195649210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70"/>
</div>

&emsp;&emsp;总结来说，const_cast 通常是无奈之举，只是 C++ 提供了一种修改 const 变量的方式，但这种方式并没有什么实质性的用处，还是不用的好。const 的变量不要让它变。
### 3）reinterpret_cast
&emsp;&emsp;reinterpret_cast 主要有三种强制转换用途： \
&emsp;&emsp;**1、改变指针或引用的类型\
&emsp;&emsp;2、将指针或引用转换为一个足够长度的整形\
&emsp;&emsp;3、将整型转换为指针或引用类型**。
&emsp;&emsp;用法为 **<font color=#FF0000 > reinterpret_cast < type-id > (expression)</font>**。
&emsp;&emsp;type-id 必须是一个指针、引用、算术类型、函数针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。
&emsp;&emsp;我们映射到的类型仅仅是为了故弄玄虚和其他目的，这是所有映射中最危险的。(这句话是C++编程思想中的原话)。因此, 你需要谨慎使用 reinterpret_cast。
### 4）dynamic_cast
&emsp;&emsp;用法为 **<font color=#FF0000 > dynamic_cast < type-id > (expression)</font>**。
&emsp;&emsp;几个特点如下：\
&emsp;&emsp;（1）==其他三种都是编译时完成的==，dynamic_cast 是运行时处理的，运行时要进行类型检查。\
&emsp;&emsp;（2）不能用于内置的基本数据类型的强制转换\
&emsp;&emsp;（3）dynamic_cast 要求 <> 内所描述的目标类型必须为指针或引用。dynamic_cast 转换如果成功的话返回的是指向类的指针或引用，转换失败的话则会返回 nullptr\
&emsp;&emsp;（4）在类的转换时，在类层次间进行==上行转换==（子类指针指向父类指针）时，dynamic_cast 和 static_cast 的效果是一样的。在进行==下行转换==（父类指针转化为子类指针）时，dynamic_cast 具有类型检查的功能，比 static_cast 更安全。 向下转换的成功与否还与将要转换的类型有关，即要转换的指针指向的对象的实际类型与转换以后的对象类型一定要相同，否则转换失败。在C++中，编译期的类型转换有可能会在运行时出现错误，特别是涉及到类对象的指针或引用操作时，更容易产生错误。Dynamic_cast操作符则可以在运行期对可能产生问题的类型转换进行测试。\
&emsp;&emsp;（5）使用 dynamic_cast 进行转换的，**<font color=#FF0000 >基类中一定要有虚函数</font>**，否则编译不通过（类中存在虚函数，就说明它有想要让基类指针或引用指向派生类对象的情况，此时转换才有意义）。这是由于运行时类型检查需要==运行时类型信息==，而这个信息存储在类的==虚函数表中==，只有定义了虚函数的类才有虚函数表（C++中的虚函数基本原理这篇文章写得不错，https://blog.csdn.net/xiejingfa/article/details/50454819）。
```cpp
class base {
public:
	void print1() { cout << "in class base" << endl; }
};

class derived : public base {
public:
	void print2() { cout << "in class derived" << endl; }
};

int main() {
	derived *p, *q;
	// p = new base;	//  Compilr Error: 无法从 "base * " 转换为 "derived * "

	// Compile Error: Cannot cast from 'base*' to 'derived*' via dynamic_cast: expression type is not polymorphic(多态的)
	// p = dynamic_cast<derived *>(new base);

	q = static_cast<derived*>(new base);	// ok, but not recommended

	q->print1();	// in class base
	q->print2();	// in class derived
}
```
&emsp;&emsp;从上边的代码可以看出==用一个派生类的指针是不能直接指向一个基类的对象的==，会出现编译错误。用 dynamic_cast 的话也会编译错误，提示我们基类不是多态的，也就是基类中没有虚函数。可以看到 static_cast 是可以编译通过的，且输出结果看起来都是对的，但是 VS 还是会提示说 Do not use static_cast to downcast from a base to a derived class，如下图所示。
<div align=center><img src="https://img-blog.csdnimg.cn/20190228203528655.png"/>
</div>

&emsp;&emsp;static_cast 强制类型转换时并不具有保证类型安全的功能，而 C++ 提供的 dynamic_cast 却能解决这一问题，dynamic_cast 可以在程序运行时检测类型转换是否类型安全。当然 dynamic_cast 使用起来也是有条件的，它要求所转换的 expression 必须包含==多态类类型==（即至少包含一个虚函数的类）。
```cpp
class A {
public:
	virtual void print(){
		cout << "in class A" << endl;
	};
};

class B :public A {
public:
	void print(){
		cout << "in class B" << endl;
	};
};

class C {
	void pp(){
		return;
	}
};

int main() {
	A *a1 = new B; // a1是A类型的指针指向一个B类型的对象
	A *a2 = new A; // a2是A类型的指针指向一个A类型的对象
	B *b1, *b2, *b3, *b4;
	C *c1, c2;
	b1 = dynamic_cast<B*>(a1);	// not null，向下转换成功，a1 之前指向的就是 B 类型的对象，所以可以转换成 B 类型的指针。
	if (b1 == nullptr) cout << "b1 is null" << endl;
	else               cout << "b1 is not null" << endl;

	b2 = dynamic_cast<B*>(a2);	// null，向下转换失败
	if (b2 == nullptr) cout << "b2 is null" << endl;
	else               cout << "b2 is not null" << endl;

	// 用 static_cast，Resharper C++ 会提示修改为 dynamic_cast
	b3 = static_cast<B*>(a1);	// not null
	if (b3 == nullptr) cout << "b3 is null" << endl;
	else               cout << "b3 is not null" << endl;

	b4 = static_cast<B*>(a2);	// not null
	if (b4 == nullptr) cout << "b4 is null" << endl;
	else               cout << "b4 is not null" << endl;

	a1->print();	// in class B
	a2->print();	// in class A
	
	b1->print();	// in class B
	// b2->print(); // null 引发异常
	b3->print();	// in class B
	b4->print();	// in class A

	c1 = dynamic_cast<C*>(a1);	// 结果为null，向下转换失败
	if (c1 == nullptr) cout << "c1 is null" << endl;
	else               cout << "c1 is not null" << endl;

	// c2 = static_cast<C*>(a1);	// 类型转换无效, Cannot cast from 'A*' to 'C*' via static_cast

	// delete 省略
}
```
