&emsp;&emsp;在C++中，类的实例化可以 A a; 或者 A *p = new A; ，有的时候想只能用其中一种方式实例化。
### 一、只能静态分配类对象，即 A a; 的方式进行实例化
&emsp;&emsp;这样需要将类的 new 和 delete 操作符重载为 protected，new[] 、delete[] 同理，如下：

```cpp
class A {
protected:	// ***
	void* operator new(size_t) { return nullptr; };	// 函数的第一个参数与返回值类型是固定的
	void operator delete(void*) { };		// 重载了new，就要重载delete

	void* operator new[](size_t) { return nullptr; };
	void operator delete[](void*) { };
};

int main() {
	A *a1 = new A;		// not ok, A::operator new   不可访问
	A *a2 = new A[];	// not ok, A::operator new[] 不可访问
	A a3;			// ok
}
```

### 二、只能动态分配类对象，即 A *p = new A;  的方式进行实例化
&emsp;&emsp;把类的构造函数和析构函数设为protected属性。类对象不能访问，但是派生类可以继承，也可以访问。还需要创建 create 和 destroy 两个函数，用于创建和销毁类的对象，不然 new A 的时候会报错说构造函数是 non-public 的不可访问。设为 static 是因为创建对象时候只能用类名访问，即 A::create();。

```cpp
class A {
protected:
	A() = default;
	~A() = default;

public:
	static A* create() {
		return new A();
	}
	static void destroy(A* p) {
		delete p;	// 不要用 delete this;
	}
};

int main() {
	A *p = A::create();
	A::destroy(p);
}

```
