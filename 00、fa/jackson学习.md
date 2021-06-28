[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958

[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958

[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958


[TOC]

---

# 基本信息

### 关于jackson

本文是《jackson学习》系列的第一篇，先来一起了解jackson：

1. jackson的github地址：https://github.com/FasterXML/jackson
2. 按照官网所述，jackson是java技术栈内最好的JSON解析工具(best JSON parser for Java)；
3. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)；

### 版本信息

1. jackson共有1.x和2.x两个版本系列，其中1.x已废弃不再有版本发布，2.x是活跃版本；
2. 1.x和2.x不兼容，如果您的代码已经使用了1.x，现在想改用2.x，您就必须修改使用jackson的那部分代码；
3. 虽然不兼容，但是1.x和2.x不冲突，您的项目可以在pom.xml中同时依赖这两个版本，假设您原有三处代码调用了1.x的API，现在可以把一处改成2.x的，另外两处维持不变，这个特性适合将项目逐步从1.x升级到2.x(This is by design and was chosen as the strategy to allow smoother migration from 1.x to 2.x.)；
4. 2.x系列版本中，有的版本已关闭(除非bug或者安全问题才会发布新的小版本)，有的版本还处于活跃状态，如下图，您可以在这个地址获取最新情况：https://github.com/FasterXML/jackson/wiki/Jackson-Releases
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705113648423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)

### 三个核心模块

jackson有三个核心模块，如下，括号内是maven的artifactId：

1. Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. Annotations（jackson-annotations）：jackson注解；
3. Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现；

### 低阶API库的作用

1. 当我们用jackson做JSON操作时，常用的是Databind模块的ObjectMapper类，对处于核心位置的jackson-core反倒是很少直接用到，那么该模块有什么作用呢？
2. 如下图，BeanSerializer是jackson-databind的功能类，其serialize方法负责将java对象转为JSON，方法中的处理逻辑就是调用JsonGenerator的API，而JsonGenerator就是jackson-core中负责序列化的主要功能类：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200705134343695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70)
3. 可见Databind模块的ObjectMapper类提供给我们的API，其底层操作是基于jackson-core实现的；
   至此，我们对jackson已有了基本了解，接下来的文章会开始一系列的实战，通过实战来掌握和理解这套优秀的工具；

# jackson-core

### 关于jackson-core

1. 本文主要内容是jackson-core库，这是个低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator；
2. 在日常的序列化和反序列化处理中，最常用的是jackson-annotations和jackson-databind，而jackson-core由于它提供的API过于基础，我们大多数情况下是用不上的；
3. 尽管jackson-databind负责序列化和反序列化处理，但它的底层实现是调用了jackson-core的API；
4. 本着万丈高楼平地起的原则，本文咱们通过实战了解神秘的jackson-core，了解整个jackson的序列化和反序列化基本原理；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135844.png)

### 创建父子工程

创建名为jacksondemo的maven工程，这是个父子结构的工程，其pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>jacksondemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>core</module>
        <module>beans</module>
        <module>databind</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>1.7.25</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.7</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 新增子工程beans

1. 在父工程jscksondemo下新增名为beans的子工程，这里面是一些常量和Pojo类；
2. 增加定义常量的类Constant.java：

```java
package com.bolingcavalry.jacksondemo.beans;

public class Constant {
    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    public final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";
    /**
     * 用来验证反序列化的JSON字符串
     */
    public final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";
    /**
     * 用来验证序列化的TwitterEntry实例
     */
    public final static TwitterEntry TEST_OBJECT = new TwitterEntry();
    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }}
```

1. 增加一个Pojo，对应的是一条推特消息：

```java
package com.bolingcavalry.jacksondemo.beans;
/**
 * @Description: 推特消息bean
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 16:24
 */
public class TwitterEntry {
    /**
     * 推特消息id
     */
    long id;
    /**
     * 消息内容
     */
    String text;    /**
     * 消息创建者
     */
    int fromUserId;
    /**
     * 消息接收者
     */
    int toUserId;
    /**
     * 语言类型
     */
    String languageCode;    public long getId() {
        return id;
    }    public void setId(long id) {
        this.id = id;
    }    public String getText() {
        return text;
    }    public void setText(String text) {
        this.text = text;
    }    public int getFromUserId() {
        return fromUserId;
    }    public void setFromUserId(int fromUserId) {
        this.fromUserId = fromUserId;
    }    public int getToUserId() {
        return toUserId;
    }    public void setToUserId(int toUserId) {
        this.toUserId = toUserId;
    }    public String getLanguageCode() {
        return languageCode;
    }    public void setLanguageCode(String languageCode) {
        this.languageCode = languageCode;
    }    public TwitterEntry() {
    }    public String toString() {
        return "[Tweet, id: "+id+", text='"+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
    }}
```

1. 以上就是准备工作了，接下来开始实战jackson-core；

### JsonFactory线程安全吗?

1. JsonFactory是否是线程安全的，这是编码前要弄清楚的问题，因为JsonParser和JsonGenerator的创建都离不开JsonFactory；
2. 如下图红框所示，jackson官方文档中明确指出JsonFactory是线程安全的，可以放心的作为全局变量给多线程同时使用：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135905.png)
3. 官方文档地址：http://fasterxml.github.io/jackson-core/javadoc/2.11/

### jackson-core实战

1. 新建子工程core，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>core</artifactId>
    <name>core</name>
    <description>Demo project for jackson core use</description>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.bolingcavalry</groupId>
            <artifactId>beans</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

1. 新建StreamingDemo类，这里面是调用jackson-core的API进行序列化和反序列化的所有demo，如下：

```java
package com.bolingcavalry.jacksondemo.core;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URL;

/**
 * @Description: jackson低阶方法的使用
 * @author: willzhao E-mail: zq2599@gmail.com
 * @date: 2020/7/4 15:50
 */
public class StreamingDemo {

    private static final Logger logger = LoggerFactory.getLogger(StreamingDemo.class);

    JsonFactory jsonFactory = new JsonFactory();

    /**
     * 该字符串的值是个网络地址，该地址对应的内容是个JSON
     */
    final static String TEST_JSON_DATA_URL = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

    /**
     * 用来验证反序列化的JSON字符串
     */
    final static String TEST_JSON_STR = "{\n" +
            "  \"id\":1125687077,\n" +
            "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
            "  \"fromUserId\":855523, \n" +
            "  \"toUserId\":815309,\n" +
            "  \"languageCode\":\"en\"\n" +
            "}";

    /**
     * 用来验证序列化的TwitterEntry实例
     */
    final static TwitterEntry TEST_OBJECT = new TwitterEntry();

    /**
     * 准备好TEST_OBJECT对象的各个参数
     */
    static {
        TEST_OBJECT.setId(123456L);
        TEST_OBJECT.setFromUserId(101);
        TEST_OBJECT.setToUserId(102);
        TEST_OBJECT.setText("this is a message for serializer test");
        TEST_OBJECT.setLanguageCode("zh");
    }


    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param json JSON字符串
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONStr(String json) throws IOException {

        JsonParser jsonParser = jsonFactory.createParser(json);

        if (jsonParser.nextToken() != JsonToken.START_OBJECT) {
            jsonParser.close();
            logger.error("起始位置没有大括号");
            throw new IOException("起始位置没有大括号");
        }

        TwitterEntry result = new TwitterEntry();

        try {
            // Iterate over object fields:
            while (jsonParser.nextToken() != JsonToken.END_OBJECT) {

                String fieldName = jsonParser.getCurrentName();

                logger.info("正在解析字段 [{}]", jsonParser.getCurrentName());

                // 解析下一个
                jsonParser.nextToken();

                switch (fieldName) {
                    case "id":
                        result.setId(jsonParser.getLongValue());
                        break;
                    case "text":
                        result.setText(jsonParser.getText());
                        break;
                    case "fromUserId":
                        result.setFromUserId(jsonParser.getIntValue());
                        break;
                    case "toUserId":
                        result.setToUserId(jsonParser.getIntValue());
                        break;
                    case "languageCode":
                        result.setLanguageCode(jsonParser.getText());
                        break;
                    default:
                        logger.error("未知字段 '" + fieldName + "'");
                        throw new IOException("未知字段 '" + fieldName + "'");
                }
            }
        } catch (IOException e) {
            logger.error("反序列化出现异常 :", e);
        } finally {
            jsonParser.close(); // important to close both parser and underlying File reader
        }

        return result;
    }

    /**
     * 反序列化测试(JSON -> Object)，入参是JSON字符串
     * @param url JSON字符串的网络地址
     * @return
     * @throws IOException
     */
    public TwitterEntry deserializeJSONFromUrl(String url) throws IOException {
        // 从网络上取得JSON字符串
        String json = IOUtils.toString(new URL(TEST_JSON_DATA_URL), JsonEncoding.UTF8.name());

        logger.info("从网络取得JSON数据 :\n{}", json);

        if(StringUtils.isNotBlank(json)) {
            return deserializeJSONStr(json);
        } else {
            logger.error("从网络获取JSON数据失败");
            return null;
        }
    }


    /**
     * 序列化测试(Object -> JSON)
     * @param twitterEntry
     * @return 由对象序列化得到的JSON字符串
     */
    public String serialize(TwitterEntry twitterEntry) throws IOException{
        String rlt = null;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        JsonGenerator jsonGenerator = jsonFactory.createGenerator(byteArrayOutputStream, JsonEncoding.UTF8);

        try {
            jsonGenerator.useDefaultPrettyPrinter();

            jsonGenerator.writeStartObject();
            jsonGenerator.writeNumberField("id", twitterEntry.getId());
            jsonGenerator.writeStringField("text", twitterEntry.getText());
            jsonGenerator.writeNumberField("fromUserId", twitterEntry.getFromUserId());
            jsonGenerator.writeNumberField("toUserId", twitterEntry.getToUserId());
            jsonGenerator.writeStringField("languageCode", twitterEntry.getLanguageCode());
            jsonGenerator.writeEndObject();
        } catch (IOException e) {
            logger.error("序列化出现异常 :", e);
        } finally {
            jsonGenerator.close();
        }

        // 一定要在
        rlt = byteArrayOutputStream.toString();

        return rlt;
    }


    public static void main(String[] args) throws Exception {

        StreamingDemo streamingDemo = new StreamingDemo();

        // 执行一次对象转JSON操作
        logger.info("********************执行一次对象转JSON操作********************");
        String serializeResult = streamingDemo.serialize(TEST_OBJECT);
        logger.info("序列化结果是JSON字符串 : \n{}\n\n", serializeResult);

        // 用本地字符串执行一次JSON转对象操作
        logger.info("********************执行一次本地JSON反序列化操作********************");
        TwitterEntry deserializeResult = streamingDemo.deserializeJSONStr(TEST_JSON_STR);
        logger.info("\n本地JSON反序列化结果是个java实例 : \n{}\n\n", deserializeResult);

        // 用网络地址执行一次JSON转对象操作
        logger.info("********************执行一次网络JSON反序列化操作********************");
        deserializeResult = streamingDemo.deserializeJSONFromUrl(TEST_JSON_DATA_URL);
        logger.info("\n网络JSON反序列化结果是个java实例 : \n{}", deserializeResult);

        ObjectMapper a;
    }
}
```

1. 上述代码可见JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容；
2. JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段；
3. 不论是JsonParser还是JsonGenerator，大家都可以感觉到工作量很大，需要开发者自己动手实现对象和JSON字段的关系映射，实际应用中不需要咱们这样辛苦的编码，jackson的另外两个库(annonation的databind)已经帮我们完成了大量工作，上述代码只是揭示最基础的jackson执行原理；
4. 执行StreamingDemo类，得到结果如下，序列化和反序列化都成功了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204135920.png)

- 以上就是jackson-core的基本功能，咱们了解了jackson最底层的工作原理，接下来的文章会继续实践更多操作；

# 常用API操作

### 本篇概览

本文是《jackson学习》系列的第三篇，前面咱们学习了jackson的低阶API，知道了底层原理，本篇开始学习平时最常用的基本功能，涉及内容如下：

1. 体验最常用的操作，内容如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140031.png)
2. 介绍常用的可配置属性，以便按需要来设置；
3. 接下来进入快速浏览的环节，咱们一起先把各个API过一遍；

### 单个对象序列化

先看常用的序列化API：

1. 对象转字符串：

```java
String jsonStr = mapper.writeValueAsString(twitterEntry);
```

1. 对象转文件：

```java
mapper.writeValue(new File("twitter.json"), twitterEntry);
```

1. 对象转byte数组：

```java
byte[] array = mapper.writeValueAsBytes(twitterEntry);
```

### 单个对象反序列化

1. 字符串转对象：

```java
TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
```

1. 文件转对象：

```java
TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
```

1. byte数组转对象：

```java
TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
```

1. 字符串网络地址转对象：

```java
String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
```

### 集合序列化

1. HashMap转字符串：

```java
String mapJsonStr = mapper.writeValueAsString(map);
```

### 集合反序列化

1. 字符串转HashMap：

```java
Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
```

### JsonNode

1. 如果您不想使用XXX.class来做反序列化，也能使用JsonNode来操作：

```java
JsonNode jsonNode = mapper.readTree(mapJsonStr);
String name = jsonNode.get("name").asText();
int age = jsonNode.get("age").asInt();
String city = jsonNode.get("addr").get("city").asText();
String street = jsonNode.get("addr").get("street").asText();
```

### 时间字段格式化

1. 对于Date字段，默认的反序列化是时间戳，可以修改配置：

```java
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
dateMapStr = mapper.writeValueAsString(dateMap);
```

### JSON数组的反序列化

假设jsonArrayStr是个json数组格式的字符串：

1. JSON数组转对象数组：

```java
TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
```

1. JSON数组转对象集合(ArrayList)：

```java
List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
```

### 完整代码

1. 上述所有常用API用法的完整代码如下：

```java
package com.bolingcavalry.jacksondemo.databind;

import com.bolingcavalry.jacksondemo.beans.TwitterEntry;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.File;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleDemo {

    private static final Logger logger = LoggerFactory.getLogger(SimpleDemo.class);

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        logger.info("以下是序列化操作");

        // 对象 -> 字符串
        TwitterEntry twitterEntry = new TwitterEntry();
        twitterEntry.setId(123456L);
        twitterEntry.setFromUserId(101);
        twitterEntry.setToUserId(102);
        twitterEntry.setText("this is a message for serializer test");
        twitterEntry.setLanguageCode("zh");

        String jsonStr = mapper.writeValueAsString(twitterEntry);
        logger.info("序列化的字符串：{}", jsonStr);

        // 对象 -> 文件
        mapper.writeValue(new File("twitter.json"), twitterEntry);

        // 对象 -> byte数组
        byte[] array = mapper.writeValueAsBytes(twitterEntry);

        logger.info("\n\n以下是反序列化操作");

        // 字符串 -> 对象
        String objectJsonStr = "{\n" +
                "  \"id\":1125687077,\n" +
                "  \"text\":\"@stroughtonsmith You need to add a \\\"Favourites\\\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?\",\n" +
                "  \"fromUserId\":855523, \n" +
                "  \"toUserId\":815309,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}";


        TwitterEntry tFromStr = mapper.readValue(objectJsonStr, TwitterEntry.class);
        logger.info("从字符串反序列化的对象：{}", tFromStr);

        // 文件 -> 对象
        TwitterEntry tFromFile = mapper.readValue(new File("twitter.json"), TwitterEntry.class);
        logger.info("从文件反序列化的对象：{}", tFromStr);

        // byte数组 -> 对象
        TwitterEntry tFromBytes = mapper.readValue(array, TwitterEntry.class);
        logger.info("从byte数组反序列化的对象：{}", tFromBytes);

        // 字符串网络地址 -> 对象
        String testJsonDataUrl = "https://raw.githubusercontent.com/zq2599/blog_demos/master/files/twitteer_message.json";

        TwitterEntry tFromUrl = mapper.readValue(new URL(testJsonDataUrl), TwitterEntry.class);
        logger.info("从网络地址反序列化的对象：{}", tFromUrl);


        logger.info("\n\n以下是集合序列化操作");

        Map<String, Object> map = new HashMap<>();
        map.put("name", "tom");
        map.put("age", 11);

        Map<String, String> addr = new HashMap<>();
        addr.put("city","深圳");
        addr.put("street", "粤海");

        map.put("addr", addr);

        String mapJsonStr = mapper.writeValueAsString(map);
        logger.info("HashMap序列化的字符串：{}", mapJsonStr);

        logger.info("\n\n以下是集合反序列化操作");
        Map<String, Object> mapFromStr = mapper.readValue(mapJsonStr, new TypeReference<Map<String, Object>>() {});
        logger.info("从字符串反序列化的HashMap对象：{}", mapFromStr);

        // JsonNode类型操作
        JsonNode jsonNode = mapper.readTree(mapJsonStr);
        String name = jsonNode.get("name").asText();
        int age = jsonNode.get("age").asInt();
        String city = jsonNode.get("addr").get("city").asText();
        String street = jsonNode.get("addr").get("street").asText();

        logger.info("用JsonNode对象和API反序列化得到的数：name[{}]、age[{}]、city[{}]、street[{}]", name, age, city, street);

        // 时间类型格式

        Map<String, Object> dateMap = new HashMap<>();
        dateMap.put("today", new Date());

        String dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("默认的时间序列化：{}", dateMapStr);

        // 设置时间格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        dateMapStr = mapper.writeValueAsString(dateMap);
        logger.info("自定义的时间序列化：{}", dateMapStr);

        System.out.println(objectJsonStr);

        // json数组
        String jsonArrayStr = "[{\n" +
                "  \"id\":1,\n" +
                "  \"text\":\"text1\",\n" +
                "  \"fromUserId\":11, \n" +
                "  \"toUserId\":111,\n" +
                "  \"languageCode\":\"en\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":2,\n" +
                "  \"text\":\"text2\",\n" +
                "  \"fromUserId\":22, \n" +
                "  \"toUserId\":222,\n" +
                "  \"languageCode\":\"zh\"\n" +
                "},\n" +
                "{\n" +
                "  \"id\":3,\n" +
                "  \"text\":\"text3\",\n" +
                "  \"fromUserId\":33, \n" +
                "  \"toUserId\":333,\n" +
                "  \"languageCode\":\"en\"\n" +
                "}]";

        // json数组 -> 对象数组
        TwitterEntry[] twitterEntryArray = mapper.readValue(jsonArrayStr, TwitterEntry[].class);
        logger.info("json数组反序列化成对象数组：{}", Arrays.toString(twitterEntryArray));

        // json数组 -> 对象集合
        List<TwitterEntry> twitterEntryList = mapper.readValue(jsonArrayStr, new TypeReference<List<TwitterEntry>>() {});
        logger.info("json数组反序列化成对象集合：{}", twitterEntryList);
    }
}
```

1. 执行结果如下：

```shell
C:\jdk\bin\java.exe -javaagent:C:\sofware\JetBrains\IntelliJIDEA\lib\idea_rt.jar=64570:C:\sofware\JetBrains\IntelliJIDEA\bin -Dfile.encoding=UTF-8 -classpath C:\jdk\jre\lib\charsets.jar;C:\jdk\jre\lib\deploy.jar;C:\jdk\jre\lib\ext\access-bridge-64.jar;C:\jdk\jre\lib\ext\cldrdata.jar;C:\jdk\jre\lib\ext\dnsns.jar;C:\jdk\jre\lib\ext\jaccess.jar;C:\jdk\jre\lib\ext\jfxrt.jar;C:\jdk\jre\lib\ext\localedata.jar;C:\jdk\jre\lib\ext\nashorn.jar;C:\jdk\jre\lib\ext\sunec.jar;C:\jdk\jre\lib\ext\sunjce_provider.jar;C:\jdk\jre\lib\ext\sunmscapi.jar;C:\jdk\jre\lib\ext\sunpkcs11.jar;C:\jdk\jre\lib\ext\zipfs.jar;C:\jdk\jre\lib\javaws.jar;C:\jdk\jre\lib\jce.jar;C:\jdk\jre\lib\jfr.jar;C:\jdk\jre\lib\jfxswt.jar;C:\jdk\jre\lib\jsse.jar;C:\jdk\jre\lib\management-agent.jar;C:\jdk\jre\lib\plugin.jar;C:\jdk\jre\lib\resources.jar;C:\jdk\jre\lib\rt.jar;D:\github\blog_demos\jacksondemo\databind\target\classes;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.11.0\jackson-databind-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.11.0\jackson-annotations-2.11.0.jar;C:\Users\12167\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.11.0\jackson-core-2.11.0.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\12167\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\12167\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\12167\.m2\repository\commons-io\commons-io\2.7\commons-io-2.7.jar;C:\Users\12167\.m2\repository\org\apache\commons\commons-lang3\3.10\commons-lang3-3.10.jar;D:\github\blog_demos\jacksondemo\beans\target\classes com.bolingcavalry.jacksondemo.databind.SimpleDemo
2020-08-28 07:53:01 INFO  SimpleDemo:27 - 以下是序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:38 - 序列化的字符串：{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
2020-08-28 07:53:01 INFO  SimpleDemo:47 - 

以下是反序列化操作
2020-08-28 07:53:01 INFO  SimpleDemo:60 - 从字符串反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:64 - 从文件反序列化的对象：[Tweet, id: 1125687077, text='@stroughtonsmith You need to add a "Favourites" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?', from: 855523, to: 815309, lang: en]
2020-08-28 07:53:01 INFO  SimpleDemo:68 - 从byte数组反序列化的对象：[Tweet, id: 123456, text='this is a message for serializer test', from: 101, to: 102, lang: zh]
2020-08-28 07:53:04 INFO  SimpleDemo:74 - 从网络地址反序列化的对象：[Tweet, id: 112233445566, text='this is a message from zq2599's github', from: 201, to: 202, lang: en]
2020-08-28 07:53:04 INFO  SimpleDemo:77 - 

以下是集合序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:90 - HashMap序列化的字符串：{"name":"tom","addr":{"city":"深圳","street":"粤海"},"age":11}
2020-08-28 07:53:04 INFO  SimpleDemo:92 - 

以下是集合反序列化操作
2020-08-28 07:53:04 INFO  SimpleDemo:94 - 从字符串反序列化的HashMap对象：{name=tom, addr={city=深圳, street=粤海}, age=11}
2020-08-28 07:53:04 INFO  SimpleDemo:103 - 用JsonNode对象和API反序列化得到的数：name[tom]、age[11]、city[深圳]、street[粤海]
2020-08-28 07:53:04 INFO  SimpleDemo:111 - 默认的时间序列化：{"today":1598572384838}
2020-08-28 07:53:04 INFO  SimpleDemo:116 - 自定义的时间序列化：{"today":"2020-08-28 07:53:04"}
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
2020-08-28 07:53:04 INFO  SimpleDemo:145 - json数组反序列化成对象数组：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]
2020-08-28 07:53:04 INFO  SimpleDemo:149 - json数组反序列化成对象集合：[[Tweet, id: 1, text='text1', from: 11, to: 111, lang: en], [Tweet, id: 2, text='text2', from: 22, to: 222, lang: zh], [Tweet, id: 3, text='text3', from: 33, to: 333, lang: en]]

Process finished with exit code 0
```

1. 还会产生名为twitter.json的文件，内容如下：

```shell
{"id":123456,"text":"this is a message for serializer test","fromUserId":101,"toUserId":102,"languageCode":"zh"}
```

### 常用配置

下面是平时可能用到的自定义配置项目：

1. 序列化结果格式化：

```java
mapper.enable(SerializationFeature.INDENT_OUTPUT);
```

1. 空对象不要抛出异常：

```java
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
```

1. Date、Calendar等序列化为时间格式的字符串(如果不执行以下设置，就会序列化成时间戳格式)：

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

1. 反序列化时，遇到未知属性不要抛出异常：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

1. 反序列化时，空字符串对于的实例属性为null：

```java
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

1. 允许C和C++样式注释：

```java
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

1. 允许字段名没有引号（可以进一步减小json体积）：

```java
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
```

1. 允许单引号：

```java
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### 特殊配置：在json对象最外层再包裹一层

1. 最后要说的是个特殊配置，先来看看正常情况一个普通的序列化结果：

```java
{
  "id" : 1,
  "text" : "aabbcc",
  "fromUserId" : 456,
  "toUserId" : 0,
  "languageCode" : "zh"
}
```

1. 接下来咱们做两件事，首先，是给上述json对应的实例类添加一个注解，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140130.png)
2. 其次，执行以下配置：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
```

1. 然后再次执行TwitterEntry实例的序列化，得到的结果如下，可见和之前的序列化结果相比，之前的整个json都变成了一个value，此value对应的key就是注解JsonRootName的value属性：

```java
{
  "aaa" : {
    "id" : 1,
    "text" : "aabbcc",
    "fromUserId" : 456,
    "toUserId" : 0,
    "languageCode" : "zh"
  }
}
```

- 至此，开发中常用的API和配置都已经介绍完成

# WRAP_ROOT_VALUE（root对象）

### 关于root对象（WRAP_ROOT_VALUE）

1. 对于只有id和name两个字段的POJO实例来说，正常的序列化结果如下：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

1. jackson在序列化时，可以在上述json外面再包裹一层，官方叫做WRAP_ROOT_VALUE，本文中叫做root对象，如下所示，整个json的只有一个键值对，key是aaabbbccc，value内部才是POJO实例的id和name字段的值：

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

### 提前小结

root对象特性提前做个小结，这样如果您时间有限，仅看这一节即可：

- 先看序列化场景：

1. 执行下面代码，jackson在序列化时会增加root对象：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时的序列化结果：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

- 再看反序列化场景：

1. 执行下面代码，jackson在反序列化时会先解析root对象：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. root对象的key，默认是实例的类名，如果实例有JsonRootName注解，就是该注解的value值；
2. root对象的value如下所示，相当于不支持root对象时用来反序列化的json字符串：

```java
{
  "id" : 1,
  "name" : "book"
}
1234
```

### 准备两个POJO类

用对比的方式可以更清楚了解JsonRootName的作用，接下来的学习咱们准备两个POJO类，一个没有JsonRootName注解，另一个有JsonRootName注解：

1. 名为Order1.java的，没有JsonRootName注解：

```java
public class Order1 {
    private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456
```

1. 名为Order2.java的，有JsonRootName注解，value值为aaabbbccc：

```java
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value = "aaabbbccc")
public class Order2 {
	private int id;
    private String name;
	// 省去get、set、toString方法
	...
}
123456789
```

- 可见Order1和Order2的代码是一致的，唯一的不同是Order2带有注解JsonRootName；

### 序列化

1. 需要设置WRAP_ROOT_VALUE属性，jackson才会支持root对象，JsonRootName注解才会发挥作用，设置代码如下：

```java
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
1
```

1. 写一段代码，在不开启WRAP_ROOT_VALUE属性的时候执行序列化，再开启WRAP_ROOT_VALUE属性执行序列化，对比试试：

```java
public static void main(String[] args) throws Exception {
        // 实例化Order1和Order2
        Order1 order1 = new Order1();
        order1. setId(1);
        order1.setName("book");

        Order2 order2 = new Order2();
        order2. setId(2);
        order2.setName("food");

        // 没有开启WRAP_ROOT_VALUE的时候
        logger.info("没有开启WRAP_ROOT_VALUE\n");
        ObjectMapper mapper1 = new ObjectMapper();
        // 美化输出
        mapper1.enable(SerializationFeature.INDENT_OUTPUT);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper1.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}\n\n\n\n", mapper1.writeValueAsString(order2));

        // 开启了WRAP_ROOT_VALUE的时候
        logger.info("开启了WRAP_ROOT_VALUE\n");
        ObjectMapper mapper2 = new ObjectMapper();
        // 美化输出
        mapper2.enable(SerializationFeature.INDENT_OUTPUT);
        // 序列化的时候支持JsonRootName注解
        mapper2.enable(SerializationFeature.WRAP_ROOT_VALUE);

        logger.info("没有JsonRootName注解类，序列化结果：\n\n{}\n\n", mapper2.writeValueAsString(order1));
        logger.info("有JsonRootName注解的类，序列化结果：\n\n{}", mapper2.writeValueAsString(order2));
    }
123456789101112131415161718192021222324252627282930
```

1. 执行结果如下，JsonRootName在序列化时的作用一目了然：指定了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140507.jpeg)

### 反序列化（默认设置）

1. 在没有做任何设置的时候，下面这个字符串用来反序列化成Order2对象，会成功吗？

```json
{
  "id" : 2,
  "name" : "food"
}
1234
```

1. 试了下是可以的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830223950284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 那下面这个字符串能反序列化成Order2对象吗？

```java
{
  "aaabbbccc" : {
    "id" : 2,
    "name" : "food"
  }
}
123456
```

1. 代码和结果如下图所示，反序列化时jackson并不认识aaabbbccc这个key，因为jackson此时并不支持root对象：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140509.jpeg)

- 小结：默认情况下，反序列化时json字符串不能有root对象；

### 反序列化（开启UNWRAP_ROOT_VALUE属性）

1. 如果开启了UNWRAP_ROOT_VALUE属性，用于反序列化的json字符串就必须要有root对象了，开启UNWRAP_ROOT_VALUE属性的代码如下：

```java
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
1
```

1. 代码和结果如下图，可见带有root对象的json字符串，可以反序列化成功，root对象的key就是JsonRootName注解的value属性：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140514.jpeg)
2. 值得注意的是，上述json字符串中，root对象的key为aaabbbccc，这和Order2的JsonRootName注解的value值是一致的，如果不一致就会反序列化失败，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140511.jpeg)

- 至此，jackson的WRAP_ROOT_VALUE特性就学习完成了，在web开发时这是个很常用的功能，用于在最外面包裹一层，以便整体上添加额外的内容，希望能给您带来参考；

# JsonInclude注解

### 本篇概览

1. 本文是《jackson学习》系列第五篇，来熟悉一个常用的注解JsonInclude，该注解的仅在序列化操作时有用，用于控制方法、属性等是否应该被序列化；
2. 之所以用单独的一篇来写JsonInclude注解，是因为该注解的值有多种，每种都有不同效果，最好的学习方法就是编码实战；
3. 先对注解的所有取值做个简介：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140615.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的jsoninclude这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140618.jpeg)

- 接下来逐个学习这些属性的效果；

### ALWAYS

ALWAYS表示全部序列化，如下图，null和空字符串都会序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140621.jpeg)

### NON_NULL

NON_NULL好理解，就是值为null就不序列化：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140624.jpeg)

### NON_ABSENT

1. NON_ABSENT略为复杂，当实例化的对象有Optional或AtomicReference类型的成员变量时，如果Optional引用的实例为空，用NON_ABSENT能使该字段不做序列化；
2. Optional是java用来优雅处理空指针的一个特性，本文中不做过多说明，请您自行查阅相关文档；
3. 要让Jackson支持Optional特性，必须做两件事，首先是在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.0</version>
</dependency>
12345
```

1. 其次是代码中执行以下设置：

```java
mapper.registerModule(new Jdk8Module());
1
```

1. 咱们先看看设置成NON_NULL时jackson对Optional和AtomicReference的处理，下面的代码中，Optional和AtomicReference的引用都是空，但还是被序列化出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140628.jpeg)
2. 代码不变，将NON_NULL改为NON_ABSENT试试，如下图，可见field2和field3都没有序列化了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140633.jpeg)
   小结NON_ABSENT的效果：
   a. 自身为null的字段不会被序列化；
   b. Optional类型的字段，如果引用值为null，该字段不会被序列化；
   c. AtomicReference类型的字段，如果引用值为null，该字段不会被序列化；

### NON_EMPTY

NON_EMPTY好理解，以下情况都不会被序列化：

1. null
2. 空字符串
3. 空集合
4. 空数组
5. Optional类型的，其引用为空
6. AtomicReference类型的，其引用为空
7. 演示代码和结果如下图，可见上述场景全部没有被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140637.jpeg)

### NON_DEFAULT

1. 设置为NON_DEFAULT后，对保持默认值的字段不做序列化，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140642.jpeg)

### CUSTOM

1. 相对其他类型，CUSTOM略为复杂，这个值要配合valueFilter属性一起使用；
2. 如下所示，JsonInclude的value等于CUSTOM时，在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化

```java
@JsonInclude(value = JsonInclude.Include.CUSTOM, 
                valueFilter = CustomFilter.class)
        private String field0;
123
```

1. 来看看CustomFilter类的代码，如下所示，只有equals方法，可见：null、非字符串、长度大于2这三种情况都返回true，也就是说这三种情况下都不会被序列化：

```java
static class CustomFilter {
        @Override
        public boolean equals(Object obj) {
            // null，或者不是字符串就返回true，意味着不被序列化
            if(null==obj || !(obj instanceof String)) {
                return true;
            }

            // 长度大于2就返回true，意味着不被序列化
            return ((String) obj).length() > 2;
        }
    }
123456789101112
```

1. 下面贴出完整代码和结果，您就一目了然了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140647.jpeg)

- 再次强调：valueFilter的equals方法返回true，意味着该字段不会被序列化！！！

### USE_DEFAULTS

USE_DEFAULTS的用法也有点绕，咱们通过对比的方法来学习；

1. 代码如下所示，在类和成员变量上都有JsonInclude注解，序列化field0的时候，是哪个注解生效呢？：

```java
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    static class Test {

        @JsonInclude(JsonInclude.Include.NON_NULL)
        private List<String> field0;

        public List<String> getField0() { return field0; }

        public void setField0(List<String> field0) { this.field0 = field0; }
    }
12345678910
```

1. 把field0设置为空集合，运行代码试试，如果类上的注解生效，那么field0就不会被序列化（NON_EMPTY会过滤掉空集合），如果成员变量上的注解生效，field0就会被序列化（NON_NULL只过滤null，空集合不是null），执行结果如下图，可见是成员变量上的注解生效了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140652.jpeg)
2. 接下来保持上述代码不变，仅在getField0方法上添加JsonInclude注释，值是USE_DEFAULTS，这样在序列化过程中，调用getField0方法时，就用类注解JsonInclude的值了，即NON_EMPTY：

```java
@JsonInclude(JsonInclude.Include.USE_DEFAULTS)
public List<String> getField0() { 
	return field0; 
}
1234
```

1. 执行修改后的代码，如下图所示，此时用的成员变量field0上的注解就不生效了，而是类注解生效，导致空集合不被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140655.jpeg)
   小结USE_DEFAULTS的作用如下：
   a. 类注解和成员变量注解同时存在时，以成员变量注解为准；
   b. 如果对应的get方法也使用了JsonInclude注解，并且值是USE_DEFAULTS，此时以类注解为准；

至此，JsonInclude注解的学习和实战就完成了，希望本文能给您提供参考，助您熟练使用注解来指定更精确的序列化过滤策略；

# 常用类注解

### 本篇概览

1. 本文是《jackson学习》系列的第六篇，继续学习jackson强大的注解能力，本篇学习的是常用的类注解，并通过实例来加深印象，下图是常用类注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140830.png)
2. 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140834.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的classannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140837.png)

### JsonRootName

1. JsonRootName的设置如下：

```java
@JsonRootName(value = "aaabbbccc")
static class Test {
	private String field0;
	public String getField0() { return field0; }
    public void setField0(String field0) { this.field0 = field0; }
}
123456
```

1. 开启root对象特性的代码以及序列化结果如下图，可见JsonRootName注解的value值aaabbbccc成了root对象的key：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140842.png)
2. 开启root对象的反序列化特性后，用上述红框3中的json字符串可反序列化成Test类的实例；
3. 关于root对象的序列化和反序列化特性，可以参考[jackson学习之四：WRAP_ROOT_VALUE（root对象）](https://blog.csdn.net/boling_cavalry/article/details/108298858)；

### JsonIgnoreProperties

1. 该注解用于指定序列化和反序列化时要忽略的字段，如下所示，Test类的field1和field2被设置为不参与序列化和反序列化操作：

```java
    @JsonIgnoreProperties({"field1", "field2"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 演示代码是JsonIgnorePropertiesSeriallization.java，执行结果如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140845.png)
2. 保持Test.java的JsonIgnoreProperties注解不变，再试试反序列化，对应的代码在JsonIgnorePropertiesDeserializer.java，如下图，反序列化后field1和field2依然是null，也就是说反序列化操作中，field1和field2都被忽略了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914215048482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### JsonIgnoreType

1. 被该注解修饰的类，作为其他类的成员变量时，不论是序列化还是反序列化都被忽略了；
2. 来验证一下，如下所示，TestChild类被JsonIgnoreType注解修饰：

```java
    @JsonIgnoreType
    static class TestChild {
        private int value;
        // 省去get、set、toString方法
1234
```

1. 如下所示，再把TestChild作为Test类的成员变量：

```java
    static class Test {
        private String field0;
        private TestChild field1;
        // 省去get、set、toString方法
1234
```

1. 序列化操作的代码是JsonIgnoreTypeSerialization.java，执行结果如下图，可见类型为TestChild的field1字段，在序列化的时候被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140849.png)
2. 再来试试反序列化，代码在JsonIgnoreTypeDeserializer.java，如下图，可见带有注解JsonIgnoreType的类作为成员变量，在反序列化时会被忽略：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204140853.png)

### JsonAutoDetect

1. 序列化和反序列化时自动识别的范围，如下：

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.PUBLIC_ONLY)
public class College {
    private String name;
    private String city;
    protected int age = 100;
12345
```

1. fieldVisibility属性有以下值可选：

```java
ANY // 所有
NON_PRIVATE // private之外的
PROTECTED_AND_PUBLIC // protected和public的(此时privte和默认的package access时不能被自动识别的)
PUBLIC_ONLY // public的
NONE // 禁止自动识别
DEFAULT // 默认的，用于继承父类的自动识别的范围
123456
```

1. 验证，如下图，College类设置了注解，fieldVisibility是PUBLIC_ONLY，红框中显示age字段是protected类型的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914220107887.png#pic_center)
2. 序列化结果如下图红框，age字段不是public，所以没有输出：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141607.png)
3. fieldVisibility改成NON_PRIVATE再试试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141612.png)
4. 如下图红框，age不是private，所以可以被序列化：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141615.png)

### JsonPropertyOrder

1. 这个注解好理解，就是指定序列化的顺序，注意该注解仅在序列化场景有效；
2. 先看看没有JsonPropertyOrder注解时的序列化顺序，Test.java如下所示，是和代码的顺序一致的：

```java
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
12345
```

1. 此时对Test的实例做序列化操作，结果如下图，顺序和代码顺序一致：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141619.png)
2. 现在给Test类加上JsonPropertyOrder注解，顺序是field2、field0、field1：

```java
    @JsonPropertyOrder({"field2", "field0", "field1"})
    static class Test {
        private String field0;
        private String field1;
        private String field2;
        // 省去get、set、toString方法
123456
```

1. 执行结果如下图所示，也是field2、field0、field1：
   ![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204141622.png)

### JsonInclude

1. 注解JsonInclude仅在序列化场景有效；
2. 通过该注解控制某些字段不被序列化（例如空字符串不被序列化）；
3. 可以设置以下几种限制：

```java
ALWAYS // 默认策略，任何情况都执行序列化
NON_NULL // 非空
NON_ABSENT // null的不会序列化，但如果类型是AtomicReference，依然会被序列化
NON_EMPTY // null、集合数组等没有内容、空字符串等，都不会被序列化
NON_DEFAULT // 如果字段是默认值，就不会被序列化
CUSTOM // 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化
USE_DEFAULTS // 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置  
1234567
```

1. JsonInclude涉及的知识点较多，已在一篇单独文章中详细说明，请参考[《jackson学习之五：JsonInclude注解》](https://blog.csdn.net/boling_cavalry/article/details/108412558)；

- 至此，Jackson的常用类注解的学习和实战就完成了，接下来的文章我们学习常用的属性注解；

# 常用Field注解

### 本篇概览

1. 本文是《jackson学习》系列的第七篇，继续学习jackson强大的注解能力，本篇学习的是常用的Field注解，并通过实例来加深印象，下图是常用Field注解的简介：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142611.png)
2. 接下来逐个学习；

### 不止是Filed

- 虽然标题说是常用Field注解，其实上图中的这些注解也能用在方法上，只不过多数情况下这些注解修饰在field上更好理解一些，例如JsonIgnore，放在field上和get方法上都是可以的；
- 接下来逐个学习；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142618.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的fieldannonation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142621.png)

### JsonProperty

1. JsonProperty可以作用在成员变量和方法上，作用是在序列化和反序列化操作中指定json字段的名称；
2. 先来看序列化操作（JsonPropertySerialization.java），如下所示，JsonProperty修饰了私有成员变量field0和公共方法getField1，并且field0没有get和set方法，是通过构造方法设置的，另外还要注意JsonProperty注解的index属性，用来指定序列化结果中的顺序，这里故意将field1的顺序设置得比field0靠前：

```java
    static class Test {

        @JsonProperty(value="json_field0", index = 1)
        private String field0;

        @JsonProperty(value="json_field1", index = 0)
        public String getField1() {
            return "111";
        }

        public Test(String field0) {
            this.field0 = field0;
        }
    }
1234567891011121314
```

1. 执行结果如下图红框所示，可见JsonProperty的value就是序列化后的属性名，另外带有JsonProperty注解的成员变量，即使是私有而且没有get和set方法，也能被成功序列化，而且顺序也和index属性对应：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142626.png)
2. 接下来看反序列化操作（JsonPropertyDeserialization.java），注解相关代码如下，field0是私有且没有get和set方法，另外setField1方法也有JsonProperty注解：

```java
    static class Test {

        @JsonProperty(value = "json_field0")
        private String field0;

        private String field1;

        @JsonProperty(value = "json_field1")
        public void setField1(String field1) {
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
1234567891011121314151617181920
```

1. 用json字符串尝试反序列化，结果如下，可见field0和field1都能被正确赋值：

![6.](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142630.png)

### JsonIgnore

1. JsonIgnore好理解，作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作；
2. 先看序列化操作（JsonIgnoreSerialization.java），如下所示，Test类的field1字段和getField2方法都有JsonIgnore注解：

```java
    static class Test {

        private String field0;

        @JsonIgnore
        private String field1;

        private String field2;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
        public void setField2(String field2) { this.field2 = field2; }

        @JsonIgnore
        public String getField2() { return field2; }
    }
123456789101112131415161718
```

1. 给field0、field1、field2三个字段都赋值，再看序列化结果，如下图，可见field0和field2都被忽略了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142635.png)
2. 再来尝试JsonIgnore注解在反序列化场景的作用，注意反序列化的时候，JsonIgnore作用的方法应该是set了，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142641.png)
3. 另外实测发现，反序列化的时候，JsonIgnore注解在get方法上也可以让对应字段被忽略；

### JacksonInject

1. JacksonInject的作用是在反序列化的时候，将配置好的值注入被JacksonInject注解的字段；
2. 如下所示，Test类的field1和field2都有JacksonInject注解，不同的是field1指定了注入值的key为defaultField1，而field2由于没有指定key，只能按照类型注入：

```java
    static class Test {
        private String field0;
        @JacksonInject(value = "defaultField1")
        private String field1;
        @JacksonInject
        private String field2;
123456
```

1. 注入时所需的数据来自哪里呢？如下所示，通过代码配置的，可以指定key对应的注入值，也可以指定类型对应的注入值：

```java
        InjectableValues.Std injectableValues = new InjectableValues.Std();
        // 指定key为"defaultField1"对应的注入参数
        injectableValues.addValue("defaultField1","field1 default value");
        // 指定String类型对应的注入参数
        injectableValues.addValue(String.class,"String type default value");
        ObjectMapper mapper = new ObjectMapper();        // 把注入参数的配置设置给mapper
        mapper.setInjectableValues(injectableValues);
1234567
```

1. 反序列化结果如下图，可见field1和field2的值都是被注入的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142646.png)

### JsonSerialize

1. JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类；
2. 执行序列化操作的类，需要继承自JsonSerializer，如下所示，Date2LongSerialize的作用是将Date类型转成long类型：

```java
    static class Date2LongSerialize extends JsonSerializer<Date> {

        @Override
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeNumber(value.getTime());
        }
    }

12345678
```

1. Test类的field0字段是私有的，且没有get和set方法，但是添加了注释JsonDeserialize就能被反序列化了，并且使用Date2LongSerialize类对将json中的long型转成field0所需的Date型：

```java
    static class Test {
        @JsonDeserialize(using = Long2DateDeserialize.class)
        private Date field0;
        @Override
        public String toString() { return "Test{" + "field0='" + field0 + '\'' + '}'; }
    }
123456
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142651.png)

### JsonDeserialize

1. JsonDeserialize用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；
2. 执行反序列化操作的类需要继承自JsonDeserializer，如下所示，Long2DateDeserialize的作用是将Long类型转成field0字段对应的Date类型：

```java
    static class Long2DateDeserialize extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

            if(null!=p && null!=ctxt && p.getLongValue()>0L ) {
                return new Date(p.getLongValue());
            }

            return null;
        }
    }
123456789101112
```

1. 测试反序列化，结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142657.png)

### JsonRawValue

最后要介绍的是JsonRawValue，使用该注解的字段或者方法，都会被序列化，但是序列化结果是原始值，例如字符串是不带双引号的：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142700.png)

- 至此，常用的Filed注解就操作完毕了，希望能带给您一些参考，助您更精确控制自己的序列化和反序列化操作；

# 常用方法注解

### 本篇概览

- 本文是《jackson学习》系列的第八篇，继续学习jackson强大的注解能力，本篇学习常用的方法注解，并通过实例来加深印象，下图是常用方法注解的简介：
  ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142752.jpeg)

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142758.png)
2. jacksondemo是父子结构的工程，本篇的代码在annotation子工程中，里面的methodannotation这个package下，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142801.jpeg)

### JsonValue

1. 在序列化时起作用，可以用来注解get方法或者成员变量；
2. 一个类中，JsonValue只允许出现一次；
3. 如果注解的是get方法，那么该方法的返回值就是整个实例的序列化结果；
4. 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；
5. 下面是用来测试的Pojo类，JsonValue注解放在getField0方法上，此方法的返回值已经写死了"abc"：

```java
    static class Test {

        private String field0;

        private String field1;

        @JsonValue
        public String getField0() { return "abc"; }

        public void setField0(String field0) { this.field0 = field0; }
        public String getField1() { return field1; }
        public void setField1(String field1) { this.field1 = field1; }
    }
12345678910111213
```

1. Test类的序列化结果如下，即getField0方法的返回值：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142808.jpeg)

### JsonCreator

1. 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系：

```java
    static class Test {

        private String field0;
        private String field1;


        public Test(String field0) {
            this.field0 = field0;
        }

        // 通过JsonCreator指定反序列化的时候使用这个构造方法
        // 通过JsonProperty指定字段关系
        @JsonCreator
        public Test(@JsonProperty("field0") String field0,
                    @JsonProperty("field1") String field1) {
            this.field0 = field0;
            this.field1 = field1;
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", field1='" + field1 + '\'' +
                    '}';
        }
    }
123456789101112131415161718192021222324252627
```

1. 反序列化结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142811.jpeg)

### JsonSetter

1. JsonSetter注解在set方法上，被用来在反序列化时指定set方法对应json的哪个属性；
2. JsonSetter源码中，推荐使用JsonProperty来取代JsonSetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142814.jpeg)
3. 测试代码和结果如下，可见反序列化时，是按照JsonSetter的value去json中查找属性的：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142817.jpeg)

### JsonGetter

1. JsonGetter只能作为方法注解；
2. 在序列化时，被JsonGetter注解的get方法，对应的json字段名是JsonGetter的value；
3. JsonGetter源码中，推荐使用JsonProperty来取代JsonGetter：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142820.jpeg)
4. 测试代码和结果如下，可见序列化时JsonGetter的value会被作为json字段名：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142823.jpeg)

### JsonAnyGetter

1. JsonAnyGetter的作用有些特别：在序列化时，用Map对象的键值对转成json的字段和值；
2. 理解JsonAnyGetter最好的办法，是对比使用前后序列化结果的变化，先来看以下这段代码，是没有JsonAnyGetter注解的，Test有两个成员变量，其中map字段是HashMap类型的：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterSerialization {

    static class Test {
        private String field0;
        private Map<String, Object> map;

        public String getField0() { return field0; }
        public void setField0(String field0) { this.field0 = field0; }
        public void setMap(Map<String, Object> map) { this.map = map; }
        public Map<String, Object> getMap() { return map; }
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        // 新增一个HashMap，里面放入两个元素
        Map<String, Object> map = new HashMap<>();
        map.put("aaa", "value_aaa");
        map.put("bbb", "value_bbb");

        Test test = new Test();
        test.setField0("000");

        // map赋值给test.map
        test.setMap(map);

        System.out.println(mapper.writeValueAsString(test));
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940
```

1. 上述代码的执行结果如下，其实很好理解，就是field0和map两个字段而已：

```JSON
{
  "field0" : "000",
  "map" : {
    "aaa" : "value_aaa",
    "bbb" : "value_bbb"
  }
}
1234567
```

1. 接下来，对上述代码做一处改动，如下图红框所示，给getMap方法增加JsonAnyGetter注解：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204142827.jpeg)
2. 修改后的执行结果如下，原来的map字段没有了，map内部的所有键值对都成了json的字段：

```JSON
{
  "field0" : "000",
  "aaa" : "value_aaa",
  "bbb" : "value_bbb"
}
12345
```

1. 至此，可以品味出JsonAnyGetter的作用了：序列化时，将Map中的键值对全部作为JSON的字段输出；

### JsonAnySetter

1. 弄懂了前面的JsonAnyGetter，对于JsonAnySetter的作用想必您也能大致猜到：反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理；
2. 测试的代码如下，Test类的setValue方法被JsonAnySetter注解，在反序列化时，json中的aaa和bbb字段，都会交给setValue方法处理，也就是放入map中：

```java
package com.bolingcavalry.jacksondemo.annotation.methodannotation;

import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;

public class JsonAnySetterDeserialization {

    static class Test {

        private String field0;
        
        private Map<String, Object> map = new HashMap<>();

        @JsonAnySetter
        public void setValue(String key, Object value) {
            map.put(key, value);
        }

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {
        String jsonStr = "{\n" +
                "  \"field0\" : \"000\",\n" +
                "  \"aaa\" : \"value_aaa\",\n" +
                "  \"bbb\" : \"value_bbb\"\n" +
                "}";

        System.out.println(new ObjectMapper().readValue(jsonStr, Test.class));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142
```

1. 执行结果如下，可见aaa、bbb都被放入了map中：

```java
Test{field0='null', map={aaa=value_aaa, field0=000, bbb=value_bbb}}
1
```

1. 另外JsonAnySetter还可以作用在成员变量上，上面的代码中，去掉setValue方法，在成员变量map上增加JsonAnySetter注解，修改后如下，执行结果也是一模一样的：

```java
    static class Test {

        private String field0;

        @JsonAnySetter
        private Map<String, Object> map = new HashMap<>();

        @Override
        public String toString() {
            return "Test{" +
                    "field0='" + field0 + '\'' +
                    ", map=" + map +
                    '}';
        }
    }
123456789101112131415
```

1. 注意，JsonAnySetter作用在成员变量上时，该成员变量必须是java.util.Map的实现类；

- 至此，Jackson常用注解已全部实战完毕，希望这些丰富的注解能助您制定出各种灵活的序列化和反序列化策略；

# springboot整合(配置文件)

### 本篇概览

今天实战内容如下：

1. 开发springboot应用，体验springboot默认支持jackson，包括jackson注解和ObjectMapper实例的注入；
2. 在application.yml中添加jackson配置，验证是否生效；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145625.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootproperties子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145628.jpeg)

### 开始实战

1. 由于同属于《jackson学习》系列文章，因此本篇的springboot工程作为jacksondemo的子工程存在，pom.xml如下，需要注意的是parent不能使用spring-boot-starter-parent，而是通过dependencyManagement节点来引入springboot依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootproperties</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootproperties</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374
```

1. 启动类很平常：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootpropertiesApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootpropertiesApplication.class, args);
    }
}
123456789101112
```

1. 由于用到了swagger，因此要添加swagger配置：

```java
package com.bolingcavalry.springbootproperties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootproperties.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 序列化和反序列化用到的Bean类，可见使用了JsonProperty属性来设置序列化和反序列化时的json属性名，field0字段刻意没有get方法，是为了验证JsonProperty的序列化能力：

```java
package com.bolingcavalry.springbootproperties.bean;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(description = "JsonProperty注解测试类")
public class Test {

    @ApiModelProperty(value = "私有成员变量")
    @JsonProperty(value = "json_field0", index = 1)
    private Date field0 = new Date();

    public void setField0(Date field0) {
        this.field0 = field0;
    }

    @ApiModelProperty(value = "来自get方法的字符串")
    @JsonProperty(value = "json_field1", index = 0)
    public String getField1() {
        return "111";
    }

    @Override
    public String toString() {
        return "Test{" +
                "field0=" + field0 +
                '}';
    }
}
1234567891011121314151617181920212223242526272829303132
```

1. 测试用的Controller代码如下，很简单只有两个接口，serialization返回序列化结果，deserialization接受客户端请求参数，反序列化成实例，通过toString()来检查反序列化的结果，另外，还通过Autowired注解从spring容器中将ObjectMapper实例直接拿来用：

```java
package com.bolingcavalry.springbootproperties.controller;

import com.bolingcavalry.springbootproperties.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 验证(不用配置文件)

1. 先来看看没有配置文件时，默认的jackson配置的表现，直接在IDEA上运行SpringbootpropertiesApplication；
2. 浏览器访问http://localhost:8080/swagger-ui.html ，如下图红框1，json_field0和json_field1都是JsonProperty注释，出现在了swagger的model中，这证明jackson注解已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145637.jpeg)
3. 点击上图的红框2，看看springboot引用返回的序列化结果，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145640.jpeg)
4. 另外，上述红框中的json格式，每个属性单独一行，像是做了格式化调整的，这是springboot做的？还是swagger展示的时候做的？用浏览器访问http://localhost:8080/jsonproperty/serialization ，结果如下，可见springboot返回的是未经过格式化的json：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145645.jpeg)

- 接下来咱们添加jackson相关的配置信息并验证是否生效；

### 添加配置文件并验证

1. 在resources目录新增application.yml文件，内容如下：

```yml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true
123456789101112131415161718192021
```

1. 将鼠标放置下图红框位置，再按住Ctlr键，IDEA会弹出一个浮层，提示该配置对应的jackson代码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145649.jpeg)
2. 在上图中，按住Ctlr键，用鼠标点击红框位置即可打开此配置对应的jackson源码，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145651.jpeg)
3. 重新运行springboot应用，用浏览器访问：http://localhost:8080/jsonproperty/serialization ，结果如下图，可见json_field0的格式变成了yyyy-MM-dd HH:mm:ss，而且json输出也做了格式化，证明application.yml中的配置已经生效：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145655.jpeg)
4. 再来试试反序列化，打开swagger页面，操作和响应如下图所示，注意红框1里面请求参数的格式：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145657.jpeg)

- 至此，在springboot中通过yml配置jackson的操作实战就完成了，接下来的章节，咱们在配置类中用代码来完成yml的配置；

# springboot整合(配置类)

### 本篇概览

- 本文是《jackson学习》系列的终篇，经过前面的一系列实战，相信您已可以熟练使用jackson灵活的执行各种json序列化和反序列化操作，那么，本篇就以轻松的方式来完成整个系列吧；
- 上一篇介绍的是在springboot中通过配置文件对jackson做设置，今天要聊的是另一种常用的jackson配置方式：配置类，就是自己编写代码实例化和配置springboot全局使用的ObjectMapper实例；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在jacksondemo文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145704.png)
2. jacksondemo是父子结构的工程，本篇的代码在springbootconfigbean子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145707.jpeg)

### 编码

1. 在父工程jacksondemo下新增子工程springbootconfigbean，pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>jacksondemo</artifactId>
        <groupId>com.bolingcavalry</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>springbootconfigbean</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootconfigbean</name>
    <description>Demo project for Spring Boot with Jackson, configuration from config bean</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--不用spring-boot-starter-parent作为parent时的配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- swagger依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
        </dependency>
        <!-- swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

1. 本文最重要的代码是配置类JacksonConfig.java，如下，需要ConditionalOnMissingBean注解避免冲突，另外还给实例指定了名称customizeObjectMapper，如果应用中通过Autowired使用此实例，需要指定这个名字，避免报错"There is more than one bean of 'ObjectMapper ’ type"：

```java
@Configuration
public class JacksonConfig {

    @Bean("customizeObjectMapper")
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper getObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.build();

        // 日期格式
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

        // 美化输出
        mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
123456789101112131415161718
```

1. 对于JacksonConfig.getObjectMapper方法内的设置，如果您想做更多设置，请参考[《jackson学习之三：常用API操作》](https://xinchen.blog.csdn.net/article/details/108192174)里面的设置内容；

- 启动类依然很简单：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootConfigBeanApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigBeanApplication.class, args);
    }

}
12345678910111213
```

1. swagger配置：

```java
package com.bolingcavalry.springbootconfigbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.Tag;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .tags(new Tag("JsonPropertySerializationController", "JsonProperty相关测试"))
                .select()
                // 当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.bolingcavalry.springbootconfigbean.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("SpringBoot整合Jackson(基于配置文件)")
                //创建人
                .contact(new Contact("程序员欣宸", "https://github.com/zq2599/blog_demos", "zq2599@gmail.com"))
                //版本号
                .version("1.0")
                //描述
                .description("API 描述")
                .build();
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344
```

1. 最后是测试用的Controller类，要注意的是在使用ObjectMapper实例的地方，用Autowired注解的时候，记得带上Qualifier注解：

```java
package com.bolingcavalry.springbootconfigbean.controller;

import com.bolingcavalry.springbootconfigbean.bean.Test;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/jsonproperty")
@Api(tags = {"JsonPropertySerializationController"})
public class JsonPropertySerializationController {

    private static final Logger logger = LoggerFactory.getLogger(JsonPropertySerializationController.class);

    @Qualifier("customizeObjectMapper")
    @Autowired
    ObjectMapper mapper;

    @ApiOperation(value = "测试序列化", notes = "测试序列化")
    @RequestMapping(value = "/serialization", method = RequestMethod.GET)
    public Test serialization() throws JsonProcessingException {

        Test test = new Test();
        logger.info(mapper.writeValueAsString(test));

        return test;
    }

    @ApiOperation(value = "测试反序列化", notes="测试反序列化")
    @RequestMapping(value = "/deserialization",method = RequestMethod.PUT)
    public String deserialization(@RequestBody Test test) {
        return test.toString();
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

### 验证

1. 启动SpringbootConfigBeanApplication后，浏览器打开：http://localhost:8080/swagger-ui.html
2. 先验证序列化接口/jsonproperty/serialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145714.jpeg)
3. 再验证反序列化接口 /jsonproperty/deserialization：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204145716.jpeg)

- 至此，整个《jackson学习》系列就全部完成了，希望这十篇内容能够给您带来一些参考，助您在编码过程中更加得心应手的使用Jackson；

# 参考

https://github.com/zq2599/blog_demos

https://xinchen.blog.csdn.net/article/details/107135958