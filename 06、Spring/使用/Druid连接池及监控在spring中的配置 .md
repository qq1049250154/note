偶尔的机会解释Druid连接池，后起之秀，但是评价不错，另外由于是阿里淘宝使用过的所以还是蛮看好的。

Druid集连接池，监控于一体整好复合当前项目的需要，项目是ssh结构，之前是用C3p0的，现在换一个连接池也是很简单的，首先spring配置DataSource，配置如下：

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
    <!-- 基本属性 url、user、password -->
    <property name="url" value="${jdbc_url}" />
    <property name="username" value="${jdbc_user}" />
    <property name="password" value="${jdbc_password}" />
    <!-- 配置初始化大小、最小、最大 -->
    <property name="initialSize" value="1" />
    <property name="minIdle" value="1" /> 
    <property name="maxActive" value="20" />

    <!-- 配置获取连接等待超时的时间 -->
    <property name="maxWait" value="60000" />

    <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
    <property name="timeBetweenEvictionRunsMillis" value="60000" />

    <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
    <property name="minEvictableIdleTimeMillis" value="300000" />

    <property name="validationQuery" value="SELECT 'x'" />
    <property name="testWhileIdle" value="true" />
    <property name="testOnBorrow" value="false" />
    <property name="testOnReturn" value="false" />

    <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
    <property name="poolPreparedStatements" value="true" />
    <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />

    <!-- 配置监控统计拦截的filters，去掉后监控界面sql无法统计 -->
    <property name="filters" value="stat" /> 
</bean>
```


目前这样的配置已经能够使用连接池，注意不要忘记了jar文件，下载地址：http://code.alibabatech.com/mvn/releases/com/alibaba/druid/

然后是监控的配置：

web.xml

```xml
<filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
        <param-name>exclusions</param-name>
        <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```
filter可以监控webURl 访问

```xml
<servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>
</servlet-mapping>
```
该配置可以访问监控界面

配置好后访问 http://ip：port/projectName/druid/index.html

经过上面的配置，我们已经能够达到连接池的使用和监控，这个只是简单的入门，如果还要更详细的学习，还得论坛上多多交流。
————————————————
版权声明：本文为CSDN博主「兮风」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/pk490525/article/details/12621649