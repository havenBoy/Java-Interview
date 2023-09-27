### 序列化与反序列化

> 总体来说，序列化就是保存，把对象转化为字节序列，反序列化就是读取，把字节序列恢复为对象；

* 作用：

  - 把对象的字节序列保存在硬盘上，存放在文件中，需要的时候再从硬盘中还原为原对象；
  - 在网络上传输字节流，网络上的信息传输还是以二进制的字节流为基础的；

* 代码演示：

  首先创建一个对象Person，使其实现Serializer接口，以下代码的作用是先把Person 对象序列化到D盘，然后再从D盘中读取出对象；

  ~~~java
  	public static void SerializePer() throws Exception {
  		ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(new File("D://person.txt")));
  		objectOutputStream.writeObject(new Person("xiong",10,"zhuhai"));
  		System.out.println("序列化成功-------");
  		objectOutputStream.close();
  	}
  	
  	public static Person DeSerializePer() throws Exception {
  		ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(new File("D://person.txt")));
  		Person person = (Person) objectInputStream.readObject();
  		System.out.println("Person反序列化成功-----");
  		objectInputStream.close();
  		return person;
  	}
  	
  	public static void main(String[] args) throws Exception {
  		SerializePer();
  		Person person = DeSerializePer();
  		System.out.println(person.getAddress());
  	}
  ~~~

* serialVersionUID的作用

  如果有需求在序列化后，增加一个方法或者属性，自己指定serialVersionUID，就能保证在反序列化的时候可以正常还原对象，如果不加此ID值的话，就会出现InvalidClassException的错误；类似于一个加密的文件，如果拿不到这个秘钥，就会报错；

* 实现序列化的2种方式

  * 实现Serializable接口，调用writeObject方法，把对象转化为字节存储在文件中，调用readObject方法，把存储在文件中的字节流转化为对象；
  * 实现Externalnalizable接口，其中的writeExternal和readExternal方法与上述的相似，其中writeExternal方法需要手动指定需要序列化的属性；

* 注意：

  **static和transient字段不能被序列化**？

  - 首先关键字transient是为了对象在序列化不想被序列化的属性值，有可能是因为这个字段涉及到密码或者秘钥等私密性问题，这个关键字就是保证了该字段不能被序列化，只能用来修饰属性，不能用来修饰方法或者类；


  - 序列化本身就是针对堆内存，不能针对方法区内存，所以static也是不能被序列化的；

* 序列化会破坏对象的单例

  * 在对象的序列化与反序列化操作中，会创建出一个新的对象；
  * 在调用readObject方法时，底层使用反射的技术，使用对象的无参构造器new出一个新的对象；

* 序列化为什么是不安全的？

  序列化只是把对象简单的转化为二进制存储，过程完全可逆，所有的字段或者属性在传输过程中都是明文；