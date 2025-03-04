
 
 
 Map是C++STL中众多的Container（容器）之一，与python的字典略类似，Map作为一个关联容器，将key与value相互关联，其中key为关键字，是不可更改的，而value是key值的相对应值。Map所提供的一对一的数据映射关系，在很多时候可以提供编程的极大便利。
Map内部通过自建红黑树（一种非严格意义上的平衡二叉树）实现，可以对数据自动排序，因而在map内部的所有数据是有序存放的。Map具有的一大特点是增加和删除节点对迭代器的影响很小，除了那个操作节点，对其他的节点都没有什么影响

# 容器属性
---
- 元素由键引用
- 自动排序
- 映射
- 键唯一
- alocator动态分配

# 模板参数
- `map<key_type , mapped_type , [key_compare]>`
- 键类型、值类型

# 成员函数
- 构造
- 析构
- 复制=
## 迭代器
- begin()
- end()
- rbegin()
- rend()
- cbegin()
- cend()
- crbegin()
- crend()

## 容量
- empty()：判空
- size()
- max_size()

## 元素访问
- []
- at

## 修改
- insert
- erase
- swap
- clear
- emplace
- emplace_hint

## obervers
- key_comp
- value_comp

## operations
- find
- count
- lower_bound
- upper_bound



# 成员
- 可以使用 first、second来指示key、value
```c++
map<int,int> m;
```


# 构造
- map[key]=value
	- key可以任意，不需提前申请，会自动创建
- m.insert(pair<int,int>(1,2))
	- 返回一个pair，第一项是一个map迭代器，第二项是bool，若插入成功为True否则为False