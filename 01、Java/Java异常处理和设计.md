## 一.什么是异常                                       

　　异常的英文单词是exception，字面翻译就是“意外、例外”的意思，也就是非正常情况。事实上，异常本质上是程序上的错误，包括程序逻辑错误和系统错误。比如使用空的引用、数组下标越界、内存溢出错误等，这些都是意外的情况，背离我们程序本身的意图。错误在我们编写程序的过程中会经常发生，包括编译期间和运行期间的错误，在编译期间出现的错误有编译器帮助我们一起修正，然而运行期间的错误便不是编译器力所能及了，并且运行期间的错误往往是难以预料的。假若程序在运行期间出现了错误，如果置之不理，程序便会终止或直接导致系统崩溃，显然这不是我们希望看到的结果。因此，如何对运行期间出现的错误进行处理和补救呢？Java提供了异常机制来进行处理，通过异常机制来处理程序运行期间出现的错误。通过异常机制，我们可以更好地提升程序的健壮性。

　　在Java中异常被当做对象来处理，根类是java.lang.Throwable类，在Java中定义了很多异常类（如OutOfMemoryError、NullPointerException、IndexOutOfBoundsException等），这些异常类分为两大类：Error和Exception。

　　Error是无法处理的异常，比如OutOfMemoryError，一般发生这种异常，JVM会选择终止程序。因此我们编写程序时不需要关心这类异常。

　　Exception，也就是我们经常见到的一些异常情况，比如NullPointerException、IndexOutOfBoundsException，这些异常是我们可以处理的异常。

　　Exception类的异常包括checked exception和unchecked exception（unchecked exception也称运行时异常RuntimeException，当然这里的运行时异常并不是前面我所说的运行期间的异常，只是Java中用运行时异常这个术语来表示，Exception类的异常都是在运行期间发生的）。

　　unchecked exception（非检查异常），也称运行时异常（RuntimeException），比如常见的NullPointerException、IndexOutOfBoundsException。对于运行时异常，java编译器不要求必须进行异常捕获处理或者抛出声明，由程序员自行决定。

　　checked exception（检查异常），也称非运行时异常（运行时异常以外的异常就是非运行时异常），java编译器强制程序员必须进行捕获处理，比如常见的IOExeption和SQLException。对于非运行时异常如果不进行捕获或者抛出声明处理，编译都不会通过。

　　在Java中，异常类的结构层次图如下图所示：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201145647.jpg)

　　在Java中，所有异常类的父类是Throwable类，Error类是error类型异常的父类，Exception类是exception类型异常的父类，RuntimeException类是所有运行时异常的父类，RuntimeException以外的并且继承Exception的类是非运行时异常。

　　典型的RuntimeException包括NullPointerException、IndexOutOfBoundsException、IllegalArgumentException等。

　　典型的非RuntimeException包括IOException、SQLException等。

## 二.Java中如何处理异常                                 

 　在Java中如果需要处理异常，必须先对异常进行捕获，然后再对异常情况进行处理。如何对可能发生异常的代码进行异常捕获和处理呢？使用try和catch关键字即可，如下面一段代码所示：

```java
try { 
    File file = new File("d:/a.txt"); 
    if(!file.exists())  
        file.createNewFile();
}
catch (IOException e) { 
    // TODO: handle exception
}
```

　　被try块包围的代码说明这段代码可能会发生异常，一旦发生异常，异常便会被catch捕获到，然后需要在catch块中进行异常处理。

　　这是一种处理异常的方式。在Java中还提供了另一种异常处理方式即抛出异常，顾名思义，也就是说一旦发生异常，我把这个异常抛出去，让调用者去进行处理，自己不进行具体的处理，此时需要用到throw和throws关键字。　

　　下面看一个示例：

```java
public class Main { 
    public static void main(String[] args) {
        try {     
            createFile();
        } catch (Exception e) { 
            // TODO: handle exception  
        } 
    }    
    public static void createFile() throws IOException{  
        File file = new File("d:/a.txt"); 
        if(!file.exists())    
            file.createNewFile();
    }
}
```

　　这段代码和上面一段代码的区别是，在实际的createFile方法中并没有捕获异常，而是用throws关键字声明抛出异常，即告知这个方法的调用者此方法可能会抛出IOException。那么在main方法中调用createFile方法的时候，采用try...catch块进行了异常捕获处理。

　　当然还可以采用throw关键字手动来抛出异常对象。下面看一个例子：

```java
public class Main { 
    public static void main(String[] args) {   
        try {    
            int[] data = new int[]{1,2,3};   
            System.out.println(getDataByIndex(-1,data)); 
        } catch (Exception e) {    
            System.out.println(e.getMessage());  
        }      
    }   
    public static int getDataByIndex(int index,int[] data) {
        if(index<0||index>=data.length)   
            throw new ArrayIndexOutOfBoundsException("数组下标越界");    
        return data[index]; 
    }
}
```

　　然后在catch块中进行捕获。

　　也就说在Java中进行异常处理的话，对于可能会发生异常的代码，可以选择三种方法来进行异常处理：

　　1）对代码块用try..catch进行异常捕获处理；

　　2）在 该代码的方法体外用throws进行抛出声明，告知此方法的调用者这段代码可能会出现这些异常，你需要谨慎处理。此时有两种情况：

　　　　如果声明抛出的异常是非运行时异常，此方法的调用者必须显示地用try..catch块进行捕获或者继续向上层抛出异常。

　　　　如果声明抛出的异常是运行时异常，此方法的调用者可以选择地进行异常捕获处理。

　　3）在代码块用throw手动抛出一个异常对象，此时也有两种情况，跟2）中的类似：

　　　　如果抛出的异常对象是非运行时异常，此方法的调用者必须显示地用try..catch块进行捕获或者继续向上层抛出异常。

　　　　如果抛出的异常对象是运行时异常，此方法的调用者可以选择地进行异常捕获处理。

　　（如果最终将异常抛给main方法，则相当于交给jvm自动处理，此时jvm会简单地打印异常信息）

## 三.深刻理解try,catch,finally,throws,throw五个关键字         　

 　下面我们来看一下异常机制中五个关键字的用法以及需要注意的地方。

1.try,catch,finally

　　try关键字用来包围可能会出现异常的逻辑代码，它单独无法使用，必须配合catch或者finally使用。Java编译器允许的组合使用形式只有以下三种形式：

　　try...catch...;    try....finally......;  try....catch...finally...

　　当然catch块可以有多个，注意try块只能有一个,finally块是可选的（但是最多只能有一个finally块）。

　　三个块执行的顺序为try—>catch—>finally。

　　当然如果没有发生异常，则catch块不会执行。但是finally块无论在什么情况下都是会执行的（这点要非常注意，因此部分情况下，都会将释放资源的操作放在finally块中进行）。

　　在有多个catch块的时候，是按照catch块的先后顺序进行匹配的，一旦异常类型被一个catch块匹配，则不会与后面的catch块进行匹配。

　　在使用try..catch..finally块的时候，注意千万不要在finally块中使用return，因为finally中的return会覆盖已有的返回值。下面看一个例子：

```java
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException; 
public class Main {  
    public static void main(String[] args) {  
        String str = new Main().openFile();   
        System.out.println(str);    
    }    
    public String openFile() {  
        try {     
            FileInputStream inputStream = new FileInputStream("d:/a.txt"); 
            int ch = inputStream.read();   
            System.out.println("aaa");  
            return "step1";    
        } catch (FileNotFoundException e) { 
            System.out.println("file not found");   
            return "step2";   
        }catch (IOException e) {  
            System.out.println("io exception");    
            return "step3";  
        }finally{    
            System.out.println("finally block");  
            //return "finally";
        }
    }
}
```

　　这段程序的输出结果为：![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201145655.jpg)

　　可以看出，在try块中发生FileNotFoundException之后，就跳到第一个catch块，打印"file not found"信息，并将"step2"赋值给返回值，然后执行finally块，最后将返回值返回。

　　从这个例子说明，无论try块或者catch块中是否包含return语句，都会执行finally块。

　　如果将这个程序稍微修改一下，将finally块中的return语句注释去掉，运行结果是：

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201145658.jpg)

　　最后打印出的是"finally"，返回值被重新覆盖了。

　　因此如果方法有返回值，切忌不要再finally中使用return，这样会使得程序结构变得混乱。

 2.throws和thow关键字

　　1）throws出现在方法的声明中，表示该方法可能会抛出的异常，然后交给上层调用它的方法程序处理，允许throws后面跟着多个异常类型；

　　2）一般会用于程序出现某种逻辑时程序员主动抛出某种特定类型的异常。throw只会出现在方法体中，当方法在执行过程中遇到异常情况时，将异常信息封装为异常对象，然后throw出去。throw关键字的一个非常重要的作用就是 异常类型的转换（会在后面阐述道）。

　　throws表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某种异常对象。两者都是消极处理异常的方式（这里的消极并不是说这种方式不好），只是抛出或者可能抛出异常，但是不会由方法去处理异常，真正的处理异常由此方法的上层调用处理。

## 四.在类继承的时候，方法覆盖时如何进行异常抛出声明            

 　本小节讨论子类重写父类方法的时候，如何确定异常抛出声明的类型。下面是三点原则：

　　1）父类的方法没有声明异常，子类在重写该方法的时候不能声明异常；

　　2）如果父类的方法声明一个异常exception1，则子类在重写该方法的时候声明的异常不能是exception1的父类；

　　3）如果父类的方法声明的异常类型只有非运行时异常（运行时异常），则子类在重写该方法的时候声明的异常也只能有非运行时异常（运行时异常），不能含有运行时异常（非运行时异常）。

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201145702.jpg)

## 五.异常处理和设计的几个建议                             

 　以下是根据前人总结的一些异常处理的建议：

1.只在必要使用异常的地方才使用异常，不要用异常去控制程序的流程

　　谨慎地使用异常，异常捕获的代价非常高昂，异常使用过多会严重影响程序的性能。如果在程序中能够用if语句和Boolean变量来进行逻辑判断，那么尽量减少异常的使用，从而避免不必要的异常捕获和处理。比如下面这段经典的程序：

```java
public void useExceptionsForFlowControl() { 
    try {  
        while (true) { 
            increaseCount();
        } 
    } catch (MaximumCountReachedException ex) {
    }  //Continue execution 
}  
public void increaseCount() throws MaximumCountReachedException { 
    if (count >= 5000)  
        throw new MaximumCountReachedException();
}
```

　　上边的useExceptionsForFlowControl()用一个无限循环来增加count直到抛出异常，这种做法并没有说让代码不易读，而是使得程序执行效率降低。

2.切忌使用空catch块

　　在捕获了异常之后什么都不做，相当于忽略了这个异常。千万不要使用空的catch块，空的catch块意味着你在程序中隐藏了错误和异常，并且很可能导致程序出现不可控的执行结果。如果你非常肯定捕获到的异常不会以任何方式对程序造成影响，最好用Log日志将该异常进行记录，以便日后方便更新和维护。

3.检查异常和非检查异常的选择

　　一旦你决定抛出异常，你就要决定抛出什么异常。这里面的主要问题就是抛出检查异常还是非检查异常。

　　检查异常导致了太多的try…catch代码，可能有很多检查异常对开发人员来说是无法合理地进行处理的，比如SQLException，而开发人员却不得不去进行try…catch，这样就会导致经常出现这样一种情况：逻辑代码只有很少的几行，而进行异常捕获和处理的代码却有很多行。这样不仅导致逻辑代码阅读起来晦涩难懂，而且降低了程序的性能。

　　我个人建议尽量避免检查异常的使用，如果确实该异常情况的出现很普遍，需要提醒调用者注意处理的话，就使用检查异常；否则使用非检查异常。

　　因此，在一般情况下，我觉得尽量将检查异常转变为非检查异常交给上层处理。

4.注意catch块的顺序

　　不要把上层类的异常放在最前面的catch块。比如下面这段代码：

```java
try {  
    FileInputStream inputStream = new FileInputStream("d:/a.txt");   
    int ch = inputStream.read();  
    System.out.println("aaa");  
    return "step1";  
} catch (IOException e) {　
    System.out.println("io exception");　　 
    return "step2";
}catch (FileNotFoundException e) {  
    System.out.println("file not found");　
    return "step3"; 
}finally{  
    System.out.println("finally block");    //return "finally"; 
}
```

　　

　　第二个catch的FileNotFoundException将永远不会被捕获到，因为FileNotFoundException是IOException的子类。

5.不要将提供给用户看的信息放在异常信息里

　　比如下面这段代码：

```java
public class Main {  
    public static void main(String[] args) {  
        try {    
            String user = null; 
            String pwd = null;   
            login(user,pwd);  
        } catch (Exception e) {  
            System.out.println(e.getMessage());   
        }    
    }   
    public static void login(String user,String pwd) {  
        if(user==null||pwd==null)  
            throw new NullPointerException("用户名或者密码为空");   
        //... 
    }
}
```

　　展示给用户错误提示信息最好不要跟程序混淆一起，比较好的方式是将所有错误提示信息放在一个配置文件中统一管理。

6.避免多次在日志信息中记录同一个异常

　　只在异常最开始发生的地方进行日志信息记录。很多情况下异常都是层层向上跑出的，如果在每次向上抛出的时候，都Log到日志系统中，则会导致无从查找异常发生的根源。

\7. 异常处理尽量放在高层进行

　　尽量将异常统一抛给上层调用者，由上层调用者统一之时如何进行处理。如果在每个出现异常的地方都直接进行处理，会导致程序异常处理流程混乱，不利于后期维护和异常错误排查。由上层统一进行处理会使得整个程序的流程清晰易懂。

\8. 在finally中释放资源

　　如果有使用文件读取、网络操作以及数据库操作等，记得在finally中释放资源。这样不仅会使得程序占用更少的资源，也会避免不必要的由于资源未释放而发生的异常情况。                  

## 参考资料

http://www.cnblogs.com/dolphin0520/

http://blessht.iteye.com/blog/908286

http://www.oschina.net/translate/10-exception-handling-best-practices-in-java-programming

http://blog.csdn.net/snow_fox_yaya/article/details/1823205

http://www.iteye.com/topic/72170

http://www.blogjava.net/gdws/archive/2010/04/25/319342.html

http://www.2cto.com/kf/201403/284166.html

http://www.iteye.com/topic/857443

http://developer.51cto.com/art/200808/85625.htm

http://www.cnblogs.com/JavaVillage/articles/384483.html

http://tech.e800.com.cn/articles/2009/79/1247105040929_1.html

http://blog.csdn.net/zhouyong80/article/details/1907799

http://blog.csdn.net/luoweifu/article/details/10721543                 

《Effective Java中文版》  
