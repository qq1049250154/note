## 1.修饰类

　　默认访问权限（包访问权限）：用来修饰类的话，表示该类只对同一个包中的其他类可见。

　　public：用来修饰类的话，表示该类对其他所有的类都可见。

　　下面通过几个例子来看一下两者的区别：

例1：

Main.java:

```java
package com.cxh.test1; public class Main { 
    /**   * @param args   */  
    public static void main(String[] args) {    
        // TODO Auto-generated method stub     
        People people = new People("Tom");   
        System.out.println(people.getName()); 
    } 
}
```

People.java

```java
package com.cxh.test1; 
class People {    
    //默认访问权限（包访问权限）
    private String name = null;    
    public People(String name) {    
        this.name = name;
    }    
    public String getName() {
        return name;
    }   
    public void setName(String name) {  
        this.name = name; 
    }
}
```

　　从代码可以看出，修饰People类采用的是默认访问权限，而由于People类和Main类在同一个包中，因此People类对于Main类是可见的。

　　程序运行结果：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201151719.jpg)

例子2：

People.java

```java
package com.cxh.test2; 
class People {     
    //默认访问权限（包访问权限） 
    private String name = null;   
    public People(String name) { 
        this.name = name;  
    }   
    public String getName() {  
        return name;  
    }   
    public void setName(String name) {  
        this.name = name;
    }
}
```

　　此时People类和Main类不在同一个包中，会发生什么情况呢？

　　下面是Main类中的提示的错误：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201151722.jpg)

　　提示Peolple类在Main类中不可视。从这里就可以看出，如果用默认访问权限去修饰一个类，该类只对同一个包中的其他类可见，对于不同包中的类是不可见的。

　　正如上图的快速修正提示所示，将People类的默认访问权限更改为public的话，People类对于Main类便可见了。

## 2.修饰类的方法和变量

　　默认访问权限（包访问权限）：如果一个类的方法或变量被包访问权限修饰，也就意味着只能在同一个包中的其他类中显示地调用该类的方法或者变量，在不同包中的类中不能显示地调用该类的方法或变量。

　　private：如果一个类的方法或者变量被private修饰，那么这个类的方法或者变量只能在该类本身中被访问，在类外以及其他类中都不能显示地进行访问。

　　protected：如果一个类的方法或者变量被protected修饰，对于同一个包的类，这个类的方法或变量是可以被访问的。对于不同包的类，只有继承于该类的类才可以访问到该类的方法或者变量。

　　public：被public修饰的方法或者变量，在任何地方都是可见的。

下面再通过几个例子来看一下它们作用域类的方法和变量时的区别：

例3：

Main.java没有变化

People.java

```java
package com.cxh.test1; 
public class People {    
    private String name = null;     
    public People(String name) {   
        this.name = name;  
    }   
    String getName() { 
        //默认访问权限（包访问权限）
        return name;
    }   
    void setName(String name) {  
        //默认访问权限（包访问权限）  
        this.name = name;
    }
}
```

　　此时在Main类是可以显示调用方法getName和setName的。

但是如果People类和Main类不在同一个包中：

```java
package com.cxh.test2;  
//与Main类处于不同包中 
public class People {     
    private String name = null; 
    public People(String name) {   
        this.name = name;
    }    
    String getName() {
        //默认访问权限（包访问权限）   
        return name; 
    }    
    void setName(String name) {  
        //默认访问权限（包访问权限）  
        this.name = name; 
    }
}
```

　　此时在Main类中会提示错误：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201151725.jpg)

　　由此可以看出，如果用默认访问权限来修饰类的方法或者变量，则只能在同一个包的其他类中进行访问。

 例4:

People.java

```java
package com.cxh.test1; 
public class People {     
    private String name = null; 
    public People(String name) { 
        this.name = name;  
    }  
    protected String getName() {  
        return name; 
    }    
    protected void setName(String name) {  
        this.name = name; 
    }
}
```

　　此时是可以在Main中显示调用方法getName和setName的。

如果People类和Main类处于不同包中：

```java
package com.cxh.test2;   
public class People {  
    private String name = null;
    public People(String name) {  
        this.name = name; 
    }  
    protected String getName() {     
        return name; 
    }     
    protected void setName(String name) {    
        this.name = name;
    }
}
```

　　则会在Main中报错：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201151729.jpg)

　　如果在com.cxh.test1中定一个类Man继承People，则可以在类Man中显示调用方法getName和setName：

```java
package com.cxh.test1;
import com.cxh.test2.People; 
public class Man extends People{  
    public Man(String name){   
        super(name); 
    }    
    public String toString() { 
        return getName();
    }
}
```

　　下面补充一些关于Java包和类文件的知识：

　　1）Java中的包主要是为了防止类文件命名冲突以及方便进行代码组织和管理；

　　2）对于一个Java源代码文件，如果存在public类的话，只能有一个public类，且此时源代码文件的名称必须和public类的名称完全相同，另外，如果还存在其他类，这些类在包外是不可见的。如果源代码文件没有public类，则源代码文件的名称可以随意命名。

作者：[Matrix海子](http://www.cnblogs.com/dolphin0520/)

出处：http://www.cnblogs.com/dolphin0520/