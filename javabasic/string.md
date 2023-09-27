### String  StringBuilder  StringBuffer

- String类是final类，不可以被继承，且它的成员方法也是final方法，当一个字符串对象进行操作操作时，任何的改变不会影响到这个对象，而是生成一个新的对象，操作是针对这个新的对象的；

- 对于下边程序的理解：

  ~~~java
  public class Main {
           
      public static void main(String[] args) {
          String str1 = "hello world";
          String str2 = new String("hello world");
          String str3 = "hello world";
          String str4 = new String("hello world");
           
          System.out.println(str1==str2);//false
          System.out.println(str1==str3);//true
          System.out.println(str2==str4);//false
      }
  }
  ~~~

  ​

- 后二者存在的意义：

  - 二者均是可变的，String是不可变的
  - StringBuilder是线程不安全的，而StringBuffer是线程安全的，内部采用Synchronize关键字进行同步；

- 区别：

  ~~~java
  public class Main {
      public static void main(String[] args) {
          String str1 = "I";
          //str1 += "love"+"java";        1)
          str1 = str1+"love"+"java";      //2)
           
      }
  }
  ~~~

  - 1方法要比2的效率高，因为编译器在前期会对1进行优化，直接一次append("lovejava")
  - 方法2则要进行2次的append()操作；