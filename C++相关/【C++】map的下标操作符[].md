&emsp;&emsp;C++ 中，set、map、multiset、multimap 都是使用 RB-Tree 作为底层数据结构的，所以他们的代码其实就是封装了一层的红黑树区别在于：
* set 和 multiset 的键值就是实值，所以都不允许修改实值，因为会导致排序混乱。set 不允许有两个相同键值，multiset 则允许，所以 set 和 multiset 的区别就在于，set 的insert 调用的是红黑树的 insert_unique()，即不允许节点键值重复，而 multiset 调用的是红黑树的 insert_equal() 接口；
* map 和 multimap 存储的都是 pair，first 是键值，second 是实值，这样就可以修改实值了，因为不会影响键值，map 和 multimap 的区别和上边 set 和multiset 的区别一样，就是 insert 所调用的红黑树的接口不用。

&emsp;&emsp;这篇博客要记录的是关于 map 的下标操作符的东西，从下边代码可以看出：\
&emsp;&emsp;1、map 的 [] 既可以作为**左值引用**（即内容可以被修改），又可以作为**右值引用**（即内容不可以被修改）\
&emsp;&emsp;2、map<string, int> mmp; mmp["hello"] 输出 0，说明默认都是 0，这样就可以按照很常见的写法直接 ++mmp["hello"] 了，完全不需要管 "hello" 是否存在\
&emsp;&emsp;3、输出 mm["hello"] 为 0 后，发现 map 中已经插入了 'hello"
```cpp
	map<string, int> mmp;
	if (mmp.find("hello") != mmp.end())	// 没找到
		cout << "found" << endl;

	cout << mmp["hello"] << endl;	// 0

	if (mmp.find("hello") != mmp.end())	// 找到
		cout << "found" << endl;

	++mmp["abc"];
	cout << mmp["abc"] << endl;	// 1
```
&emsp;&emsp;这些都是因为 map 的下标操作符对左值、右值都适用，这关键就在于，<font color=red>**map 的 [] 返回值采用的是 by reference 传递形式**</font>，如下：
```cpp
		// CLASS TEMPLATE map
template<class _Kty,			// 键值
	class _Ty,					// 实值
	class _Pr = less<_Kty>,		// 比较函数
	class _Alloc = allocator<pair<const _Kty, _Ty>>>	// 存储结构，注意const
	class map
		: public _Tree<_Tmap_traits<_Kty, _Ty, _Pr, _Alloc, false>>
	{	// ordered red-black tree of {key, mapped} values, unique keys
public:
	...
	using key_type = _Kty;
	using mapped_type = _Ty;
	...
	mapped_type& operator[](key_type&& _Keyval)
	{	// find element matching _Keyval or insert with default mapped
		return (try_emplace(_STD move(_Keyval)).first->second);
	}
```
&emsp;&emsp;这是 C++ 11 之后的写法，用到了 emplace 和 std::move（移动语义），可能不是很好理解，看《剑指offer》上写的更好理解一点：
```cpp
template<class Key, class T,
		 class Compare = less<Key>,
		 class Alloc = alloc>
class map {
public:
	typedef Key key_type;	// 键值
	typedef pair<const Key, T> value_type;	// 元素型别（键值/实值）
	...
	T& operator[](const key_type& k) {
		return (*(insert(value_type(k, T()))).first).second;
	}
};
```
&emsp;&emsp;这个返回值看起来很复杂，但是理解起来简单些，首先就是一直没有变的 **by reference** 形式的返回类型，在传入参数中，因为没有用到移动语义，所以不需要上边的 &&，而是单纯的 **const reference** 形式。对于 return 这一句，首先是用传入的键值和临时对象 T() 创建一个临时 value_type（也就是 pair<const Key, T>），然后将这个 pair 插入到 map 中，要注意，这个 insert 如果插入的键值在 map 中没有，就插入到合适位置并返回，如果已经存在，就直接返回已经存在的。要注意，这里 insert 调用的是红黑树的  insert_uniquem，实际返回的是一个 pair (如下是 insert_unique 的一个返回路径)，first 是迭代器，second 是是否插入新值。
```cpp
	return pair<iterator, bool>(j, false);
```
&emsp;&emsp;所以 ***(insert(value_type(k, T()))).first** 的目的就是获取到这个迭代器，然后最后 **.second** 返回实值，下标操作符结束。
&emsp;&emsp;所以说，上边的例子中，++mmp["abc"]; 一句，其实是两个操作，先用 int 默认的构造函数，创建一个临时 int，同于创建 pair，然后插入到 map中，这个临时的 int 默认为 0，插入之后，执行 ++ 操作，对已经存在于 map 中的这个迭代器（刚创建的）的实值进行 ++。
