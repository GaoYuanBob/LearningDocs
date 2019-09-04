&emsp;&emsp;本文分为以下几个部分内容：
* 什么是内存对齐，为什么要内存对齐
* C++的空类，以及没有虚函数和非静态变量的类
* C++类的内存分布（成员变量）
* C++类的内存分布（虚函数）
	* 一个类的情况
	* 继承关系中的情况

### 一、什么是内存对齐，为什么要内存对齐
#### 1.1 什么是内存对齐：
&emsp;&emsp;内存对齐是从硬件层面出现的概念。可执行程序是由一系列CPU指令构成的，其中有一些指令是需要访问内存的。在很多CPU架构下，这些指令都要求操作的内存地址（更准确的说，操作内存的<font color=red>**起始地址**</font>）能够被操作的内存大小整除，满足这个要求的内存访问叫做<font color=red>**对齐内存的访问**</font>（**aligned memory access**），否则就是<font color=red>**未对齐内存的访问**</font>（**unaligned memory access**）。如果访问未对齐的内存会出现什么结果呢？这个要看CPU。
* 有些CPU架构可以访问未对齐的内存，但是会有性能上的影响。典型的就是 x86 架构CPU
* 有些CPU会抛出异常
* 有些CPU不会抛出任何异常，会静默地访问错误的地址
* 近几年也有些CPU的一部分指令可以正常访问未对齐的内存，同时不会有性能影响

&emsp;&emsp;因为每个CPU对未对齐内存的访问的处理方式都不一样，所以访问未对齐的内存是要尽量避免的。所以就出现了 C/C++ 的内存对齐机制。
#### 1.2 为什么要内存对齐：
1. 平台原因（移植原因）：不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。 
2. 性能原因：数据结构（尤其是栈）应该尽可能地在自然边界上对齐。原因在于，为了访问未对齐的内存，处理器需要作两次内存访问；而对齐的内存访问仅需要一次访问。
### 二、C++的空类，以及没有虚函数和非静态变量的类
#### 2.1 C++ 空类大小：
```cpp
class A{
};

int main() {
	cout << sizeof(A) << endl;	// 1
}
```
&emsp;&emsp;对于一个什么都没有的空类，实际并不是空的，因为有默认的函数，具体可以参考 （待填入网址），大小是 1，这是因为需要有一个地址，C++ 不允许两个不同的对象有相同的地址，所以 C++ 中空的类和结构体大小都是 1。
#### 2.1 加入成员函数、静态成员函数、静态成员变量：
&emsp;&emsp;当我们显示加入了新的成员函数、静态成员函数、静态成员变量后，类的大小还是 1：
```cpp
class A{
	A(){}
	~A(){}
	void print() { printf("print()\n"); }
	void foo() { printf("print()\n"); }

	static void sprint() { printf("sprint()\n"); }
};

int main() {
	cout << sizeof(A) << endl;	// 1
}
```
&emsp;&emsp;也就是成员函数、静态成员函数、静态成员变量都是不占用类的内存的，这是因为这些东西都不是类的，而不是每个对象分别存储。static 变量就是存储在全局静态区。
### 三、C++类的内存分布（变量）
&emsp;&emsp;C++ 中会影响一个类的对象的大小的，就是<font color=red>**非静态成员变量**</font>和<font color=red>**虚函数**</font>。
&emsp;&emsp;在 C++ 中每个类型都有两个属性，一个是大小（**size**），还有一个就是对齐要求（**alignment requirement**），或称之为对齐量（**alignment**）。C++标准并没有规定每个类型的对齐量，但是一般都会有这样的规律：
* 所有基础类型的对齐量等于这个类型的大小。
* struct, class, union 类型的对齐量等于其中**非静态成员变量**中**最大的对齐量**。
* 标准规定所有的对齐量必须是 2 的幂次。

&emsp;&emsp;编译器在给一个变量分配内存时，都要算出并满足这个类型的对齐要求。struct 和 class 类型的非静态成员变量的字节数偏移（**offset**）也要满足各自类型的对齐要求。
&emsp;&emsp;从下边的例子中我们就可以看到：
```cpp
class A{		// size		pos range
	int a;		// 4		0 - 3
	double b;	// 8		8 - 15
	short c;	// 2		16 - 17
};

class B {
	int a;		// 4		0 - 3
	short c;	// 2		4 - 5
	double b;	// 8		8 - 15
};

int main() {
	cout << sizeof(A) << endl;	// 24
	cout << sizeof(B) << endl;	// 16
}
```
&emsp;&emsp;**类 A 和类 B 的对齐量都是 sizeof(double) = 8**，<font color=red>**就好比这个类都是 8 大小的盒子，每个变量都是按声明的前后顺序往盒子里放，当前盒子放不下，就放下一个全新的空盒子中**</font>。所以上边的类 A 中 int a; 占了第一个盒子的一半，double b; 发现只有 4 大小的盒子放不下，就往下一盒子盒子中放了（<font color=red>**全新的盒子一定放的下，因为盒子大小是所有变量中最大的！**</font>），而类 B 中 int a; 放在第一个盒子后，short c; 只需要 2 的大小，所以还是可以和 int a; 放在一个盒子中（所以类 B 中的 short c; 换成 int c; 不会影响类的大小，因为第一个盒子还是放得下）。
&emsp;&emsp;多尝试尝试能有更好的理解：

### 四、C++类的内存分布（虚函数）
#### 4.1 一个类中有虚函数时内存分布
&emsp;&emsp;C++ 的类中，没有除了虚函数以外的所有函数，都是不占类的内存的，但是如果类有了虚函数，类内就会有一个<font color=red>**虚函数表的指针 _vptr**</font>，指向自己的虚函数表，<font color=red>**vptr 一般都是在类的最前边**</font>，如下所示。
<div align=center><img src="https://img-blog.csdnimg.cn/20190904134505152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "70%"></div>

&emsp;&emsp;由于只是存一个指向虚函数表的指针，所以不管有多少个虚函数，都是 4 字节大小（一个指针的大小），比如下边这个类 A，size 就是 4：
<div align=center><img src="https://img-blog.csdnimg.cn/20190904140018758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "60%"></div>

#### 4.2 继承关系中的有虚函数时的内存分布
&emsp;&emsp;用下边这段代码看，内存分布如图所示：
```cpp
class A {
public:
	A(){}
	virtual ~A(){}
	virtual void foo(){}
	virtual void print() {}
};

class B : public A {
	double d;
	void print() override { cout << "B print()" << endl; }
};

int main() {
	A a;	// sizeof(A) = 4  (_vptr: 4)
	B b;	// sizeof(B) = 16 (_vptr: 4 + 空: 4 + double: 8)
}
```
<div align=center><img src="https://img-blog.csdnimg.cn/20190904142027386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "70%"></div>

&emsp;&emsp;最关键的一个点就是，<font color=red>**对于没有 override 的虚函数，基类和子类中 _vptr 指向的虚函数表中，这个虚函数的地址是一样的**</font>，也就是上边的 **foo()** 函数，而对于重写了的或者默认重写的析构函数来说，**_vptr** 指向的虚函数表中，函数地址是不一样的（当然两个类的 **_vptr** 地址也是不一样的，这是肯定的），这就能窥探到多态的实现了。
