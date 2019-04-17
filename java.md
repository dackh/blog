# **基本类型**
|数据类型|大小/bit|
|--|--|
|byte |8位|
|short |16位|
|int |32位|
|long |64位|
|float|32位|
|double |64位|
|char|16位|
|boolean|1位|
# **Integer**
- new Integer(int)：返回一个全新的Integer对象。
- Integer.parseInt(String s):返回基本类型int
- Integer.valueOf(String s):返回一个Integer对象，会用到cache缓存（-128~127），效率可能比new Integer()要高。
- new Integer(String s)：源码即调用parseInt方法。

# **String**
**可变与不可变**
- String类中使用字符数组来保存字符串，因为有final修饰符，所以string对象是不可变的。一旦创建，就不能修改她的值，对于已经存在的String对象修改都是创建一个新的对象。

```
        private final char value[];

```

- StringBuffer和StringBuilder都是继承AbstractStringBuilder类，在类中也是使用字符数组来保存字符串，但没有final修饰，故可以改变。

```
        char[] value;

```

**是否线程安全**
- String类对象是不可变的，故线程安全。
- AbstractStringBuilder是StringBuffer跟StringBuilder的公共父类，定义了一些字符串的基本操作，如expandCapacity、append、insert、indexOf等方法。
- StringBuffer对方法加了同步锁或对调用的方法加了同步锁，所以是线程安全的。

```
       public   synchronized  StringBuffer reverse() {
           super .reverse();
           return   this ;
      }
      
       public   int  indexOf(String str) {
           return  indexOf(str, 0);         //存在 public synchronized int indexOf(String str, int fromIndex) 方法
      }
```
- StringBuilder并没有对方法进行加同步锁，所以是 非线程安全的 。



### **StringBuilder跟StringBuffer扩容**
当char[] 数组大小即capacity不足时，StringBuilder的扩容为原来的capacity*2+2；

![clipboard.png](![clipboard.png](https://segmentfault.com/img/bVbgCe0)
/img/bVbgCe0)

- length()方法返回已使用的长度
- capacity()方法返回char[]数组的长度
# **Object的通用方法**
### **概览**

```
public final native Class<?> getClass()

public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

protected void finalize() throws Throwable {}
```
### **equals()**
#### **与null的比较**
对任何不是null的对象x调用x.equals(null)结果都为false。

#### **equals()与==**
- 对于基本类型，== 判断两个值是否相等，基本类型没有equals()方法。
- 对于引用类型，==判断两个实例是否引用同一对象，而equals()判断对象的引用是否等价，根据对象引用equals()方法的具体实现来进行比较。

```
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```
#### **实现**
- 判断是否为同一对象的引用，如果是直接返回true。
- 判断是否是同一类型，如果不是直接返回false。
- 将Object实例进行转型。
- 判断每个关键域是否相等。

```
public class EqualExample {
    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

### **hashCode()**
hashCode()返回散列值，而equals()是用来判断两个实例是否等价，等价的两个实例散列值一定要相等，但是散列值相同的两个实例不一定等价。

在覆盖equals()方法时应当总是覆盖hashCode()方法，保证等价的两个实例散列值也相等。

下面代码中，新建了两个等价的实例，并将它们添加到HashSet中，我们希望将这两个实例当成一样的，只在集合中添加一个实例，但是因为EqualExample没有实现hashCode()方法，因此这两个实例的散列值是不相同的，最终导致集合添加了两个等价的实例。

```
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```
理想的散列函数应当具有均匀性，既不相等的实例应当分布在所有可能的散列值上。这就要求散列值要把所有的值都考虑进来，可以将每个域都当成R进制的某一位，然后组成一个R进程的整数。R一般取31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息会丢失，因为与2相乘相当于向左移一位。

一个数与 31 相乘可以转换成移位和减法：31*x == (x<<5)-x，编译器会自动进行这个优化。

```
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```
### **toString()**
 
默认返回ToStringExample@4554617c 这种形式，其中@后面的数值为散列值的无符号十六进制表示。

```
public class ToStringExample {
    private int number;

    public ToStringExample(int number) {
        this.number = number;
    }
}

ToStringExample example = new ToStringExample(123);
System.out.println(example.toString());

ToStringExample@4554617c
```

### **clone()**
#### **cloneable**
clone()是Object的protect方法，它不是public，一个类不显示区重写clone()，其他类就不能直接去调用该类实例的clone()方法。

```
public class CloneExample {
    private int a;
    private int b;
}
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```

重写clone()得到以下实现：

```
public class CloneExample {
    private int a;
    private int b;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}

CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}

java.lang.CloneNotSupportedException: CloneExample
```
以上抛出了CloneNotSupportedException，这个因为CloneExample没有实现Cloneable接口。

```
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```
应该注意的是，clone()方法并不是Cloneable接口的方法，而是Object的一个protect方法。Cloneable接口只是规定，如果一个类没有实现Cloneable接口又调用了clone()方法，就会抛出CloneNotSupportException。

#### **深拷贝和浅拷贝**
 - 浅拷贝：拷贝实例和原始实例的引用类型引用同一对象。
 - 深拷贝：拷贝实例和原始实例的引用类型引用不同对象。

```
public class ShallowCloneExample implements Cloneable {
    private int[] arr;

    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}


ShallowCloneExample e1 = new ShallowCloneExample();
ShallowCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 222
```

```
public class DeepCloneExample implements Cloneable {
    private int[] arr;

    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}


DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```
使用clone()方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。最好不要去使用clone()而是使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```
public class CloneConstructorExample {
    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

# **面向对象的三个特性**
### **封装**
利用抽象数据类型将数据和基本数据的操作封装在一起，使其构成一个不可分割的独立实体。数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口与外部发生联系。

### **继承**
继承实现了 IS-A 关系，例如 Cat 和 Animal 就是一种 IS-A 关系，因此 Cat 可以继承自 Animal，从而获得 Animal 非 private 的属性和方法。

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。


### **多态**
多态分为编译时多态和运行时多态，编译时多态指方法的重载，运行时多态指程序中定义的对象引用所指向的具体类型在运行时才确定。
运行时多态的三个条件：
- 继承
- 覆盖
- 向上转型

如：
乐器类（Instrument）有两个子类：Wind 和 Percussion，它们都覆盖了父类的 play() 方法，并且在 main() 方法中使用父类 Instrument 来引用 Wind 和 Percussion 对象。在 Instrument 引用调用 play() 方法时，会执行实际引用对象所在类的 play() 方法，而不是 Instrument 类的方法。

# **内部类**
每个内部里都能独立地继承一个（接口的）实现所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

在我们程序设计中有时候会存在一些使用接口很难解决的问题，这个时候我们可以利用内部类提供的、可以继承多个具体的或者抽象的类的能力来解决这个程序设计问题。

也就是说接口只是解决了部分问题，而内部类使用多重继承的解决方案变得更加完整。 
###[Java内部类详解][1]


# **泛型**
### **<T>跟<?>**
类型参数<T>，无解通配符<?>
- <T>：声明一个泛型类或者泛型方法，一种约束，保证类中通过T定义的参数都是同一类型。
- <?>：使用泛型类或泛型方法。

# **NIO**
新的输入/输出库是JDK1.4中引入的，弥补了原来的I/O的不足，提供了高速的、面向块的I/O.

### **流与块**
I/O跟NIO最重要的区别是数据打包和传输的方式，I/O以流的方式处理数据，而NIO以块的方式处理数据。

面向流的I/O一次处理一个字节数据，一个输入流产生一个字节数据，一个输出流产生一个字节数据。为流式创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责处理机制的一部分。不利的一面是，面向流的I/O通常相当慢。

面向块的I/O一个处理一个数据块，按块处理数据比按流处理数据要快得多。但是面向块的I/O缺少一些面向流的I/O所具有的优雅性和简单性。

I/O包和NIO已经很好地集成了，java.io.*已经以NIO为基础重新实现了，所以现在它可以利用NIO的一些特性，例如：java.io.*包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

### **通道和缓冲区**
#### **通道**
通道Channel是对原来I/O包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动（一个流必须是InputStream或者OutputStream的子类），而通道是双向的，可以用于读、写或者同时读写。

通道包括以下类型：
- FileChannel：从文件中读写数据。
- DatagramChannel：通过UDP读写网络中的数据。
- SocketChannel：通过TCP读写网络中的数据。
- ServerSocketChannel：可以监听新进来的TCP链接，对每一个新进来的连接都会创建一个SockerChannel。

#### **缓冲区**
发送给一个通道的所有数据都必须先经过缓冲区，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区的类型：
- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer




# **注解**
JDK5.0中的类型：类、接口、枚举、注解。注解是跟其他三种类型一样，都可以定义、使用，以及包含自己的属性。

### **注解的分类**
- 标记注解：注解内部没有属性，称作标记注解。
- 单值注解：注解内部只有一个属性，称为单值注解。使用方法：@注解名(属性名=属性值)或@注解名(属性值)
- 多值注解：注解内部有多个属性，称作多值注解。使用方法：@注解名(属性名=属性值,属性名=属性值)

### **元注解**
负责注解其他注解
#### **@Target**
作用：定义注解的使用范围
取值：(ElementType)
- CONSTRUCTOR：用于描述构造器
- FIELD：成员变量
- LOCAL_VARIABLE：局部变量
- METHOD：方法
- PACKAGE：包
- PARAAMETER：参数
- TYPE：类、接口、枚举
#### **@Tetantion**
作用：该注解的保留时间
取值：(RetantionPoicy)
- SOURCE：源文件保留
- CLASS：class文件保留
- RUNTIME：运行是保留
#### **@Documented**
作用：用于描述其他类型的注解应该被标注为程序成员的API，因此可以被javadoc此类的工具文档化。是一个标记注解，没有成员。
#### **@Inherited**
作用：@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。 



# **Java序列化**
参考：[Java序列化 | 菜鸟教程][2]

# **Java网络IO**
不是特别了解，再补上

# **ArrayList如何扩容**
默认数组大小为10，当超出数组大小时，1.5扩容，vector扩容元数组的2倍。

# **JDK动态代理跟CGLIB代理的区别**
### **JDK动态代理**
代理对象和目标对象实现了相同的接口，目标对象作为代理对象的一个属性，具体的实现接口中，可以在调用目标对象相应方法前后加上其他业务处理逻辑。
### **GCLIB动态代理**
CGLIB代理是针对类实现代理。
主要是对指定的类生成一个子类，覆盖起所有方法，所以该类或方法不能声明为final。
### **应用**
AOP（Aspect-OrientedProgramming，面向切面编程），AOP包括切面（aspect）、通知（advice）、连接点（joinpoint），实现方式就是通过对目标对象的代理在连接点前后加入通知，完成统一的切面操作。

参考：[JDK和CGLIB生成动态代理类的区别][3]


  [1]: https://www.cnblogs.com/dolphin0520/p/3811445.html
  [2]: http://www.runoob.com/java/java-serialization.html
  [3]: http://www.cnblogs.com/binyue/p/4519652.html
