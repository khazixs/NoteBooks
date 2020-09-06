## 权限控制和登录流程

认证–>授权–>鉴权–>权限控制，这是权限控制的流程

认证是解决我是我的过程；授权是对我在这个环境中所扮演的角色所拥有的的权利进行授予；鉴权是对授权真实性的判断，**对授权所颁发授信媒介进行解析，确认其真实性**；之后就是权限控制即只允许目标在其角色权限范围内对资源进行操作。

- **认证是确认声明者的本身身份，其作为授权的上游衔接而存在**
- **鉴权是对声明者所声明的真实性进行确认的过程，其作为授权的下游衔接而存在**



重要的几个类：

 `UserDetailsServiceAutoConfiguration`——SpringBoot下的SpringSecurity自动配置类，会根据用户配置的spring.security.user属性（未配置该属性就自动创建一个名为user，密码为随机的用户）生成一个内存级别的对象`InMemoryUserDetailsManager`该对象用于提供认证的用户信息。



可以通过创建配置类继承于`WebSecurityConfigurerAdapter`实现自定义配置SpringSecurity，主要需要重写以下几个方法来实现权限控制：



1. `configure(AuthenticationManagerBuilder auth)`实现AuthenticationManage认证管理器，这里主要功能是进行基于内存的用户认证

   ```java
   // SecurityConfig.java
   @Overrideprotected void configure(AuthenticationManagerBuilder auth) throws Exception {    auth.            
               // <X> 使用内存中的 InMemoryUserDetailsManager            
               InMemoryAuthentication()       
               // <Y> 不使用 PasswordEncoder 密码编码器  
       		.passwordEncoder(NoOpPasswordEncoder.getInstance())            
       		// <Z> 配置 admin 用户      
       		.withUser("admin").password("admin").roles("ADMIN")  
       		// <Z> 配置 normal 用户
       		.and().withUser("normal").password("normal").roles("NORMAL");}
   ```

2. `configure(HttpSecurity http)`这里主要是配置URL的权限控制。实际代码如图所示：

   ```java
   protected void configure(HttpSecurity http) throws Exception {
       http.csrf().disable().exceptionHandling()
           
          	.authenticationEntryPoint(
                       (request, response, authException) -> response.sendError(HttpServletResponse.SC_UNAUTHORIZED))
           
          	.and()
           	.authorizeRequests()
           	// 下面是放开权限的url
               .antMatchers(PermitAllUrl.permitAllUrl("/api/anon/**", "/index","/api/user/current", "/api/**/alipay/**")).permitAll() 
           	//除了放开权限的其他任何URL需要登录才能访问
               .anyRequest().authenticated() 
           
           //下面是使用表单登录的配置
           .and()
           	.formLogin().loginProcessingUrl("/api/login")
           	//自定义登出界面
           	.loginPage("/user/login")
           	//成功登录的处理器
           	.successHandler((request, response, authentication) -> {
                   RestResponse res = RestResponse.ok(authentication.getPrincipal());
                   response.getWriter().write(JSON.toJSONString(res));
                   response.setContentType("text/json");
                   response.setCharacterEncoding("utf-8");
               })
           	//登录失败的处理器
           	.failureHandler((request, response, exception) -> {
                   RestResponse res = RestResponse.fail(exception.getMessage());
                   response.getWriter().write(JSON.toJSONString(res));
                   response.setContentType("text/json");
                   response.setCharacterEncoding("utf-8");
               })
           
          //配置登出
          	.and()
           	.logout().logoutUrl("/api/logout")
   		//配置免登录
           .and()
           	.rememberMe()
           	.rememberMeParameter("rememberMe")
           	.tokenValiditySeconds(1209600).tokenRepository(tokenRepository())
           
            .and()
           	.httpBasic();
   }
   ```



### 原理及流程

#### “记住我”的原理

![img](https://img2018.cnblogs.com/blog/647585/201811/647585-20181109110908290-328952476.png)

#### SpringSecurity实现原理

![640?wx_fmt=png](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYngjiaicZ6UqibkEHRhzUg8JYLz2G6ILGaaibJe3fOl7LSDyBmHFJy3wdJqmVKdYKUYxQALGibSef9QJRA/640?wx_fmt=png)

Spring Security使用了责任链设计模式，这个模块是基于拦截器实现的。



## Spring Security + Json Web Token + Spring Boot的登录和权限控制方案

### jwt

所谓jwt就是将用户的身份信息（账号名字）、其他信息（不固定，根据需要增加）在用户登陆时提取出来，并且通过加密手段加工成一串密文，在用户登陆成功时带在返回结果发送给用户。以后用户每次请求时均带上这串密文，服务器根据解析这段密文判断用户是否有权限访问相关资源，并返回相应结果。

jwt的优点：

1. 相比于session，它无需保存在服务器，**不占用服务器内存开销**。
2. **无状态、可拓展性强**：比如有3台机器（A、B、C）组成服务器集群，若session存在机器A上，session只能保存在其中一台服务器，此时你便不能访问机器B、C，因为B、C上没有存放该Session，而使用token就能够验证用户请求合法性，并且我再加几台机器也没事，所以可拓展性好就是这个意思。
3. 由 2 知，这样做可就支持了跨域访问。

### 本次技术方案中 Spring Security 执行流程

![本次技术方案中 Spring Security 执行流程](https://camo.githubusercontent.com/c5bd6b9b3be3c19caa2c3d2c08de9aadc94c7f3d/687474703a2f2f7765616e2d626c6f672e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f254536253943254143254536254143254131254536253841253830254536253943254146254536253936254239254536254131253838254534254238254144253230537072696e6725323053656375726974792532302545362538392541372545382541312538432545362542352538312545372541382538422e706e67)

这里我使用配置类的方式配置SpringSecurity，新建配置类继承于WebSecurityConfigurerAdapter,并加上@Configuration @EnableWebSecurity,注解声明配置类和启用Spring Security web安全的功能，在新建类中覆写configure方法，以HttpSecurity 作为参数，在其中通过sessionManagement()和sessionCreationPolicy()方法定制自己的session策略，传入SessionCreationPolicy.STATELESS策略，让SpringSecurity不创建和使用session。

然后引入一个jwt的轮子，创建自己的token操作类，把用户名和当前时间封装在token的主题claims中，时间用来判断当前token是否是在修改密码之前生成的。再计算token过期的时间写入到token中，对轮子token进行撒盐加密，生成一串字符串，即我们定制的 token。