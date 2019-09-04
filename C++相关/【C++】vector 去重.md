### 记录一下C++ STL 中 vector 的去除重复元素的方法
#### 1、对于一个已经 sorted（`升序降序皆可`）的vector来说，去重只需要使用 unique 函数
&emsp;&emsp;unique 函数是从前向后遍历，将所有和前一个元素相同的元素放到 vector 尾部（`“remove each matching previous”`），然后返回指向第一个重复元素的迭代器（如果没有重复的，返回 finish，也就是 end() 迭代器），所以升序降序皆可。再使用 erase 方法出去这个元素以及后边所有元素即可去除重复元素。如果 vector 没有排序直接使用 unique 会出现如下错误。
```cpp
#include <vector>
#include <algorithm>	// unique、sort 所需头文件
using namespace std;

int main() {
	vector<int> v = { 2, 3, 7, 7, 3, 1, 2, 4, 4, 1, 5 };
	//const auto u_idx = unique(v.begin(), v.end());  //vector<int>::const_iterator
	//for(auto ite = cbegin(v); ite != cend(v); ++ite)
	//{
	//	if (ite == u_idx) printf("u_idx ");
	//	printf("%d ", *ite);	// 输出 2 3 7 3 1 2 4 1 5 u_idx 1 5 "，少了4、7，多了1、5
	//}
	sort(begin(v), end(v));		// 使用 unique 必须先排序
	// sort(begin(v), end(v), [](const int& i, const int& j) { return (i > j); });	// 降序也可

	const auto u_idx = unique(begin(v), end(v));	// unique 作用: "remove each matching previous"
	for (auto ite = begin(v); ite != end(v); ++ite) {
		if (ite == u_idx)  printf("unique_idx ");
		printf("%d ", *ite); // 输出 "1 2 3 4 5 7 unique_idx 4 4 5 7 7 "
	}
	printf("\n");

	v.erase(u_idx, end(v));	 // 去重直接 v.erase(unique(begin(v), end(v)), end(v)); 即可
	for (auto tv : v)
		printf("%d ", tv);	 // 输出 "1 2 3 4 5 7 "
	printf("\n");
}
```

#### 2、对于没有排序又不想调整顺序的 vector，使用 unordered_set 即可
&emsp;&emsp;很多情况下 vector 不想改变元素原有相对顺序，可以使用 unordered_set 来完成（`使用 set 和方法1效果一样，结果是默认升序`），如下所示。
```cpp
#include <vector>
#include <unordered_set>
using namespace std;

int main() {
	vector<int> v = { 2, 3, 7, 7, 3, 1, 2, 4, 4, 1, 5 };
	unordered_set<int> s(begin(v), end(v));
	v = vector<int>(begin(s), end(s));
	for (auto tv : v)
		printf("%d ", tv);		// 输出 "2 3 7 1 4 5 "
	printf("\n");
}
```
