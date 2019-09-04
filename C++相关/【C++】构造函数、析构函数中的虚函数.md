&emsp;&emsp;构造函数或者析构函数中有虚函数，那实际执行的是哪个函数，测试代码如下：

```cpp
#include <iostream>
class A {
public:
	A() {
		printf("*** A() begin ***\n");
		f();
		g();
		printf("*** A() end ***\n\n");
	}
	void g() { printf("A g()\n"); }
	virtual void f() { printf("A f()\n"); }
	virtual ~A()	// 不加 virtual 会只执行父类的析构
	{
		printf("*** ~A() begin ***\n");
		f();
		g();
		printf("*** ~A() end ***\n\n");
	}
};

class B : public A {
public:
	B() {
		printf("*** B() begin ***\n");
		f();
		g();
		printf("*** B() end ***\n\n");
	}
	void g() { printf("B g()\n"); }
	void f() override { printf("B f()\n"); }
	~B() {
		printf("*** ~B() begin ***\n");
		f();
		g();
		printf("*** ~B() end ***\n\n");
	}
};

int main() {
	A* p = new B;

	printf("p->f() -> ");
	p->f();

	printf("p->g() -> ");
	p->g();

	printf("\n");
	delete p;
}
```

&emsp;&emsp;输出如下：
> *** A() begin ***\
> A f()\
> A g()\
> *** A() end ***\
> \
> *** B() begin ***\
> B f()\
> B g()\
> *** B() end ***\
> \
> p->f() -> B f()\
> p->g() -> A g()\
> \
> // A 中析构函数如果不是虚函数，则没有这一段输出\
> *** ~B() begin ***\
> B f()\
> B g()\
> *** ~B() end ***\
> // A 中析构函数如果不是虚函数，则没有这一段输出\
> \
> *** ~A() begin ***\
> A f()\
> A g()\
> *** ~A() end ***

&emsp;&emsp;从上边的代码和结果可以看出：\
&emsp;&emsp;1、构造函数执行顺序是：先父类，后子类；析构函数的执行顺序是：先子类，后父类。\
&emsp;&emsp;2、父类的析构函数必须是虚函数，否则 delete 时候不会执行==子类析构函数==，会内存泄漏。\
&emsp;&emsp;3、同名，同参虚函数能正常达到多态效果，不是虚函数就会出现 ==“隐藏”==，父类指针只能调用父类函数，ReSharper提示如下。

<div align=center><img src="https://img-blog.csdnimg.cn/20190319191653171.png">
</div>

&emsp;&emsp;4、类的构造函数和析构函数中，虚函数表现不出多态！\
&emsp;&emsp;在VS中，ReSharper 会提示构造函数和析构函数中的虚函数会被<font color=red>**静态处理**</font>，也就是用这个类自己的，如下图。

<div align=center><img src="https://img-blog.csdnimg.cn/2019031919112047.png">
</div>

<div align=center><img src="https://img-blog.csdnimg.cn/2019031919155381.png">
</div>
