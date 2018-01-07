+++
date = "2015-03-27T14:15:22+08:00"
menu = "main"
tags = ["java"]
title = "关于java的序列化"

+++

##### 关于java的序列化

今天面试官问到 java 的一些高级特性，其中就有序列化这一知识点，通俗的来讲，java 中的序列化就是将 java 中的对象变成字节序列，以方便永久存储或者进行网络通信，同样的，java 的反序列话就是将那些字节序列恢复为对象。

被序列化的类必须继承 Serializable 或者实现 Externalizable 接口。在 java 中有一些对象是无法被序列化的，比如说 Thread 和 Runnable 之类的，如果不在他们前面加上 transient 关键字程序就会报错。 还有就是有些人希望对象中的一些成员不要被序列化，比如说密码之类的。 还有就是static关键字不能别序列化。 所以就有了一下两个概念：控制序列化和控制反序列化。

对于 Serializable 来讲，要被序列化的类需要继承这个类，（重写一些方法？），不想被序列化或者不能别序列化的成员变量使用 transient 关键字进行修饰。而对于 Externalizable 这个接口来讲，需要被序列化的对象实现这个方法，并且重写 `writeExternal(ObjectInput input)（控制序列化）` 和 `readExternal(ObjectOutput output)（控制反序列化）` 。

##### Serializable 使用

枚举类型，性别

	public enum Gender{
		MALE, FEMALE
	}

能够序列化的类，人

	public class Person implements Serializable {
	
	    private String name = null;
	
	    private Integer age = null;
	
	    private Gender gender = null;
	
	    public Person() {
	        System.out.println("none-arg constructor");
	    }
	
	    public Person(String name, Integer age, Gender gender) {
	        System.out.println("arg constructor");
	        this.name = name;
	        this.age = age;
	        this.gender = gender;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public Integer getAge() {
	        return age;
	    }
	
	    public void setAge(Integer age) {
	        this.age = age;
	    }
	
	    public Gender getGender() {
	        return gender;
	    }
	
	    public void setGender(Gender gender) {
	        this.gender = gender;
	    }
	
	    @Override
	    public String toString() {
	        return "[" + name + ", " + age + ", " + gender + "]";
	    }
	}

使用 SimpleSerail 进行序列化，先将 Person 对象保存到 person.out 中，然后再从该文件中读出被存储的 Person 对象，并打印出来

	public class SimpleSerial {
	
	    public static void main(String[] args) throws Exception {
	        File file = new File("person.out");
	
	        ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
	        Person person = new Person("John", 101, Gender.MALE);
	        oout.writeObject(person);
	        oout.close();
	
	        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
	        Object newPerson = oin.readObject(); // 没有强制转换到Person类型
	        oin.close();
	        System.out.println(newPerson);
	    }
	}

##### Why

一个类实现了 Serializable 接口之后就可以被序列化，在上述的实例中，我们使用 ObjectOutputStream 来序列化对象， 部分代码如下

	private void writeObject0(Object obj, boolean unshared) throws IOException {
	    
	    if (obj instanceof String) {
	        writeString((String) obj, unshared);
	    } else if (cl.isArray()) {
	        writeArray(obj, desc, unshared);
	    } else if (obj instanceof Enum) {
	        writeEnum((Enum) obj, desc, unshared);
	    } else if (obj instanceof Serializable) {
	        writeOrdinaryObject(obj, desc, unshared);
	    } else {
	        if (extendedDebugInfo) {
	            throw new NotSerializableException(cl.getName() + "\n"
	                    + debugInfoStack.toString());
	        } else {
	            throw new NotSerializableException(cl.getName());
	        }
	    }
	    
	}

从上述代码我们可以得知，当被写的对象是 String, Array, Enum 或者 Serializable时， 会对该对象进行持久化，否则会抛出 `NotSerializableException`

##### trasient 关键字

如果让一个类 X 去实现 Serializable， 而不对它进行处理的话，当对 X 进行序列化时，X 所引用到的对象也会被序列化，所以当 X 的对象中包含容器时，序列化的开销很大的。 

所以我们使用 transient 关键字过滤掉部分敏感数据

	public class Person {
		...
		transient private Integer age = null;
		...
	}

上述的实例将不会把 age 进行序列化

##### writeObject() 和 readObject() 函数

这两个函数可以控制部分序列化

	public class Person implements Serializable {
	    
	    transient private Integer age = null；
	
	    private void writeObject(ObjectOutputStream out) throws IOException {
	        out.defaultWriteObject();
	        out.writeInt(age);
	    }
	
	    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
	        in.defaultReadObject();
	        age = in.readInt();
	    }
	}

在 `writeObject()` 方法中会先调用 ObjectOutputStream 中的 ` defaultWriteObject()` 方法，该方法会执行默认的序列化机制，然后再调用 `writeInt()` 方法显式的将 age 字段写到 ObejctOutputStream 中，`readObject()` 的作用则是针对对象的兑取，其原理与 `writeObject()` 方法相同。

`writeObject()` 和 `readObject` 都是 private 方法，它们是使用反射被调用的。（详情可见 ObjectOutputStream 中的  `writeSerialData()` 方法，以及ObjectInputStream中的 `readSerialData()` 方法）。

##### Externalizable接口

Externalizable 是 java 提供的另一个可以实现序列化的类，当一个类 X 实现 Externalizable 接口时，需要重写的方法是 `readExternal()` 和 `writeExternal()` 方法，而且没有默认的序列化机制，所有的序列化细节都需要自己去完成。

另外当使用 Externalizable 进行序列化。 当读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后将被保存的字段的值分别填充在新的对象中。 因此我们实现 Externalizale 接口的类必须提供一个无参的构造器，且访问权限是 public 。

	public class Person implements Externalizable {
	
	    private String name = null;
	
	    transient private Integer age = null;
	
	    private Gender gender = null;
	
	    public Person() {
	        System.out.println("none-arg constructor");
	    }
	
	    public Person(String name, Integer age, Gender gender) {
	        System.out.println("arg constructor");
	        this.name = name;
	        this.age = age;
	        this.gender = gender;
	    }
	
	    private void writeObject(ObjectOutputStream out) throws IOException {
	        out.defaultWriteObject();
	        out.writeInt(age);
	    }
	
	    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
	        in.defaultReadObject();
	        age = in.readInt();
	    }
	
	    @Override
	    public void writeExternal(ObjectOutput out) throws IOException {
	        out.writeObject(name);
	        out.writeInt(age);
	    }
	
	    @Override
	    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
	        name = (String) in.readObject();
	        age = in.readInt();
	    }
	    
	}

在上述的程序中我们虽然使用了 `writeObject()` 和 `readObject()` 函数，但是并没有什么乱用，使用 SimpleSerial 类打印出来的结果还是

	arg constructor
	none-arg constructor
	[John, 31, null]

##### readResolve() 方法

当我们使用单例模式时，我们希望这个类的对象是唯一的，如果这个类又是可序列化的，就需要更加复杂一些。

正常的单例模式

	public class Person implements Serializable {
	
	    private static class InstanceHolder {
	        private static final Person instatnce = new Person("John", 31, Gender.MALE);
	    }
	
	    public static Person getInstance() {
	        return InstanceHolder.instatnce;
	    }
	
	    private String name = null;
	
	    private Integer age = null;
	
	    private Gender gender = null;
	
	    private Person() {
	        System.out.println("none-arg constructor");
	    }
	
	    private Person(String name, Integer age, Gender gender) {
	        System.out.println("arg constructor");
	        this.name = name;
	        this.age = age;
	        this.gender = gender;
	    }
	    
	}

同时我们修改 SimpleSerial 应用，使得能够保存/获取上述单例对象，并对对象的相等性进行比较，代码如下

	public class SimpleSerial {
	
	    public static void main(String[] args) throws Exception {
	        File file = new File("person.out");
	        ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
	        oout.writeObject(Person.getInstance()); // 保存单例对象
	        oout.close();
	
	        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
	        Object newPerson = oin.readObject();
	        oin.close();
	        System.out.println(newPerson);
	
	        System.out.println(Person.getInstance() == newPerson); // 将获取的对象与Person类中的单例对象进行相等性比较
	    }
	}

结果如下

	arg constructor
	[John, 31, MALE]
	false

因此我们得知，从文件 person.out 中获取的 Person 对象与 Person 类中的单例对象并不相等。 为了能在序列化中仍然保持单例的特性，可以在 Person 类中添加一个 `readResolve()` 方法，在该方法中直接返回 Person 的单例对象。

	public class Person implements Serializable {
	
	    private static class InstanceHolder {
	        private static final Person instatnce = new Person("John", 31, Gender.MALE);
	    }
	
	    public static Person getInstance() {
	        return InstanceHolder.instatnce;
	    }
	
	    private String name = null;
	
	    private Integer age = null;
	
	    private Gender gender = null;
	
	    private Person() {
	        System.out.println("none-arg constructor");
	    }
	
	    private Person(String name, Integer age, Gender gender) {
	        System.out.println("arg constructor");
	        this.name = name;
	        this.age = age;
	        this.gender = gender;
	    }
	
	    private Object readResolve() throws ObjectStreamException {
	        return InstanceHolder.instatnce;
	    }
	    
	}

再次使用 SimpleSerial 输出后结果如下

	arg constructor
	[John, 31, MALE]
	true

无论是实现 Serializable 接口，还是实现 Externalizable 接口，当从 I/O 中读取对象时， `readResolve()` 方法总会被调用到。 实际上就是用 `readResolve()` 方法返回的对象替换在反序列化过程中创建的对象，而被创建的对象则会被垃圾回收掉。