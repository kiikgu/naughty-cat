---
title: serialize和deserialize
date: 2017-11-03 08:15:53
categories: [java]
tags: [序列化,反序列化,Serializable]
toc: true

---

主要介绍Java解序列化和反序列化基础知识
<!--more-->

## Java序列化和反序列化

### 怎么实现java对象的序列化和反序列化

#### 实现java.io.Serializable接口
* 没有实现Serializable接口的类，它们的状态不会被序列化和反序列化的
* 可序列化类的所有子类都是可序列化和可反序列化的
* 不可序列化类的子类，如果实现了Serializable接口，那么子类负责保存和恢复父类对象的域，且父类必须要包含无参构造函数，用于反序列化时初始化父类对象的域

```
public class Person {
    private String name;
    private int age;
    
    public Person() {
        this.name = "new";
        this.age = 5;
    }
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    //get/set方法
}

public class Student extends Person implements Serializable {

    private String className;

    public Student(String name, int age, String className) {
        super(name, age);
        this.className = className;
    }
    //get/set方法
}

//序列化、反序列化测试
public static void test2() {
        Student s = new Student("p1", 10, "2");
        doSerialize(s, "student.txt");
        Student s1 = doDeserialize("student.txt");
        System.out.println(s1);
 }

 public static <T> void doSerialize(T obj, String file) {
        try {
            FileOutputStream fos = new FileOutputStream("./" + file);
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(obj);
            oos.flush();
            oos.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```
序列化结果：

![序列化结果](/img/20171103/serializable001.png)

反序列化结果：

```
Student{className='2'Person{name='new', age=5}}
```

从结果中看到，Person类不是可序列化的，所以在子类对象序列化时Person对象的状态没有保存下来；子类对象反序列化时，调用了Person的无参构造函数来初始化父类对象。

如果父类不提供无参构造函数，那么反序列化时会抛出运行时异常，java.io.InvalidClassException。

#### writeObject()
对象序列化时可以通过writeObject方法实现特殊化处理，类必须实现writeObject方法：

```
private void writeObject(java.io.ObjectOutputStream out) 
	throws IOException
```

**示例**

```
private void writeObject(ObjectOutputStream outputStream) throws IOException {
    outputStream.defaultWriteObject();
    outputStream.writeObject("hello world");
}
```

**结果**

![writeObject结果](/img/20171103/serializable002.png)

从序列化结果中可以看到，除了保存对象的状态，额外的“hello world”字符串也被保存下来。writeObject方法传入ObjectOutputStream对象，通过调用defaultWriteObject方法保存对象的非static、非transient变量；static变量不能序列化和反序列化。

#### readObject()

readObject是在对象反序列化时，实现特殊处理的方法，类必须实现readObject方法：

```
private void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException;
```

**示例**

```
private void readObject(ObjectInputStream inputStream) 
	throws IOException, ClassNotFoundException {
    inputStream.defaultReadObject();
    System.out.println(inputStream.readObject());
}

结果：
hello world
```
同样readObject传入ObjectInputStream对象，实现对象非static、非transient对象的恢复。

#### readObjectNoData()
readObjectNoData用作处理反序列化时出现的2种情况：

* 类定义发生变化，继承了新的类
* 序列化信息被篡改

**序列化Person对象**

```
public class Person implements Serializable {
    private static final long serialVersionUID = 6767364611710948742L;

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    ...
}


执行序列化：
Person p = new Person("p1", 10);
doSerialize(p, "person.txt");

public static <T> void doSerialize(T obj, String file) {
    try {
        FileOutputStream fos = new FileOutputStream("./" + file);
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(obj);
        oos.flush();
        oos.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**Person类继承新类Life**

```
public class Life implements Serializable {
    private String type;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
    
    private void readObjectNoData() {
        this.type = "people";
    }
}

public class Person extends Life {
	...
}

执行反序列化：
Person p = doDeserialize("person.txt");
System.out.println(JSON.toJSONString(p, SerializerFeature.WriteMapNullValue));


执行结果：
{"age":10,"name":"p1","type":"people"}
```
Person继承了新类Life，但是旧的Person对象序列化结果中没有Life对象的信息，所以在进行反序列化时Life中的变量可以通过readObjectNoData方法来进行初始化。

#### writeReplace()
用作替换序列化对象，需要实现方法：

```
ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
```

**示例**
```
public class Person implements Serializable {
    private static final long serialVersionUID = 6767364611710948742L;

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }


    public Object writeReplace() {
        return 1;
    }
}
```

**序列化结果**

![writeReplace结果](/img/20171103/serializable003.png)

序列化后保存的对象已经不是Person对象，而是被替换成1，此时这个序列化是不可逆转的，不会被反序列化为Person对象。如果实现了writeReplace方法，则writeObject、readObject方法不会生效。
为什么要有这个方法？

#### readResolve()

用作替换反序列化对象，可以保护性恢复单例和枚举值，需要实现方法：

```
ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
```

**示例**

```
public class Color implements Serializable {
    private int value;

    Color(int value) {
        this.value = value;
    }

    public static final Color RED = new Color(1);
    public static final Color BLUE = new Color(2);
}

测试
Color red = Color.RED;
doSerialize(red, "color.txt");
Color red1 = doDeserialize("color.txt");
System.out.println(red == red1);

结果输出：false
```

我们需要实现readResove方法，在反序列化时对单例做保护性恢复

```
public class Color implements Serializable {
    private int value;

    Color(int value) {
        this.value = value;
    }

    public static final Color RED = new Color(1);
    public static final Color BLUE = new Color(2);

    public Object readResolve() {
        if (value == 1) {
            return RED;
        } else if (value == 2) {
            return BLUE;
        }
        return null;
    }
}
```

#### serialVersionUID
在序列化时，每个序列化的对象都会关联一个serialVersionUID，在反序列化时会校验接收类的serialVersionUID和序列化类的serialVersionUID是否一致，如果不一致则抛出 InvalidClassException。

如果没有显式指定serialVersionUID，序列化时会自动生成一个serialVersionUID，同一个类，经过不同编译器进行编译后，自动生成的serialVersionUID可能不同，所以最好实践是显式指定一个serialVersionUID。


### 总结

本文主要介绍了Java的序列化和反序列化，及相关的一些方法的使用。

### 参考

[https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)

[http://blog.csdn.net/lirx_tech/article/details/51303966](http://blog.csdn.net/lirx_tech/article/details/51303966)

[http://blog.csdn.net/dont27/article/details/38309061](http://blog.csdn.net/dont27/article/details/38309061)

[http://blog.csdn.net/yangxiangyuibm/article/details/43227457](http://blog.csdn.net/yangxiangyuibm/article/details/43227457)

版权声明：本文为博主原创文章，转载请注明出处