# ArrayList/Vector/linkedList

> 主要介绍三个集合以及性能比较及其用法

- 三个都是实现java.util.List接口
- 从集合是否线程安全来说，vector是基于synchronize的线程安全的，而ArrayList与LinkedList是线程不安全的，从性能上vector比ArrayList与LinkedList差；
- ArrayList与vector都是基于object的动态数组的实现，扩容的机制：ArrayList是把容量增加原来的1倍，Vector是增加原来容量的1/2；
- LinkedList是基于双向链表的结构，没有具体的扩容机制；
- ArrayList与LinkedList区别：
  * 对于随机访问元素的性能：LinkedList要比ArrayList效率低，因为LinkedList要遍历一遍链表，ArrayList可以按照下标直接访问元素；
  * 对于添加或者删除元素：LinkedList要比ArrayList效率高，因为ArrayList需要在插入或者删除元素的后方移动元素，有较大的时间与空间的消耗；
- ArrayList添加元素时首先确定是否需要扩容，扩容时首先把容量变成原来的1.5倍，后把之前的元素通过arraycopy到现在的空间；
  ```java
   public void add(int index, E element) {
        rangeCheckForAdd(index);
  
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    private void grow(int minCapacity) {  //增长的策略
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
  ```
- vector是线程安全的，扩容时也会校验是否需要扩容，如果需要，扩容是为原来的1.5倍，
  ```java
      public synchronized void addElement(E obj) { //synchronized确保线程安全性
        modCount++;
        ensureCapacityHelper(elementCount + 1);//是否需要扩容
        elementData[elementCount++] = obj;
    }
      private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity); //扩增原来的0.5空间
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
  ```
  ## 综述
  - 结合自己实际的项目需要，选择合适的集合