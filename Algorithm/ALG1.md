### 递归算法

> 递归算法就是直接或者间接的调用自己的算法

- 常见的问题

  斐波那契数列，汉诺塔问题等

- 程序

  ```java
  //对于斐波那契数列的递归解法 
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

     考虑使用循环代替递归

  2. 返回值溢出

     ​