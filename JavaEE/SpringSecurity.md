# Spring Security秋招学习笔记

## 一、概述

### 1.1 说在前面

这个笔记讲述的内容非常的浅显，只涉及到一点点的使用，并不涉及底层原理。什么？为什么不看原理？...因为面试不咋问啊，时间不够啊！

### 1.2 什么是SpringSecurity

- 在web开发中，安全应该是第一位的。在设计之初就应该要考虑安全。

- 安全的框架有shiro、SpringSecurity

- 主要完成两个功能，认证、授权

- 当然过滤器、拦截器也可以做到，但是非常繁琐~


SpringSecurity是和SpringBoot齐名的项目，Spring Security是一个**功能强大且高度可定制的身份验证和访问控制框架**。它是用于保护基于Spring的应用程序的实际标准。Spring Security是一个框架，致力于为Java应用程序提供**身份验证和授权**。与所有Spring项目一样，Spring Security的真正强大之处在于可以轻松扩展以满足自定义要求。

好的，说了这么多，其实Spring Security就是干验证、授权。在说白一点就是用户的登录、注销、登录的权限、自动登录等等这些内容。当然可能、应该还有其他方面的功能吧？不管了，反正项目里只用到了这些。



## 二、Spring Security的使用

使用Spring Boot来集成，怎么导入依赖不用说了吧？

### 2.1 配置类

首先需要有一个类来继承WebSecurityConfigurerAdapter类，并重写一些方法。

这里只用到了configure方法，至于其他方法干了什么，日后有精力了再谈吧。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{}
```

### 2.2 配置用户权限

配置访问权限，也就是哪些内容在没有登录就可以访问，哪些需要登录之后访问，或者vip等级来访问，当需要权限验证时，就会跳转到登录页。然后是登录页的配置。

默认只是开启登录页的话，那么登录页面会采用一个默认的登录页，然后可以通过一些方法修改默认配置，具体使用到的方法在注释中写明了用途

```java
protected void configure(HttpSecurity http) throws Exception {
    // .denyAll();    //拒绝访问
    // .authenticated();    //需认证通过
    // .permitAll();    //无条件允许访问
    // 访问权限
    http.authorizeRequests()
            .antMatchers("/","/index").permitAll()//允许访问
            .antMatchers("/register","/login","/toLogin").permitAll()//允许访问
            .antMatchers("/*").authenticated();//需要验证

    // 没有权限进入，进入登录
    http.formLogin()
            .usernameParameter("username")//用户名的name
            .passwordParameter("password")//密码的name
            .loginPage("/toLogin")//登录的页面
            .loginProcessingUrl("/login") // 登陆表单提交请求地址
            .defaultSuccessUrl("/index"); // 设置默认登录成功后跳转的页面
}
```

### 2.3 配置用户认证

那么登录怎么去判定是否有效呢，以及用户的权限呢？使用configure的另一个重载，新版的Spring Security需要有一个密码的加密方式。

认证的方法有大概几个：直接在代码里写死、使用jdbc、或者自己重写细节

很明显第一种方式非常不合理，通常使用第二种或者第三种，实际业务中重写细节会比较多一点

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    //使用userDetailsService可以使用自定义的方式来做登录验证
    auth.userDetailsService(userService).passwordEncoder(passwordEncoder());
}
```

### 2.4 配置加密

新版的Spring Security需要有一个密码的加密方式，也可以直接new一个对象作为需要的方法的参数，不过也可以使用为一个Bean交给Spring托管

```java
@Bean
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}
```

### 2.5 注销

这个是写在2.2那个类里的，通过logout方法注销，默认路径为/logout，然后logoutSuccessUrl的值为登出后跳转的页面

```java
http.logout().logoutSuccessUrl("/");
```

### 2.6 记住我

这个也是写在2.2那个类里的，通过rememberMe方法，如果要自定义的话，使用rememberMeParameter方法来接受前端的参数。其实现原理其实还是使用cookie。

```java
http.rememberMe().rememberMeParameter("remember");
```



## 三、自定义登录细节

### 3.1 源码分析

可以看到我们使用自定义登录类的操作的方法userDetailsService，他的参数是一个UserDetailsService的子类，那么什么是UserDetailsService？

```java
public <T extends UserDetailsService> DaoAuthenticationConfigurer<AuthenticationManagerBuilder, T> userDetailsService(T userDetailsService) throws Exception {
   this.defaultUserDetailsService = userDetailsService;
   return apply(new DaoAuthenticationConfigurer<>(
         userDetailsService));
}
```

### 3.2 UserDetailsService

这个是一个接口，security包里的接口，里面只有一个方法，loadUserByUsername，因此想要实现自定义细节，那么就需要实现这个方法。

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
   UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

### 3.3 具体的实现

来源于一个开源项目，其实这里也没有做出登录的细节，只不过是给出了一些数据，账号、密码、权限之类的内容，因此真正的登录细节是在spring security中的。

```java
@Override
public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
    // 通过用户名查询用户
    User user = userService.getOne(new QueryWrapper<User>().eq("username", s));

    // 放入session
    session.setAttribute("loginUser",user);

    //创建一个新的UserDetails对象，最后验证登陆的需要
    UserDetails userDetails=null;
    if(user!=null){
        // 登录后会将登录密码进行加密，然后比对数据库中的密码，数据库密码需要加密存储！
        String password = user.getPassword();

        //创建一个集合来存放权限
        Collection<GrantedAuthority> authorities = getAuthorities(user);
        //实例化UserDetails对象
        userDetails=new org.springframework.security.core.userdetails.User(s,password,
                        true,
                        true,
                        true,
                        true, authorities);
    }
    return userDetails;
}
```



## 四、使用体会

其实这个框架的内容还是比较单纯的，就是为了解决用户的授权、权限的问题。说白了就是登录、注销、权限这些内容，原本呢都是通过过滤器、拦截器做的，但是呢，太繁琐了。虽然没有确认过，不过我觉得他底层肯定还是过滤器、拦截器。

然后就是如何触发这些内容，其实这些内容都有一个默认的路径，比如退出登录的/logout，当然这个路径可以被修改。说白了，使用很简单，就是两步，第一步配置，第二步使用，具体实现就交给框架了。

我对Spring Security的使用也是刚接触，还有很多未知。不过在秋招之前还有大量的工作要做，然后这个框架只是在项目中有所运用，没有必要去了解太多。Spring系列的框架真的要读源码，其难度比JDK的源码高太多了。目前的基本功完全不支持这么做。

### 4.1 Shiro

另外还要一个安全框架就是Shiro了，emmm没时间学了，在网上搜了一下基本的区别

- Shiro：简单灵活、权限控制粒度较粗，可以独立于Spring
- Spring Security:具有Shiro的全部功能，Spring Security的权限细粒度更高，不能脱离spring

孰是孰非、无从判断。以后找到工作了，投入真实环境的开发后，应该能找到答案了。