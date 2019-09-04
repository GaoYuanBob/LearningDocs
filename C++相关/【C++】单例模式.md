&emsp;&emsp;设计模式是面试中经常会问到的问题，而单例模式（**Singleton Pattern**）又是最常用的几个之一。其意图是保证一个类**最多仅有一个实例**，并提供一个访问它的**全局访问点**，该实例被所有程序模块共享。\
&emsp;&emsp;定义一个单例类需要：\
&emsp;&emsp;1、私有化它的构造函数，以防止外界创建单例类的对象\
&emsp;&emsp;2、使用类的私有静态变量表示类的唯一实例\
&emsp;&emsp;3、使用一个公有的静态方法获取该实例

&emsp;&emsp;分为懒汉版（**Lazy Singleton**）和懒汉版（**Eager Singleton**）：
### 一、懒汉版（Lazy Singleton）
&emsp;&emsp;单例实例在第一次被使用时才进行初始化，这叫做<font color=red>**延迟初始化**</font>。

```cpp
class Singleton
{
private:
	static Singleton* instance;

	Singleton() {};
	~Singleton() {};
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
	
public:
	static Singleton* getInstance() {
		if(instance == NULL) 
			instance = new Singleton();
		return instance;
	}
};

// init static member
Singleton* Singleton::instance = NULL;
```
&emsp;&emsp;这个方式在多线程下会有问题。在<font color=red>**C++11之后，类内静态成员变量是线程安全的**</font>（C++11规定，在一个线程开始local static 对象的初始化后到完成初始化前，其他线程执行到这个local static对象的初始化语句就会等待，**直到该local static 对象初始化完成**）。也有很多博客测试了，类内静态成员变量的构造是线程安全的，但是访问不是线程安全的，不过这里只需要保证构造是线程安全的就行，所以就有以下称为 <font color=red>**Meyers' Singleton**</font> 方式的单例模式：
```cpp
class Singleton {
public:
    static Singleton& Instance() {
        static Singleton theSingleton;
        return theSingleton;
    }

private:
    Singleton();                            // ctor 		hidden
    Singleton(Singleton const&);            // copy ctor 	hidden
    Singleton& operator=(Singleton const&); // assign op. 	hidden
    ~Singleton();                           // dtor 		hidden
};
```
&emsp;&emsp;注意这种方式只有在 C++11 及以后才是线程安全的。
### 二、饿汉版（Eager Singleton）
&emsp;&emsp;指单例实例<font color=red>**在程序运行时被立即执行初始化**</font>。
```cpp
class Singleton
{
private:
	static Singleton instance;

	Singleton();
	~Singleton();
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
	
public:
	static Singleton& getInstance() {
		return instance;
	}
}

// initialize defaultly
Singleton Singleton::instance;
```
&emsp;&emsp;由于在main函数之前初始化，所以没有线程安全的问题。但是潜在问题在于no-local static对象（函数外的static对象）在不同编译单元中的初始化顺序是未定义的。也即，static Singleton instance;和static Singleton& getInstance()二者的初始化顺序不确定，如果在初始化完成之前调用 getInstance() 方法会返回一个未定义的实例。

&emsp;&emsp;总的来说，还是用C++11及之后的 **Meyers' Singleton** 最好。
