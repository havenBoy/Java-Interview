## 24种设计模式

* #### 一、创建型模式
    * ##### 抽象工厂模式(Abstract factory pattern)
    * ##### 生成器模式(Builder pattern)
    * ##### 工厂模式(factory method pattern)
    * ##### 原型模式(prototype pattern)
    * ##### 单例了模式(Singleton pattern)
        * 饿汉：饿汉是指在程序启动或者单例所在类加载的时候就加载了（能很好的保证线程的安全）
        ```java
           public class SingletonPatteran {
              
              private static final SingletonPatteran instance = new SingletonPatteran();
              private SingletonPatteran() {};
              
              public static SingletonPatteran getInstance() {
                  return instance;
              }
           }
        ```
        * 懒汉：懒汉是指需要使用的时候才创建加载初始化（需要很好的运用同步，在多线程下才能保证线程安全）
        ```java
           public class SingletonPatteran {
              
              private SingletonPatteran instance = null;
              private SingletonPatteran() {};
              
              public static SingletonPatteran getInstance() {
                  if (instance == null) {       
                      synchronized(instance.class) {
                          if (instance == null) {
                              return instance = new SingletonPatteran();
                          }
                      }
                  }
                  return instance;
              }
           }
        ```
    * ##### 多例模式(Multition pattern)

* #### 结构型模式
    * ##### 适配器模式(Adapter pattern)
    * ##### 桥接模式(Bridge pattern)
    * ##### 组合模式(composite pattern)
    * ##### 装饰者模式(decorator pattern)
    * ##### 外观模式(facade pattern)
    * ##### 亨元模式(Flyweight Pattern)
    * ##### 代理模式(Proxy pattern)

* #### 行为型模式
    * ##### 责任链模式(Chain of responsibility pattern)
    * ##### 命令模式(Command pattern)
    * ##### 解释器模式(Interpreter pattern)
    * ##### 迭代器模式(iterator pattern)
    * ##### 中介者模式(Mediator pattern) 
    * ##### 备忘录模式(Memento pattern)
    * ##### 观察者模式(observer pattern)
    * ##### 状态模式(State pattern)
    * ##### 策略模式(strategy pattern)
    * ##### 模板方法模式(Template pattern)
    * ##### 访问者模式(visitor pattern)


## 7大设计原则
* #### 单一原则【SINGLE RESPONSIBILITY PRINCIPLE】
    * 一个类负责一项职责.
* #### 里氏替换原则【LISKOV SUBSTITUTION PRINCIPLE】
    * 继承与派生的规则.
* #### 依赖倒置原则【DEPENDENCE INVERSION PRINCIPLE】
    * 高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。即针对接口编程，不要针对实现编程.
* #### 接口隔离原则【INTERFACE SEGREGATION PRINCIPLE】
    * 建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少.
* #### 迪米特法则【LOW OF DEMETER】
    * 低耦合，高内聚.
* #### 开闭原则【OPEN CLOSE PRINCIPLE】
    * 一个软件实体如类、模块和函数应该对扩展开放，对修改关闭.
* #### 组合/聚合复用原则【Composition/Aggregation Reuse Principle(CARP) 】
    * 尽量使用组合和聚合少使用继承的关系来达到复用的原则.