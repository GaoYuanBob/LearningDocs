&emsp;&emsp;C++ 11 中新加入的容器 unordered_map 和 unordered_set 底层都是**哈希表**实现的，那么对于内置类型，肯定是可以自动计算出 hash 值的，但是对于像 pair\<int, int\> 或者 vector\<int\> 这样的，或者自定义的类这种复杂类型，就不能自动算出 hash 值了，编译会提示 **The C++ Standard doesn’t provide a hash for this type**。那么就需要自己去定义 hash function 了，自定义 hash 函数可以参考 [https://blog.csdn.net/Bob__yuan/article/details/96737222](https://blog.csdn.net/Bob__yuan/article/details/96737222)，这里不讨论这个问题。
&emsp;&emsp;本文学习并记录一下 C++ 中 string 的 hash 计算方式。首先我们定义一个 unordered_map 如 **unordered_set\<string\> hs;**，然后跳到 unordered_map 的定义，可以看到模板参数中第二个参数就是 hash 函数，<font color=red>**默认使用 hash\<_Kty\>**</font>。 
<div align=center><img src="https://img-blog.csdnimg.cn/20190822154103917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "55%"></div>

&emsp;&emsp;然后再跳到 hash\<_Kty\> 的定义，如下，有一个模板实现，以及一些<font color=red>**模板特化**</font>，但是没有针对 string 的特化。
<div align=center><img src="https://img-blog.csdnimg.cn/20190822154806484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "60%"></div>

&emsp;&emsp;随着C++发展，这部分代码肯定会变，就像上文链接中我是用vs2015里边写的，hash 的结构定义就和上边这个截图（vs2017中）不一样。不过意思都是差不多的。
&emsp;&emsp;跳到 **_Hash_representation** 中发现有跳到了别的函数 **_Fnv1a_append_value**。
<div align=center><img src="https://img-blog.csdnimg.cn/20190822155038347.png" WIDTH = "60%"></div>

&emsp;&emsp;再跳过去，这里有提示 <font color=red>**"Only trivial types can be directly hashed."**</font>。
<div align=center><img src="https://img-blog.csdnimg.cn/2019082215513414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "70%"></div>

&emsp;&emsp;再跳到 **_Fnv1a_append_bytes**。
<div align=center><img src="https://img-blog.csdnimg.cn/2019082215533424.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYl9feXVhbg==,size_16,color_FFFFFF,t_70" WIDTH = "70%"></div>

&emsp;&emsp;**就是这里了！**这里的返回值就是 hash 值，这个函数也很简单，就是从前到后遍历这个传入的参数，对每一位，先转换为 size_t，然后和 _Val 进行异或（注意 _Val 传入就是 **_FNV_offset_basis**），然后 **_Val *= _FNV_prime;**。也就是传入的是 string 的话，对每个字符，转换成 size_t（应该就是 ascii 码），异或，乘一个常量，循环。 <font color=red>**重点就是下边这两句**</font>：
```cpp
	_Val ^= static_cast<size_t>(_First[_Idx]);
	_Val *= _FNV_prime;
```
下面我们验证一下。
```cpp
#include <iostream>
#include <string>
using namespace std;

constexpr size_t tmp_FNV_offset_basis = 2166136261U;
constexpr size_t tmp_FNV_prime = 16777619U;

int main(){
	// 输出 hash 函数计算出的 hash 值
	cout << "hash<string>()(\'a\') = " << hash<char>()('a') << endl;		// 3826002220
	cout << "hash<string>()(\"a\") = "   << hash<string>()("a")   << endl;	// 3826002220
	cout << "hash<string>()(\"abc\") = " << hash<string>()("abc") << endl;	// 440920331

	// 手动计算 'a' 的 hash 值
	size_t val1 = tmp_FNV_offset_basis;
	val1 ^= static_cast<size_t>('a');
	val1 *= tmp_FNV_prime;
	cout << "val1 = " << val1 << endl;

	// 手动计算 'abc' 的 hash 值
	string str = "abc";
	size_t val2 = tmp_FNV_offset_basis;
	for(int i = 0; i < str.length(); ++i) {
		val2 ^= static_cast<size_t>(str[i]);
		val2 *= tmp_FNV_prime;
	}
	cout << "val2 = " << val2 << endl;
}
```
&emsp;&emsp;结果如下：
> hash<string>()('a') = 3826002220
hash<string>()("a") = 3826002220
hash<string>()("abc") = 440920331
val1 = 3826002220
val2 = 440920331

&emsp;&emsp;说明 string 的 hash 值确实是这么计算的，字符也是这么处理，就相当于是长度为 1 的字符串。

&emsp;&emsp;不过《STL源码剖析》里讲，SGI STL 中，string 是按照如下方式计算的：
```cpp
// 以下定义于 <stl_hash_fun.h>
template <class Key> struct hash { };
inline size_t __stl_hash_string(const char* s)
{
	unsigned long h = 0;
	for ( ; *s; ++s)
		h = 5 * h + *s;
	return size_t(h);
}
```