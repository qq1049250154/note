# Spring Security 自动踢掉前一个登录用户，一个配置搞定！

登录成功后，自动踢掉前一个登录用户，松哥第一次见到这个功能，就是在扣扣里边见到的，当时觉得挺好玩的。

## 1.需求分析

在同一个系统中，我们可能只允许一个用户在一个终端上登录，一般来说这可能是出于安全方面的考虑，但是也有一些情况是出于业务上的考虑，松哥之前遇到的需求就是业务原因要求一个用户只能在一个设备上登录。

要实现一个用户不可以同时在两台设备上登录，我们有两种思路：

- 后来的登录自动踢掉前面的登录，就像大家在扣扣中看到的效果。
- 如果用户已经登录，则不允许后来者登录。

这种思路都能实现这个功能，具体使用哪一个，还要看我们具体的需求。

在 Spring Security 中，这两种都很好实现，一个配置就可以搞定。

## 2.具体实现

### 2.1 踢掉已经登录用户

想要用新的登录踢掉旧的登录，我们只需要将最大会话数设置为 1 即可，配置如下：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .permitAll()
            .and()
            .csrf().disable()
            .sessionManagement()
            .maximumSessions(1);
}
```

maximumSessions 表示配置最大会话数为 1，这样后面的登录就会自动踢掉前面的登录。这里其他的配置都是我们前面文章讲过的，我就不再重复介绍，文末可以下载案例完整代码。

配置完成后，分别用 Chrome 和 Firefox 两个浏览器进行测试（或者使用 Chrome 中的多用户功能）。

1. Chrome 上登录成功后，访问 /hello 接口。
2. Firefox 上登录成功后，访问 /hello 接口。
3. 在 Chrome 上再次访问 /hello 接口，此时会看到如下提示：

```
This session has been expired (possibly due to multiple concurrent logins being attempted as the same user).
```

可以看到，这里说这个 session 已经过期，原因则是由于使用同一个用户进行并发登录。

### 2.2 禁止新的登录

如果相同的用户已经登录了，你不想踢掉他，而是想禁止新的登录操作，那也好办，配置方式如下：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .permitAll()
            .and()
            .csrf().disable()
            .sessionManagement()
            .maximumSessions(1)
            .maxSessionsPreventsLogin(true);
}
```

添加 maxSessionsPreventsLogin 配置即可。此时一个浏览器登录成功后，另外一个浏览器就登录不了了。

是不是很简单？

不过还没完，我们还需要再提供一个 Bean：

```
@Bean
HttpSessionEventPublisher httpSessionEventPublisher() {
    return new HttpSessionEventPublisher();
}
```

为什么要加这个 Bean 呢？因为在 Spring Security 中，它是通过监听 session 的销毁事件，来及时的清理 session 的记录。用户从不同的浏览器登录后，都会有对应的 session，当用户注销登录之后，session 就会失效，但是默认的失效是通过调用 StandardSession#invalidate 方法来实现的，这一个失效事件无法被 Spring 容器感知到，进而导致当用户注销登录之后，Spring Security 没有及时清理会话信息表，以为用户还在线，进而导致用户无法重新登录进来（小伙伴们可以自行尝试不添加上面的 Bean，然后让用户注销登录之后再重新登录）。

为了解决这一问题，我们提供一个 HttpSessionEventPublisher ，这个类实现了 HttpSessionListener 接口，在该 Bean 中，可以将 session 创建以及销毁的事件及时感知到，并且调用 Spring 中的事件机制将相关的创建和销毁事件发布出去，进而被 Spring Security 感知到，该类部分源码如下：

```
public void sessionCreated(HttpSessionEvent event) {
	HttpSessionCreatedEvent e = new HttpSessionCreatedEvent(event.getSession());
	getContext(event.getSession().getServletContext()).publishEvent(e);
}
public void sessionDestroyed(HttpSessionEvent event) {
	HttpSessionDestroyedEvent e = new HttpSessionDestroyedEvent(event.getSession());
	getContext(event.getSession().getServletContext()).publishEvent(e);
}
```

OK，虽然多了一个配置，但是依然很简单！

## 3.实现原理

上面这个功能，在 Spring Security 中是怎么实现的呢？我们来稍微分析一下源码。

首先我们知道，在用户登录的过程中，会经过 UsernamePasswordAuthenticationFilter（参考：[松哥手把手带你捋一遍 Spring Security 登录流程](https://mp.weixin.qq.com/s/z6GeR5O-vBzY3SHehmccVA)），而 UsernamePasswordAuthenticationFilter 中过滤方法的调用是在 AbstractAuthenticationProcessingFilter 中触发的，我们来看下 AbstractAuthenticationProcessingFilter#doFilter 方法的调用：

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
		throws IOException, ServletException {
	HttpServletRequest request = (HttpServletRequest) req;
	HttpServletResponse response = (HttpServletResponse) res;
	if (!requiresAuthentication(request, response)) {
		chain.doFilter(request, response);
		return;
	}
	Authentication authResult;
	try {
		authResult = attemptAuthentication(request, response);
		if (authResult == null) {
			return;
		}
		sessionStrategy.onAuthentication(authResult, request, response);
	}
	catch (InternalAuthenticationServiceException failed) {
		unsuccessfulAuthentication(request, response, failed);
		return;
	}
	catch (AuthenticationException failed) {
		unsuccessfulAuthentication(request, response, failed);
		return;
	}
	// Authentication success
	if (continueChainBeforeSuccessfulAuthentication) {
		chain.doFilter(request, response);
	}
	successfulAuthentication(request, response, chain, authResult);
```

在这段代码中，我们可以看到，调用 attemptAuthentication 方法走完认证流程之后，回来之后，接下来就是调用 sessionStrategy.onAuthentication 方法，这个方法就是用来处理 session 的并发问题的。具体在：

```
public class ConcurrentSessionControlAuthenticationStrategy implements
		MessageSourceAware, SessionAuthenticationStrategy {
	public void onAuthentication(Authentication authentication,
			HttpServletRequest request, HttpServletResponse response) {

		final List<SessionInformation> sessions = sessionRegistry.getAllSessions(
				authentication.getPrincipal(), false);

		int sessionCount = sessions.size();
		int allowedSessions = getMaximumSessionsForThisUser(authentication);

		if (sessionCount < allowedSessions) {
			// They haven't got too many login sessions running at present
			return;
		}

		if (allowedSessions == -1) {
			// We permit unlimited logins
			return;
		}

		if (sessionCount == allowedSessions) {
			HttpSession session = request.getSession(false);

			if (session != null) {
				// Only permit it though if this request is associated with one of the
				// already registered sessions
				for (SessionInformation si : sessions) {
					if (si.getSessionId().equals(session.getId())) {
						return;
					}
				}
			}
			// If the session is null, a new one will be created by the parent class,
			// exceeding the allowed number
		}

		allowableSessionsExceeded(sessions, allowedSessions, sessionRegistry);
	}
	protected void allowableSessionsExceeded(List<SessionInformation> sessions,
			int allowableSessions, SessionRegistry registry)
			throws SessionAuthenticationException {
		if (exceptionIfMaximumExceeded || (sessions == null)) {
			throw new SessionAuthenticationException(messages.getMessage(
					"ConcurrentSessionControlAuthenticationStrategy.exceededAllowed",
					new Object[] {allowableSessions},
					"Maximum sessions of {0} for this principal exceeded"));
		}

		// Determine least recently used sessions, and mark them for invalidation
		sessions.sort(Comparator.comparing(SessionInformation::getLastRequest));
		int maximumSessionsExceededBy = sessions.size() - allowableSessions + 1;
		List<SessionInformation> sessionsToBeExpired = sessions.subList(0, maximumSessionsExceededBy);
		for (SessionInformation session: sessionsToBeExpired) {
			session.expireNow();
		}
	}
}
```

这段核心代码我来给大家稍微解释下：

1. 首先调用 sessionRegistry.getAllSessions 方法获取当前用户的所有 session，该方法在调用时，传递两个参数，一个是当前用户的 authentication，另一个参数 false 表示不包含已经过期的 session（在用户登录成功后，会将用户的 sessionid 存起来，其中 key 是用户的主体（principal），value 则是该主题对应的 sessionid 组成的一个集合）。
2. 接下来计算出当前用户已经有几个有效 session 了，同时获取允许的 session 并发数。
3. 如果当前 session 数（sessionCount）小于 session 并发数（allowedSessions），则不做任何处理；如果 allowedSessions 的值为 -1，表示对 session 数量不做任何限制。
4. 如果当前 session 数（sessionCount）等于 session 并发数（allowedSessions），那就先看看当前 session 是否不为 null，并且已经存在于 sessions 中了，如果已经存在了，那都是自家人，不做任何处理；如果当前 session 为 null，那么意味着将有一个新的 session 被创建出来，届时当前 session 数（sessionCount）就会超过 session 并发数（allowedSessions）。
5. 如果前面的代码中都没能 return 掉，那么将进入策略判断方法 allowableSessionsExceeded 中。
6. allowableSessionsExceeded 方法中，首先会有 exceptionIfMaximumExceeded 属性，这就是我们在 SecurityConfig 中配置的 maxSessionsPreventsLogin 的值，默认为 false，如果为 true，就直接抛出异常，那么这次登录就失败了（对应 2.2 小节的效果），如果为 false，则对 sessions 按照请求时间进行排序，然后再使多余的 session 过期即可（对应 2.1 小节的效果）。

## 4.小结

如此，两行简单的配置就实现了 Spring Security 中 session 的并发管理。是不是很简单？不过这里还有一个小小的坑，松哥将在下篇文章中继续和大家分析。

本文案例大家可以从 GitHub 上下载：https://github.com/lenve/spring-security-samples

# Spring Boot + Vue 前后端分离项目，如何踢掉已登录用户？

但是有一个不太完美的地方，就是我们的用户是配置在内存中的用户，我们没有将用户放到数据库中去。正常情况下，松哥在 Spring Security 系列中讲的其他配置，大家只需要参考[Spring Security+Spring Data Jpa 强强联手，安全管理只有更简单！](https://mp.weixin.qq.com/s/VWJvINbi1DB3fF-Mcx7mGg)一文，将数据切换为数据库中的数据即可。

但是，在做 Spring Security 的 session 并发处理时，直接将内存中的用户切换为数据库中的用户会有问题，今天我们就来说说这个问题，顺便把这个功能应用到微人事中（https://github.com/lenve/vhr）。

本文的案例将基于[Spring Security+Spring Data Jpa 强强联手，安全管理只有更简单！](https://mp.weixin.qq.com/s/VWJvINbi1DB3fF-Mcx7mGg)一文来构建，所以重复的代码我就不写了，小伙伴们要是不熟悉可以参考该篇文章。

## 1.环境准备

首先，我们打开[Spring Security+Spring Data Jpa 强强联手，安全管理只有更简单！](https://mp.weixin.qq.com/s/VWJvINbi1DB3fF-Mcx7mGg)一文中的案例，这个案例结合 Spring Data Jpa 将用户数据存储到数据库中去了。

然后我们将上篇文章中涉及到的登录页面拷贝到项目中（文末可以下载完整案例）：

[![img](http://img.itboyhub.com/2020/04/20200506204420.png)](http://img.itboyhub.com/2020/04/20200506204420.png)

并在 SecurityConfig 中对登录页面稍作配置：

```
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/js/**", "/css/**", "/images/**");
}
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            ...
            .and()
            .formLogin()
            .loginPage("/login.html")
            .loginProcessingUrl("/doLogin")
            ...
            .and()
            .sessionManagement()
            .maximumSessions(1);
}
```

这里都是常规配置，我就不再多说。注意最后面我们将 session 数量设置为 1。

好了，配置完成后，我们启动项目，并行性多端登录测试。

打开多个浏览器，分别进行多端登录测试，我们惊讶的发现，每个浏览器都能登录成功，每次登录成功也不会踢掉已经登录的用户！

这是怎么回事？

## 2.问题分析

要搞清楚这个问题，我们就要先搞明白 Spring Security 是怎么保存用户对象和 session 的。

Spring Security 中通过 SessionRegistryImpl 类来实现对会话信息的统一管理，我们来看下这个类的源码（部分）：

```
public class SessionRegistryImpl implements SessionRegistry,
		ApplicationListener<SessionDestroyedEvent> {
	/** <principal:Object,SessionIdSet> */
	private final ConcurrentMap<Object, Set<String>> principals;
	/** <sessionId:Object,SessionInformation> */
	private final Map<String, SessionInformation> sessionIds;
	public void registerNewSession(String sessionId, Object principal) {
		if (getSessionInformation(sessionId) != null) {
			removeSessionInformation(sessionId);
		}
		sessionIds.put(sessionId,
				new SessionInformation(principal, sessionId, new Date()));

		principals.compute(principal, (key, sessionsUsedByPrincipal) -> {
			if (sessionsUsedByPrincipal == null) {
				sessionsUsedByPrincipal = new CopyOnWriteArraySet<>();
			}
			sessionsUsedByPrincipal.add(sessionId);
			return sessionsUsedByPrincipal;
		});
	}
	public void removeSessionInformation(String sessionId) {
		SessionInformation info = getSessionInformation(sessionId);
		if (info == null) {
			return;
		}
		sessionIds.remove(sessionId);
		principals.computeIfPresent(info.getPrincipal(), (key, sessionsUsedByPrincipal) -> {
			sessionsUsedByPrincipal.remove(sessionId);
			if (sessionsUsedByPrincipal.isEmpty()) {
				sessionsUsedByPrincipal = null;
			}
			return sessionsUsedByPrincipal;
		});
	}

}
```

这个类的源码还是比较长，我这里提取出来一些比较关键的部分：

1. 首先大家看到，一上来声明了一个 principals 对象，这是一个支持并发访问的 map 集合，集合的 key 就是用户的主体（principal），正常来说，用户的 principal 其实就是用户对象，松哥在之前的文章中也和大家讲过 principal 是怎么样存入到 Authentication 中的（参见：[松哥手把手带你捋一遍 Spring Security 登录流程](https://mp.weixin.qq.com/s/z6GeR5O-vBzY3SHehmccVA)），而集合的 value 则是一个 set 集合，这个 set 集合中保存了这个用户对应的 sessionid。
2. 如有新的 session 需要添加，就在 registerNewSession 方法中进行添加，具体是调用 principals.compute 方法进行添加，key 就是 principal。
3. 如果用户注销登录，sessionid 需要移除，相关操作在 removeSessionInformation 方法中完成，具体也是调用 principals.computeIfPresent 方法，这些关于集合的基本操作我就不再赘述了。

看到这里，大家发现一个问题，ConcurrentMap 集合的 key 是 principal 对象，用对象做 key，一定要重写 equals 方法和 hashCode 方法，否则第一次存完数据，下次就找不到了，这是 JavaSE 方面的知识，我就不用多说了。

如果我们使用了基于内存的用户，我们来看下 Spring Security 中的定义：

```
public class User implements UserDetails, CredentialsContainer {
	private String password;
	private final String username;
	private final Set<GrantedAuthority> authorities;
	private final boolean accountNonExpired;
	private final boolean accountNonLocked;
	private final boolean credentialsNonExpired;
	private final boolean enabled;
	@Override
	public boolean equals(Object rhs) {
		if (rhs instanceof User) {
			return username.equals(((User) rhs).username);
		}
		return false;
	}
	@Override
	public int hashCode() {
		return username.hashCode();
	}
}
```

可以看到，他自己实际上是重写了 equals 和 hashCode 方法了。

所以我们使用基于内存的用户时没有问题，而我们使用自定义的用户就有问题了。

找到了问题所在，那么解决问题就很容易了，重写 User 类的 equals 方法和 hashCode 方法即可：

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
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(username, user.username);
    }

    @Override
    public int hashCode() {
        return Objects.hash(username);
    }
    ...
    ...
}
```

配置完成后，重启项目，再去进行多端登录测试，发现就可以成功踢掉已经登录的用户了。

如果你使用了 MyBatis 而不是 Jpa，也是一样的处理方案，只需要重写登录用户的 equals 方法和 hashCode 方法即可。

## 3.微人事应用

### 3.1 存在的问题

由于微人事目前是采用了 JSON 格式登录，所以如果项目控制 session 并发数，就会有一些额外的问题要处理。

最大的问题在于我们用自定义的过滤器代替了 UsernamePasswordAuthenticationFilter，进而导致前面所讲的关于 session 的配置，统统失效。所有相关的配置我们都要在新的过滤器 LoginFilter 中进行配置 ，包括 SessionAuthenticationStrategy 也需要我们自己手动配置了。

这虽然带来了一些工作量，但是做完之后，相信大家对于 Spring Security 的理解又会更上一层楼。

### 3.2 具体应用

我们来看下具体怎么实现，我这里主要列出来一些关键代码，**完整代码大家可以从 GitHub 上下载**：https://github.com/lenve/vhr。

首先第一步，我们重写 Hr 类的 equals 和 hashCode 方法，如下：

```
public class Hr implements UserDetails {
    ...
    ...
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Hr hr = (Hr) o;
        return Objects.equals(username, hr.username);
    }

    @Override
    public int hashCode() {
        return Objects.hash(username);
    }
    ...
    ...
}
```

接下来在 SecurityConfig 中进行配置。

这里我们要自己提供 SessionAuthenticationStrategy，而前面处理 session 并发的是 ConcurrentSessionControlAuthenticationStrategy，也就是说，我们需要自己提供一个 ConcurrentSessionControlAuthenticationStrategy 的实例，然后配置给 LoginFilter，但是在创建 ConcurrentSessionControlAuthenticationStrategy 实例的过程中，还需要有一个 SessionRegistryImpl 对象。

前面我们说过，SessionRegistryImpl 对象是用来维护会话信息的，现在这个东西也要我们自己来提供，SessionRegistryImpl 实例很好创建，如下：

```
@Bean
SessionRegistryImpl sessionRegistry() {
    return new SessionRegistryImpl();
}
```

然后在 LoginFilter 中配置 SessionAuthenticationStrategy，如下：

```
@Bean
LoginFilter loginFilter() throws Exception {
    LoginFilter loginFilter = new LoginFilter();
    loginFilter.setAuthenticationSuccessHandler((request, response, authentication) -> {
                //省略
            }
    );
    loginFilter.setAuthenticationFailureHandler((request, response, exception) -> {
                //省略
            }
    );
    loginFilter.setAuthenticationManager(authenticationManagerBean());
    loginFilter.setFilterProcessesUrl("/doLogin");
    ConcurrentSessionControlAuthenticationStrategy sessionStrategy = new ConcurrentSessionControlAuthenticationStrategy(sessionRegistry());
    sessionStrategy.setMaximumSessions(1);
    loginFilter.setSessionAuthenticationStrategy(sessionStrategy);
    return loginFilter;
}
```

我们在这里自己手动构建 ConcurrentSessionControlAuthenticationStrategy 实例，构建时传递 SessionRegistryImpl 参数，然后设置 session 的并发数为 1，最后再将 sessionStrategy 配置给 LoginFilter。

> 其实[上篇文章](https://mp.weixin.qq.com/s/9f2e4Ua2_fxEd-S9Y7DDtA)中，我们的配置方案，最终也是像上面这样，只不过现在我们自己把这个写出来了而已。

这就配置完了吗？没有！session 处理还有一个关键的过滤器叫做 ConcurrentSessionFilter，本来这个过滤器是不需要我们管的，但是这个过滤器中也用到了 SessionRegistryImpl，而 SessionRegistryImpl 现在是由我们自己来定义的，所以，该过滤器我们也要重新配置一下，如下：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            ...
    http.addFilterAt(new ConcurrentSessionFilter(sessionRegistry(), event -> {
        HttpServletResponse resp = event.getResponse();
        resp.setContentType("application/json;charset=utf-8");
        resp.setStatus(401);
        PrintWriter out = resp.getWriter();
        out.write(new ObjectMapper().writeValueAsString(RespBean.error("您已在另一台设备登录，本次登录已下线!")));
        out.flush();
        out.close();
    }), ConcurrentSessionFilter.class);
    http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
}
```

在这里，我们重新创建一个 ConcurrentSessionFilter 的实例，代替系统默认的即可。在创建新的 ConcurrentSessionFilter 实例时，需要两个参数：

1. sessionRegistry 就是我们前面提供的 SessionRegistryImpl 实例。
2. 第二个参数，是一个处理 session 过期后的回调函数，也就是说，当用户被另外一个登录踢下线之后，你要给什么样的下线提示，就在这里来完成。

最后，我们还需要在处理完登录数据之后，手动向 SessionRegistryImpl 中添加一条记录：

```
public class LoginFilter extends UsernamePasswordAuthenticationFilter {
    @Autowired
    SessionRegistry sessionRegistry;
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        //省略
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
                    username, password);
            setDetails(request, authRequest);
            Hr principal = new Hr();
            principal.setUsername(username);
            sessionRegistry.registerNewSession(request.getSession(true).getId(), principal);
            return this.getAuthenticationManager().authenticate(authRequest);
        } 
        ...
        ...
    }
}
```

在这里，我们手动调用 sessionRegistry.registerNewSession 方法，向 SessionRegistryImpl 中添加一条 session 记录。

OK，如此之后，我们的项目就配置完成了。

接下来，重启 vhr 项目，进行多端登录测试，如果自己被人踢下线了，就会看到如下提示：

[![img](http://img.itboyhub.com/2020/04/20200507231051.png)](http://img.itboyhub.com/2020/04/20200507231051.png)

完整的代码，我已经更新到 vhr 上了，大家可以下载学习。

如果小伙伴们对松哥录制的 vhr 项目视频感兴趣，不妨看看这里：[微人事项目视频教程](https://mp.weixin.qq.com/s/1k4CZ6_re11fQM_6_00jCw)

## 4.小结

好了，本文主要和小伙伴们介绍了一个在 Spring Security 中处理 session 并发问题时，可能遇到的一个坑，以及在前后端分离情况下，如何处理 session 并发问题。不知道小伙伴们有没有 GET 到呢？

本文第二小节的案例大家可以从 GitHub 上下载：https://github.com/lenve/spring-security-samples

# Spring Security 自带防火墙！

之前有小伙伴表示，看 Spring Security 这么麻烦，不如自己写一个 Filter 拦截请求，简单实用。



自己写当然也可以实现，但是大部分情况下，大家都不是专业的 Web 安全工程师，所以考虑问题也不过就是认证和授权，这两个问题处理好了，似乎系统就很安全了。

其实不是这样的！

各种各样的 Web 攻击每天都在发生，什么固定会话攻击、csrf 攻击等等，如果不了解这些攻击，那么做出来的系统肯定也不能防御这些攻击。

使用 Spring Security 的好处就是，即使不了解这些攻击，也不用担心这些攻击，因为 Spring Security 已经帮你做好防御工作了。

我们常说相比于 Shiro，Spring Security 更加重量级，重量级有重量级的好处，比如功能全，安全管理更加完备。用了 Spring Security，你都不知道自己的系统有多安全！

今天我就来和大家聊一聊 Spring Security 中自带的防火墙机制。



## 1.HttpFirewall

在 Spring Security 中提供了一个 HttpFirewall，看名字就知道这是一个请求防火墙，它可以自动处理掉一些非法请求。

HttpFirewall 目前一共有两个实现类：

[![img](http://img.itboyhub.com/2020/05/HttpFirewall.png)](http://img.itboyhub.com/2020/05/HttpFirewall.png)

一个是严格模式的防火墙设置，还有一个默认防火墙设置。

DefaultHttpFirewall 的限制相对于 StrictHttpFirewall 要宽松一些，当然也意味着安全性不如 StrictHttpFirewall。

Spring Security 中默认使用的是 StrictHttpFirewall。

## 2.防护措施

那么 StrictHttpFirewall 都是从哪些方面来保护我们的应用呢？我们来挨个看下。

### 2.1 只允许白名单中的方法

首先，对于请求的方法，只允许白名单中的方法，也就是说，不是所有的 HTTP 请求方法都可以执行。

这点我们可以从 StrictHttpFirewall 的源码中看出来：

```
public class StrictHttpFirewall implements HttpFirewall {
	private Set<String> allowedHttpMethods = createDefaultAllowedHttpMethods();
	private static Set<String> createDefaultAllowedHttpMethods() {
		Set<String> result = new HashSet<>();
		result.add(HttpMethod.DELETE.name());
		result.add(HttpMethod.GET.name());
		result.add(HttpMethod.HEAD.name());
		result.add(HttpMethod.OPTIONS.name());
		result.add(HttpMethod.PATCH.name());
		result.add(HttpMethod.POST.name());
		result.add(HttpMethod.PUT.name());
		return result;
	}
	private void rejectForbiddenHttpMethod(HttpServletRequest request) {
		if (this.allowedHttpMethods == ALLOW_ANY_HTTP_METHOD) {
			return;
		}
		if (!this.allowedHttpMethods.contains(request.getMethod())) {
			throw new RequestRejectedException("The request was rejected because the HTTP method \"" +
					request.getMethod() +
					"\" was not included within the whitelist " +
					this.allowedHttpMethods);
		}
	}
}
```

从这段代码中我们看出来，你的 HTTP 请求方法必须是 DELETE、GET、HEAD、OPTIONS、PATCH、POST 以及 PUT 中的一个，请求才能发送成功，否则的话，就会抛出 RequestRejectedException 异常。

那如果你想发送其他 HTTP 请求方法，例如 TRACE ，该怎么办呢？我们只需要自己重新提供一个 StrictHttpFirewall 实例即可，如下：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setUnsafeAllowAnyHttpMethod(true);
    return firewall;
}
```

其中，setUnsafeAllowAnyHttpMethod 方法表示不做 HTTP 请求方法校验，也就是什么方法都可以过。或者也可以通过 setAllowedHttpMethods 方法来重新定义可以通过的方法。

### 2.2 请求地址不能有分号

不知掉大家有没有试过，如果你使用了 Spring Security，请求地址是不能有 `;` 的，如果请求地址有 `;` ，就会自动跳转到如下页面：

[![img](http://img.itboyhub.com/2020/05/20200511152104.png)](http://img.itboyhub.com/2020/05/20200511152104.png)

可以看到，页面的提示中已经说了，因为你的请求地址中包含 `;`，所以请求失败。

什么时候请求地址中会包含 `;` 呢？不知道小伙伴们在使用 Shiro 的时候，有没有注意到，如果你禁用了 Cookie，那么 jsessionid 就会出现在地址栏里，像下面这样：

```
http://localhost:8080/hello;jsessionid=xx
```

这种传递 jsessionid 的方式实际上是非常不安全的（松哥后面的文章会和大家细聊这个问题），所以在 Spring Security 中，这种传参方式默认就禁用了。

当然，如果你希望地址栏能够被允许出现 `;` ，那么可以按照如下方式设置：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowSemicolon(true);
    return firewall;
}
```

设置完成之后，再去访问相同的接口，可以看到，此时虽然还是报错，但是错误是 404 了，而不是一开始那个不允许 `;` 的错了。

[![img](http://img.itboyhub.com/2020/05/20200511153022.png)](http://img.itboyhub.com/2020/05/20200511153022.png)

**注意，在 URL 地址中，`;` 编码之后是 `%3b` 或者 `%3B`，所以地址中同样不能出现 `%3b` 或者 `%3B`**

#### 题外话

有的小伙伴可能不知道或者没用过，Spring3.2 开始，带来了一种全新的传参方式 @MatrixVariable。

@MatrixVariable 是 Spring3.2 中带来的功能，这种方式拓展了请求参数的传递格式，使得参数之间可以用 `;` 隔开，这种传参方式真是哪壶不开提哪壶。因为 Spring Security 默认就是禁止这种传参方式，所以一般情况下，如果你需要使用 @MatrixVariable 来标记参数，就得在 Spring Security 中额外放行。

接下来我通过一个简单的例子来和大家演示一下 @MatrixVariable 的用法。

我们新建一个 `/hello` 方法：

```
@RequestMapping(value = "/hello/{id}")
public void hello(@PathVariable Integer id,@MatrixVariable String name) {
    System.out.println("id = " + id);
    System.out.println("name = " + name);
}
```

另外我们还需要配置一下 SpringMVC，使 `;` 不要被自动移除了：

```
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    @Override
    protected void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

然后启动项目(注意，Spring Security 中也已经配置了允许 URL 中存在 `;`)，浏览器发送如下请求：

```
http://localhost:8080/hello/123;name=javaboy
```

控制台打印信息如下：

```
id = 123
name = javaboy
```

可以看到，@MatrixVariable 注解已经生效了。

### 2.3 必须是标准化 URL

请求地址必须是标准化 URL。

什么是标准化 URL？标准化 URL 主要从四个方面来判断，我们来看下源码：

StrictHttpFirewall#isNormalized：

```
private static boolean isNormalized(HttpServletRequest request) {
	if (!isNormalized(request.getRequestURI())) {
		return false;
	}
	if (!isNormalized(request.getContextPath())) {
		return false;
	}
	if (!isNormalized(request.getServletPath())) {
		return false;
	}
	if (!isNormalized(request.getPathInfo())) {
		return false;
	}
	return true;
}
```

getRequestURI 就是获取请求协议之外的字符；getContextPath 是获取上下文路径，相当于是 project 的名字；getServletPath 这个就是请求的 servlet 路径，getPathInfo 则是除过 contextPath 和 servletPath 之后剩余的部分。

这四种路径中，都不能包含如下字符串：

```
"./", "/../" or "/."
```

### 2.4 必须是可打印的 ASCII 字符

如果请求地址中包含不可打印的 ASCII 字符，请求则会被拒绝，我们可以从源码中看出端倪：

StrictHttpFirewall#containsOnlyPrintableAsciiCharacters

```
private static boolean containsOnlyPrintableAsciiCharacters(String uri) {
	int length = uri.length();
	for (int i = 0; i < length; i++) {
		char c = uri.charAt(i);
		if (c < '\u0020' || c > '\u007e') {
			return false;
		}
	}
	return true;
}
```

### 2.5 双斜杠不被允许

如果请求地址中出现双斜杠，这个请求也将被拒绝。双斜杠 `//` 使用 URL 地址编码之后，是 %2F%2F，其中 F 大小写无所谓，所以请求地址中也能不出现 “%2f%2f”, “%2f%2F”, “%2F%2f”, “%2F%2F”。

如果你希望请求地址中可以出现 `//` ，可以按照如下方式配置：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowUrlEncodedDoubleSlash(true);
    return firewall;
}
```

### 2.6 % 不被允许

如果请求地址中出现 %，这个请求也将被拒绝。URL 编码后的 % 是 %25，所以 %25 也不能出现在 URL 地址中。

如果希望请求地址中可以出现 %，可以按照如下方式修改：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowUrlEncodedPercent(true);
    return firewall;
}
```

### 2.7 正反斜杠不被允许

如果请求地址中包含斜杠编码后的字符 %2F 或者 %2f ，则请求将被拒绝。

如果请求地址中包含反斜杠 \ 或者反斜杠编码后的字符 %5C 或者 %5c ，则请求将被拒绝。

如果希望去掉如上两条限制，可以按照如下方式来配置：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowBackSlash(true);
    firewall.setAllowUrlEncodedSlash(true);
    return firewall;
}
```

### 2.8 `.` 不被允许

如果请求地址中存在 `.` 编码之后的字符 `%2e`、`%2E`，则请求将被拒绝。

如需支持，按照如下方式进行配置：

```
@Bean
HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowUrlEncodedPeriod(true);
    return firewall;
}
```

### 2.9 小结

需要强调一点，上面所说的这些限制，都是针对请求的 requestURI 进行的限制，而不是针对请求参数。例如你的请求格式是：

```
http://localhost:8080/hello?param=aa%2ebb
```

那么 2.7 小节说的限制和你没关系。

这个大家从 StrictHttpFirewall 源码中很容易看到：

```
public class StrictHttpFirewall implements HttpFirewall {
	@Override
	public FirewalledRequest getFirewalledRequest(HttpServletRequest request) throws RequestRejectedException {
		rejectForbiddenHttpMethod(request);
		rejectedBlacklistedUrls(request);
		rejectedUntrustedHosts(request);

		if (!isNormalized(request)) {
			throw new RequestRejectedException("The request was rejected because the URL was not normalized.");
		}

		String requestUri = request.getRequestURI();
		if (!containsOnlyPrintableAsciiCharacters(requestUri)) {
			throw new RequestRejectedException("The requestURI was rejected because it can only contain printable ASCII characters.");
		}
		return new FirewalledRequest(request) {
			@Override
			public void reset() {
			}
		};
	}
	private void rejectedBlacklistedUrls(HttpServletRequest request) {
		for (String forbidden : this.encodedUrlBlacklist) {
			if (encodedUrlContains(request, forbidden)) {
				throw new RequestRejectedException("The request was rejected because the URL contained a potentially malicious String \"" + forbidden + "\"");
			}
		}
		for (String forbidden : this.decodedUrlBlacklist) {
			if (decodedUrlContains(request, forbidden)) {
				throw new RequestRejectedException("The request was rejected because the URL contained a potentially malicious String \"" + forbidden + "\"");
			}
		}
	}
	private static boolean encodedUrlContains(HttpServletRequest request, String value) {
		if (valueContains(request.getContextPath(), value)) {
			return true;
		}
		return valueContains(request.getRequestURI(), value);
	}

	private static boolean decodedUrlContains(HttpServletRequest request, String value) {
		if (valueContains(request.getServletPath(), value)) {
			return true;
		}
		if (valueContains(request.getPathInfo(), value)) {
			return true;
		}
		return false;
	}
	private static boolean valueContains(String value, String contains) {
		return value != null && value.contains(contains);
	}
}
```

rejectedBlacklistedUrls 方法就是校验 URL 的，该方法逻辑很简单，我就不再赘述了。

**注意：虽然我们可以手动修改 Spring Security 中的这些限制，但是松哥不建议大家做任何修改，每一条限制都有它的原由，每放开一个限制，就会带来未知的安全风险。后面松哥在和大家分享 Web 中的安全攻击时，也会再次提到这些限制的作用，请小伙伴们保持关注哦。**

## 3.总结

没想到吧？Spring Security 竟然为你做了这么多事情！正好应了那句鸡汤：

> 你所谓的岁月静好,不过是有人在替你负重前行。

# 1