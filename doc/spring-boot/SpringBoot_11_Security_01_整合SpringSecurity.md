[TOC]





# 一、基于内存的认证

## 0.基本思路

> - 引入依赖
> - 通过 @EnableWebSecurity 注解开启Spring Security 的功能
> - 继承 WebSecurityConfigurerAdapter ，并重写它的方法来设置一些web安全的细节
>   - 配置 WebSecurity
>   - 配置 HttpSecurity
>   - 配置 AuthenticationManager



见`EnableWebSecurity`注解的源码注释：使用 @EnableWebSecurity 注解修饰一个继承了`WebSecurityConfigurerAdapter`的配置类，即可开启Spring Security 的功能



```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 构建认证管理器
     }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 定义对资源的保护策略
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        // 忽略对静态资源的处理
    }
}
```





## 1. 创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-11-spring-security-standard'
```



## 2.引入依赖

### 2.1 继承父工程依赖

在父工程`spring-boot-seeds` 的 `settings.gradle`加入子工程

```properties
rootProject.name = 'spring-boot-seeds'
include 'spring-boot-01-helloworld'
include 'spring-boot-02-restful-test'
include 'spring-boot-03-thymeleaf'
include 'spring-boot-04-swagger2'
include 'spring-boot-05-jpa'
include 'spring-boot-05-mybatis'
include 'spring-boot-05-tk-mybatis'
include 'spring-boot-06-nosql-redis'
include 'spring-boot-06-nosql-mongodb'
include 'spring-boot-07-cache-concurrentmap'
include 'spring-boot-07-cache-ehcache'
include 'spring-boot-07-cache-caffeine'
include 'spring-boot-07-cache-redis'
include 'spring-boot-08-messaging-rabbitmq'
include 'spring-boot-11-spring-security-standard'
```



这样，子工程`spring-boot-11-spring-security-standard`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`Spring Security`依赖

将子模块`spring-boot-11-spring-security-standard` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
}

```



## 3.配置WebSecurity

```java
package com.ray.study.springboot11springsecuritystandard.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * WebSecurity配置类
 *
 * @author shira 2019/08/30 17:05
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
        //return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(passwordEncoder()).withUser("admin").password(passwordEncoder().encode("admin")).roles("ADMIN");
        auth.inMemoryAuthentication().passwordEncoder(passwordEncoder()).withUser("zhangsan").password(passwordEncoder().encode("zhangsan")).roles("USER");
     }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .anyRequest().authenticated()
                .and()
                .logout().permitAll()
                .and()
                .formLogin();
        http.csrf().disable();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/js/**", "/css/**", "/images/**");
    }

}

```

## 4.测试类

```java
package com.ray.study.springboot11springsecuritystandard.controller;

import org.springframework.security.core.userdetails.User;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * description
 *
 * @author shira 2019/08/30 17:02
 */

@RestController
public class HelloController {

    @RequestMapping("/")
    public String home(){
        return "welcome to study spring security!";
    }

    @RequestMapping("/hello")
    public String hello() {
        return "hello world";
    }
}

```







# 二、方法权限认证

## 1.开启方法权限认证

通过 @EnableGlobalMethodSecurity 注解，可开启 Spring Security 方法权限注解



（1）@EnableGlobalMethodSecurity(securedEnabled=true) 

开启@Secured 注解过滤权限

@Secured认证是否有权限访问，例如:

> @Secured("IS_AUTHENTICATED_ANONYMOUSLY") public Account readAccount(Long id);
>
> @Secured("ROLE_TELLER")



（2）@EnableGlobalMethodSecurity(jsr250Enabled=true)

 开启JSR-250注解：

> - @DenyAll：拒绝所有访问
> - @RolesAllowed({"USER", "ADMIN"})  ：该方法只要具有"USER", "ADMIN"任意一种权限就可以访问。这里可以省 略前缀ROLE_，实际的权限可能是ROLE_ADMIN
> - @PermitAll：允许所有访问



（3）@EnableGlobalMethodSecurity(prePostEnabled=true) 
         启用如下4个注解

> - @PreAuthorize：在方法调用之前,基于表达式的计算结果来限制对方法的访问
>
> - *@PostAuthorize：允许方法调用,但是如果表达式计算结果为false,将抛出一个安全性异常*
> - *@PostFilter ：允许方法调用,但必须按照表达式来过滤方法的结果*
> - *@PreFilter：允许方法调用,但必须在进入方法之前过滤输入值*



## 2.使用方法权限认证注解

（1）首先我们通过 @EnableGlobalMethodSecurity 注解，来开启 Spring Security 方法权限注解

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

```



（2）在HelloController中加上如下方法

```java
    // 需要拥有ADMIN的角色才能访问，否则会报403
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @RequestMapping("/roleAuth")
    public String role() {
        return "admin auth";
    }

```





# 三、基于数据库的认证

通常，用户和权限信息是保存在数据库中的，下面我们将实现基于数据库的认证



## 1.引入依赖

将子模块`spring-boot-11-spring-security-standard` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    // spring security
    implementation 'org.springframework.boot:spring-boot-starter-security'

    // jpa 、mysql
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'mysql:mysql-connector-java'

    // thymeleaf
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}

```



## 2.修改`application.yml`

创建数据库spring-security-study，然后修改 application.yml 

```yml
server:
  port: 8088
  servlet:
    context-path: /

spring:
  datasource:       # 配置数据源
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/spring-security-study?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: root
  jpa:                 # 配置jpa
    database: mysql       # 数据库类型
    show-sql: true        # 打印sql语句
    open-in-view: false
    hibernate:
      ddl-auto: update    # 加载 Hibernate时， 自动更新数据库结构
  thymeleaf:
    mode: HTML5     # 设置模板模式，支持 HTML, XML TEXT JAVASCRIPT
    cache: false    # 禁用模板缓存：开发配置为false,避免修改模板还要重启服务器
```



## 3.entity

（1）UserDO

```java
package com.ray.study.springboot11springsecuritystandard.entity;

import lombok.Data;

import javax.persistence.*;
import java.util.List;

/**
 * 用户
 *
 * @author shira 2019/09/02 14:55
 */
@Data
@Entity
@Table(name = "user")
public class UserDO {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String username;

    private String password;

    @ManyToMany(cascade = {CascadeType.REFRESH}, fetch = FetchType.EAGER)
    @JoinTable(name = "user_role",
            joinColumns = {@JoinColumn(name = "user_id", referencedColumnName = "id")},
            inverseJoinColumns = {@JoinColumn(name = "role_id", referencedColumnName = "id")})
    private List<RoleDO> roleList;

}

```





（2）RoleDO

```java
package com.ray.study.springboot11springsecuritystandard.entity;

import lombok.Data;

import javax.persistence.*;

/**
 * 角色
 *
 * @author shira 2019/09/02 14:57
 */
@Data
@Entity
@Table(name = "role")
public class RoleDO {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;
}

```



## 4.repository

```java
package com.ray.study.springboot11springsecuritystandard.repository;

import com.ray.study.springboot11springsecuritystandard.entity.UserDO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

/**
 * UserRepository
 *
 * @author shira 2019/04/26 10:15
 */
@Repository
public interface UserRepository extends JpaRepository<UserDO, Long> {

    UserDO findByUsername(String username);
}

```



（2）RoleRepository

```java
package com.ray.study.springboot11springsecuritystandard.repository;

import com.ray.study.springboot11springsecuritystandard.entity.RoleDO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * RoleRepository
 *
 * @author shira 2019/09/02 19:11
 */
@Repository
public interface RoleRepository extends JpaRepository<RoleDO, Long> {

    List<RoleDO> findByName(String name);
}

```







## 5.Security配置

### 5.1 UserDetailsService实现类



```java
package com.ray.study.springboot11springsecuritystandard.security;

/**
 * description
 *
 * @author shira 2019/09/02 15:08
 */
import com.ray.study.springboot11springsecuritystandard.entity.RoleDO;
import com.ray.study.springboot11springsecuritystandard.entity.UserDO;
import com.ray.study.springboot11springsecuritystandard.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;

/**
 * UserDetailsServiceImpl
 *
 * @author shira 2019/04/26 10:15
 */
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;


    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        UserDO userDO = userRepository.findByUsername(s);
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        for (RoleDO roleDO : userDO.getRoleList()) {
            authorities.add(new SimpleGrantedAuthority(roleDO.getName()));
        }
        return new User(userDO.getUsername(), userDO.getPassword(), authorities);
    }

}

```





UserDetailsServiceImpl 还有另一种实现:

> - 用户类：
>
>   ```java
>   @Entity
>   @Table(name = "user")
>   @Getter
>   @Setter
>   public class Member implements UserDetails {
>   
>       @Id
>       @GeneratedValue
>       private Integer id;
>   
>       private String username;
>   
>       private String password;
>   
>       @ManyToMany(cascade = {CascadeType.REFRESH},fetch = FetchType.EAGER)
>       private List<Role> roles;
>   
>       @Override
>       public Collection<? extends GrantedAuthority> getAuthorities() {
>           List<GrantedAuthority> auths = new ArrayList<>();
>           List<Role> roles = this.getRoles();
>           for (Role role : roles) {
>               auths.add(new SimpleGrantedAuthority(role.getMark()));
>           }
>           return auths;
>       }
>   
>       @Override
>       public String getPassword() {
>           return this.password;
>       }
>   
>       @Override
>       public String getUsername() {
>           return this.username;
>       }
>   
>       @Override
>       public boolean isAccountNonExpired() {
>           return true;
>       }
>   
>       @Override
>       public boolean isAccountNonLocked() {
>           return true;
>       }
>   
>       @Override
>       public boolean isCredentialsNonExpired() {
>           return true;
>       }
>   
>       @Override
>       public boolean isEnabled() {
>           return true;
>       }
>   }
>   ```
>
> 
>
> - UserDetailsService实现类*
>
>   ```java
>   public class MemberService implements UserDetailsService {
>   
>       @Autowired
>       MemberRepository memberRepository;
>   
>       @Override
>       public UserDetails loadUserByUsername(String name) {
>   
>           Member member = memberRepository.findByUsername(name);
>           if (member == null) {
>               throw new UsernameNotFoundException("用户名不存在");
>           }
>   
>           return member;
>       }
>   }
>   ```
>
>   





### 5.2 WebSecurityConfiguration

构建认证管理器时，指定userDetailsService即可

```java
  @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService( new UserDetailsServiceImpl())
                .passwordEncoder(passwordEncoder());
    }
```





完整配置如下：

```java
package com.ray.study.springboot11springsecuritystandard.config;

import com.ray.study.springboot11springsecuritystandard.security.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * WebSecurity配置类
 *
 * @author shira 2019/08/30 17:05
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
        //return new BCryptPasswordEncoder();
    }


    /**
     * 基于内存的认证
     *      即认证信息保存在内存中
     * @param auth
     * @throws Exception
     */
    private void inMemoryAuthentication(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(passwordEncoder()).withUser("admin").password(passwordEncoder().encode("admin")).roles("ADMIN");
        auth.inMemoryAuthentication().passwordEncoder(passwordEncoder()).withUser("zhangsan").password(passwordEncoder().encode("zhangsan")).roles("USER");
    }


    @Bean
    UserDetailsService detailsService() {
        return new UserDetailsServiceImpl();
    }


    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 1.基于内存的认证
        //inMemoryAuthentication(auth);

        // 2.基于数据库的认证
        auth.userDetailsService(detailsService())
                .passwordEncoder(passwordEncoder());
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/sign-up","/registry").permitAll()
                .anyRequest().authenticated()
            .and().formLogin()
                .loginPage("/sign-in")
                .loginProcessingUrl("/login").defaultSuccessUrl("/personal-center",true)
                .failureUrl("/sign-in?error").permitAll()
            .and().logout().logoutSuccessUrl("/sign-in").permitAll()
            .and().rememberMe().tokenValiditySeconds(1209600)
            .and().csrf().disable();
    }


    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/js/**", "/css/**", "/images/**");
    }


}

```





## 6.Controller

```java
package com.ray.study.springboot11springsecuritystandard.controller;

import com.ray.study.springboot11springsecuritystandard.entity.RoleDO;
import com.ray.study.springboot11springsecuritystandard.entity.UserDO;
import com.ray.study.springboot11springsecuritystandard.repository.RoleRepository;
import com.ray.study.springboot11springsecuritystandard.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import java.util.ArrayList;
import java.util.List;

/**
 * description
 *
 * @author shira 2019/09/02 15:50
 */
@Controller
public class IndexController {

    @Autowired
    UserRepository userRepository;

    @Autowired
    RoleRepository roleRepository;

    @Autowired
    PasswordEncoder passwordEncoder;


    @RequestMapping("/sign-up")
    public String signUp() {
        return "registry";
    }

    @RequestMapping("/sign-in")
    public String signIn() {
        return "login";
    }



    @RequestMapping("/registry")
    public String registry(UserDO user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        if(user.getUsername().startsWith("admin")){
            user.setRoleList(roleRepository.findByName("ROLE_ADMIN"));
        }else{
            user.setRoleList(roleRepository.findByName("ROLE_USER"));
        }
        userRepository.save(user);
        return "login";
    }


    @ResponseBody
    @RequestMapping("/personal-center")
    public void login(HttpServletRequest request) {
        System.out.println("登录成功");
    }


}

```





## 7.视图

### 7.1 registry.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/registry" method="post">
    姓名：<input type="text" name="username">
    密码：<input type="password" name="password">
    <button type="submit">注册</button>
</form>
</body>
</html>
```





### 7.2 login.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/login" method="post">
    姓名：<input type="text" name="username">
    密码：<input type="password" name="password">
    <button type="submit">登录</button>
</form>
</body>
</html>
```





## 8.测试

（1）启动项目，并初始化role表数据

启动项目时，jpa会自动根据实体类创建数据库表结构，表创建好之后，我们插入两条角色以供下面的测试

```sql
INSERT INTO `role` VALUES (1, 'ROLE_ADMIN');
INSERT INTO `role` VALUES (2, 'ROLE_USER');
```



（2）注册

访问 `/sign-up` ，会前往注册页面（registry.html），然后输入用户名、密码来注册用户



（3）登录

注册成功后，会前往登录页面，使用前面注册的用户名密码登录即可





# 参考资料

1. [SpringBoot2.x整合Security5（完美解决 There is no PasswordEncoder mapped for the id "null"）](https://blog.csdn.net/SWPU_Lipan/article/details/80586054)
2. [springboot2.0--结合spring security5.0进行权限控制，从数据库中取权限信息及增加验证码](https://blog.csdn.net/fycghy0803/article/details/80535287)
3. [springboot2.1.4 与security5用户认证学习笔记](https://blog.51cto.com/357712148/2388932)
4. [SpringBoot + Spring Security 学习笔记（四）记住我功能实现](https://www.cnblogs.com/mujingyu/p/10702324.html)
5. [Spring Security 实现记住我](https://www.cnblogs.com/jonban/p/rememberMe.html)
6. [精通Spring Boot—— 第二十篇：Spring Security记住我功能](https://msd.misuland.com/pd/2884250034537237580)
7. 









# 附录一：异常信息

## 1.There is no PasswordEncoder mapped for the id "null"

### 1.1 异常现象

使用内存用户登录时，抛出异常信息：

```
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
	at org.springframework.security.crypto.password.DelegatingPasswordEncoder$UnmappedIdPasswordEncoder.matches(DelegatingPasswordEncoder.java:244) ~[spring-security-core-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at org.springframework.security.crypto.password.DelegatingPasswordEncoder.matches(DelegatingPasswordEncoder.java:198) ~[spring-security-core-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter$LazyPasswordEncoder.matches(WebSecurityConfigurerAdapter.java:594) ~[spring-security-config-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.additionalAuthenticationChecks(DaoAuthenticationProvider.java:90) ~[spring-security-core-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider.authenticate(AbstractUserDetailsAuthenticationProvider.java:166) ~[spring-security-core-5.1.5.RELEASE.jar:5.1.5.RELEASE]
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:175) ~[spring-security-core-5.1.5.RELEASE.jar:5.1.5.RELEASE]

```



### 1.2 异常原因

> 参考：[解决java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"](https://blog.csdn.net/weixin_39220472/article/details/80865411)



这是因为当前引用的 security 依赖是 spring security 5.X版本，此版本需要提供一个PasswordEncorder的实例，否则会抛出此异常



### 1.3 异常解决

提供一个PasswordEncorder的实例即可

```
    @Bean
    PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
        //return new BCryptPasswordEncoder();
    }
```

