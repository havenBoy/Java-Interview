## 时间复杂度与空间复杂度
> 衡量一个算法的基本手段
> 主要讲述各自的概念、如何计算一个算法的时间与空间复杂度

#### 概念
- 时间复杂度：算法执行语句的次数；
- 空间复杂度：执行此算法所需要的空间；

#### 计算方法
- 首先选取相对增高的最高的项，即最高指数的项；
- 把最高项的系数换成1；
- 如果最高项是常数，则复杂度是1；

#### 时间复杂度的计算：
> 通常，如果没有说明，我们一般默认说的是算法的最坏情况下的时间复杂度
- 解决问题的代码与问题的规模n有关，如果无关的话，时间复杂度始终为O(1);
- 若干个循环语句的嵌套，与嵌套的层数有关，假设嵌套的层数是常数k,那么时间复杂度为O(n^k);
- 特殊的，类似于二分查找法时间复杂度为log2(n)；

#### 空间复杂度的计算：
- 一般常数级的运算空间复杂度是O(1);
- 递归算法空间复杂度 = 递归的层数 * 每次递归需要的空间；
- 默认情况下指的是需要最大空间复杂度的数量级，从而来存储递归最深级别的空间大小；

#### 常见算法的时间复杂度以及空间复杂度
- 二分查找算法 O(log(n))     O(1)
- 斐波那契数列   递归算法： O()    O()   ----   O()    O();
- 冒泡排序：
- 快排：
- 归并排序：
- 堆排序：