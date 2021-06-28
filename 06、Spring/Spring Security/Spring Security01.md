# Spring Security 介绍

Java 领域老牌的权限管理框架当属 Shiro 了。

Shiro 有着众多的优点，例如轻量、简单、易于集成等。

当然 Shiro 也有不足，例如对 OAuth2 支持不够，在 Spring Boot 面前无法充分展示自己的优势等等，特别是随着现在 Spring Boot 和 Spring Cloud 的流行，Spring Security 正在走向舞台舞台中央。

## 陈年旧事

Spring Security 最早不叫 Spring Security ，叫 Acegi Security，叫 Acegi Security 并不是说它和 Spring 就没有关系了，它依然是为 Spring 框架提供安全支持的。事实上，Java 领域的框架，很少有框架能够脱离 Spring 框架独立存在。

Acegi Security 基于 Spring，可以帮助我们为项目建立丰富的角色与权限管理，但是最广为人诟病的则是它臃肿繁琐的配置，这一问题最终也遗传给了 Spring Security。

在 Acegi Security 时代，网上流传一句话：“每当有人要使用 Acegi Security，就会有一个精灵死去。”足见 Acegi Security 的配置是多么可怕。

当 Acegi Security 投入 Spring 怀抱之后，先把这个名字改了，这就是大家所见到的 Spring Security 了，然后配置也得到了极大的简化。

但是和 Shiro 相比，人们对 Spring Security 的评价依然中重量级、配置繁琐。

直到有一天 Spring Boot 像谜一般出现在江湖边缘，彻底颠覆了 JavaEE 的世界。一人得道鸡犬升天，Spring Security 也因此飞上枝头变凤凰。

到现在，要不要学习 Spring Security 已经不是问题了，无论是 Spring Boot 还是 Spring Cloud，你都有足够多的机会接触到 Spring Security，现在的问题是如何快速掌握 Spring Security？那么看松哥的教程就对了。

##  核心功能

对于一个权限管理框架而言，无论是 Shiro 还是 Spring Security，最最核心的功能，无非就是两方面：

- 认证
- 授权

通俗点说，认证就是我们常说的登录，授权就是权限鉴别，看看请求是否具备相应的权限。

虽然就是一个简简单单的登录，可是也能玩出很多花样来。

Spring Security 支持多种不同的认证方式，这些认证方式有的是 Spring Security 自己提供的认证功能，有的是第三方标准组织制订的，主要有如下一些：

一些比较常见的认证方式：

- HTTP BASIC authentication headers：基于IETF RFC 标准。
- HTTP Digest authentication headers：基于IETF RFC 标准。
- HTTP X.509 client certificate exchange：基于IETF RFC 标准。
- LDAP：跨平台身份验证。
- Form-based authentication：基于表单的身份验证。
- Run-as authentication：用户用户临时以某一个身份登录。
- OpenID authentication：去中心化认证。

除了这些常见的认证方式之外，一些比较冷门的认证方式，Spring Security 也提供了支持。

- Jasig Central Authentication Service：单点登录。
- Automatic “remember-me” authentication：记住我登录（允许一些非敏感操作）。
- Anonymous authentication：匿名登录。
- ……

作为一个开放的平台，Spring Security 提供的认证机制不仅仅是上面这些。如果上面这些认证机制依然无法满足你的需求，我们也可以自己定制认证逻辑。当我们需要和一些“老破旧”的系统进行集成时，自定义认证逻辑就显得非常重要了。

除了认证，剩下的就是授权了。

Spring Security 支持基于 URL 的请求授权、支持方法访问授权以及对象访问授权。

# 快速开始

## 1.新建项目

首先新建一个 Spring Boot 项目，创建时引入 Spring Security 依赖和 web 依赖，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100911.png)](http://img.itboyhub.com//2020/03/spring-security-2-1.png)

项目创建成功后，Spring Security 的依赖就添加进来了，在 Spring Boot 中我们加入的是 `spring-boot-starter-security` ，其实主要是这两个：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100906.png)](http://img.itboyhub.com//2020/03/spring-security-2-2.png)

项目创建成功后，我们添加一个测试的 HelloController，内容如下：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

接下来什么事情都不用做，我们直接来启动项目。

在项目启动过程中，我们会看到如下一行日志：

```
Using generated security password: 30abfb1f-36e1-446a-a79b-f70024f589ab
```

这就是 Spring Security 为默认用户 user 生成的临时密码，是一个 UUID 字符串。

接下来我们去访问 `http://localhost:8080/hello` 接口，就可以看到自动重定向到登录页面了：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100903.png)](http://img.itboyhub.com//2020/03/spring-security-2-3.png)

在登录页面，默认的用户名就是 user，默认的登录密码则是项目启动时控制台打印出来的密码，输入用户名密码之后，就登录成功了，登录成功后，我们就可以访问到 /hello 接口了。

在 Spring Security 中，默认的登录页面和登录接口，都是 `/login` ，只不过一个是 get 请求（登录页面），另一个是 post 请求（登录接口）。

**大家可以看到，非常方便，一个依赖就保护了所有接口。**

有人说，你怎么知道知道生成的默认密码是一个 UUID 呢？

这个其实很好判断。

和用户相关的自动化配置类在 `UserDetailsServiceAutoConfiguration` 里边，在该类的 `getOrDeducePassword` 方法中，我们看到如下一行日志：

```
if (user.isPasswordGenerated()) {
	logger.info(String.format("%n%nUsing generated security password: %s%n", user.getPassword()));
}
```

毫无疑问，我们在控制台看到的日志就是从这里打印出来的。打印的条件是 isPasswordGenerated 方法返回 true，即密码是默认生成的。

进而我们发现，user.getPassword 出现在 SecurityProperties 中，在 SecurityProperties 中我们看到如下定义：

```
/**
 * Default user name.
 */
private String name = "user";
/**
 * Password for the default user name.
 */
private String password = UUID.randomUUID().toString();
private boolean passwordGenerated = true;
```

可以看到，默认的用户名就是 user，默认的密码则是 UUID，而默认情况下，passwordGenerated 也为 true。

## 2.用户配置

默认的密码有一个问题就是每次重启项目都会变，这很不方便。

### 2.1 配置文件

我们可以在 application.properties 中配置默认的用户名密码。

怎么配置呢？大家还记得上一小节我们说的 SecurityProperties，默认的用户就定义在它里边，是一个静态内部类，我们如果要定义自己的用户名密码，必然是要去覆盖默认配置，我们先来看下 SecurityProperties 的定义：

```
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {
```

这就很清晰了，我们只需要以 spring.security.user 为前缀，去定义用户名密码即可：

```
spring.security.user.name=javaboy
spring.security.user.password=123
```

这就是我们新定义的用户名密码。

在 properties 中定义的用户名密码最终是通过 set 方法注入到属性中去的，这里我们顺便来看下 SecurityProperties.User#setPassword 方法:

```
public void setPassword(String password) {
	if (!StringUtils.hasLength(password)) {
		return;
	}
	this.passwordGenerated = false;
	this.password = password;
}
```

从这里我们可以看到，application.properties 中定义的密码在注入进来之后，还顺便设置了 passwordGenerated 属性为 false，这个属性设置为 false 之后，控制台就不会打印默认的密码了。

此时重启项目，就可以使用自己定义的用户名/密码登录了。

### 2.2 配置类

除了上面的配置文件这种方式之外，我们也可以在配置类中配置用户名/密码。

在配置类中配置，我们就要指定 PasswordEncoder 了，这是一个非常关键的东西。

考虑到有的小伙伴对于 PasswordEncoder 还不太熟悉，因此，我这里先稍微给大家介绍一下 PasswordEncoder 到底是干嘛用的。要说 PasswordEncoder ，就得先说密码加密。

#### 2.2.1 为什么要加密

2011 年 12 月 21 日，有人在网络上公开了一个包含 600 万个 CSDN 用户资料的数据库，数据全部为明文储存，包含用户名、密码以及注册邮箱。事件发生后 CSDN 在微博、官方网站等渠道发出了声明，解释说此数据库系 2009 年备份所用，因不明原因泄露，已经向警方报案，后又在官网发出了公开道歉信。在接下来的十多天里，金山、网易、京东、当当、新浪等多家公司被卷入到这次事件中。整个事件中最触目惊心的莫过于 CSDN 把用户密码明文存储，由于很多用户是多个网站共用一个密码，因此一个网站密码泄露就会造成很大的安全隐患。由于有了这么多前车之鉴，我们现在做系统时，密码都要加密处理。

这次泄密，也留下了一些有趣的事情，特别是对于广大程序员设置密码这一项。人们从 CSDN 泄密的文件中，发现了一些好玩的密码，例如如下这些：

- `ppnn13%dkstFeb.1st` 这段密码的中文解析是：娉娉袅袅十三余，豆蔻梢头二月初。
- `csbt34.ydhl12s` 这段密码的中文解析是：池上碧苔三四点，叶底黄鹂一两声
- …

等等不一而足，你会发现很多程序员的人文素养还是非常高的，让人啧啧称奇。

#### 2.2.2 加密方案

密码加密我们一般会用到散列函数，又称散列算法、哈希函数，这是一种从任何数据中创建数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来，然后将数据打乱混合，重新创建一个散列值。散列值通常用一个短的随机字母和数字组成的字符串来代表。好的散列函数在输入域中很少出现散列冲突。在散列表和数据处理中，不抑制冲突来区别数据，会使得数据库记录更难找到。我们常用的散列函数有 MD5 消息摘要算法、安全散列算法（Secure Hash Algorithm）。

但是仅仅使用散列函数还不够，为了增加密码的安全性，一般在密码加密过程中还需要加盐，所谓的盐可以是一个随机数也可以是用户名，加盐之后，即使密码明文相同的用户生成的密码密文也不相同，这可以极大的提高密码的安全性。但是传统的加盐方式需要在数据库中有专门的字段来记录盐值，这个字段可能是用户名字段（因为用户名唯一），也可能是一个专门记录盐值的字段，这样的配置比较繁琐。

Spring Security 提供了多种密码加密方案，官方推荐使用 BCryptPasswordEncoder，BCryptPasswordEncoder 使用 BCrypt 强哈希函数，开发者在使用时可以选择提供 strength 和 SecureRandom 实例。strength 越大，密钥的迭代次数越多，密钥迭代次数为 2^strength。strength 取值在 4~31 之间，默认为 10。

不同于 Shiro 中需要自己处理密码加盐，在 Spring Security 中，BCryptPasswordEncoder 就自带了盐，处理起来非常方便。

而 BCryptPasswordEncoder 就是 PasswordEncoder 接口的实现类。

#### 2.2.3 PasswordEncoder

PasswordEncoder 这个接口中就定义了三个方法：

```
public interface PasswordEncoder {
	String encode(CharSequence rawPassword);
	boolean matches(CharSequence rawPassword, String encodedPassword);
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}
}
```

1. encode 方法用来对明文密码进行加密，返回加密之后的密文。
2. matches 方法是一个密码校对方法，在用户登录的时候，将用户传来的明文密码和数据库中保存的密文密码作为参数，传入到这个方法中去，根据返回的 Boolean 值判断用户密码是否输入正确。
3. upgradeEncoding 是否还要进行再次加密，这个一般来说就不用了。

通过下图我们可以看到 PasswordEncoder 的实现类：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100839.png)](http://img.itboyhub.com//2020/03/spring-security-2-4.png)

#### 2.2.4 配置

预备知识讲完后，接下来我们来看具体如何配置：

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("javaboy.org")
                .password("123").roles("admin");
    }
}
```

1. 首先我们自定义 SecurityConfig 继承自 WebSecurityConfigurerAdapter，重写里边的 configure 方法。
2. 首先我们提供了一个 PasswordEncoder 的实例，因为目前的案例还比较简单，因此我暂时先不给密码进行加密，所以返回 NoOpPasswordEncoder 的实例即可。
3. configure 方法中，我们通过 inMemoryAuthentication 来开启在内存中定义用户，withUser 中是用户名，password 中则是用户密码，roles 中是用户角色。
4. 如果需要配置多个用户，用 and 相连。

为什么用 and 相连呢？

> 在没有 Spring Boot 的时候，我们都是 SSM 中使用 Spring Security，这种时候都是在 XML 文件中配置 Spring Security，既然是 XML 文件，标签就有开始有结束，现在的 and 符号相当于就是 XML 标签的结束符，表示结束当前标签，这是个时候上下文会回到 inMemoryAuthentication 方法中，然后开启新用户的配置。

配置完成后，再次启动项目，Java 代码中的配置会覆盖掉 XML 文件中的配置，此时再去访问 /hello 接口，就会发现只有 Java 代码中的用户名/密码才能访问成功。

## 3.自定义表单登录页

默认的表单登录有点丑（实际上现在默认的表单登录比以前的好多了，以前的更丑）。

但是很多时候我们依然绝对这个登录页面有点丑，那我们可以自定义一个登录页面。

一起来看下。

### 3.1 服务端定义

然后接下来我们继续完善前面的 SecurityConfig 类，继续重写它的 `configure(WebSecurity web)` 和 `configure(HttpSecurity http)`方法，如下：

```
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/js/**", "/css/**","/images/**");
}
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .permitAll()
            .and()
            .csrf().disable();
}
```

1. web.ignoring() 用来配置忽略掉的 URL 地址，一般对于静态文件，我们可以采用此操作。
2. 如果我们使用 XML 来配置 Spring Security ，里边会有一个重要的标签 `<http>`，HttpSecurity 提供的配置方法 都对应了该标签。
3. authorizeRequests 对应了 `<intercept-url>`。
4. formLogin 对应了 `<formlogin>`。
5. and 方法表示结束当前标签，上下文回到HttpSecurity，开启新一轮的配置。
6. permitAll 表示登录相关的页面/接口不要被拦截。
7. 最后记得关闭 csrf ，关于 csrf 问题我到后面专门和大家说。

当我们定义了登录页面为 /login.html 的时候，Spring Security 也会帮我们自动注册一个 /login.html 的接口，这个接口是 POST 请求，用来处理登录逻辑。

### 3.2 前端定义

松哥这里准备了一个还过得去的登录页面，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100833.png)](http://img.itboyhub.com//2020/03/spring-security-2-6.png)

我们将登录页面的相关静态文件拷贝到 Spring Boot 项目的 resources/static 目录下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204100827.png)](http://img.itboyhub.com//2020/03/spring-security-2-7.png)

前端页面比较长，这里我把核心部分列出来（完整代码我会上传到 GitHub：https://github.com/lenve/spring-security-samples）：

```
<form action="/login.html" method="post">
    <div class="input">
        <label for="name">用户名</label>
        <input type="text" name="username" id="name">
        <span class="spin"></span>
    </div>
    <div class="input">
        <label for="pass">密码</label>
        <input type="password" name="password" id="pass">
        <span class="spin"></span>
    </div>
    <div class="button login">
        <button type="submit">
            <span>登录</span>
            <i class="fa fa-check"></i>
        </button>
    </div>
</form>
```

form 表单中，注意 action 为 `/login.html` ，其他的都是常规操作，我就不重复介绍了。

好了，配置完成后，再去重启项目，此时访问任意页面，就会自动重定向到我们定义的这个页面上来，输入用户名密码就可以重新登录了。

# 定制 Spring Security 中的表单登录

## 1.登录接口

很多初学者分不清登录接口和登录页面，这个我也很郁闷。我还是在这里稍微说一下。

登录页面就是你看到的浏览器展示出来的页面，像下面这个：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204103038.png)](http://img.itboyhub.com//2020/03/spring-security-2-6.png)

登录接口则是提交登录数据的地方，就是登录页面里边的 form 表单的 action 属性对应的值。

在 Spring Security 中，如果我们不做任何配置，默认的登录页面和登录接口的地址都是 `/login`，也就是说，默认会存在如下两个请求：

- GET http://localhost:8080/login
- POST http://localhost:8080/login

如果是 GET 请求表示你想访问登录页面，如果是 POST 请求，表示你想提交登录数据。

在[上篇文章](https://mp.weixin.qq.com/s/Q0GkUb1Nt6ynV22LFHuQrQ)中，我们在 SecurityConfig 中自定定义了登录页面地址，如下：

```
.and()
.formLogin()
.loginPage("/login.html")
.permitAll()
.and()
```

当我们配置了 loginPage 为 `/login.html` 之后，这个配置从字面上理解，就是设置登录页面的地址为 `/login.html`。

实际上它还有一个隐藏的操作，就是登录接口地址也设置成 `/login.html` 了。换句话说，新的登录页面和登录接口地址都是 `/login.html`，现在存在如下两个请求：

- GET http://localhost:8080/login.html
- POST http://localhost:8080/login.html

前面的 GET 请求用来获取登录页面，后面的 POST 请求用来提交登录数据。

有的小伙伴会感到奇怪？为什么登录页面和登录接口不能分开配置呢？

其实是可以分开配置的！

在 SecurityConfig 中，我们可以通过 loginProcessingUrl 方法来指定登录接口地址，如下：

```
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.permitAll()
.and()
```

这样配置之后，登录页面地址和登录接口地址就分开了，各是各的。

此时我们还需要修改登录页面里边的 action 属性，改为 `/doLogin`，如下：

```
<form action="/doLogin" method="post">
<!--省略-->
</form>
```

此时，启动项目重新进行登录，我们发现依然可以登录成功。

那么为什么默认情况下两个配置地址是一样的呢？

我们知道，form 表单的相关配置在 FormLoginConfigurer 中，该类继承自 AbstractAuthenticationFilterConfigurer ，所以当 FormLoginConfigurer 初始化的时候，AbstractAuthenticationFilterConfigurer 也会初始化，在 AbstractAuthenticationFilterConfigurer 的构造方法中，我们可以看到：

```
protected AbstractAuthenticationFilterConfigurer() {
	setLoginPage("/login");
}
```

这就是配置默认的 loginPage 为 `/login`。

另一方面，FormLoginConfigurer 的初始化方法 init 方法中也调用了父类的 init 方法：

```
public void init(H http) throws Exception {
	super.init(http);
	initDefaultLoginFilter(http);
}
```

而在父类的 init 方法中，又调用了 updateAuthenticationDefaults，我们来看下这个方法：

```
protected final void updateAuthenticationDefaults() {
	if (loginProcessingUrl == null) {
		loginProcessingUrl(loginPage);
	}
	//省略
}
```

从这个方法的逻辑中我们就可以看到，如果用户没有给 loginProcessingUrl 设置值的话，默认就使用 loginPage 作为 loginProcessingUrl。

而如果用户配置了 loginPage，在配置完 loginPage 之后，updateAuthenticationDefaults 方法还是会被调用，此时如果没有配置 loginProcessingUrl，则使用新配置的 loginPage 作为 loginProcessingUrl。

好了，看到这里，相信小伙伴就明白了为什么一开始的登录接口和登录页面地址一样了。

## 2.登录参数

说完登录接口，我们再来说登录参数。

在[上篇文章](https://mp.weixin.qq.com/s/Q0GkUb1Nt6ynV22LFHuQrQ)中，我们的登录表单中的参数是 username 和 password，注意，默认情况下，这个不能变：

```
<form action="/login.html" method="post">
    <input type="text" name="username" id="name">
    <input type="password" name="password" id="pass">
    <button type="submit">
      <span>登录</span>
    </button>
</form>
```

那么为什么是这样呢？

还是回到 FormLoginConfigurer 类中，在它的构造方法中，我们可以看到有两个配置用户名密码的方法：

```
public FormLoginConfigurer() {
	super(new UsernamePasswordAuthenticationFilter(), null);
	usernameParameter("username");
	passwordParameter("password");
}
```

在这里，首先 super 调用了父类的构造方法，传入了 UsernamePasswordAuthenticationFilter 实例，该实例将被赋值给父类的 authFilter 属性。

接下来 usernameParameter 方法如下：

```
public FormLoginConfigurer<H> usernameParameter(String usernameParameter) {
	getAuthenticationFilter().setUsernameParameter(usernameParameter);
	return this;
}
```

getAuthenticationFilter 实际上是父类的方法，在这个方法中返回了 authFilter 属性，也就是一开始设置的 UsernamePasswordAuthenticationFilter 实例，然后调用该实例的 setUsernameParameter 方法去设置登录用户名的参数：

```
public void setUsernameParameter(String usernameParameter) {
	this.usernameParameter = usernameParameter;
}
```

这里的设置有什么用呢？当登录请求从浏览器来到服务端之后，我们要从请求的 HttpServletRequest 中取出来用户的登录用户名和登录密码，怎么取呢？还是在 UsernamePasswordAuthenticationFilter 类中，有如下两个方法：

```
protected String obtainPassword(HttpServletRequest request) {
	return request.getParameter(passwordParameter);
}
protected String obtainUsername(HttpServletRequest request) {
	return request.getParameter(usernameParameter);
}
```

可以看到，这个时候，就用到默认配置的 username 和 password 了。

当然，这两个参数我们也可以自己配置，自己配置方式如下：

```
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.usernameParameter("name")
.passwordParameter("passwd")
.permitAll()
.and()
```

配置完成后，也要修改一下前端页面：

```
<form action="/doLogin" method="post">
    <div class="input">
        <label for="name">用户名</label>
        <input type="text" name="name" id="name">
        <span class="spin"></span>
    </div>
    <div class="input">
        <label for="pass">密码</label>
        <input type="password" name="passwd" id="pass">
        <span class="spin"></span>
    </div>
    <div class="button login">
        <button type="submit">
            <span>登录</span>
            <i class="fa fa-check"></i>
        </button>
    </div>
</form>
```

注意修改 input 的 name 属性值和服务端的对应。

配置完成后，重启进行登录测试。

## 3.登录回调

在登录成功之后，我们就要分情况处理了，大体上来说，无非就是分为两种情况：

- 前后端分离登录
- 前后端不分登录

两种情况的处理方式不一样。本文我们先来卡第二种前后端不分的登录，前后端分离的登录回调我在下篇文章中再来和大家细说。

### 3.1 登录成功回调

在 Spring Security 中，和登录成功重定向 URL 相关的方法有两个：

- defaultSuccessUrl
- successForwardUrl

这两个咋看没什么区别，实际上内藏乾坤。

首先我们在配置的时候，defaultSuccessUrl 和 successForwardUrl 只需要配置一个即可，具体配置哪个，则要看你的需求，两个的区别如下：

1. defaultSuccessUrl 有一个重载的方法，我们先说一个参数的 defaultSuccessUrl 方法。如果我们在 defaultSuccessUrl 中指定登录成功的跳转页面为 `/index`，此时分两种情况，如果你是直接在浏览器中输入的登录地址，登录成功后，就直接跳转到 `/index`，如果你是在浏览器中输入了其他地址，例如 `http://localhost:8080/hello`，结果因为没有登录，又重定向到登录页面，此时登录成功后，就不会来到 `/index` ，而是来到 `/hello` 页面。
2. defaultSuccessUrl 还有一个重载的方法，第二个参数如果不设置默认为 false，也就是我们上面的的情况，如果手动设置第二个参数为 true，则 defaultSuccessUrl 的效果和 successForwardUrl 一致。
3. successForwardUrl 表示不管你是从哪里来的，登录后一律跳转到 successForwardUrl 指定的地址。例如 successForwardUrl 指定的地址为 `/index` ，你在浏览器地址栏输入 `http://localhost:8080/hello`，结果因为没有登录，重定向到登录页面，当你登录成功之后，就会服务端跳转到 `/index` 页面；或者你直接就在浏览器输入了登录页面地址，登录成功后也是来到 `/index`。

相关配置如下：

```
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.usernameParameter("name")
.passwordParameter("passwd")
.defaultSuccessUrl("/index")
.successForwardUrl("/index")
.permitAll()
.and()
```

**注意：实际操作中，defaultSuccessUrl 和 successForwardUrl 只需要配置一个即可。**

### 3.2 登录失败回调

与登录成功相似，登录失败也是有两个方法：

- failureForwardUrl
- failureUrl

**这两个方法在设置的时候也是设置一个即可**。failureForwardUrl 是登录失败之后会发生服务端跳转，failureUrl 则在登录失败之后，会发生重定向。

## 4.注销登录

注销登录的默认接口是 `/logout`，我们也可以配置。

```
.and()
.logout()
.logoutUrl("/logout")
.logoutRequestMatcher(new AntPathRequestMatcher("/logout","POST"))
.logoutSuccessUrl("/index")
.deleteCookies()
.clearAuthentication(true)
.invalidateHttpSession(true)
.permitAll()
.and()
```

注销登录的配置我来说一下：

1. 默认注销的 URL 是 `/logout`，是一个 GET 请求，我们可以通过 logoutUrl 方法来修改默认的注销 URL。
2. logoutRequestMatcher 方法不仅可以修改注销 URL，还可以修改请求方式，实际项目中，这个方法和 logoutUrl 任意设置一个即可。
3. logoutSuccessUrl 表示注销成功后要跳转的页面。
4. deleteCookies 用来清除 cookie。
5. clearAuthentication 和 invalidateHttpSession 分别表示清除认证信息和使 HttpSession 失效，默认可以不用配置，默认就会清除。

# 授权操作

## 1.授权

所谓的授权，就是用户如果要访问某一个资源，我们要去检查用户是否具备这样的权限，如果具备就允许访问，如果不具备，则不允许访问。

## 2.准备测试用户

因为我们现在还没有连接数据库，所以测试用户还是基于内存来配置。

基于内存配置测试用户，我们有两种方式，第一种就是我们本系列前面几篇文章用的配置方式，如下：

```
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            .withUser("javaboy")
            .password("123").roles("admin")
            .and()
            .withUser("江南一点雨")
            .password("123")
            .roles("user");
}
```

这是一种配置方式。

由于 Spring Security 支持多种数据源，例如内存、数据库、LDAP 等，这些不同来源的数据被共同封装成了一个 UserDetailService 接口，任何实现了该接口的对象都可以作为认证数据源。

因此我们还可以通过重写 WebSecurityConfigurerAdapter 中的 userDetailsService 方法来提供一个 UserDetailService 实例进而配置多个用户：

```
@Bean
protected UserDetailsService userDetailsService() {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withUsername("javaboy").password("123").roles("admin").build());
    manager.createUser(User.withUsername("江南一点雨").password("123").roles("user").build());
    return manager;
}
```

两种基于内存定义用户的方法，大家任选一个。

## 3.准备测试接口

测试用户准备好了，接下来我们准备三个测试接口。如下：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/admin/hello")
    public String admin() {
        return "admin";
    }

    @GetMapping("/user/hello")
    public String user() {
        return "user";
    }
}
```

这三个测试接口，我们的规划是这样的：

1. /hello 是任何人都可以访问的接口
2. /admin/hello 是具有 admin 身份的人才能访问的接口
3. /user/hello 是具有 user 身份的人才能访问的接口
4. 所有 user 能够访问的资源，admin 都能够访问

**注意第四条规范意味着所有具备 admin 身份的人自动具备 user 身份。**

## 4.配置

接下来我们来配置权限的拦截规则，在 Spring Security 的 configure(HttpSecurity http) 方法中，代码如下：

```
http.authorizeRequests()
        .antMatchers("/admin/**").hasRole("admin")
        .antMatchers("/user/**").hasRole("user")
        .anyRequest().authenticated()
        .and()
        ...
        ...
```

这里的匹配规则我们采用了 Ant 风格的路径匹配符，Ant 风格的路径匹配符在 Spring 家族中使用非常广泛，它的匹配规则也非常简单：

| 通配符 | 含义             |
| :----- | :--------------- |
| **     | 匹配多层路径     |
| *      | 匹配一层路径     |
| ?      | 匹配任意单个字符 |

上面配置的含义是：

1. 如果请求路径满足 `/admin/**` 格式，则用户需要具备 admin 角色。
2. 如果请求路径满足 `/user/**` 格式，则用户需要具备 user 角色。
3. 剩余的其他格式的请求路径，只需要认证（登录）后就可以访问。

注意代码中配置的三条规则的顺序非常重要，和 Shiro 类似，Spring Security 在匹配的时候也是按照从上往下的顺序来匹配，一旦匹配到了就不继续匹配了，**所以拦截规则的顺序不能写错**。

另一方面，如果你强制将 anyRequest 配置在 antMatchers 前面，像下面这样：

```
http.authorizeRequests()
        .anyRequest().authenticated()
        .antMatchers("/admin/**").hasRole("admin")
        .antMatchers("/user/**").hasRole("user")
        .and()
```

此时项目在启动的时候，就会报错，会提示不能在 anyRequest 之后添加 antMatchers：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204103831.png)](http://img.itboyhub.com/2020/04/security-05-01.png)

这从语义上很好理解，anyRequest 已经包含了其他请求了，在它之后如果还配置其他请求也没有任何意义。

从语义上理解，anyRequest 应该放在最后，表示除了前面拦截规则之外，剩下的请求要如何处理。

在拦截规则的配置类 AbstractRequestMatcherRegistry 中，我们可以看到如下一些代码（部分源码）：

```java
public abstract class AbstractRequestMatcherRegistry<C> {
	private boolean anyRequestConfigured = false;
	public C anyRequest() {
		Assert.state(!this.anyRequestConfigured, "Can't configure anyRequest after itself");
		this.anyRequestConfigured = true;
		return configurer;
	}
	public C antMatchers(HttpMethod method, String... antPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure antMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.antMatchers(method, antPatterns));
	}
	public C antMatchers(String... antPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure antMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.antMatchers(antPatterns));
	}
	protected final List<MvcRequestMatcher> createMvcMatchers(HttpMethod method,
			String... mvcPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure mvcMatchers after anyRequest");
		return matchers;
	}
	public C regexMatchers(HttpMethod method, String... regexPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure regexMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.regexMatchers(method, regexPatterns));
	}
	public C regexMatchers(String... regexPatterns) {
		Assert.state(!this.anyRequestConfigured, "Can't configure regexMatchers after anyRequest");
		return chainRequestMatchers(RequestMatchers.regexMatchers(regexPatterns));
	}
	public C requestMatchers(RequestMatcher... requestMatchers) {
		Assert.state(!this.anyRequestConfigured, "Can't configure requestMatchers after anyRequest");
		return chainRequestMatchers(Arrays.asList(requestMatchers));
	}
}
```

从这段源码中，我们可以看到，在任何拦截规则之前（包括 anyRequest 自身），都会先判断 anyRequest 是否已经配置，如果已经配置，则会抛出异常，系统启动失败。

这样大家就理解了为什么 anyRequest 一定要放在最后。

## 5.启动测试

接下来，我们启动项目进行测试。

项目启动成功后，我们首先以 江南一点雨的身份进行登录：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204103837.png)](http://img.itboyhub.com/2020/04/security-5-2.png)

登录成功后，分别访问 `/hello`，`/admin/hello` 以及 `/user/hello` 三个接口，其中：

1. `/hello` 因为登录后就可以访问，这个接口访问成功。
2. `/admin/hello` 需要 admin 身份，所以访问失败。
3. `/user/hello` 需要 user 身份，所以访问成功。

具体测试效果小伙伴们可以参考松哥的视频，我就不截图了。

按照相同的方式，大家也可以测试 javaboy 用户。

## 6.角色继承

在前面松哥提到过一点，所有 user 能够访问的资源，admin 都能够访问，很明显我们目前的代码还不具备这样的功能。

要实现所有 user 能够访问的资源，admin 都能够访问，这涉及到另外一个知识点，叫做角色继承。

这在实际开发中非常有用。

上级可能具备下级的所有权限，如果使用角色继承，这个功能就很好实现，我们只需要在 SecurityConfig 中添加如下代码来配置角色继承关系即可：

```
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_admin > ROLE_user");
    return hierarchy;
}
```

注意，在配置时，需要给角色手动加上 `ROLE_` 前缀。上面的配置表示 `ROLE_admin` 自动具备 `ROLE_user` 的权限。

配置完成后，重启项目，此时我们发现 javaboy 也能访问 `/user/hello` 这个接口了。

# 前后端分离，使用 JSON 格式登录

## 1.服务端接口调整

首先大家知道，用户登录的用户名/密码是在 `UsernamePasswordAuthenticationFilter` 类中处理的，具体的处理代码如下：

```
public Authentication attemptAuthentication(HttpServletRequest request,
		HttpServletResponse response) throws AuthenticationException {
	String username = obtainUsername(request);
	String password = obtainPassword(request);
    //省略
}
protected String obtainPassword(HttpServletRequest request) {
	return request.getParameter(passwordParameter);
}
protected String obtainUsername(HttpServletRequest request) {
	return request.getParameter(usernameParameter);
}
```

从这段代码中，我们就可以看出来为什么 Spring Security 默认是通过 key/value 的形式来传递登录参数，因为它处理的方式就是 request.getParameter。

所以我们要定义成 JSON 的，思路很简单，就是自定义来定义一个过滤器代替 `UsernamePasswordAuthenticationFilter` ，然后在获取参数的时候，换一种方式就行了。

**这里有一个额外的点需要注意，就是我们的微人事现在还有验证码的功能，所以如果自定义过滤器，要连同验证码一起处理掉。**

## 2.自定义过滤器

接下来我们来自定义一个过滤器代替 `UsernamePasswordAuthenticationFilter` ，如下：

```
public class LoginFilter extends UsernamePasswordAuthenticationFilter {
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (!request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
        String verify_code = (String) request.getSession().getAttribute("verify_code");
        if (request.getContentType().equals(MediaType.APPLICATION_JSON_VALUE) || request.getContentType().equals(MediaType.APPLICATION_JSON_UTF8_VALUE)) {
            Map<String, String> loginData = new HashMap<>();
            try {
                loginData = new ObjectMapper().readValue(request.getInputStream(), Map.class);
            } catch (IOException e) {
            }finally {
                String code = loginData.get("code");
                checkCode(response, code, verify_code);
            }
            String username = loginData.get(getUsernameParameter());
            String password = loginData.get(getPasswordParameter());
            if (username == null) {
                username = "";
            }
            if (password == null) {
                password = "";
            }
            username = username.trim();
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
                    username, password);
            setDetails(request, authRequest);
            return this.getAuthenticationManager().authenticate(authRequest);
        } else {
            checkCode(response, request.getParameter("code"), verify_code);
            return super.attemptAuthentication(request, response);
        }
    }

    public void checkCode(HttpServletResponse resp, String code, String verify_code) {
        if (code == null || verify_code == null || "".equals(code) || !verify_code.toLowerCase().equals(code.toLowerCase())) {
            //验证码不正确
            throw new AuthenticationServiceException("验证码不正确");
        }
    }
}
```

这段逻辑我们基本上是模仿官方提供的 `UsernamePasswordAuthenticationFilter` 来写的，我来给大家稍微解释下：

1. 首先登录请求肯定是 POST，如果不是 POST ，直接抛出异常，后面的也不处理了。
2. 因为要在这里处理验证码，所以第二步从 session 中把已经下发过的验证码的值拿出来。
3. 接下来通过 contentType 来判断当前请求是否通过 JSON 来传递参数，如果是通过 JSON 传递参数，则按照 JSON 的方式解析，如果不是，则调用 super.attemptAuthentication 方法，进入父类的处理逻辑中，也就是说，我们自定义的这个类，既支持 JSON 形式传递参数，也支持 key/value 形式传递参数。
4. 如果是 JSON 形式的数据，我们就通过读取 request 中的 I/O 流，将 JSON 映射到一个 Map 上。
5. 从 Map 中取出 code，先去判断验证码是否正确，如果验证码有错，则直接抛出异常。
6. 接下来从 Map 中取出 username 和 password，构造 UsernamePasswordAuthenticationToken 对象并作校验。

过滤器定义完成后，接下来用我们自定义的过滤器代替默认的 `UsernamePasswordAuthenticationFilter`，首先我们需要提供一个 LoginFilter 的实例：

```
@Bean
LoginFilter loginFilter() throws Exception {
    LoginFilter loginFilter = new LoginFilter();
    loginFilter.setAuthenticationSuccessHandler(new AuthenticationSuccessHandler() {
        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            response.setContentType("application/json;charset=utf-8");
            PrintWriter out = response.getWriter();
            Hr hr = (Hr) authentication.getPrincipal();
            hr.setPassword(null);
            RespBean ok = RespBean.ok("登录成功!", hr);
            String s = new ObjectMapper().writeValueAsString(ok);
            out.write(s);
            out.flush();
            out.close();
        }
    });
    loginFilter.setAuthenticationFailureHandler(new AuthenticationFailureHandler() {
        @Override
        public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
            response.setContentType("application/json;charset=utf-8");
            PrintWriter out = response.getWriter();
            RespBean respBean = RespBean.error(exception.getMessage());
            if (exception instanceof LockedException) {
                respBean.setMsg("账户被锁定，请联系管理员!");
            } else if (exception instanceof CredentialsExpiredException) {
                respBean.setMsg("密码过期，请联系管理员!");
            } else if (exception instanceof AccountExpiredException) {
                respBean.setMsg("账户过期，请联系管理员!");
            } else if (exception instanceof DisabledException) {
                respBean.setMsg("账户被禁用，请联系管理员!");
            } else if (exception instanceof BadCredentialsException) {
                respBean.setMsg("用户名或者密码输入错误，请重新输入!");
            }
            out.write(new ObjectMapper().writeValueAsString(respBean));
            out.flush();
            out.close();
        }
    });
    loginFilter.setAuthenticationManager(authenticationManagerBean());
    loginFilter.setFilterProcessesUrl("/doLogin");
    return loginFilter;
}
```

当我们代替了 `UsernamePasswordAuthenticationFilter` 之后，原本在 SecurityConfig#configure 方法中关于 form 表单的配置就会失效，那些失效的属性，都可以在配置 LoginFilter 实例的时候配置。

另外记得配置一个 AuthenticationManager，根据 WebSecurityConfigurerAdapter 中提供的配置即可。

FilterProcessUrl 则可以根据实际情况配置，如果不配置，默认的就是 `/login`。

最后，我们用自定义的 LoginFilter 实例代替 `UsernamePasswordAuthenticationFilter`，如下：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        ...
        //省略
    http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
}
```

调用 addFilterAt 方法完成替换操作。

篇幅原因，我这里只展示了部分代码，完整代码小伙伴们可以在 GitHub 上看到：https://github.com/lenve/vhr。

配置完成后，重启后端，先用 POSTMAN 测试登录接口，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104006.png)](http://img.itboyhub.com/2020/03/spring-security-json-1.png)

## 3.前端修改

原本我们的前端登录代码是这样的：

```
this.$refs.loginForm.validate((valid) => {
    if (valid) {
        this.loading = true;
        this.postKeyValueRequest('/doLogin', this.loginForm).then(resp => {
            this.loading = false;
            //省略
        })
    } else {
        return false;
    }
});
```

首先我们去校验数据，在校验成功之后，通过 postKeyValueRequest 方法来发送登录请求，这个方法是我自己封装的通过 key/value 形式传递参数的 POST 请求，如下：

```
export const postKeyValueRequest = (url, params) => {
    return axios({
        method: 'post',
        url: `${base}${url}`,
        data: params,
        transformRequest: [function (data) {
            let ret = '';
            for (let i in data) {
                ret += encodeURIComponent(i) + '=' + encodeURIComponent(data[i]) + '&'
            }
            return ret;
        }],
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        }
    });
}
export const postRequest = (url, params) => {
    return axios({
        method: 'post',
        url: `${base}${url}`,
        data: params
    })
}
```

postKeyValueRequest 是我封装的通过 key/value 形式传递参数，postRequest 则是通过 JSON 形式传递参数。

所以，前端我们只需要对登录请求稍作调整，如下：

```
this.$refs.loginForm.validate((valid) => {
    if (valid) {
        this.loading = true;
        this.postRequest('/doLogin', this.loginForm).then(resp => {
            this.loading = false;
            //省略
        })
    } else {
        return false;
    }
});
```

配置完成后，再去登录，浏览器按 F12 ，就可以看到登录请求的参数形式了：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104010.png)](http://img.itboyhub.com/2020/03/spring-security-json-2.png)

# Spring Security 如何将用户数据存入数据库？

## 1.UserDetailService

Spring Security 支持多种不同的数据源，这些不同的数据源最终都将被封装成 UserDetailsService 的实例，在微人事（https://github.com/lenve/vhr）项目中，我们是自己来创建一个类实现 UserDetailsService 接口，除了自己封装，我们也可以使用系统默认提供的 UserDetailsService 实例，例如上篇文章和大家介绍的 InMemoryUserDetailsManager 。

我们来看下 UserDetailsService 都有哪些实现类：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104217.png)](http://img.itboyhub.com/2020/04/springsecurity-6-3.png)

可以看到，在几个能直接使用的实现类中，除了 InMemoryUserDetailsManager 之外，还有一个 JdbcUserDetailsManager，使用 JdbcUserDetailsManager 可以让我们通过 JDBC 的方式将数据库和 Spring Security 连接起来。

## 2.JdbcUserDetailsManager

JdbcUserDetailsManager 自己提供了一个数据库模型，这个数据库模型保存在如下位置：

```
org/springframework/security/core/userdetails/jdbc/users.ddl
```

这里存储的脚本内容如下：

```
create table users(username varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not null,enabled boolean not null);
create table authorities (username varchar_ignorecase(50) not null,authority varchar_ignorecase(50) not null,constraint fk_authorities_users foreign key(username) references users(username));
create unique index ix_auth_username on authorities (username,authority);
```

可以看到，脚本中有一种数据类型 varchar_ignorecase，这个其实是针对 HSQLDB 数据库创建的，而我们使用的 MySQL 并不支持这种数据类型，所以这里需要大家手动调整一下数据类型，将 varchar_ignorecase 改为 varchar 即可。

修改完成后，创建数据库，执行完成后的脚本。

执行完 SQL 脚本后，我们可以看到一共创建了两张表：users 和 authorities。

- users 表中保存用户的基本信息，包括用户名、用户密码以及账户是否可用。
- authorities 中保存了用户的角色。
- authorities 和 users 通过 username 关联起来。

配置完成后，接下来，我们将上篇文章中通过 InMemoryUserDetailsManager 提供的用户数据用 JdbcUserDetailsManager 代替掉，如下：

```
@Autowired
DataSource dataSource;
@Override
@Bean
protected UserDetailsService userDetailsService() {
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager();
    manager.setDataSource(dataSource);
    if (!manager.userExists("javaboy")) {
        manager.createUser(User.withUsername("javaboy").password("123").roles("admin").build());
    }
    if (!manager.userExists("江南一点雨")) {
        manager.createUser(User.withUsername("江南一点雨").password("123").roles("user").build());
    }
    return manager;
}
```

这段配置的含义如下：

1. 首先构建一个 JdbcUserDetailsManager 实例。
2. 给 JdbcUserDetailsManager 实例添加一个 DataSource 对象。
3. 调用 userExists 方法判断用户是否存在，如果不存在，就创建一个新的用户出来（因为每次项目启动时这段代码都会执行，所以加一个判断，避免重复创建用户）。
4. 用户的创建方法和我们之前 InMemoryUserDetailsManager 中的创建方法基本一致。

这里的 createUser 或者 userExists 方法其实都是调用写好的 SQL 去判断的，我们从它的源码里就能看出来（部分）：

```
public class JdbcUserDetailsManager extends JdbcDaoImpl implements UserDetailsManager,
		GroupManager {
	public static final String DEF_USER_EXISTS_SQL = "select username from users where username = ?";

	private String userExistsSql = DEF_USER_EXISTS_SQL;

	public boolean userExists(String username) {
		List<String> users = getJdbcTemplate().queryForList(userExistsSql,
				new String[] { username }, String.class);

		if (users.size() > 1) {
			throw new IncorrectResultSizeDataAccessException(
					"More than one user found with name '" + username + "'", 1);
		}

		return users.size() == 1;
	}
}
```

从这段源码中就可以看出来，userExists 方法的执行逻辑其实就是调用 JdbcTemplate 来执行预定义好的 SQL 脚本，进而判断出用户是否存在，其他的判断方法都是类似，我就不再赘述。

## 3.数据库支持

通过前面的代码，大家看到这里需要数据库支持，所以我们在项目中添加如下两个依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

然后再在 application.properties 中配置一下数据库连接：

```
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.url=jdbc:mysql:///security?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
```

配置完成后，就可以启动项目。

项目启动成功后，我们就可以看到数据库中自动添加了两个用户进来，并且用户都配置了角色。如下图：

[![img](http://img.itboyhub.com/2020/04/security-6-4.png)](http://img.itboyhub.com/2020/04/security-6-4.png)
[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104233.png)](http://img.itboyhub.com/2020/04/security-6-5.png)

## 4.测试

接下来我们就可以进行测试了。

我们首先以 江南一点雨的身份进行登录：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104244.png)](http://img.itboyhub.com/2020/04/security-5-2.png)

登录成功后，分别访问 `/hello`，`/admin/hello` 以及 `/user/hello` 三个接口，其中：

1. `/hello` 因为登录后就可以访问，这个接口访问成功。
2. `/admin/hello` 需要 admin 身份，所以访问失败。
3. `/user/hello` 需要 user 身份，所以访问成功。

具体测试效果小伙伴们可以参考松哥的视频，我就不截图了。

在测试的过程中，如果在数据库中将用户的 enabled 属性设置为 false，表示禁用该账户，此时再使用该账户登录就会登录失败。

# Spring Security使用Spring Data Jpa 

## 1.创建工程

首先我们创建一个新的 Spring Boot 工程，添加如下依赖：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104404.png)](http://img.itboyhub.com/2020/04/spring-security-7-1.png)

注意，除了 Spring Security 依赖之外，我们还需要数据依赖和 Spring Data Jpa 依赖。

工程创建完成后，我们再在数据库中创建一个空的库，就叫做 withjpa，里边什么都不用做，这样我们的准备工作就算完成了。

## 2.准备模型

接下来我们创建两个实体类，分别表示用户角色了用户类：

用户角色：

```
@Entity(name = "t_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String nameZh;
    //省略 getter/setter
}
```

这个实体类用来描述用户角色信息，有角色 id、角色名称（英文、中文），@Entity 表示这是一个实体类，项目启动后，将会根据实体类的属性在数据库中自动创建一个角色表。

用户实体类：

```
@Entity(name = "t_user")
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    private boolean enabled;
    @ManyToMany(fetch = FetchType.EAGER,cascade = CascadeType.PERSIST)
    private List<Role> roles;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        for (Role role : getRoles()) {
            authorities.add(new SimpleGrantedAuthority(role.getName()));
        }
        return authorities;
    }
    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
    //省略其他 get/set 方法
}
```

用户实体类主要需要实现 UserDetails 接口，并实现接口中的方法。

这里的字段基本都好理解，几个特殊的我来稍微说一下：

1. accountNonExpired、accountNonLocked、credentialsNonExpired、enabled 这四个属性分别用来描述用户的状态，表示账户是否没有过期、账户是否没有被锁定、密码是否没有过期、以及账户是否可用。
2. roles 属性表示用户的角色，User 和 Role 是多对多关系，用一个 @ManyToMany 注解来描述。
3. getAuthorities 方法返回用户的角色信息，我们在这个方法中把自己的 Role 稍微转化一下即可。

## 3.配置

数据模型准备好之后，我们再来定义一个 UserDao：

```
public interface UserDao extends JpaRepository<User,Long> {
    User findUserByUsername(String username);
}
```

这里的东西很简单，我们只需要继承 JpaRepository 然后提供一个根据 username 查询 user 的方法即可。如果小伙伴们不熟悉 Spring Data Jpa 的操作，可以在公众号后台回复 springboot 获取松哥手敲的 Spring Boot 教程，里边有 jpa 相关操作，也可以看看松哥录制的视频教程：[Spring Boot + Vue 系列视频教程](https://mp.weixin.qq.com/s/1k4CZ6_re11fQM_6_00jCw)。

在接下来定义 UserService ，如下：

```
@Service
public class UserService implements UserDetailsService {
    @Autowired
    UserDao userDao;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userDao.findUserByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        return user;
    }
}
```

我们自己定义的 UserService 需要实现 UserDetailsService 接口，实现该接口，就要实现接口中的方法，也就是 loadUserByUsername ，这个方法的参数就是用户在登录的时候传入的用户名，根据用户名去查询用户信息（查出来之后，系统会自动进行密码比对）。

配置完成后，接下来我们在 Spring Security 中稍作配置，Spring Security 和测试用的 HelloController 我还是沿用之前文章中的（[Spring Security 如何将用户数据存入数据库？](https://mp.weixin.qq.com/s/EurEXmU0M9AKuUs4Jh_V5g)），主要列出来需要修改的地方。

在 SecurityConfig 中，我们通过如下方式来配置用户：

```
@Autowired
UserService userService;
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userService);
}
```

大家注意，还是重写 configure 方法，只不过这次我们不是基于内存，也不是基于 JdbcUserDetailsManager，而是使用自定义的 UserService，就这样配置就 OK 了。

最后，我们再在 application.properties 中配置一下数据库和 JPA 的基本信息，如下：

```
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.url=jdbc:mysql:///withjpa?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai

spring.jpa.database=mysql
spring.jpa.database-platform=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

都是常规配置，我们就不再重复解释了。

这一套组合拳下来，我们的 Spring Security 就算是接入数据库了，接下来我们来进行测试，测试的 HelloController 参考[上篇文章](https://mp.weixin.qq.com/s/EurEXmU0M9AKuUs4Jh_V5g)，我就不重复写了。

## 4.测试

首先我们来添加两条测试数据，在单元测试中添加如下方法：

```
@Autowired
UserDao userDao;
@Test
void contextLoads() {
    User u1 = new User();
    u1.setUsername("javaboy");
    u1.setPassword("123");
    u1.setAccountNonExpired(true);
    u1.setAccountNonLocked(true);
    u1.setCredentialsNonExpired(true);
    u1.setEnabled(true);
    List<Role> rs1 = new ArrayList<>();
    Role r1 = new Role();
    r1.setName("ROLE_admin");
    r1.setNameZh("管理员");
    rs1.add(r1);
    u1.setRoles(rs1);
    userDao.save(u1);
    User u2 = new User();
    u2.setUsername("江南一点雨");
    u2.setPassword("123");
    u2.setAccountNonExpired(true);
    u2.setAccountNonLocked(true);
    u2.setCredentialsNonExpired(true);
    u2.setEnabled(true);
    List<Role> rs2 = new ArrayList<>();
    Role r2 = new Role();
    r2.setName("ROLE_user");
    r2.setNameZh("普通用户");
    rs2.add(r2);
    u2.setRoles(rs2);
    userDao.save(u2);
}
```

运行该方法后，我们会发现数据库中多了三张表：这就是根据我们的实体类自动创建出来的。

我们来查看一下表中的数据。

用户表：![](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104903.png)

角色表：![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104928.png)

用户和角色关联表：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204104333.png)](http://img.itboyhub.com/2020/04/spring-security-7-8.png)

有了数据，接下来启动项目，我们来进行测试。

我们首先以 江南一点雨的身份进行登录：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105016.png)登录成功后，分别访问 `/hello`，`/admin/hello` 以及 `/user/hello` 三个接口，其中：

1. `/hello` 因为登录后就可以访问，这个接口访问成功。
2. `/admin/hello` 需要 admin 身份，所以访问失败。
3. `/user/hello` 需要 user 身份，所以访问成功。

具体测试效果小伙伴们可以参考松哥的视频，我就不截图了。

在测试的过程中，如果在数据库中将用户的 enabled 属性设置为 false，表示禁用该账户，此时再使用该账户登录就会登录失败。

按照相同的方式，大家也可以测试 javaboy 用户。

# Spring Boot + Spring Security 实现自动登录功能

自动登录是我们在软件开发时一个非常常见的功能，例如我们登录 QQ 邮箱：



[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105153.png)](http://img.itboyhub.com/2020/04/security-8-1.png)

很多网站我们在登录的时候都会看到类似的选项，毕竟总让用户输入用户名密码是一件很麻烦的事。

自动登录功能就是，用户在登录成功后，在某一段时间内，如果用户关闭了浏览器并重新打开，或者服务器重启了，都不需要用户重新登录了，用户依然可以直接访问接口数据。

## 1.实战代码

首先，要实现记住我这个功能，其实只需要其实只需要在 Spring Security 的配置中，添加如下代码即可：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .and()
            .csrf().disable();
}
```

大家看到，这里只需要添加一个 `.rememberMe()` 即可，自动登录功能就成功添加进来了。

接下来我们随意添加一个测试接口：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

重启项目，我们访问 hello 接口，此时会自动跳转到登录页面：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105214.png)](http://img.itboyhub.com/2020/04/security-8-2.png)

这个时候大家发现，默认的登录页面多了一个选项，就是记住我。我们输入用户名密码，并且勾选上记住我这个框，然后点击登录按钮执行登录操作：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105220.png)](http://img.itboyhub.com/2020/04/20200426174808.png)

可以看到，登录数据中，除了 username 和 password 之外，还有一个 remember-me，之所以给大家看这个，是想告诉大家，如果你你需要自定义登录页面，RememberMe 这个选项的 key 该怎么写。

登录成功之后，就会自动跳转到 hello 接口了。我们注意，系统访问 hello 接口的时候，携带的 cookie：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105223.png)](http://img.itboyhub.com/2020/04/20200426175634.png)

大家注意到，这里多了一个 remember-me，这就是这里实现的核心，关于这个 remember-me 我一会解释，我们先来测试效果。

接下来，我们关闭浏览器，再重新打开浏览器。正常情况下，浏览器关闭再重新打开，如果需要再次访问 hello 接口，就需要我们重新登录了。但是此时，我们再去访问 hello 接口，发现不用重新登录了，直接就能访问到，这就说明我们的 RememberMe 配置生效了（即下次自动登录功能生效了）。

## 2.原理分析

按理说，浏览器关闭再重新打开，就要重新登录，现在竟然不用等了，那么这个功能到底是怎么实现的呢？

首先我们来分析一下 cookie 中多出来的这个 remember-me，这个值一看就是一个 Base64 转码后的字符串，我们可以使用网上的一些在线工具来解码，可以自己简单写两行代码来解码：

```
@Test
void contextLoads() throws UnsupportedEncodingException {
    String s = new String(Base64.getDecoder().decode("amF2YWJveToxNTg5MTA0MDU1MzczOjI1NzhmZmJjMjY0ODVjNTM0YTJlZjkyOWFjMmVmYzQ3"), "UTF-8");
    System.out.println("s = " + s);
}
```

执行这段代码，输出结果如下：

```
s = javaboy:1589104055373:2578ffbc26485c534a2ef929ac2efc47
```

可以看到，这段 Base64 字符串实际上用 `:` 隔开，分成了三部分：

1. 第一段是用户名，这个无需质疑。
2. 第二段看起来是一个时间戳，我们通过在线工具或者 Java 代码解析后发现，这是一个两周后的数据。
3. 第三段我就不卖关子了，这是使用 MD5 散列函数算出来的值，他的明文格式是 `username + ":" + tokenExpiryTime + ":" + password + ":" + key`，最后的 key 是一个散列盐值，可以用来防治令牌被修改。

了解到 cookie 中 remember-me 的含义之后，那么我们对于记住我的登录流程也就很容易猜到了了。

在浏览器关闭后，并重新打开之后，用户再去访问 hello 接口，此时会携带着 cookie 中的 remember-me 到服务端，服务到拿到值之后，可以方便的计算出用户名和过期时间，再根据用户名查询到用户密码，然后通过 MD5 散列函数计算出散列值，再将计算出的散列值和浏览器传递来的散列值进行对比，就能确认这个令牌是否有效。

流程就是这么个流程，接下来我们通过分析源码来验证一下这个流程对不对。

## 3.源码分析

接下来，我们通过源码来验证一下我们上面说的对不对。

这里主要从两个方面来介绍，一个是 remember-me 这个令牌生成的过程，另一个则是它解析的过程。

### 3.1 生成

生成的核心处理方法在：`TokenBasedRememberMeServices#onLoginSuccess`：

```
@Override
public void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,
		Authentication successfulAuthentication) {
	String username = retrieveUserName(successfulAuthentication);
	String password = retrievePassword(successfulAuthentication);
	if (!StringUtils.hasLength(password)) {
		UserDetails user = getUserDetailsService().loadUserByUsername(username);
		password = user.getPassword();
	}
	int tokenLifetime = calculateLoginLifetime(request, successfulAuthentication);
	long expiryTime = System.currentTimeMillis();
	expiryTime += 1000L * (tokenLifetime < 0 ? TWO_WEEKS_S : tokenLifetime);
	String signatureValue = makeTokenSignature(expiryTime, username, password);
	setCookie(new String[] { username, Long.toString(expiryTime), signatureValue },
			tokenLifetime, request, response);
}
protected String makeTokenSignature(long tokenExpiryTime, String username,
		String password) {
	String data = username + ":" + tokenExpiryTime + ":" + password + ":" + getKey();
	MessageDigest digest;
	digest = MessageDigest.getInstance("MD5");
	return new String(Hex.encode(digest.digest(data.getBytes())));
}
```

这段方法的逻辑其实很好理解：

1. 首先从登录成功的 Authentication 中提取出用户名/密码。
2. 由于登录成功之后，密码可能被擦除了，所以，如果一开始没有拿到密码，就再从 UserDetailsService 中重新加载用户并重新获取密码。
3. 再接下来去获取令牌的有效期，令牌有效期默认就是两周。
4. 再接下来调用 makeTokenSignature 方法去计算散列值，实际上就是根据 username、令牌有效期以及 password、key 一起计算一个散列值。如果我们没有自己去设置这个 key，默认是在 RememberMeConfigurer#getKey 方法中进行设置的，它的值是一个 UUID 字符串。
5. 最后，将用户名、令牌有效期以及计算得到的散列值放入 Cookie 中。

关于第四点，我这里再说一下。

由于我们自己没有设置 key，key 默认值是一个 UUID 字符串，这样会带来一个问题，就是如果服务端重启，这个 key 会变，这样就导致之前派发出去的所有 remember-me 自动登录令牌失效，所以，我们可以指定这个 key。指定方式如下：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .key("javaboy")
            .and()
            .csrf().disable();
}
```

如果自己配置了 key，**即使服务端重启，即使浏览器打开再关闭**，也依然能够访问到 hello 接口。

这是 remember-me 令牌生成的过程。至于是如何走到 onLoginSuccess 方法的，大家可以参考松哥之前的文章：[松哥手把手带你捋一遍 Spring Security 登录流程](https://mp.weixin.qq.com/s/z6GeR5O-vBzY3SHehmccVA)。这里可以给大家稍微提醒一下思路：

AbstractAuthenticationProcessingFilter#doFilter -> AbstractAuthenticationProcessingFilter#successfulAuthentication -> AbstractRememberMeServices#loginSuccess -> TokenBasedRememberMeServices#onLoginSuccess。

### 3.2 解析

那么当用户关掉并打开浏览器之后，重新访问 /hello 接口，此时的认证流程又是怎么样的呢？

我们之前说过，Spring Security 中的一系列功能都是通过一个过滤器链实现的，RememberMe 这个功能当然也不例外。

Spring Security 中提供了 RememberMeAuthenticationFilter 类专门用来做相关的事情，我们来看下 RememberMeAuthenticationFilter 的 doFilter 方法：

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
		throws IOException, ServletException {
	HttpServletRequest request = (HttpServletRequest) req;
	HttpServletResponse response = (HttpServletResponse) res;
	if (SecurityContextHolder.getContext().getAuthentication() == null) {
		Authentication rememberMeAuth = rememberMeServices.autoLogin(request,
				response);
		if (rememberMeAuth != null) {
				rememberMeAuth = authenticationManager.authenticate(rememberMeAuth);
				SecurityContextHolder.getContext().setAuthentication(rememberMeAuth);
				onSuccessfulAuthentication(request, response, rememberMeAuth);
				if (this.eventPublisher != null) {
					eventPublisher
							.publishEvent(new InteractiveAuthenticationSuccessEvent(
									SecurityContextHolder.getContext()
											.getAuthentication(), this.getClass()));
				}
				if (successHandler != null) {
					successHandler.onAuthenticationSuccess(request, response,
							rememberMeAuth);
					return;
				}
			}
		chain.doFilter(request, response);
	}
	else {
		chain.doFilter(request, response);
	}
}
```

可以看到，就是在这里实现的。

这个方法最关键的地方在于，如果从 SecurityContextHolder 中无法获取到当前登录用户实例，那么就调用 rememberMeServices.autoLogin 逻辑进行登录，我们来看下这个方法：

```
public final Authentication autoLogin(HttpServletRequest request,
		HttpServletResponse response) {
	String rememberMeCookie = extractRememberMeCookie(request);
	if (rememberMeCookie == null) {
		return null;
	}
	logger.debug("Remember-me cookie detected");
	if (rememberMeCookie.length() == 0) {
		logger.debug("Cookie was empty");
		cancelCookie(request, response);
		return null;
	}
	UserDetails user = null;
	try {
		String[] cookieTokens = decodeCookie(rememberMeCookie);
		user = processAutoLoginCookie(cookieTokens, request, response);
		userDetailsChecker.check(user);
		logger.debug("Remember-me cookie accepted");
		return createSuccessfulAuthentication(request, user);
	}
	catch (CookieTheftException cte) {
		
		throw cte;
	}
	cancelCookie(request, response);
	return null;
}
```

可以看到，这里就是提取出 cookie 信息，并对 cookie 信息进行解码，解码之后，再调用 processAutoLoginCookie 方法去做校验，processAutoLoginCookie 方法的代码我就不贴了，核心流程就是首先获取用户名和过期时间，再根据用户名查询到用户密码，然后通过 MD5 散列函数计算出散列值，再将拿到的散列值和浏览器传递来的散列值进行对比，就能确认这个令牌是否有效，进而确认登录是否有效。

好了，这里的流程我也根据大家大致上梳理了一下。

## 4.总结

看了上面的文章，大家可能已经发现，如果我们开启了 RememberMe 功能，最最核心的东西就是放在 cookie 中的令牌了，这个令牌突破了 session 的限制，即使服务器重启、即使浏览器关闭又重新打开，只要这个令牌没有过期，就能访问到数据。

一旦令牌丢失，别人就可以拿着这个令牌随意登录我们的系统了，这是一个非常危险的操作。

但是实际上这是一段悖论，为了提高用户体验（少登录），我们的系统不可避免的引出了一些安全问题，不过我们可以通过技术将安全风险降低到最小。

# Spring Boot 自动登录，安全风险要怎么控制？松哥教你两招

## 1.持久化令牌

### 1.1 原理

持久化令牌就是在基本的自动登录功能基础上，又增加了新的校验参数，来提高系统的安全性，这一些都是由开发者在后台完成的，对于用户来说，登录体验和普通的自动登录体验是一样的。

在持久化令牌中，新增了两个经过 MD5 散列函数计算的校验参数，一个是 series，另一个是 token。其中，series 只有当用户在使用用户名/密码登录时，才会生成或者更新，而 token 只要有新的会话，就会重新生成，这样就可以避免一个用户同时在多端登录，就像手机 QQ ，一个手机上登录了，就会踢掉另外一个手机的登录，这样用户就会很容易发现账户是否泄漏（之前看到松哥交流群里有小伙伴在讨论如何禁止多端登录，其实就可以借鉴这里的思路）。

持久化令牌的具体处理类在 PersistentTokenBasedRememberMeServices 中，[上篇文章](https://mp.weixin.qq.com/s/aSsGNBSWMTsAEXjn9wQnYQ)我们讲到的自动化登录具体的处理类是在 TokenBasedRememberMeServices 中，它们有一个共同的父类：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105336.png)](http://img.itboyhub.com/2020/04/RememberMeServices.png)

而用来保存令牌的处理类则是 PersistentRememberMeToken，该类的定义也很简洁命令：

```
public class PersistentRememberMeToken {
	private final String username;
	private final String series;
	private final String tokenValue;
	private final Date date;
    //省略 getter
}
```

这里的 Date 表示上一次使用自动登录的时间。

### 1.2 代码演示

接下来，我通过代码来给大家演示一下持久化令牌的具体用法。

首先我们需要一张表来记录令牌信息，这张表我们可以完全自定义，也可以使用系统默认提供的 JDBC 来操作，如果使用默认的 JDBC，即 JdbcTokenRepositoryImpl，我们可以来分析一下该类的定义：

```
public class JdbcTokenRepositoryImpl extends JdbcDaoSupport implements
		PersistentTokenRepository {
	public static final String CREATE_TABLE_SQL = "create table persistent_logins (username varchar(64) not null, series varchar(64) primary key, "
			+ "token varchar(64) not null, last_used timestamp not null)";
	public static final String DEF_TOKEN_BY_SERIES_SQL = "select username,series,token,last_used from persistent_logins where series = ?";
	public static final String DEF_INSERT_TOKEN_SQL = "insert into persistent_logins (username, series, token, last_used) values(?,?,?,?)";
	public static final String DEF_UPDATE_TOKEN_SQL = "update persistent_logins set token = ?, last_used = ? where series = ?";
	public static final String DEF_REMOVE_USER_TOKENS_SQL = "delete from persistent_logins where username = ?";
}
```

根据这段 SQL 定义，我们就可以分析出来表的结构，松哥这里给出一段 SQL 脚本：

```
CREATE TABLE `persistent_logins` (
  `username` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `series` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `token` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `last_used` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

首先我们在数据库中准备好这张表。

既然要连接数据库，我们还需要准备 jdbc 和 mysql 依赖，如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

然后修改 application.properties ，配置数据库连接信息：

```
spring.datasource.url=jdbc:mysql:///oauth2?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123
```

接下来，我们修改 SecurityConfig，如下：

```
@Autowired
DataSource dataSource;
@Bean
JdbcTokenRepositoryImpl jdbcTokenRepository() {
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    return jdbcTokenRepository;
}
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .key("javaboy")
            .tokenRepository(jdbcTokenRepository())
            .and()
            .csrf().disable();
}
```

提供一个 JdbcTokenRepositoryImpl 实例，并给其配置 DataSource 数据源，最后通过 tokenRepository 将 JdbcTokenRepositoryImpl 实例纳入配置中。

OK，做完这一切，我们就可以测试了。

### 1.3 测试

我们还是先去访问 `/hello` 接口，此时会自动跳转到登录页面，然后我们执行登录操作，记得勾选上“记住我”这个选项，登录成功后，我们可以重启服务器、然后关闭浏览器再打开，再去访问 /hello 接口，发现依然能够访问到，说明我们的持久化令牌配置已经生效。

查看 remember-me 的令牌，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105345.png)](http://img.itboyhub.com/2020/04/20200427102713.png)

这个令牌经过解析之后，格式如下：

```
emhqATk3ZDBdR8862WP4Ig%3D%3D:ZAEv6EIWqA7CkGbYewCh8g%3D%3D
```

这其中，%3D 表示 `=`，所以上面的字符实际上可以翻译成下面这样：

```
emhqATk3ZDBdR8862WP4Ig==:ZAEv6EIWqA7CkGbYewCh8g==
```

此时，查看数据库，我们发现之前的表中生成了一条记录：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105353.png)](http://img.itboyhub.com/2020/04/20200427102351.png)

数据库中的记录和我们看到的 remember-me 令牌解析后是一致的。

### 1.4 源码分析

这里的源码分析和[上篇文章](https://mp.weixin.qq.com/s/aSsGNBSWMTsAEXjn9wQnYQ)的流程基本一致，只不过实现类变了，也就是生成令牌/解析令牌的实现变了，所以这里我主要和大家展示不一样的地方，流程问题，大家可以参考[上篇文章](https://mp.weixin.qq.com/s/aSsGNBSWMTsAEXjn9wQnYQ)。

这次的实现类主要是：PersistentTokenBasedRememberMeServices，我们先来看里边几个和令牌生成相关的方法：

```
protected void onLoginSuccess(HttpServletRequest request,
		HttpServletResponse response, Authentication successfulAuthentication) {
	String username = successfulAuthentication.getName();
	PersistentRememberMeToken persistentToken = new PersistentRememberMeToken(
			username, generateSeriesData(), generateTokenData(), new Date());
	tokenRepository.createNewToken(persistentToken);
	addCookie(persistentToken, request, response);
}
protected String generateSeriesData() {
	byte[] newSeries = new byte[seriesLength];
	random.nextBytes(newSeries);
	return new String(Base64.getEncoder().encode(newSeries));
}
protected String generateTokenData() {
	byte[] newToken = new byte[tokenLength];
	random.nextBytes(newToken);
	return new String(Base64.getEncoder().encode(newToken));
}
private void addCookie(PersistentRememberMeToken token, HttpServletRequest request,
		HttpServletResponse response) {
	setCookie(new String[] { token.getSeries(), token.getTokenValue() },
			getTokenValiditySeconds(), request, response);
}
```

可以看到：

1. 在登录成功后，首先还是获取到用户名，即 username。
2. 接下来构造一个 PersistentRememberMeToken 实例，generateSeriesData 和 generateTokenData 方法分别用来获取 series 和 token，具体的生成过程实际上就是调用 SecureRandom 生成随机数再进行 Base64 编码，不同于我们以前用的 Math.random 或者 java.util.Random 这种伪随机数，SecureRandom 则采用的是类似于密码学的随机数生成规则，其输出结果较难预测，适合在登录这样的场景下使用。
3. 调用 tokenRepository 实例中的 createNewToken 方法，tokenRepository 实际上就是我们一开始配置的 JdbcTokenRepositoryImpl，所以这行代码实际上就是将 PersistentRememberMeToken 存入数据库中。
4. 最后 addCookie，大家可以看到，就是添加了 series 和 token。

这是令牌生成的过程，还有令牌校验的过程，也在该类中，方法是：processAutoLoginCookie：

```
protected UserDetails processAutoLoginCookie(String[] cookieTokens,
		HttpServletRequest request, HttpServletResponse response) {
	final String presentedSeries = cookieTokens[0];
	final String presentedToken = cookieTokens[1];
	PersistentRememberMeToken token = tokenRepository
			.getTokenForSeries(presentedSeries);
	if (!presentedToken.equals(token.getTokenValue())) {
		tokenRepository.removeUserTokens(token.getUsername());
		throw new CookieTheftException(
				messages.getMessage(
						"PersistentTokenBasedRememberMeServices.cookieStolen",
						"Invalid remember-me token (Series/token) mismatch. Implies previous cookie theft attack."));
	}
	if (token.getDate().getTime() + getTokenValiditySeconds() * 1000L < System
			.currentTimeMillis()) {
		throw new RememberMeAuthenticationException("Remember-me login has expired");
	}
	PersistentRememberMeToken newToken = new PersistentRememberMeToken(
			token.getUsername(), token.getSeries(), generateTokenData(), new Date());
	tokenRepository.updateToken(newToken.getSeries(), newToken.getTokenValue(),
				newToken.getDate());
	addCookie(newToken, request, response);
	return getUserDetailsService().loadUserByUsername(token.getUsername());
}
```

这段逻辑也比较简单：

1. 首先从前端传来的 cookie 中解析出 series 和 token。
2. 根据 series 从数据库中查询出一个 PersistentRememberMeToken 实例。
3. 如果查出来的 token 和前端传来的 token 不相同，说明账号可能被人盗用（别人用你的令牌登录之后，token 会变）。此时根据用户名移除相关的 token，相当于必须要重新输入用户名密码登录才能获取新的自动登录权限。
4. 接下来校验 token 是否过期。
5. 构造新的 PersistentRememberMeToken 对象，并且更新数据库中的 token（这就是我们文章开头说的，新的会话都会对应一个新的 token）。
6. 将新的令牌重新添加到 cookie 中返回。
7. 根据用户名查询用户信息，再走一波登录流程。

OK，这里和小伙伴们简单理了一下令牌生成和校验的过程，具体的流程，大家可以参考[上篇文章](https://mp.weixin.qq.com/s/aSsGNBSWMTsAEXjn9wQnYQ)。

## 2.二次校验

相比于[上篇文章](https://mp.weixin.qq.com/s/aSsGNBSWMTsAEXjn9wQnYQ)，持久化令牌的方式其实已经安全很多了，但是依然存在用户身份被盗用的问题，这个问题实际上很难完美解决，我们能做的，只能是当发生用户身份被盗用这样的事情时，将损失降低到最小。

因此，我们来看下另一种方案，就是二次校验。

二次校验这块，实现起来要稍微复杂一点，我先来和大家说说思路。

为了让用户使用方便，我们开通了自动登录功能，但是自动登录功能又带来了安全风险，一个规避的办法就是如果用户使用了自动登录功能，我们可以只让他做一些常规的不敏感操作，例如数据浏览、查看，但是不允许他做任何修改、删除操作，如果用户点击了修改、删除按钮，我们可以跳转回登录页面，让用户重新输入密码确认身份，然后再允许他执行敏感操作。

这个功能在 Shiro 中有一个比较方便的过滤器可以配置，Spring Security 当然也一样，例如我现在提供三个访问接口：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
    @GetMapping("/admin")
    public String admin() {
        return "admin";
    }
    @GetMapping("/rememberme")
    public String rememberme() {
        return "rememberme";
    }
}
```

1. 第一个 /hello 接口，只要认证后就可以访问，无论是通过用户名密码认证还是通过自动登录认证，只要认证了，就可以访问。
2. 第二个 /admin 接口，必须要用户名密码认证之后才能访问，如果用户是通过自动登录认证的，则必须重新输入用户名密码才能访问该接口。
3. 第三个 /rememberme 接口，必须是通过自动登录认证后才能访问，如果用户是通过用户名/密码认证的，则无法访问该接口。

好了，我们来看下接口的访问要怎么配置：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .antMatchers("/rememberme").rememberMe()
            .antMatchers("/admin").fullyAuthenticated()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .key("javaboy")
            .tokenRepository(jdbcTokenRepository())
            .and()
            .csrf().disable();
}
```

可以看到：

1. /rememberme 接口是需要 rememberMe 才能访问。
2. /admin 是需要 fullyAuthenticated，fullyAuthenticated 不同于 authenticated，fullyAuthenticated 不包含自动登录的形式，而 authenticated 包含自动登录的形式。
3. 最后剩余的接口（/hello）都是 authenticated 就能访问。

OK，配置完成后，重启测试，测试过程我就不再赘述了。

# 在微服务项目中，Spring Security 比 Shiro 强在哪？

虽然目前 Spring Security 一片火热，但是 Shiro 的市场依然存在，今天我就来稍微的说一说这两个框架的，方便大家在实际项目中选择适合自己的安全管理框架。

首先我要声明一点，框架无所谓好坏，关键是适合当前项目场景，作为一个年轻的程序员更不应该厚此薄彼，或者拒绝学习某一个框架。

小孩子才做选择题，成年人两个都要学！

所以接下来主要结合我自己的经验来说一说这两个框架的优缺点，没有提到的地方也欢迎大家留言补充。

## 1. Spring Security

### 1.1 因为 SpringBoot 而火

Spring Security 并非一个新生的事物，它最早不叫 Spring Security ，叫 Acegi Security，叫 Acegi Security 并不是说它和 Spring 就没有关系了，它依然是为 Spring 框架提供安全支持的。事实上，Java 领域的框架，很少有框架能够脱离 Spring 框架独立存在。

当 Spring Security 还叫 Acegi Security 的时候，虽然功能也还可以，但是实际上这个东西并没有广泛流行开来。最重要的原因就是它的配置太过于繁琐，当时网上流传一句话：“每当有人要使用 Acegi Security，就会有一个精灵死去。” 足见 Acegi Security 的配置是多么可怕。直到今天，当人们谈起 Spring Security 的时候，依然在吐槽它的配置繁琐。

后来 Acegi Security 投入 Spring 的怀抱，改名叫 Spring Security，事情才慢慢开始发生变化。新的开发团队一直在尽力简化 Spring Security 的配置，Spring Security 的配置相比 Acegi Security 确实简化了很多。但是在最初的几年里，Spring Security 依然无法得到广泛的使用。

直到有一天 Spring Boot 像谜一般出现在江湖边缘，彻底颠覆了 JavaEE 的世界。一人得道鸡犬升天，自从 Spring Boot 火了之后，Spring 家族的产品都被带了一把，Spring Security 就是受益者之一，从此飞上枝头变凤凰。

Spring Boot/Spring Cloud 现在作为 Java 开发领域最最主流的技术栈，这一点大家应该都没有什么异议，而在 Spring Boot/Spring Cloud 中做安全管理，Spring Security 无疑是最方便的。

你想保护 Spring Boot 中的接口，添加一个 Spring Security 的依赖即可，事情就搞定了，所有接口就保护起来了，甚至不需要一行配置。

有小伙伴可能觉得这个太笼统了，我再举一个实际点的例子。

在微服务架构的项目中，我们可能使用 Eureka 做服务注册中心，默认情况下，Eureka 没有做安全管理，如果你想给 Eureka 添加安全管理，只需要添加 Spring Security 依赖，然后在 application.properties 中配置一下用户名密码即可，Eureka 就自动被保护起来了，别人无法轻易访问；然后各个微服务在注册的时候，只需要把注册地址改为 http://username:password@localhost:8080/eureka 即可。类似的例子还有 Spring Cloud Config 中的安全管理。

在微服务这种场景下，如果你想用 Shiro 代替 Spring Security，那 Shiro 代码量绝对非常可观，Spring Security 则可以非常容易的集成到现在流行的 Spring Boot/Spring Cloud 技术栈中，可以和 Spring Boot、Spring Cloud、Spring Social、WebSocket 等非常方便的整合。

所以我说，因为 Spring Boot/Spring Cloud 火爆，让 Spring Security 跟着沾了一把光。

### 1.2 配置臃肿吗？

**有的人觉得 Spring Security 配置臃肿。**

如果是 SSM + Spring Security 的话，我觉得这话有一定道理。

但是如果是 Spring Boot 项目的话，其实并不见得臃肿。Spring Boot 中，通过自动化配置 starter 已经极大的简化了 Spring Security 的配置，我们只需要做少量的定制的就可以实现认证和授权了，这一点，大家可以参考我之前连载的 Spring Security 系列文章基本就能感受到这种便利（公众号后台回复 2020 有文章索引）。

**有人觉得 Spring Security 中概念复杂。**

这个是这样的，没错。

Spring Security 由于功能比较多，支持 OAuth2 等原因，就显得比较重量级，不像 Shiro 那样轻便。

但是如果换一个角度，你可能会有不一样的感受。

在 Spring Security 中你会学习到许多安全管理相关的概念，以及常见的安全攻击。这些安全攻击，如果你不是 web 安全方面的专家，很多可能存在的 web 攻击和漏洞你可能很难想到，而 Spring Security 则把这些安全问题都给我们罗列出来并且给出了相应的解决方案。

所以我说，我们学习 Spring Security 的过程，也是在学习 web 安全，各种各样的安全攻击、各种各样的登录方式、各种各样你能想到或者想不到的安全问题，Spring Security 都给我们罗列出来了，并且给出了解决方案，从这个角度来看，你会发现 Spring Security 好像也不是那么让人讨厌。

### 1.3 结合微服务的优势

除了前面和大家介绍的 Spring Security 优势，在微服务中，Spring 官方推出了 Spring Cloud Security 和 Spring Cloud OAuth2，结合微服务这种分布式特性，可以让我们更加方便的在微服务中使用 Spring Security 和 OAuth2，松哥前面的 OAuth2 系列实际上都是基于 Spring Cloud Security 来做的。

可以看到，Spring 官方一直在积极进取，让 Spring Security 能够更好的集成进微服务中。

## 2. Shiro

接下来我们再说说 Apache Shiro。

Apache Shiro 是一个开源安全框架，提供身份验证、授权、密码学和会话管理。Shiro 框架具有直观、易用等特性，同时也能提供健壮的安全性，虽然它的功能不如 Spring Security 那么强大，但是在常规的企业级应用中，其实也够用了。

### 2.1 由来

Shiro 的前身是 JSecurity，2004 年，Les Hazlewood 和 Jeremy Haile 创办了 Jsecurity。当时他们找不到适用于应用程序级别的合适 Java 安全框架，同时又对 JAAS 非常失望，于是就搞了这个东西。

2004 年到 2008 年期间，JSecurity 托管在 SourceForge 上，贡献者包括 Peter Ledbrook、Alan Ditzel 和 Tim Veil。

2008 年，JSecurity 项目贡献给了 Apache 软件基金会（ASF），并被接纳成为 Apache Incubator 项目，由导师管理，目标是成为一个顶级 Apache 项目。期间，Jsecurity 曾短暂更名为 Ki，随后因商标问题被社区更名为“Shiro”。随后项目持续在 Apache Incubator 中孵化，并增加了贡献者 Kalle Korhonen。

2010 年 7 月，Shiro 社区发布了 1.0 版，随后社区创建了其项目管理委员会，并选举 Les Hazlewood 为主席。2010 年 9 月 22 日，Shrio 成为 Apache 软件基金会的顶级项目（TLP）。

### 2.2 有哪些功能

Apache Shiro 是一个强大而灵活的开源安全框架，它干净利落地处理身份认证，授权，企业会话管理和加密。Apache Shiro 的首要目标是易于使用和理解。安全有时候是很复杂的，甚至是痛苦的，但它没有必要这样。框架应该尽可能掩盖复杂的地方，露出一个干净而直观的 API，来简化开发人员在应用程序安全上所花费的时间。

以下是你可以用 Apache Shiro 所做的事情：

1. 验证用户来核实他们的身份
2. 对用户执行访问控制，如：判断用户是否被分配了一个确定的安全角色；判断用户是否被允许做某事
3. 在任何环境下使用Session API，即使没有Web容器
4. 在身份验证，访问控制期间或在会话的生命周期，对事件作出反应
5. 聚集一个或多个用户安全数据的数据源，并作为一个单一的复合用户“视图”
6. 单点登录（SSO）功能
7. 为没有关联到登录的用户启用”Remember Me”服务
8. …

Apache Shiro 是一个拥有许多功能的综合性的程序安全框架。下面的图表展示了 Shiro 的重点：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204105557.png)](http://img.itboyhub.com/2020/04/20200429193053.png)

Shiro 中有四大基石——身份验证，授权，会话管理和加密。

1. Authentication：有时也简称为“登录”，这是一个证明用户是谁的行为。
2. Authorization：访问控制的过程，也就是决定“谁”去访问“什么”。
3. Session Management：管理用户特定的会话，即使在非 Web 或 EJB 应用程序。
4. Cryptography：通过使用加密算法保持数据安全同时易于使用。

除此之外，Shiro 也提供了额外的功能来解决在不同环境下所面临的安全问题，尤其是以下这些：

1. Web Support：Shiro 的 web 支持的 API 能够轻松地帮助保护 Web 应用程序。
2. Caching：缓存是 Apache Shiro 中的第一层公民，来确保安全操作快速而又高效。
3. Concurrency：Apache Shiro 利用它的并发特性来支持多线程应用程序。
4. Testing：测试支持的存在来帮助你编写单元测试和集成测试。
5. “Run As”：一个允许用户假设为另一个用户身份（如果允许）的功能，有时候在管理脚本很有用。
6. “Remember Me”：在会话中记住用户的身份，这样用户只需要在强制登录时候登录。

### 2.3 学习资料

Shiro 的学习资料并不多，没看到有相关的书籍。张开涛的《跟我学Shiro》是一个非常不错的资料，小伙伴可以搜索了解下。也可以在公众号**江南一点雨**后台回复 2TB，有相关的视频教程。

### 2.4 优势和劣势

就目前而言，Shiro 最大的问题在于和 Spring 家族的产品进行整合的时候非常不便，在 Spring Boot 推出的很长一段时间里，Shiro 都没有提供相应的 starter，后来虽然有一个 `shiro-spring-boot-web-starter` 出来，但是其实配置并没有简化多少。所以在 Spring Boot/Spring Cloud 技术栈的微服务项目中，Shiro 几乎不存在优势。

但是如果你是传统的 SSM 项目，不是微服务项目，那么无疑使用 Shiro 是最方便省事的，因为它足够简单，足够轻量级。

## 3. 如何取舍

在公司里做开发，这两个要如何取舍，还是要考虑蛮多东西的。

首先，如果是基于 Spring Boot/Spring Cloud 的微服务项目，Spring Security 无疑是最方便的。

如果是就是普通的 SSM 项目，那么 Shiro 基本上也够用。

另外，选择技术栈的时候，我们可能也要考虑团队内工程师的技术栈，如果工程师更擅长 Shiro，那么无疑 Shiro 是合适的，毕竟让工程师去学习一门新的技术，一来可能影响项目进度，而来也可能给项目埋下许多未知的雷。

# SpringSecurity 自定义认证逻辑的两种方式(高级玩法)

之前我们自定义的一个核心思路就是自定义过滤器，在过滤器中做各种各样我们想做的事：



- [Spring Security 如何添加登录验证码？松哥手把手教你给微人事添加登录验证码](https://mp.weixin.qq.com/s/aaop_dS9UIOgTtQd0hl_tw)
- [前后端分离中，使用 JSON 格式登录原来这么简单！](https://mp.weixin.qq.com/s/RHoXwIn6J-O8tbVjsYIcBQ)

上面这两篇文章都是使用了自定义过滤器的思路，这算是一种入门级的自定义认证逻辑了，不知道大家有没有想过，这种方式其实是有一些问题的。

举一个简单的例子，在添加登录验证码中，我为了校验验证码就自定义了一个过滤器，并把这个自定义的过滤器放入 SpringSecurity 过滤器链中，每次请求都会通过该过滤器。但实际上，只需要登录请求经过该过滤器即可，其他请求是不需要经过该过滤器的，这个时候，大家是不是就发现弊端了。

当然，如果你对性能没有极致追求，这种写法其实也问题不大，毕竟功能已经实现了，但是抱着学习的态度，今天松哥要在前面文章的基础上给大家介绍一个更加优雅的写法。

## 1.认证流程简析

AuthenticationProvider 定义了 Spring Security 中的验证逻辑，我们来看下 AuthenticationProvider 的定义：

```
public interface AuthenticationProvider {
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
	boolean supports(Class<?> authentication);
}
```

可以看到，AuthenticationProvider 中就两个方法：

- authenticate 方法用来做验证，就是验证用户身份。
- supports 则用来判断当前的 AuthenticationProvider 是否支持对应的 Authentication。

这里又涉及到一个东西，就是 Authentication。

玩过 Spring Security 的小伙伴都知道，在 Spring Security 中有一个非常重要的对象叫做 Authentication，我们可以在任何地方注入 Authentication 进而获取到当前登录用户信息，Authentication 本身是一个接口，它实际上对 java.security.Principal 做的进一步封装，我们来看下 Authentication 的定义：

```java
public interface Authentication extends Principal, Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

可以看到，这里接口中的方法也没几个，我来大概解释下：

1. getAuthorities 方法用来获取用户的权限。
2. getCredentials 方法用来获取用户凭证，一般来说就是密码。
3. getDetails 方法用来获取用户携带的详细信息，可能是当前请求之类的东西。
4. getPrincipal 方法用来获取当前用户，可能是一个用户名，也可能是一个用户对象。
5. isAuthenticated 当前用户是否认证成功。

Authentication 作为一个接口，它定义了用户，或者说 Principal 的一些基本行为，它有很多实现类：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204110311.png)](http://img.itboyhub.com/2020/03/authentication-1.png)

在这些实现类中，我们最常用的就是 UsernamePasswordAuthenticationToken 了，而每一个 Authentication 都有适合它的 AuthenticationProvider 去处理校验。例如处理 UsernamePasswordAuthenticationToken 的 AuthenticationProvider 是 DaoAuthenticationProvider。

所以大家在 AuthenticationProvider 中看到一个 supports 方法，就是用来判断 AuthenticationProvider 是否支持当前 Authentication。

在一次完整的认证中，可能包含多个 AuthenticationProvider，而这多个 AuthenticationProvider 则由 ProviderManager 进行统一管理，具体可以参考松哥之前的文章：[松哥手把手带你捋一遍 Spring Security 登录流程](https://mp.weixin.qq.com/s/z6GeR5O-vBzY3SHehmccVA)。

这里我们来重点看一下 DaoAuthenticationProvider，因为这是我们最常用的一个，当我们使用用户名/密码登录的时候，用的就是它，DaoAuthenticationProvider 的父类是 AbstractUserDetailsAuthenticationProvider，我们就先从它的父类看起：

```java
public abstract class AbstractUserDetailsAuthenticationProvider implements
		AuthenticationProvider, InitializingBean, MessageSourceAware {
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
				: authentication.getName();
		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);
		if (user == null) {
			cacheWasUsed = false;
			try {
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException notFound) {
				logger.debug("User '" + username + "' not found");

				if (hideUserNotFoundExceptions) {
					throw new BadCredentialsException(messages.getMessage(
							"AbstractUserDetailsAuthenticationProvider.badCredentials",
							"Bad credentials"));
				}
				else {
					throw notFound;
				}
			}
		}

		try {
			preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user,
					(UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException exception) {
			if (cacheWasUsed) {
				cacheWasUsed = false;
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
				preAuthenticationChecks.check(user);
				additionalAuthenticationChecks(user,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			else {
				throw exception;
			}
		}

		postAuthenticationChecks.check(user);

		if (!cacheWasUsed) {
			this.userCache.putUserInCache(user);
		}

		Object principalToReturn = user;

		if (forcePrincipalAsString) {
			principalToReturn = user.getUsername();
		}

		return createSuccessAuthentication(principalToReturn, authentication, user);
	}
	public boolean supports(Class<?> authentication) {
		return (UsernamePasswordAuthenticationToken.class
				.isAssignableFrom(authentication));
	}
}
```

AbstractUserDetailsAuthenticationProvider 的代码还是挺长的，这里我们重点关注两个方法：authenticate 和 supports。

authenticate 方法就是用来做认证的方法，我们来简单看下方法流程：

1. 首先从 Authentication 提取出登录用户名。
2. 然后通过拿着 username 去调用 retrieveUser 方法去获取当前用户对象，这一步会调用我们自己在登录时候的写的 loadUserByUsername 方法，所以这里返回的 user 其实就是你的登录对象，可以参考微人事的 org/javaboy/vhr/service/HrService.java#L34，也可以参考本系列之前的文章：[Spring Security+Spring Data Jpa 强强联手，安全管理只有更简单！](https://mp.weixin.qq.com/s/VWJvINbi1DB3fF-Mcx7mGg)。
3. 接下来调用 preAuthenticationChecks.check 方法去检验 user 中的各个账户状态属性是否正常，例如账户是否被禁用、账户是否被锁定、账户是否过期等等。
4. additionalAuthenticationChecks 方法则是做密码比对的，好多小伙伴好奇 Spring Security 的密码加密之后，是如何进行比较的，看这里就懂了，因为比较的逻辑很简单，我这里就不贴代码出来了。但是注意，additionalAuthenticationChecks 方法是一个抽象方法，具体的实现是在 AbstractUserDetailsAuthenticationProvider 的子类中实现的，也就是 DaoAuthenticationProvider。这个其实很好理解，因为 AbstractUserDetailsAuthenticationProvider 作为一个较通用的父类，处理一些通用的行为，我们在登录的时候，有的登录方式并不需要密码，所以 additionalAuthenticationChecks 方法一般交给它的子类去实现，在 DaoAuthenticationProvider 类中，additionalAuthenticationChecks 方法就是做密码比对的，在其他的 AuthenticationProvider 中，additionalAuthenticationChecks 方法的作用就不一定了。
5. 最后在 postAuthenticationChecks.check 方法中检查密码是否过期。
6. 接下来有一个 forcePrincipalAsString 属性，这个是是否强制将 Authentication 中的 principal 属性设置为字符串，这个属性我们一开始在 UsernamePasswordAuthenticationFilter 类中其实就是设置为字符串的（即 username），但是默认情况下，当用户登录成功之后， 这个属性的值就变成当前用户这个对象了。之所以会这样，就是因为 forcePrincipalAsString 默认为 false，不过这块其实不用改，就用 false，这样在后期获取当前用户信息的时候反而方便很多。
7. 最后，通过 createSuccessAuthentication 方法构建一个新的 UsernamePasswordAuthenticationToken。

supports 方法就比较简单了，主要用来判断当前的 Authentication 是否是 UsernamePasswordAuthenticationToken。

由于 AbstractUserDetailsAuthenticationProvider 已经把 authenticate 和 supports 方法实现了，所以在 DaoAuthenticationProvider 中，我们主要关注 additionalAuthenticationChecks 方法即可：

```
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
	@SuppressWarnings("deprecation")
	protected void additionalAuthenticationChecks(UserDetails userDetails,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		if (authentication.getCredentials() == null) {
			throw new BadCredentialsException(messages.getMessage(
					"AbstractUserDetailsAuthenticationProvider.badCredentials",
					"Bad credentials"));
		}
		String presentedPassword = authentication.getCredentials().toString();
		if (!passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
			throw new BadCredentialsException(messages.getMessage(
					"AbstractUserDetailsAuthenticationProvider.badCredentials",
					"Bad credentials"));
		}
	}
}
```

大家可以看到，additionalAuthenticationChecks 方法主要用来做密码比对的，逻辑也比较简单，就是调用 PasswordEncoder 的 matches 方法做比对，如果密码不对则直接抛出异常即可。

**正常情况下，我们使用用户名/密码登录，最终都会走到这一步。**

而 AuthenticationProvider 都是通过 ProviderManager#authenticate 方法来调用的。由于我们的一次认证可能会存在多个 AuthenticationProvider，所以，在 ProviderManager#authenticate 方法中会逐个遍历 AuthenticationProvider，并调用他们的 authenticate 方法做认证，我们来稍微瞅一眼 ProviderManager#authenticate 方法：

```
public Authentication authenticate(Authentication authentication)
		throws AuthenticationException {
	for (AuthenticationProvider provider : getProviders()) {
		result = provider.authenticate(authentication);
		if (result != null) {
			copyDetails(authentication, result);
			break;
		}
	}
    ...
    ...
}
```

可以看到，在这个方法中，会遍历所有的 AuthenticationProvider，并调用它的 authenticate 方法进行认证。

好了，大致的认证流程说完之后，相信大家已经明白了我们要从哪里下手了。

## 2.自定义认证思路

之前我们通过自定义过滤器，将自定义的过滤器加入到 Spring Security 过滤器链中，进而实现了添加登录验证码功能，但是我们也说这种方式是有弊端的，就是破坏了原有的过滤器链，请求每次都要走一遍验证码过滤器，这样不合理。

我们改进的思路也很简单。

登录请求是调用 AbstractUserDetailsAuthenticationProvider#authenticate 方法进行认证的，在该方法中，又会调用到 DaoAuthenticationProvider#additionalAuthenticationChecks 方法做进一步的校验，去校验用户登录密码。我们可以自定义一个 AuthenticationProvider 代替 DaoAuthenticationProvider，并重写它里边的 additionalAuthenticationChecks 方法，在重写的过程中，加入验证码的校验逻辑即可。

这样既不破坏原有的过滤器链，又实现了自定义认证功能。**常见的手机号码动态登录，也可以使用这种方式来认证。**

好了，不 bb 了，咱们上代码。

## 3.代码实现

首先我们需要验证码，这次我就懒得自己去实现了，我们用网上一个现成的验证码库 kaptcha，首先我们添加该库的依赖，如下：

```
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

然后我们提供一个实体类用来描述验证码的基本信息：

```
@Bean
Producer verifyCode() {
    Properties properties = new Properties();
    properties.setProperty("kaptcha.image.width", "150");
    properties.setProperty("kaptcha.image.height", "50");
    properties.setProperty("kaptcha.textproducer.char.string", "0123456789");
    properties.setProperty("kaptcha.textproducer.char.length", "4");
    Config config = new Config(properties);
    DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
    defaultKaptcha.setConfig(config);
    return defaultKaptcha;
}
```

这段配置很简单，我们就是提供了验证码图片的宽高、字符库以及生成的验证码字符长度。

接下来提供一个返回验证码图片的接口：

```
@RestController
public class VerifyCodeController {
    @Autowired
    Producer producer;
    @GetMapping("/vc.jpg")
    public void getVerifyCode(HttpServletResponse resp, HttpSession session) throws IOException {
        resp.setContentType("image/jpeg");
        String text = producer.createText();
        session.setAttribute("verify_code", text);
        BufferedImage image = producer.createImage(text);
        try(ServletOutputStream out = resp.getOutputStream()) {
            ImageIO.write(image, "jpg", out);
        }
    }
}
```

这里我们生成验证码图片，并将生成的验证码字符存入 HttpSession 中。注意这里我用到了 try-with-resources ，可以自动关闭流，有的小伙伴可能不太清楚，可以自己搜索看下。

接下来我们来自定义一个 MyAuthenticationProvider 继承自 DaoAuthenticationProvider，并重写 additionalAuthenticationChecks 方法：

```
public class MyAuthenticationProvider extends DaoAuthenticationProvider {

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        HttpServletRequest req = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String code = req.getParameter("code");
        String verify_code = (String) req.getSession().getAttribute("verify_code");
        if (code == null || verify_code == null || !code.equals(verify_code)) {
            throw new AuthenticationServiceException("验证码错误");
        }
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
```

在 additionalAuthenticationChecks 方法中：

1. 首先获取当前请求，注意这种获取方式，在基于 Spring 的 web 项目中，我们可以随时随地获取到当前请求，获取方式就是我上面给出的代码。
2. 从当前请求中拿到 code 参数，也就是用户传来的验证码。
3. 从 session 中获取生成的验证码字符串。
4. 两者进行比较，如果验证码输入错误，则直接抛出异常。
5. 最后通过 super 调用父类方法，也就是 DaoAuthenticationProvider 的 additionalAuthenticationChecks 方法，该方法中主要做密码的校验。

MyAuthenticationProvider 定义好之后，接下来主要是如何让 MyAuthenticationProvider 代替 DaoAuthenticationProvider。

前面我们说，所有的 AuthenticationProvider 都是放在 ProviderManager 中统一管理的，所以接下来我们就要自己提供 ProviderManager，然后注入自定义的 MyAuthenticationProvider，这一切操作都在 SecurityConfig 中完成：

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Bean
    MyAuthenticationProvider myAuthenticationProvider() {
        MyAuthenticationProvider myAuthenticationProvider = new MyAuthenticationProvider();
        myAuthenticationProvider.setPasswordEncoder(passwordEncoder());
        myAuthenticationProvider.setUserDetailsService(userDetailsService());
        return myAuthenticationProvider;
    }
    
    @Override
    @Bean
    protected AuthenticationManager authenticationManager() throws Exception {
        ProviderManager manager = new ProviderManager(Arrays.asList(myAuthenticationProvider()));
        return manager;
    }

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("javaboy").password("123").roles("admin").build());
        return manager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/vc.jpg").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .successHandler((req, resp, auth) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(new ObjectMapper().writeValueAsString(RespBean.ok("success", auth.getPrincipal())));
                    out.flush();
                    out.close();
                })
                .failureHandler((req, resp, e) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(new ObjectMapper().writeValueAsString(RespBean.error(e.getMessage())));
                    out.flush();
                    out.close();
                })
                .permitAll()
                .and()
                .csrf().disable();
    }
}
```

这里的代码我稍作解释：

1. 我们需要提供一个 MyAuthenticationProvider 的实例，创建该实例时，需要提供 UserDetailService 和 PasswordEncoder 实例。
2. 通过重写 authenticationManager 方法来提供一个自己的 AuthenticationManager，实际上就是 ProviderManager，在创建 ProviderManager 时，加入自己的 myAuthenticationProvider。
3. 这里为了简单，我将用户直接存在内存中，提供一个 UserDetailsService 实例即可。如果大家想将用户存在数据库中，可以参考松哥之前的文章：[Spring Security+Spring Data Jpa 强强联手，安全管理只有更简单！](https://mp.weixin.qq.com/s/VWJvINbi1DB3fF-Mcx7mGg)。
4. 最后就简单配置一下各种回调即可，另外记得设置 `/vc.jpg` 任何人都能访问。

好了，如此之后，在不需要修改原生过滤器链的情况下，我们嵌入了自己的认证逻辑。

## 4.测试

好了，接下来，启动项目，我们开始测试。

为了方便，这里我就用 POSTMAN 来测试，首先可以给一个错误的验证码，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204110343.png)](http://img.itboyhub.com/2020/04/20200502214324.png)

接下来，请求 /vc.jpg 获取验证码：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204110352.png)](http://img.itboyhub.com/2020/04/20200502214427.png)

输入正确的验证码和错误的密码，再进行登录：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204110355.png)](http://img.itboyhub.com/2020/04/20200502214513.png)

最后，所有的都输入正确，再来看下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204110403.png)](http://img.itboyhub.com/2020/04/20200502214614.png)

登录成功！

## 5.小结

上面的例子，我使用了添加登录验证码的案例，实际上，其他的登录场景也可以考虑这种方案，例如目前广为流行的手机号码动态登录，就可以使用这种方式认证。

# Spring Security 中如何快速查看登录用户 IP 地址等信息？

## 1.Authentication

Authentication 这个接口前面和大家聊过多次，今天还要再来聊一聊。

Authentication 接口用来保存我们的登录用户信息，实际上，它是对主体（java.security.Principal）做了进一步的封装。

我们来看下 Authentication 的一个定义：

```
public interface Authentication extends Principal, Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

接口的解释如下：

1. getAuthorities 方法用来获取用户的权限。
2. getCredentials 方法用来获取用户凭证，一般来说就是密码。
3. getDetails 方法用来获取用户携带的详细信息，可能是当前请求之类的东西。
4. getPrincipal 方法用来获取当前用户，可能是一个用户名，也可能是一个用户对象。
5. isAuthenticated 当前用户是否认证成功。

这里有一个比较好玩的方法，叫做 getDetails。关于这个方法，源码的解释如下：

Stores additional details about the authentication request. These might be an IP address, certificate serial number etc.

从这段解释中，我们可以看出，该方法实际上就是用来存储有关身份认证的其他信息的，例如 IP 地址、证书信息等等。

实际上，在默认情况下，这里存储的就是用户登录的 IP 地址和 sessionId。我们从源码角度来看下。

## 2.源码分析

松哥的 SpringSecurity 系列已经写到第 12 篇了，看了前面的文章，相信大家已经明白用户登录必经的一个过滤器就是 UsernamePasswordAuthenticationFilter，在该类的 attemptAuthentication 方法中，对请求参数做提取，在 attemptAuthentication 方法中，会调用到一个方法，就是 setDetails。

我们一起来看下 setDetails 方法：

```
protected void setDetails(HttpServletRequest request,
		UsernamePasswordAuthenticationToken authRequest) {
	authRequest.setDetails(authenticationDetailsSource.buildDetails(request));
}
```

UsernamePasswordAuthenticationToken 是 Authentication 的具体实现，所以这里实际上就是在设置 details，至于 details 的值，则是通过 authenticationDetailsSource 来构建的，我们来看下：

```
public class WebAuthenticationDetailsSource implements
		AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {
	public WebAuthenticationDetails buildDetails(HttpServletRequest context) {
		return new WebAuthenticationDetails(context);
	}
}
public class WebAuthenticationDetails implements Serializable {
	private final String remoteAddress;
	private final String sessionId;
	public WebAuthenticationDetails(HttpServletRequest request) {
		this.remoteAddress = request.getRemoteAddr();

		HttpSession session = request.getSession(false);
		this.sessionId = (session != null) ? session.getId() : null;
	}
    //省略其他方法
}
```

默认通过 WebAuthenticationDetailsSource 来构建 WebAuthenticationDetails，并将结果设置到 Authentication 的 details 属性中去。而 WebAuthenticationDetails 中定义的属性，大家看一下基本上就明白，这就是保存了用户登录地址和 sessionId。

那么看到这里，大家基本上就明白了，用户登录的 IP 地址实际上我们可以直接从 WebAuthenticationDetails 中获取到。

我举一个简单例子，例如我们登录成功后，可以通过如下方式随时随地拿到用户 IP：

```
@Service
public class HelloService {
    public void hello() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        WebAuthenticationDetails details = (WebAuthenticationDetails) authentication.getDetails();
        System.out.println(details);
    }
}
```

这个获取过程之所以放在 service 来做，就是为了演示**随时随地**这个特性。然后我们在 controller 中调用该方法，当访问接口时，可以看到如下日志：

```
WebAuthenticationDetails@fffc7f0c: RemoteIpAddress: 127.0.0.1; SessionId: 303C7F254DF8B86667A2B20AA0667160
```

可以看到，用户的 IP 地址和 SessionId 都给出来了。这两个属性在 WebAuthenticationDetails 中都有对应的 get 方法，也可以单独获取属性值。

## 3.定制

当然，WebAuthenticationDetails 也可以自己定制，因为默认它只提供了 IP 和 sessionid 两个信息，如果我们想保存关于 Http 请求的更多信息，就可以通过自定义 WebAuthenticationDetails 来实现。

如果我们要定制 WebAuthenticationDetails，还要连同 WebAuthenticationDetailsSource 一起重新定义。

结合[上篇文章](https://mp.weixin.qq.com/s/LeiwIJVevaU5C1Fn5nNEeg)的验证码登录，我跟大家演示一个自定义 WebAuthenticationDetails 的例子。

[上篇文章](https://mp.weixin.qq.com/s/LeiwIJVevaU5C1Fn5nNEeg)我们是在 MyAuthenticationProvider 类中进行验证码判断的，回顾一下[上篇文章](https://mp.weixin.qq.com/s/LeiwIJVevaU5C1Fn5nNEeg)的代码：

```
public class MyAuthenticationProvider extends DaoAuthenticationProvider {

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        HttpServletRequest req = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String code = req.getParameter("code");
        String verify_code = (String) req.getSession().getAttribute("verify_code");
        if (code == null || verify_code == null || !code.equals(verify_code)) {
            throw new AuthenticationServiceException("验证码错误");
        }
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
```

不过这个验证操作，我们也可以放在自定义的 WebAuthenticationDetails 中来做，我们定义如下两个类：

```
public class MyWebAuthenticationDetails extends WebAuthenticationDetails {

    private boolean isPassed;

    public MyWebAuthenticationDetails(HttpServletRequest req) {
        super(req);
        String code = req.getParameter("code");
        String verify_code = (String) req.getSession().getAttribute("verify_code");
        if (code != null && verify_code != null && code.equals(verify_code)) {
            isPassed = true;
        }
    }

    public boolean isPassed() {
        return isPassed;
    }
}
@Component
public class MyWebAuthenticationDetailsSource implements AuthenticationDetailsSource<HttpServletRequest,MyWebAuthenticationDetails> {
    @Override
    public MyWebAuthenticationDetails buildDetails(HttpServletRequest context) {
        return new MyWebAuthenticationDetails(context);
    }
}
```

首先我们定义 MyWebAuthenticationDetails，由于它的构造方法中，刚好就提供了 HttpServletRequest 对象，所以我们可以直接利用该对象进行验证码判断，并将判断结果交给 isPassed 变量保存。**如果我们想扩展属性，只需要在 MyWebAuthenticationDetails 中再去定义更多属性，然后从 HttpServletRequest 中提取出来设置给对应的属性即可，这样，在登录成功后就可以随时随地获取这些属性了。**

最后在 MyWebAuthenticationDetailsSource 中构造 MyWebAuthenticationDetails 并返回。

定义完成后，接下来，我们就可以直接在 MyAuthenticationProvider 中进行调用了：

```
public class MyAuthenticationProvider extends DaoAuthenticationProvider {

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        if (!((MyWebAuthenticationDetails) authentication.getDetails()).isPassed()) {
            throw new AuthenticationServiceException("验证码错误");
        }
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
```

直接从 authentication 中获取到 details 并调用 isPassed 方法，有问题就抛出异常即可。

最后的问题就是如何用自定义的 MyWebAuthenticationDetailsSource 代替系统默认的 WebAuthenticationDetailsSource，很简单，我们只需要在 SecurityConfig 中稍作定义即可：

```
@Autowired
MyWebAuthenticationDetailsSource myWebAuthenticationDetailsSource;
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            ...
            .and()
            .formLogin()
            .authenticationDetailsSource(myWebAuthenticationDetailsSource)
            ...
}
```

将 MyWebAuthenticationDetailsSource 注入到 SecurityConfig 中，并在 formLogin 中配置 authenticationDetailsSource 即可成功使用我们自定义的 WebAuthenticationDetails。

这样自定义完成后，WebAuthenticationDetails 中原有的功能依然保留，也就是我们还可以利用老办法继续获取用户 IP 以及 sessionId 等信息，如下：

```
@Service
public class HelloService {
    public void hello() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        MyWebAuthenticationDetails details = (MyWebAuthenticationDetails) authentication.getDetails();
        System.out.println(details);
    }
}
```

这里类型强转的时候，转为 MyWebAuthenticationDetails 即可。

# 参考

http://www.javaboy.org/springsecurity/