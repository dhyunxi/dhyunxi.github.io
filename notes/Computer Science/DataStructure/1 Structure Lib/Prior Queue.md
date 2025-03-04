
- 本节介绍优先队列
- 优先队列基于[[Heap|堆]]




# ADT
---
## 概念
- **优先队列**：特殊的队列，出队的顺序依照元素的优先权（关键字）而非进入队列的先后顺序
- 降序优先队列：更大的键有更高的优先级
- 升序优先队列：更小的键有更高的优先级

## 操作
- insert(key,data)：插入pair(key,data)，元素按key排序
- deleteMin/deleteMax：删除并返回最小/最大元素
- getMin/getMax：返回最小/最大元素
- size
- 返回队列中第i个元素

## 应用
- Huffman编码、Dijkstra、Prim、顾客排队算法