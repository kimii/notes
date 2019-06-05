# Coding
--------
## Leetcode
- 题目 -> 思路
	- 衍生题 -> 思路，对比
1. permutation(不同字符全排列)： 递归+递归中传引用逐位交换
	- permutationUnique(包含相同字符全排列)： 
		- 最好： 排序(sort)+递归+递归中传值逐位交换+交换不在换回来(已排序)
		- 不佳： 递归+递归中传引用逐位交换+set去重

## 剑指offer
### c++
- 与类型转换相关的 4 个关键字
	- static_cast： 运算符完成相关类型之间的转换　
	- reinterpret_cast： 处理互不相关类型之间的转换
	- dynamic_cast： 处理基类型到派生类型的转换
	- const_cast： 移除变量的const或volatile限定符
- 复制构造函数
	- 参数必须为常量引用，不能是指针或传值，否则会发生无限制递归
```
class A
{
private:
	int v;
public:
	A(int n){ v = n; }
	A(const A& other){ v = other.v; } /* 复制构造函数 */
	/* 赋值运算符重载 */
	A& operator = (const A& a)
	{
		v = a.v;
		return *this;
	}
};

int main(int args, int* argv[])
{
	A d(2);
	A b = d;	/* b 未实例化发生复制构造 */
	/* 若复制构造函数参数为 A other（传值），递归解释：
     *     A b = d;		->b.A(A other) ->b.A(d)  
     *     A other = d; ->other.A(d)   ->other.A(d) ...
	 */
	A c(3);		
	c = d		/* c 已实例化发生赋值，区别于上例 */

}
```
- 局部变量运行到该变量作用域之外，会自动调用析构函数释放内存
- 链表头指针传参	pHead 为指针的指针
```
/* 结构体 */
struct ListNode
{
	int v;
	ListNode* pNext;
};

/* 修改链表传参，传入 **pHead, 指向指针的指针
 * 插入新节点 
 * 修改 pHead 指向的指针 *pHead 的内容
 */ 

void addToTail(ListNode** pHead, int value)
{
	/* 1. 为新节点申请内存 */
	ListNode*pNew = new ListNode()
	pNew->v = value;
	pNew->pNext = nullptr;
	/* 2. 插入节点为头结点 */
	if(*pHead == nullptr) { *pHead = pNew; return; }
	/* 3. 插入节点不为头结点 */
	ListNode* pTemp = *pHead;
	while(pTemp->pNext != nullptr)
		pTemp = pTemp->pNext;
	pTemp->pNext = pNew;
	return;
}

```

### python 
1. 动态执行 exec/eval 使用


