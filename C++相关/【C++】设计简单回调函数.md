&emsp;&emsp;当我们写二叉树遍历是，比如写最简单的二叉树前序遍历（[二叉树遍历参考](https://blog.csdn.net/Bob__yuan/article/details/99577057)），如下：
```cpp
void inorder(TreeNode* root) {
	if (root != nullptr) {
		cout << root->val <<endl;
		inorder(root->left);
		inorder(root->right);
	}
}
```
&emsp;&emsp;这个函数现在是用来前序遍历顺序输出每个节点的 val，但是如果别人想用这个遍历的函数，但是要实现自己想实现的功能，就需要用到回调函数了，如下：

```cpp
#include <iostream>
#include <functional>
using namespace std;

int sum2;	// 为了方便演示，sum2设成他全局变量

struct TreeNode {
	int val;
	TreeNode *left, *right;
	TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

void inorder(TreeNode* root, function<void(TreeNode*)> func) {
	if (root != nullptr) {
		func(root);
		inorder(root->left, func);
		inorder(root->right, func);
	}
}

void treeSum(TreeNode* root) {
	sum2 += root->val;
}

int main(){
	TreeNode* root = new TreeNode(1);	/*	   1	 */
	root->left = new TreeNode(2);		/*	  / \	 */
	root->right = new TreeNode(3);		/*	 2   3	 */

	int sum1 = 0;
	inorder(root, [&sum1](TreeNode* root) { sum1 += root->val; });
	cout << "sum1 = " << sum1 << endl;		// 6

	sum2 = 0;
	inorder(root, treeSum);
	cout << "sum2 = " << sum2 << endl;		// 6
}
```
&emsp;&emsp;这样不管是传入<font color=red>**函数指针**</font>，还是<font color=red>**lambda表达式**</font>都可以进行相应的处理，别人要复用这段代码，只需要自己写自己想实现的方法传进来就行，<font color=red>**任何可调用对象都可以转换成std::function<>**</font>。<>里边定义了返回值为 void，传入参数为 TreeNode*，形如这样的可调用对象都可以传进来。
