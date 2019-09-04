&emsp;&emsp;unordered_map 是 C++ 11 中新加入的容器，和没有标准化的 hash_map 一个意思，使用 hash 表作为底层数据结构，那么对于键值就需要有 hash function 计算出对应的 hash 值了。\
&emsp;&emsp;对于内置函数，hash function 是自带的，不需要显示指定，如下所示，使用 int、string 作为键值是可以的，但是使用 vector 和 pair 是不行，提示就是 'The C++ Standard doesn't provide a hash for this type."。

```cpp
#include <unordered_map>
#include <vector>
using namespace std;
int main() {
	unordered_map<int, int> mmp;
	unordered_map<string, int> mmp1;
	
	// The C++ Standard doesn't provide a hash for this type.
	unordered_map<pair<int, int>, int> mmp2;	// Error
	unordered_map<vector<int>, int> mmp2;		// Error
}
```
&emsp;&emsp;那么这个 hsah function 是在哪里呢，跳到 unordered_map 定义的地方，可以看到，构造函数有很多种，unordered_map<string, int> mmp1; 这种形式进行构造的话，应该是下图这样的方式，也就是第一个模板参数是**键值**，第二个末班参数是**值**，第三个模板参数是**hash function**（hash<_Kty>），第四个模板参数是相等的比较函数（或者说**表现为函数的对象**），最后一个是分配器。
<div align=center><img src="https://img-blog.csdnimg.cn/20190721180603446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "75%"></div>

&emsp;&emsp;官方的五个参数的注释如下：
```cpp
template < class Key,                                    // unordered_map::key_type
           class T,                                      // unordered_map::mapped_type
           class Hash = hash<Key>,                       // unordered_map::hasher
           class Pred = equal_to<Key>,                   // unordered_map::key_equal
           class Alloc = allocator< pair<const Key,T> >  // unordered_map::allocator_type
         >
class unordered_map;
```

&emsp;&emsp;我们可以通过 unordered_map<int, int>::hasher hs = mmp.hash_function(); 的方式得到 unordered_map 对于不同的键值计算出的 hash 值，也可以直接利用 hash<int>()(1) 的方式进行调用，如下所示：

```cpp
int main() {
	unordered_map<int, int> mmp;
	unordered_map<int, int>::hasher hs = mmp.hash_function();
	
	cout << hs(1) << endl;				// 4218009092	
	cout << hs(2) << endl;				// 3958272823
	cout << hash<int>()(1) << endl;			// 4218009092, equal = hs(1)
	cout << hash<string>()("abc") << endl;		// 440920331
	cout << hash<float>()(3.1415926) << endl;	// 812821127
	cout << hash<double>()(3.1415926) << endl;	// 2306040251
}
```
&emsp;&emsp;我们可以调到 hash 这个结构的定义部分，如下图，对于基础类型，都给出了对应的<font color=red>**模板特化**</font>，也可以看到上边编译错误时报错的地方。
<div align=center><img src="https://img-blog.csdnimg.cn/20190721181158473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "65%"></div>

&emsp;&emsp;所以如果对于自定义类型，或者 hash 这个结构没有提供模板特化的数据类型，那就需要自己定义了，最简单的方法就是：

```cpp
struct pair_hash {
	template<class T1, class T2>
	std::size_t operator() (const std::pair<T1, T2>& p) const	{
		auto h1 = std::hash<T1>{}(p.first);
		auto h2 = std::hash<T2>{}(p.second);
		return h1 ^ h2;
	}
};

int main() {
	//unordered_map<pair<int, int>, int> error_mmp;			// error
	unordered_map<pair<int, int>, int, pair_hash> ok_mmp;	// ok
}
```
&emsp;&emsp;unordered_set 同理，都是用的**哈希表**。
