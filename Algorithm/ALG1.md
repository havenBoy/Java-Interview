### 递归算法

> 递归算法就是直接或者间接的调用自己的算法

- 常见的问题

  斐波那契数列，汉诺塔问题等

- 程序

  ```java
  //对于斐波那契数列的递归解法 
  //占用的空间大，效率低，会发生溢出
  public static int Fribonacci(int n){
     if(n<=2)
       return 1;
     else
       return Fribonacci(n-1)+Fribonacci(n-2);
  }
  //非递归解法
  public static int function fib(n){
      if(n < 1){
          throw new Error('invalid arguments');
      }
      if(n == 1 || n == 2){
          return 1;
      }
      int a = 1,
          b = 1,
          res = 0;
      for(int i=3;i<=n;i++){
          res = a + b;
          a = b;
          b = res;
      }
      return res;
  }
  ```

  ​

- 常见的问题

  1. 递归栈溢出

     考虑使用循环代替递归，例如斐波那契数列的非递归解法；

  2. 返回值溢出

     当n<47时使用基于int的计算方法；当n<93时可以使用基于long的计算方法；

     否则使用基于BigInteger

     ​