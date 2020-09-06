## Java 序列化与反序列化是什么？

Java 序列化是指把 Java 对象转换为 *字节序列* 的过程，而 Java 反序列化是指把字节序列恢复为 Java 对象的过程：

- **序列化：** 对象序列化的最主要的用处就是在 *传递* 和 *保存对象* 的时候，保证对象的完整性和可传递性。序列化是把对象转换成有序字节流，以便在网络上传输或者保存在本地文件中。核心作用是对象状态的保存与重建。
- **反序列化：** 客户端从文件中或网络上获得序列化后的对象字节流，根据字节流中所保存的对象状态及描述信息，通过反序列化重建对象。

所有可在网络上传输的对象都必须是可序列化的，比如 RMI（remote method invoke, 即远程方法调用），传入的参数或返回的对象都是可序列化的，否则会出错；所有需要保存到磁盘的 java 对象都必须是可序列化的。通常建议：程序创建的每个 JavaBean 类都实现 Serializeable 接口。

## 为什么需要序列化与反序列化？

为什么要序列化，那就是说一下序列化的好处喽，序列化有什么什么优点，所以我们要序列化。

### 对象序列化可以实现分布式对象。

主要应用例如：[RMI](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486403&idx=1&sn=3ec98d1553969ad38fbd225aef2c9807&chksm=eb538ef5dc2407e3ee906c378776baf473d2a27ae0e98ebd0970c1050eb2cb30a0467c84ceb5&scene=21#wechat_redirect)(即远程调用 Remote Method Invocation)要利用对象序列化运行远程主机上的服务，就像在本地机上运行对象时一样。

### java 对象序列化不仅保留一个对象的数据，而且递归保存对象引用的每个对象的数据。

可以将整个对象层次写入字节流中，可以保存在文件中或在网络连接上传递。利用对象序列化可以进行对象的 "深复制"，即复制对象本身及引用的对象本身。[序列化](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247484372&idx=1&sn=381af66344b4b50d033c38e51bcb0e21&chksm=eb5386e2dc240ff4fbee79af445ffeb3856da484a8f5206bcaf4531f0d510564943b213f948c&scene=21#wechat_redirect) 一个对象可能得到整个对象序列。

### 序列化可以将内存中的类写入文件或数据库中。

比如：将某个类序列化后存为文件，下次读取时只需将文件中的数据反序列化就可以将原先的类还原到内存中。也可以将类序列化为流数据进行传输。

总的来说就是将一个已经实例化的类转成文件存储，下次需要实例化的时候只要反序列化即可将类实例化到内存中并保留序列化时类中的所有变量和状态。[关于 Java 序列化你应该知道的一切](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247484372&idx=1&sn=381af66344b4b50d033c38e51bcb0e21&chksm=eb5386e2dc240ff4fbee79af445ffeb3856da484a8f5206bcaf4531f0d510564943b213f948c&scene=21#wechat_redirect)，推荐大家看下。

### 对象、文件、数据，有许多不同的格式，很难统一传输和保存。

序列化以后就都是字节流了，无论原来是什么东西，都能变成一样的东西，就可以进行通用的格式传输或保存，传输结束以后，要再次使用，就进行反序列化还原，这样对象还是对象，文件还是文件。

## 如何实现 Java 序列化与反序列化

如果需要将某个对象保存到磁盘上或者通过网络传输，那么这个类应该实现 **Serializable** 接口或者 **Externalizable** 接口之一。

### Serializable

#### 普通序列化

Serializable 接口是一个标记接口，不用实现任何方法。一旦实现了此接口，该类的对象就是可序列化的。

序列化步骤：

1. 创建一个 ObjectOutputStream 输出流；

2. 调用 ObjectOutputStream 对象的 writeObject 输出可序列化对象。

```java
public class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}


public class WriteObject {

    public static void main(String[] args) {
        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"));
            Person p = new Person("zhangsan", 20);
            oos.writeObject(p);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

反序列化步骤：

1. 创建一个 ObjectInputStream 输入流；

2. 调用 ObjectInputStream 对象的 readObject()得到序列化的对象。

```java
public class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        System.out.println("反序列化，你调用我了吗？");
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public class ReadObject {

    public static void main(String[] args) {
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("object.txt"));
            Person p = (Person) ois.readObject();
            System.out.println(p);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

// 运行结果
Person{name='zhangsan', age=20}
```

What??? 输出告诉我们，反序列化时并不会调用构造函数。反序列化的对象是由 JVM 自己生成的对象，**不通过构造函数构造。**

#### 成员是引用的序列化

如果一个可序列化的类的成员不是基本类型，也不是 String 类型，那这个引用类型也必须是可序列化的；否则，会导致此类不能序列化。

看例子，我们新增一个 Teacher 类。将 Person 去掉实现 Serializable 接口代码。

```java
public class Teacher implements Serializable {
    
    private String name;
    private Person person;

    public Teacher(String name, Person person) {
        this.name = name;
        this.person = person;
    }

    public static void main(String[] args) {
        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"));
            Person p = new Person("zhangsan", 20);
            Teacher t = new Teacher("lisi", p);
            oos.writeObject(t);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// 运行结果
java.io.NotSerializableException: cn.com.dotleo.serial.Person
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548)
	at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1509)
	at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1432)
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1178)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at cn.com.dotleo.serial.Teacher.main(Teacher.java:23)
```

我们看到程序直接报错，因为 Person 类的对象是不可序列化的，这导致了 Teacher 的对象不可序列化。

#### 同一对象序列化多次的机制

同一对象序列化多次，会将这个对象序列化多次吗？答案是 *否定* 的。

```java
public class WriteObject {

    public static void main(String[] args) {
        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("teacher.txt"));
            Person p = new Person("zhangsan", 20);
            Teacher t1 = new Teacher("lisi", p);
            Teacher t2 = new Teacher("wangwu", p);
            oos.writeObject(t1);
            oos.writeObject(t2);
            oos.writeObject(p);
            oos.writeObject(t2);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

依次将 t1、t2、person、t2 对象序列化到文件 teacher.txt 文件中。

**注意：反序列化的顺序与序列化时的顺序一致**。

```java
public class ReadObject {

    public static void main(String[] args) {
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("teacher.txt"));
            Teacher t1 = (Teacher) ois.readObject();
            Teacher t2 = (Teacher) ois.readObject();
            Person p = (Person) ois.readObject();
            Teacher t3 = (Teacher) ois.readObject();
            System.out.println(t1 == t2);
            System.out.println(t1.getPerson() == p);
            System.out.println(t2.getPerson() == p);
            System.out.println(t2 == t3);
            System.out.println(t1.getPerson() == t2.getPerson());
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

// 运行结果
false
true
true
true
true
```

从输出结果可以看出，**Java 序列化同一对象，并不会将此对象序列化多次得到多个对象。**

Java 序列化算法:

1. 所有保存到磁盘的对象都有一个序列化编码号
2. 当程序试图序列化一个对象时，会先检查此对象是否已经序列化过，只有此对象从未（在此虚拟机）被序列化过，才会将此对象序列化为字节序列输出。
3. 如果此对象已经序列化过，则直接输出编号即可。

#### Java 序列化算法潜在问题

由于 java 序利化算法不会重复序列化同一个对象，只会记录已序列化对象的编号。**如果序列化一个可变对象（对象内的内容可更改）后，更改了对象内容，再次序列化，并不会再次将此对象转换为字节序列，而只是保存序列化编号。**

```java
public class ReadObject {

    public static void main(String[] args) {
        ObjectInputStream ois = null;
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
            ois = new ObjectInputStream(new FileInputStream("person.txt"));

            Person p = new Person("zhangsan", 20);
            oos.writeObject(p);
            System.out.println(p);

            p.setName("wangwu");
            oos.writeObject(p);
            System.out.println(p);

            Person p1 = (Person) ois.readObject();
            Person p2 = (Person) ois.readObject();
            System.out.println(p1 == p2);
            System.out.println(p1.getName().equals(p2.getName()));

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

// 运行结果
Person{name='zhangsan', age=20}
Person{name='wangwu', age=20}
true
true
```

#### 可选的自定义序列化

- 有时，我们有这样的需求，某些属性不需要序列化。使用 *transient* 关键字后， *默认序列化机制* 就会忽略这个字段。  

```java
public class Person implements Serializable {
    private transient String name;
    private transient int age;
    private int height;
    private transient boolean singlehood;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
		
  	// 省略get/set方法
}

public class TransientTeset {

    public static void main(String[] args) {
        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
            ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.txt"));
            Person person = new Person("zhangsan", 20);
            person.setHeight(185);
            oos.writeObject(person);
            System.out.println(person);
            Person p1 = (Person) ois.readObject();
            System.out.println(p1);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

// 运行结果
Person{name='zhangsan', age=20, height=185, singlehood=false}
Person{name='null', age=0, height=185, singlehood=false}
```

从输出我们看到，**使用 transient 修饰的属性，java 序列化时，会忽略掉此字段，所以反序列化出的对象，被 transient 修饰的属性是默认值。对于引用类型，值是 null；基本类型，值是 0；boolean 类型，值是 false。**

- 使用 *transient* 虽然简单，但将此属性完全隔离在了序列化之外。Java 提供了 **可选的自定义序列化。** 可以进行控制序列化的方式，或者对序列化数据进行编码加密等。

```java
private void writeObject(java.io.ObjectOutputStream out) throws IOException;
private void readObject(java.io.ObjectIutputStream in) throws IOException,ClassNotFoundException;
private void readObjectNoData() throws ObjectStreamException;
```

通过重写 writeObject 与 readObject 方法，可以自己选择哪些属性需要序列化， 哪些属性不需要。

```java
public class Person implements Serializable {
    private transient String name;
    private int age;
    private int height;
    private transient boolean singlehood;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

     private void writeObject(ObjectOutputStream out) {
        try {
            out.defaultWriteObject();
            out.writeObject(this.name);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void readObject(ObjectInputStream ois) {
        try {
            ois.defaultReadObject();
            this.name = (String) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

public class TransientTeset {

    public static void main(String[] args) throws IOException, ClassNotFoundException {

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.txt"));
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.txt"));

        Person p = new Person("zhangsan", 20);
        p.setHeight(185);
        p.setSinglehood(true);
        oos.writeObject(p);
        System.out.println(p);

        Person p1 = (Person) ois.readObject();
        System.out.println(p1);

    }
}

// 运行结果
Person{name='zhangsan', age=20, height=185, singlehood=true}
Person{name='zhangsan', age=20, height=185, singlehood=false}
```

我们以 *writeObject*()为例进行说明：

1. 首先调用 `defaultWriteObject()` 来处理非 *transient* 字段，将它们依次字节化写入流中。
2. 然后调用自定义语句，根据需求序列化。

*readObject()* 中的内容也相似。



## 	SerialVersionUID



## 参考文章

1. [java 序列化，看这篇就够了](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#heading-10https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#heading-10)
2. [java 类中 serialversionuid 作用 是什么?举个例子说明](https://www.cnblogs.com/duanxz/p/3511695.html)
