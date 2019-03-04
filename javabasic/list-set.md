# list - set

 二者都是Collection的子接口或者实现类

- list
  - 可以允许重复的对象
  - 可以插入多个null对象
  - 是一个有序的容器，保证出入的有序性
  - 常用的ArrayList,linkedlist，vector
- set
  - 不允许重复的对象
  - 无序的对象，无法保证插入的顺序与出入顺序的一致性
  - 只允许一个NULL对象的存在
  - 常用的set有HashSet, LinkedHashset,TreeSet
- Map       （不是Collection的子接口，而是单独的Map接口）
  - Map的每个Entry是都持有2个对象，一个是Key，一个是Value值，value值可以多次重复，但Key值必须是唯一的
  - Map中的Key值只能由一个NULL值，Value值可以是多个NULL；
  - 常见的有HashMap，LinkedHashMap， TreeMap等