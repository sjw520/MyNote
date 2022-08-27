# Spring Security

## 1 权限管理

权限管理包括用户身份认证和授权两部分，简称认证授权，对于需要访问控制的资源用户首先经过身份认证，认证通过后用户具有该资源的访问权限方可访问

### 1.1 认证

身份认证，即使判断一个用户是否为合法用户的处理过程。最长用的简单身份认证方式是系统通过对用户输入的用户名和口令，看其是否与系统存储的该用户的用户和口令一致，来判断用户身份正确。对于采用指纹等系统，则初始指纹；对于硬件key等刷卡系统，则需要刷卡。

### 1.2 授权

**授权**，即访问控制，控制谁能访问那些资源。主体进行身份认证后需要分配权限方可访问系统的资源，对于某些资源没有权限是无法访问的。

## 2 整体架构

在的架构设计中，**认证**和**授权**是分开的，无论使用什么样的认证方式。都不会影响授权，这是两个独立的存在，这种独立带来的好处之一，就是可以非常方便地整合一些外部的解决方案。

### 2.1 认证

#### 2.1.1 AuthenicationManager

* 在Spring Security中认证是由AuthenicationManager接口来负责的

  ```java
  public interface AuthenticationManager {
      Authentication authenticate(Authentication var1) throws AuthenticationException;
  }
  ```

  
  * 返回Authentication表示认证成功
  * 返回AuthenticationException异常，表示认证失败

AuthenticationManager主要实现类为ProviderManager，在ProviderManager中管理了众多AuthenticationProvider 实例。在一次完整的认证流程中，Spring security允许存在多个AuthenticationProvider，用来实现多种认证方式，这些AuthenticationProvider 都是由ProviderManager进行统一管理的。 

#### 2.1.2 Authentication

* Authentication
  * 认证以及认证成功的信息主要是由Authentication的实现类进行保存的
    * getAuthorities 获取用户权限信息
    * getCredentials 获取用户凭证信息，一般指密码
    * getDetails 获取用户详细信息
    * getPrincipal 获取用户身份信息，用户名、用户对象等
    * isAuthenticated 用户是否认证成功

#### 2.1.3 SecurityContextHolder

* SecurityContextHolder
  * SecurityContextHolder 用来获取登录之后用户信息。Spring Security 会将登录用户数据保存在 Session 中。但是，为了使用方便,Spring Security在此基础上还做了一些改进，其中最主要的一个变化就是线程绑定。当用户登录成功后,Spring Security 会将登录成功的用户信息保存到 SecurityContextHolder 中。SecurityContextHolder 中的数据保存默认是通过ThreadLocal 来实现的，使用 ThreadLocal 创建的变量只能被当前线程访问，不能被其他线程访问和修改，也就是用户数据和请求线程绑定在一起。当登录请求处理完毕后，Spring Security 会将 SecurityContextHolder 中的数据拿出来保存到 Session 中，同时将 SecurityContexHolder 中的数据清空。以后每当有请求到来时，Spring Security 就会先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中，方便在该请求的后续处理过程中使用，同时在请求结束时将 SecurityContextHolder 中的数据拿出来保存到 Session 中，然后将 Security SecurityContextHolder 中的数据清空。这一策略非常方便用户在 Controller、Service 层以及任何代码中获取当前登录用户数据。

### 2.2 授权

​	当完成认证后接下来就是授权了。在Spring Security的授权体系中，有两个关键接口

#### 2.2.1 AccessDecisionManager

> 访问决策管理器，用来决定此次访问是否被允许

#### 2.2.2 AccessDecisionVoter

> 访问决定投票器，投票器会检查用户是否具备应有的角色，进而投出赞成，反对或者弃权票

AccesDecisionVoter 和 AccessDecisionManager 都有众多的实现类，在 AccessDecisionManager 中会换个遍历 AccessDecisionVoter，进而决定是否允许用户访问，因而 AaccesDecisionVoter 和 AccessDecisionManager 两者的关系类似于 AuthenticationProvider 和 ProviderManager 的关系。

#### 2.2.3 ConfigAttribute

> 用来保存授权时的角色信息

在 Spring Security 中，用户请求一个资源(通常是一个接口或者一个 Java 方法)需要的角色会被封装成一个 ConfigAttribute 对象，在 ConfigAttribute 中只有一个 getAttribute方法，该方法返回一个 String 字符串，就是角色的名称。一般来说，角色名称都带有一个 `ROLE_` 前缀，投票器 AccessDecisionVoter 所做的事情，其实就是比较用户所具各的角色和请求某个 资源所需的 ConfigAtuibute 之间的关系。

## 3 实现原理

### 3.1 配置相关

* 为什么引入Spring security中没有任何配置的情况下，请求会被拦截

  ```java
  @Configuration(proxyBeanMethods = false)
  @ConditionalOnDefaultWebSecurity
  @ConditionalOnWebApplication(type = Type.SERVLET)
  class SpringBootWebSecurityConfiguration {
      @Bean
      @Order(SecurityProperties.BASIC_AUTH_ORDER)
      SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) 
      throws Exception {
      //要求所有请求都要认证
      http.authorizeRequests().anyRequest().authenticated()
      .and().formLogin().and().httpBasic();
          return http.build();
      }
  }
  ```

  

## 4 基本原理

### 4.1 Spring Security本质是一个过滤链

* 从启动时可以获取到的过滤器

  ```java
  org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
  org.springframework.security.web.context.SecurityContextPersistenceFilter
  org.springframework.security.web.header.HeaderWriterFilter
  org.springframework.security.web.csrf.CsrfFilter
  org.springframework.security.web.authentication.logout.LogoutFilter
  org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
  org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter
  org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter
  org.springframework.security.web.savedrequest.RequestCacheAwareFilter
  org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter
  org.springframework.security.web.authentication.AnonymousAuthenticationFilter
  org.springframework.security.web.session.SessionManagementFilter
  org.springframework.security.web.access.ExceptionTranslationFilter
  org.springframework.security.web.access.intercept.FilterSecurityInterceptor
  ```

  

### 4.2 FilterSecurityInterceptor

* 是一个方法级的权限过滤器，基本位于过滤链的对底部



### 4.3 ExceptionTranslationFilter

* 是个异常过滤器，用来处理在认证授权过程中抛出的异常

### 4.4 UsernamePasswordAuthenticationFilter 

* 对/login的POST请求做拦截，校验表单中用户名，密码

### 4.5 过滤器如何进行加载

1. 使用SpringSecurity配置过滤器，DElegatingFilterProxy

2.  ```jav
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
           Filter delegateToUse = this.delegate;
           if (delegateToUse == null) {
               synchronized(this.delegateMonitor) {
                   delegateToUse = this.delegate;
                   if (delegateToUse == null) {
                       WebApplicationContext wac = this.findWebApplicationContext();
                       if (wac == null) {
                           throw new IllegalStateException("No WebApplicationContext found: no ContextLoaderListener or DispatcherServlet registered?");
                       }
   
                       delegateToUse = this.initDelegate(wac);
                   }
   
                   this.delegate = delegateToUse;
               }
           }
   
           this.invokeDelegate(delegateToUse, request, response, filterChain);
       }
    ```

3. ```java
       protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
           String targetBeanName = this.getTargetBeanName();
           Assert.state(targetBeanName != null, "No target bean name set");
           Filter delegate = (Filter)wac.getBean(targetBeanName, Filter.class);
           if (this.isTargetFilterLifecycle()) {
               delegate.init(this.getFilterConfig());
           }
   
           return delegate;
       }
   ```

4. FilterChainProxy

   ```java
       private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
           FirewalledRequest fwRequest = this.firewall.getFirewalledRequest((HttpServletRequest)request);
           HttpServletResponse fwResponse = this.firewall.getFirewalledResponse((HttpServletResponse)response);
           List<Filter> filters = this.getFilters((HttpServletRequest)fwRequest);
           if (filters != null && filters.size() != 0) {
               VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
               vfc.doFilter(fwRequest, fwResponse);
           } else {
               if (logger.isDebugEnabled()) {
                   logger.debug(UrlUtils.buildRequestUrl(fwRequest) + (filters == null ? " has no matching filters" : " has an empty filter list"));
               }
   
               fwRequest.reset();
               chain.doFilter(fwRequest, fwResponse);
           }
       }
   ```

   

## 5 UserDetailService接口

当什么也没有配置的时候，账号和密码是由Spring Security定义生成的。而在实际项目中账号和密码都是从数据库中查询出来的。 所以我们要通过自定义逻辑控制认证逻辑。

* 创建类继承UsernamePasswordAuthenticationFilter，重写三个方法
* 创建类实现UserDetailService，编写查询数据过程，返回User对象，这个User对象是安全框架提供对象

## 6 PasswordEncoder接口

数据加密几口，用于返回USer对象里面密码加密

## 7 设置登录的用户名和密码

### 7.1 通过配置文件

```properties
spring.security.user.name=sjw
spring.security.user.password=sjw
```



### 7.2 通过配置类

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String password = passwordEncoder.encode("123");
        auth.inMemoryAuthentication().withUser("lucy").password(password).roles("admin");
    }

    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder();
    }
}
```



### 7.3 自定义编写实现类

1. 创建配置类，设置使用哪个UserDetailService实现类

   ```java
   @Configuration
   public class SecurityConfigTest extends WebSecurityConfigurerAdapter {
   
       @Autowired
       private UserDetailsService userDetailsService;
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService).passwordEncoder(password());
       }
   
       @Bean
       PasswordEncoder password(){
           return new BCryptPasswordEncoder();
       }
   }
   ```

   

2. 编写实现类，返回User对象，User对象有用户名密码和操作权限

```java
@Service("userDetailsService")
public class MyUserDetailService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");

        return new User("marry",new BCryptPasswordEncoder().encode("123"),auths);
    }
}
```

## 8 自定义设置登录页面 不需要认证可以访问

### 8.1 在配置类中实现相关配置

```java
   @Override
    protected void configure(HttpSecurity http) throws Exception {
        //自定义自己编写的登录页面
        http.formLogin()
                .loginPage("/login.html")//登录的页面设置
                .loginProcessingUrl("/user/login")//登录访问路径 登录之后的controller
                .defaultSuccessUrl("/test/index").permitAll()//登录成功之后 跳转路径
                .and().authorizeRequests()
                .antMatchers("/","/test/hello","/user/login").permitAll()//访问这些路径不需要认证
                .anyRequest().authenticated()//所有请求都能访问
                .and().csrf().disable();//关闭csrf防护

    }
```

## 9 基于角色或权限进行访问控制

### 9.1 hasAuthority

* 如果当前的主体具有指定的权限，则放回true，否则返回false

  1. 在配置类设置钱钱访问地址有哪些权限

     ```java
      .antMatchers("/test/index").hasAuthority("admin")//当前登录用户，只有具有admin权限才可以访问路径
     ```

  2. 在UserDetailService，把返回User对象设置权限

     ```java
         @Override
         public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
             List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");//给与用户加权限
     
             return new User("marry",new BCryptPasswordEncoder().encode("123"),auths);
         }
     ```

     

### 9.2 hasAnyAuthority

如果当前的主体有任何提供的角色（给定的作为一个都逗号隔的字符串列表）的话，返回true

1. 在配置类设置钱钱访问地址有哪些权限

   ```java
   .antMatchers("/test/index").hasAnyAuthority("admin,manager")
   ```

   

2. 在UserDetailService，把返回User对象设置权限

### 9.3 hasRole

如果用户具备给定角色就允许访问，否则出现403

如果当前主体具有指定的角色，则返回true

* 底层实现

  ```java
      private static String hasRole(String role) {
          Assert.notNull(role, "role cannot be null");
          if (role.startsWith("ROLE_")) {
              throw new IllegalArgumentException("role should not start with 'ROLE_' since it is automatically inserted. Got '" + role + "'");
          } else {
              return "hasRole('ROLE_" + role + "')";
          }
      }
  ```

  

1. 在配置类设置钱钱访问地址有哪些权限

   ```java
   .antMatchers("/test/index").hasRole("sale")
   ```

   

2. 在UserDetailService，把返回User对象设置权限

   ```java
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
           List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale");//给与用户加权限
   
           return new User("marry",new BCryptPasswordEncoder().encode("123"),auths);
       }
   ```

   

### 9.4 hasAnyRole

## 10 自定义403没有权限访问页面

1. 在配置类中设置

   ```java
   http.exceptionHandling().accessDeniedPage("/unauth.html");
   ```

## 11 认证授权注解使用

### 11.1 @Secured

用户具有某个角色，可以访问方法

1. 启动类（配置类）开启注解

   `@EnableGlobalMethodSecurity(securedEnabled = true)`

2. 在controller的方法上使用注解，设置角色

   ```java
       @GetMapping("update")
       @Secured({"ROLE_admin","ROLE_manage"})
       public String update(){
           return "update";
       }
   
   ```

3. userDetailService设置用户角色

   ```java
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
           List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale");//给与用户加权限
   
           return new User("marry",new BCryptPasswordEncoder().encode("123"),auths);
       }
   ```

   

### 11.2 @PreAuthorize

注解适合进入方法前验证，可以将登录用户的roles/permissions参数传到方法中

1. 启动类上添加注解

   `@EnableGlobalMethodSecurity(securedEnabled = true,prePostEnabled = true)`

2. 在controller的方法上使用注解

   ```java
       @GetMapping("update")
       @PreAuthorize("hasAnyAuthority('admin')")
       public String update(){
           return "update";
       }
   ```

   

### 11.3 @PostAuthorize

在方法执行后在进行权限验证，适合验证有返回值的权限

1. 启动类上添加注解

   `@EnableGlobalMethodSecurity(securedEnabled = true,prePostEnabled = true)`

2. 在controller的方法上使用注解

   ```java
       @GetMapping("update")
       @PostAuthorize("hasAnyAuthority('admin')")
       public String update(){
           return "update";
       }
   ```

   

### 11.3 @PostFilter

对方法返回的数据进行过滤

### 11.4 @PreFilter

对传入方法数据进行过滤

## 12 用户注销

### 12.1 在配置类中添加退出配置

```java
http.logout().logoutUrl("login")//a标签里的退出路径
    .logoutSuccessUrl("/test/hello").permitAll();
```

