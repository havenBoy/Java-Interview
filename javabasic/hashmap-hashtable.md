### HashMap 与 HashTable

* **HashMap**
  - HashMap基于哈希表实现，每个元素都是key-value对；
  - HashMap是非线程安全的，只能用于单线程下；
  - HashMap已实现序列化，实现了克隆接口；
* **HashTable**
  - 也是key-value，基于哈希表的实现
  - 是线程安全的，可用于多线程下；
  - HashMap已实现序列化，实现了克隆接口；
* 二者的区别
  - HashMap继承AbstractMap,而HashTable继承于Dictionary类；
  - HashMap是线程不安全的，HashTable是线程安全的，仅仅是因为每个方法都有Synchronize；
  - HashTable中不予许使用null来作为可key和Value值，HashMap可以，因为是object类型；
  - HashMap重新计算了hash值作为hashCode值，而HashTable是直接作为对象的hashCode值；
  - 扩容机制：hashTable初始大小为11，而hashMap初始化为16，扩容时，Table的大小是原来的2倍加1，而HashTMap则直接为原来的2倍；

