 

## 一.什么是不可变对象

　　下面是《Effective Java》这本书对于不可变对象的定义：

```
不可变对象(Immutable Object)：对象一旦被创建后，对象所有的状态及属性在其生命周期内不会发生任何变化。
```

　　从不可变对象的定义来看，其实比较简单，就是一个对象在创建后，不能对该对象进行任何更改。比如下面这段代码：

```java
public class ImmutableObject {
    private int value;    
    public ImmutableObject(int value) {    
        this.value = value; 
    } 
    public int getValue() {  
        return this.value;
    }
}
```

　　由于ImmutableObject不提供任何setter方法，并且成员变量value是基本数据类型，getter方法返回的是value的拷贝，所以一旦ImmutableObject实例被创建后，该实例的状态无法再进行更改，因此该类具备不可变性。

　　再比如我们平时用的最多的String：

```java
public class Test {  
    public static void main(String[] args) {   
        String str = "I love java"; 
        String str1 = str;    
        System.out.println("after replace str:" + str.replace("java", "Java"));   
        System.out.println("after replace str1:" + str1); 
    }
}
```

　　输出结果：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084121154-470480675.png)

　　从输出结果可以看出，在对str进行了字符串替换替换之后，str1指向的字符串对象仍然没有发生变化。

## 二.深入理解不可变性

　　我们是否考虑过一个问题：假如Java中的String、包装器类设计成可变的ok么？如果String对象可变了，会带来哪些问题？

　　我们这一节主要来聊聊不可变对象存在的意义。

#### 1）让并发编程变得更简单

　　说到并发编程，可能很多朋友都会觉得最苦恼的事情就是如何处理共享资源的互斥访问，可能稍不留神，就会导致代码上线后出现莫名其妙的问题，并且大部分并发问题都不是太容易进行定位和复现。所以即使是非常有经验的程序员，在进行并发编程时，也会非常的小心，内心如履薄冰。

　　大多数情况下，对于资源互斥访问的场景，都是采用加锁的方式来实现对资源的串行访问，来保证并发安全，如synchronize关键字，Lock锁等。但是这种方案最大的一个难点在于：在进行加锁和解锁时需要非常地慎重。如果加锁或者解锁时机稍有一点偏差，就可能会引发重大问题，然而这个问题Java编译器无法发现，在进行单元测试、集成测试时可能也发现不了，甚至程序上线后也能正常运行，但是可能突然在某一天，它就莫名其妙地出现了。

　　既然采用串行方式来访问共享资源这么容易出现问题，那么有没有其他办法来解决呢？

　　事实上，引起线程安全问题的根本原因在于：多个线程需要同时访问同一个共享资源。

　　假如没有共享资源，那么多线程安全问题就自然解决了，Java中提供的ThreadLocal机制就是采取的这种思想。

　　然而大多数时候，线程间是需要使用共享资源互通信息的，如果共享资源在创建之后就完全不再变更，如同一个常量，而多个线程间并发读取该共享资源是不会存在线上安全问题的，因为所有线程无论何时读取该共享资源，总是能获取到一致的、完整的资源状态。

　　不可变对象就是这样一种在创建之后就不再变更的对象，这种特性使得它们天生支持线程安全，让并发编程变得更简单。

　　我们来看一个例子，这个例子来源于：http://ifeve.com/immutable-objects/

```java
public class SynchronizedRGB { 
    private int red; // 颜色对应的红色值 
    private int green; // 颜色对应的绿色值
    private int blue; // 颜色对应的蓝色值 
    private String name; // 颜色名称  
    private void check(int red, int green, int blue) {   
        if (red < 0 || red > 255 || green < 0 || green > 255        || blue < 0 || blue > 255) { 
            throw new IllegalArgumentException();  
        } 
    }  
    public SynchronizedRGB(int red, int green, int blue, String name) {  
        check(red, green, blue); 
        this.red = red;  
        this.green = green;  
        this.blue = blue;   
        this.name = name;
    }  
    public void set(int red, int green, int blue, String name) { 
        check(red, green, blue); 
        synchronized (this) {    
            this.red = red;    
            this.green = green;    
            this.blue = blue;    
            this.name = name; 
        } 
    }  
    public synchronized int getRGB() {   
        return ((red << 16) | (green << 8) | blue); 
    }  
    public synchronized String getName() {  
        return name;  
    }
}
```

　　例如一个有个线程1执行了以下代码：

```java
SynchronizedRGB color = new SynchronizedRGB(0, 0, 0, "Pitch Black");
int myColorInt = color.getRGB();   // Statement1
String myColorName = color.getName(); // Statement2
```

　　然后有另外一个线程2在Statement 1之后、Statement 2之前调用了color.set方法：

```
color.set(0, 255, 0, "Green");
```

　　那么在线程1中变量myColorInt的值和myColorName的值就会不匹配。为了避免出现这样的结果，必须要像下面这样把这两条语句绑定到一块执行：

```java
synchronized (color) { 
    int myColorInt = color.getRGB();  
    String myColorName = color.getName();
}
```

　　假如SynchronizedRGB是不可变类，那么就不会出现这个问题，比如将SynchronizedRGB改成下面这种实现方式：

```java
public class ImmutableRGB { 
    private int red;
    private int green;
    private int blue;  
    private String name;
    private void check(int red, int green, int blue) {
        if (red < 0 || red > 255 || green < 0 || green > 255        || blue < 0 || blue > 255) {  
            throw new IllegalArgumentException();   
        }
    } 
    public ImmutableRGB(int red, int green, int blue, String name) {  
        check(red, green, blue);
        this.red = red;   
        this.green = green; 
        this.blue = blue; 
        this.name = name; 
    }  
    public ImmutableRGB set(int red, int green, int blue, String name) {    
        return new ImmutableRGB(red, green, blue, name); 
    }  
    public int getRGB() {  
        return ((red << 16) | (green << 8) | blue);  
    }  
    public String getName() {  
        return name; 
    }
}
```

　　由于set方法并没有改变原来的对象，而是新创建了一个对象，所以无论线程1或者线程2怎么调用set方法，都不会出现并发访问导致的数据不一致的问题。

### 2）消除副作用

　　很多时候一些很严重的bug是由于一个很小的副作用引起的，并且由于副作用通常不容易被察觉，所以很难在编写代码以及代码review过程中发现，并且即使发现了也可能会花费很大的精力才能定位出来。

　　举个简单的例子：

```java
class Person { 
    private int age;  // 年龄 
    private String identityCardID; // 身份证号码 
    public int getAge() {   
        return age;
    }  
    public void setAge(int age) {   
        this.age = age; 
    } 
    public String getIdentityCardID() {  
        return identityCardID;
    } 
    public void setIdentityCardID(String identityCardID) {
        this.identityCardID = identityCardID; 
    }
} 
public class Test { 
    public static void main(String[] args) {  
        Person jack = new Person();  
        jack.setAge(101);  
        jack.setIdentityCardID("42118220090315234X");  
        System.out.println(validAge(jack));　　　　　　　　// 后续使用可能没有察觉到jack的age被修改了　
        // 为后续埋下了不容易察觉的问题   
    }  
    public static boolean validAge(Person person) {   
        if (person.getAge() >= 100) {  
            person.setAge(100); // 此处产生了副作用     
            return false; 
        }  
        return true;
    } 
}
```

　　validAge函数本身只是对age大小进行判断，但是在这个函数里面有一个副作用，就是对参数person指向的对象进行了修改，导致在外部的jack指向的对象也发生了变化。

　　如果Person对象是不可变的，在validAge函数中是无法对参数person进行修改的，从而避免了validAge出现副作用，减少了出错的概率。

### 3）减少容器使用过程出错的概率

　　我们在使用HashSet时，如果HashSet中元素对象的状态可变，就会出现元素丢失的情况，比如下面这个例子：

```java
class Person { 
    private int age;  // 年龄
    private String identityCardID; // 身份证号码  
    public int getAge() {    
        return age;
    }  
    public void setAge(int age) {  
        this.age = age; 
    }   
    public String getIdentityCardID() {   
        return identityCardID;
    } 
    public void setIdentityCardID(String identityCardID) { 
        this.identityCardID = identityCardID; 
    } 
    @Override  
    public boolean equals(Object obj) { 
        if (obj == null) { 
            return false;  
        }   
        if (!(obj instanceof Person)) {  
            return false; 
        }    
        Person personObj = (Person) obj;    
        return this.age == personObj.getAge() && this.identityCardID.equals(personObj.getIdentityCardID());  	}  
    @Override  
    public int hashCode() { 
        return age * 37 + identityCardID.hashCode(); 
    }
} 
public class Test { 
    public static void main(String[] args) { 
        Person jack = new Person();   
        jack.setAge(10);   
        jack.setIdentityCardID("42118220090315234X");
        Set<Person> personSet = new HashSet<Person>();   
        personSet.add(jack);   
        jack.setAge(11);   
        System.out.println(personSet.contains(jack));   
    }
}
```

　输出结果：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084552544-2140901315.png)

　　所以在Java中，对于String、包装器这些类，我们经常会用他们来作为HashMap的key，试想一下如果这些类是可变的，将会发生什么？后果不可预知，这将会大大增加Java代码编写的难度。

## 三.如何创建不可变对象

　　通常来说，创建不可变类原则有以下几条：

　　**1）所有成员变量必须是private**

　　**2）最好同时用final修饰(非必须)**

　　3**）不提供能够修改原有对象状态的方法**

- - 最常见的方式是不提供setter方法
  - 如果提供修改方法，需要新创建一个对象，并在新创建的对象上进行修改

　　4**）通过构造器初始化所有成员变量，引用类型的成员变量必须进行深拷贝(deep copy)**

　　5**）getter方法不能对外泄露this引用以及成员变量的引用**

　　**6）最好不允许类被继承(非必须)**

　　JDK中提供了一系列方法方便我们创建不可变集合，如：

```
Collections.unmodifiableList(List<? extends T> list)
```

　　另外，在Google的Guava包中也提供了一系列方法来创建不可变集合，如：

```
ImmutableList.copyOf(list)
```

　　这2种方式虽然都能创建不可变list，但是两者是有区别的，JDK自带提供的方式实际上创建出来的不是真正意义上的不可变集合，看unmodifiableList方法的实现就知道了：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084704863-1373867856.png)

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084714300-1627698632.png)

　　可以看出，实际上UnmodifiableList是将入参list的引用复制了一份，同时将所有的修改方法抛出UnsupportedOperationException。因此如果在外部修改了入参list，实际上会影响到UnmodifiableList，而Guava包提供的ImmutableList是真正意义上的不可变集合，它实际上是对入参list进行了深拷贝。看下面这段测试代码的结果便一目了然：

```java
public class Test { 
    public static void main(String[] args) { 
        List<Integer> list = new ArrayList<Integer>();
        list.add(1);  
        System.out.println(list);  
        List unmodifiableList = Collections.unmodifiableList(list);    
        ImmutableList immutableList = ImmutableList.copyOf(list); 
        list.add(2);    
        System.out.println(unmodifiableList);   
        System.out.println(immutableList);
    } 
}
```

　　输出结果：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084728875-849616707.png)

## 四.不可变对象真的"完全不可改变"吗？

　　不可变对象虽然具备不可变性，但是不是"完全不可变"的，这里打上引号是因为通过反射的手段是可以改变不可变对象的状态的。

　　大家看到这里可能有疑惑了，为什么既然能改变，为何还叫不可变对象？这里面大家不要误会不可变的本意，从不可变对象的意义分析能看出来对象的不可变性只是用来辅助帮助大家更简单地去编写代码，减少程序编写过程中出错的概率，这是不可变对象的初衷。如果真要靠通过反射来改变一个对象的状态，此时编写代码的人也应该会意识到此类在设计的时候就不希望其状态被更改，从而引起编写代码的人的注意。下面是通过反射方式改变不可变对象的例子：

```java
public class Test { 
    public static void main(String[] args) throws Exception { 
        String s = "Hello World";   
        System.out.println("s = " + s);  
        Field valueFieldOfString = String.class.getDeclaredField("value");    			                             valueFieldOfString.setAccessible(true);  
        char[] value = (char[]) valueFieldOfString.get(s);    
        value[5] = '_';  
        System.out.println("s = " + s); 
    } 
}
```

　　输出结果：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/288799-20190412084742077-227058955.png)

[ ](http://www.importnew.com/14027.html)

## 参考文章

http://ifeve.com/immutable-objects/

http://www.cnblogs.com/dolphin0520/


