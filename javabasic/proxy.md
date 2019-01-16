### 代理

> 本文主要介绍动态代理与静态代理，分析其异同

- 代理模式
  - 代理类与委托类具有相同的接口，代理类为委托类过滤消息，转发消息给委托类，以及后续的消息处理
  - 代理类的对象与委托类的对象相互管理，但代理类的本身是不会去真正实现服务，是代理类调用委托类对应的方法来提供服务


- 静态代理

  - 由程序员创建或者特定工具生成，代理类的class文件已经生成；

  - 示例：学生给老师交作业，由课代表收齐然后交给老师，这里课代表就是学生的代理类

    ~~~java
    -------Person--------------
    interface Person {
    	public void giveHomeWork();
    }
    --------Student------------
    public class Student implements Person{
    	private String name;

    	public Student(String name) {
    		this.name = name;
    	}
    	@Override
    	public void giveHomeWork() {
    		System.out.println(this.name + "上交班费");
    	}
    }
    ----StudengProxy------------
    public class StudentProxy implements Person{
    	
    	private Student student;
    	
    	public StudentProxy(Person student) {
    		if(student.getClass() == Student.class)
    			this.student = (Student)student;
    	}

    	@Override
    	public void giveHomeWork() {
            //这里可以写实现方法之前需要完成的方法
    		student.giveHomeWork();
            //这里可以写实现方法之后需要完成的方法
    	}
        //测试
        public static void main(String[] args) {
    		Student student = new Student("ming");
    		Person studentProxy = new StudentProxy(student);
    		studentProxy.giveHomeWork();
    	}
    }
    ~~~

  - 可以看出在实现代理方法之前或者之后如果需要实现某种方法的话，是非常简单的；

- 动态代理

  - 程序在运行中创立的代理方式就是动态代理，优点就是可以在代理类中统一对函数处理，而不用一一的添加对应的方法

  - 代码实现： 代理类需要实现InvocationHandler接口

    ~~~java
    public class StudentInvocation<T> implements InvocationHandler {
    	
    	private T target;
    	
        public StudentInvocation(T target) {
    		this.target = target;
    	}

    	@Override
    	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    		Object result = method.invoke(target, args);
    		return result;
    	}
    	
    	public static void main(String[] args) {
    		Person student = new Student("ming");
    		InvocationHandler studentHandler = new StudentInvocation<Person>(student);
    		Person studengProxy = (Person)           Proxy.newProxyInstance(Person.class.getClassLoader(),  new Class<?>{Person.class}, studentHandler);
    		studengProxy.giveMoney();
    	}
    }
    ~~~

    ​