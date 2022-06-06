### 红黑树

> 只要是谈及TreeMap以及HashMap，总要介绍一下红黑树；

- 简介：

  - Red-Black Tree,是一种特殊的二叉查找树，每个节点都存储了节点的颜色；

  - 其特点如下：

    1.每个节点要不是红色，要不是黑色；

    2.根节点一定是黑色的；

    3.红色节点不能连续（该节点的父亲与儿子不能为红色）

    4.对于每一个节点，从该节点到末端都有相同个数的黑色节点；

  - 其时间复杂度的是 log(n)

  - TreeMap是非线程安全的，如果需要可以使用：

    SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));