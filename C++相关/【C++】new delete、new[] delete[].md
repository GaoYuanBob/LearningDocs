&emsp;&emsp;C++ 中，new、delete 和 sizeof 一样，都**不是函数**，都是<font color=red>**操作符**</font>。面试经常回问 malloc/free 和 new/delete的区别和联系：
*  malloc/free 只是动态分配内存空间/释放空间，new/delete 除了分配空间还会调用**构造函数**和**析构函数**进行初始化与清理
*  它们都是动态管理内存的入口
*  malloc/free 是 C/C++ 标准库的函数，new/delete 是 C++ 操作符
*  malloc/free 需要**手动计算类型大小**且返回值为 **void***，new/delete 可**自动计算类型的大小**，**返回对应类型的指针**
*  malloc/free 管理内存失败会返回 0，new/delete 等的方式管理内存失败会**抛出异常**
* 从系统层面看来，真正开出空间来的还是 malloc，operator new 和 operator delete 只是 malloc 和 free 的一层封装

#### 正确写法，new[]、delete[] 配对使用
&emsp;&emsp;先设计一个类，每个对象有自己的 id，这样我们可以看到构造和析构的顺序，之后的所有有 A 这个类的代码都用这个 A：
```cpp
class A {
public:
	int id;
	explicit A(int _id) : id(_id) { printf("构造 id 为 %d 的对象\n", id); };
	~A() { printf("释放 id 为 %d 的对象\n", id); }
};
```
&emsp;&emsp;然后是正常的 new[] 构造数组，delete[] 析构数组：
```cpp
int main() {
	A* pa = new A[3]{ A(1), A(2), A(3) };
	putchar('\n');	// 用来分开构造和析构的输出
	delete[] pa;
}
```
&emsp;&emsp;上边代码输出如下：
> 构造 id 为 1 的对象
> 构造 id 为 2 的对象
> 构造 id 为 3 的对象
> \
> 释放 id 为 3 的对象
> 释放 id 为 2 的对象
> 释放 id 为 1 的对象

&emsp;&emsp;可以看到这是正常的构造和析构过程，这就引发了两个问题：
&emsp;&emsp;<font color=red>**1、delete[] 是怎么知道数组有多大的？**</font>
&emsp;&emsp;<font color=red>**2、如果不配对，new[] 之后用 delete 会怎么样？**</font>
&emsp;&emsp;<font color=red>**3、为什么析构顺序是反着的？**</font>
#### 一、delete[] 怎么知道数组有多大的？
##### 1.1 C++有析构函数的对象
&emsp;&emsp;还是上边的代码，进入调试，可以看到 operator new[] 返回的确实是 **void*** 类型，pa 则是 **A*** 类型。
<div align=center><img src="https://img-blog.csdnimg.cn/20190903151619441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "60%"></div>

&emsp;&emsp;不过我们的重点不是这里，而是值，可以看到 <font color=red>**operator new[] 返回的值（也就是指针地址）和 pa 的值相差 4**</font>，这是因为编译器用这 **4** 个字节来保存 <font color=red>**对象个数**</font>，也就是在这个 pa 指针向前 4 个字节中，记录这对象个数这个值，我们用下面代码验证一下，发现**对象个数输出确实是 5**，甚至还可以手动修改这个值，如下：
```cpp
int main() {
	A* pa = new A[5]{ A(1), A(2), A(3), A(4), A(5) };
	printf("\n对象个数: %d\n", *(pa - 1));
	*reinterpret_cast<int*>(pa - 1) = 3;	// *((int*)(pa - 1)) = 2; 也行
	delete[] pa;
}
```
&emsp;&emsp;输出如下：
> 构造 id 为 1 的对象
> 构造 id 为 2 的对象
> 构造 id 为 3 的对象
> 构造 id 为 4 的对象
> 构造 id 为 5 的对象
> \
> 对象个数: 5
> 释放 id 为 3 的对象
> 释放 id 为 2 的对象
> 释放 id 为 1 的对象

&emsp;&emsp;**这个输出很重要！** 说明 delete[] 确实是根据数组指针前 4 个字节存储的数据作为对象个数的记录的，<font color=red>**这就解释了为什么 delete[] 不需要传入参数**</font>。
##### 1.2 基础类型或者没有析构函数的类型
&emsp;&emsp;对于基础类型，没有析构函数，或者编译器会优化的类型，就不会有这个记录对象个数的值存在：
```cpp
int main() {
	int* a = new int[10];
	printf("\n对象个数: %d\n", *(a - 1));	//  -33686019
	delete[] a;
}
```
&emsp;&emsp;而且 delete a; 和 delete[] a; 也都不会崩。
&emsp;&emsp;<font color=red>**上边的类 A 中，如果把析构函数注释掉，也不会有对象个数的记录**</font>，会输出一个随机值。
&emsp;&emsp;但是还是应该保持 new[] 和 delete[] 的配对原则使用。
#### 二、如果不配对使用，new[] 之后用 delete 会怎么样？
```cpp
int main() {
	A* pa = new A[5]{ A(1), A(2), A(3), A(4), A(5) };
	delete pa;
}
```
&emsp;&emsp;比如上边这样的代码，运行起来，会输出：
> 构造 id 为 1 的对象
> 构造 id 为 2 的对象
> 构造 id 为 3 的对象
> 构造 id 为 4 的对象
> 构造 id 为 5 的对象
> 释放 id 为 1 的对象

&emsp;&emsp;然后崩掉，提示下图：
<div align=center><img src="https://img-blog.csdnimg.cn/20190903160253258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "35%"></div>

&emsp;&emsp;所以可以看到，new[] 构造出对象数组，但是只用 delete 去释放的话，会有两个问题：
&emsp;&emsp;<font color=red>**1、只会对第一个对象的调用析构函数，其他对象的内存不会释放**</font>
&emsp;&emsp;<font color=red>**2、更严重的是，有时候会直接崩掉**</font>
&emsp;&emsp;也不可以 new 之后 delete[]，总之一句话，<font color=red>**new 配 delete，new[] 配 delete[]**</font>。
&emsp;&emsp;所以面试时，如果问到如果 new[] 了之后 delete，会有什么问题，不能只回答剩下的部分不会被调用析构函数，因为可能还会导致程序崩溃（之前我就是答的内存泄漏这个问题，然后他问我还有什么问题，我就蒙了 = =）。

#### 三、为什么析构顺序是反着的？
&emsp;&emsp;没有看到官方的、靠谱的解释，可能就是 C++ 的规定，很多构造和析构都是先构造的后析构，类似栈的感觉，比如继承关系中：
```cpp
class A {
public:
	A()          { printf("构造 A 的对象\n"); }
	virtual ~A() { printf("释放 A 的对象\n"); }
};

class B : public A {
public:
	B()  { printf("构造 B 的对象\n"); }
	~B() { printf("释放 B 的对象\n"); }
};

int main() {
	A *p = new B();
	delete p;
}
```
&emsp;&emsp;输出：
> 构造 A 的对象
> 构造 B 的对象
> 释放 B 的对象
> 释放 A 的对象

&emsp;&emsp;肯定还是不太一样，如果有大神可以讲解一下，将万分感谢！！！！