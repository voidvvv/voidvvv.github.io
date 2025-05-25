---
title: SpringSecurity 踩坑记录
date: 2025-05-17 20:48:45
tags:
---


## Spring security 基础使用
整合springboot，基本可以说是开箱即用。甚至无需额外设置，只需要引入依赖:
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

然后直接运行，即可看到spring security默认的登陆页面。

<!-- more -->

## Spring Security 的相关功能
整个springsecurity的基础功能，可以归纳为三个大部分：
1. 认证 (Authenticate)
   认证简单来说就是让系统知道你是谁。一般是用前端传过来的username 和password去数据库里进行获取user信息（也可以是别的比如session缓存，jwt token），这样我们就知道了当前的用户是谁。
2. 鉴权 （Authorization）
   就是在知道你是谁了后，你有哪些权限。
3. 检查 （Check）
   在知道你是谁并且确认了你的权限后，需要去跟当前要访问的资源进行匹配。匹配成功后，才会放行，否则会阻止访问并且正常会有 ``AccessDeniedException`` 抛出

## Spring Security 的组件

### 过滤器链（SecurityFilterChain）
构成Spring Security的核心功能的，是由``SecurityFilterChain``过滤器链来实现的。其本质就是一个个一连串的过滤器，进行各自功能的check以及验证。
而在``SecurityFilterChain``过滤器链内，则是有各个功能自己的组件。
而这个 ``SecurityFilterChain``过滤器链，实际上是由一个：FilterChainProxy 过滤器链代理的类来执行的。
其主要逻辑源码如下：
```java
    private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        FirewalledRequest firewallRequest = this.firewall.getFirewalledRequest((HttpServletRequest) request);
        HttpServletResponse firewallResponse = this.firewall.getFirewalledResponse((HttpServletResponse) response);
        // 获取当前请求所对应的过滤器集合
        List<Filter> filters = getFilters(firewallRequest);
        if (filters == null || filters.isEmpty()) {
            if (logger.isTraceEnabled()) {
                logger.trace(LogMessage.of(() -> "No security for " + requestLine(firewallRequest)));
            }
            firewallRequest.reset();
            this.filterChainDecorator.decorate(chain).doFilter(firewallRequest, firewallResponse);
            return;
        }
        if (logger.isDebugEnabled()) {
            logger.debug(LogMessage.of(() -> "Securing " + requestLine(firewallRequest)));
        }
        // 这个匿名内部类的作用是在遍历完所有对应过滤器集合后，回归当前的filter chain的
        FilterChain reset = (req, res) -> {
            if (logger.isDebugEnabled()) {
                logger.debug(LogMessage.of(() -> "Secured " + requestLine(firewallRequest)));
            }
            // Deactivate path stripping as we exit the security filter chain
            firewallRequest.reset();
            chain.doFilter(req, res);
        };
        // 此处是真实的过滤器逻辑，使用了装饰者模式
        this.filterChainDecorator.decorate(reset, filters).doFilter(firewallRequest, firewallResponse);
    }

```
装饰者内部逻辑
```java
        public void doFilter(final ServletRequest request, final ServletResponse response) throws IOException, ServletException {
            // 若过滤器集合遍历完毕，则回归主filter chain
            if (this.currentPosition == this.additionalFilters.size()) {
                this.originalChain.doFilter(request, response);
            } else {
                // 循环遍历所有找到的filter
                ++this.currentPosition;
                Filter nextFilter = (Filter)this.additionalFilters.get(this.currentPosition - 1);
                nextFilter.doFilter(request, response, this);
            }

        }
```

### Security上下文（SecurityContext）
使用Context来保存上下文信息是一个常用的方法，在spring中更是有application context来保存当前应用的上下文。在springsecurity中也不例外，springsecurity有自己的securityContext。其中主要是用来保存用户的认证信息（Authentication）。
在程序中，我们可以通过 SecurityContextHolder 来获取当前线程中的 SecurityContext。
```java
SecurityContextHolder.getContext();
```
其内部是存在**threadlocal**里的

### AuthenticateManager认证管理器
这个是认证步骤的核心逻辑。其主要功能是负责认证一个用户的身份(Authentication).
```JAVA
// AuthenticateManager是一个接口，只有一个方法，目的就是认证身份
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

```
springsecurity中，我们需要从当前的请求（request）以及响应（response）中，想办法获取user的身份(Authenticate)，然后放进AuthenticateManager中认证即可。
参考demo可以看springsecurity自带的``UsernamePasswordAuthenticationFilter``
```java
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        String username = obtainUsername(request);
        username = (username != null) ? username.trim() : "";
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
                password);
        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }

```





## 问题（坑）
1. 获取``AuthenticationManager``问题。
   ### 描述
   关于如果在自己的代码中注入并使用``AuthenticationManager``，我搜到资料表示说可以直接注入，但是我使用的springboot版本3.4.4的情况下，直接注入``AuthenticationManager``会找不到bean报错。意味着spring security并没有把一个默认的``AuthenticationManager``放到spring context中。
   ![alt text](unify\spring_security\image.png)
    此时，继续按照网上的另一个注入方法：
    ```java
    @Autowired
    private AuthenticationConfiguration AuthenticationConfiguration;
    
    // 使用AuthenticationConfiguration 来获取 AuthenticationManager
    @Bean
    public AuthenticationManager authenticationManager() throws Exception {
        return AuthenticationConfiguration.getAuthenticationManager();
    }
    ```
    看起来好像没有问题，并且可以启动成功。但是使用起来还是有些问题。
    如果我们在security的config中，配置了AuthenticateProvider，如下：
    ```java
        public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
            return http.authenticationProvider(new KAuthenticationProvider()).build();
        }
    ```

    然后我们在自己的代码中，使用刚才注册的 ``AuthenticationManager``
    ```java
    
    @Autowired
    private AuthenticationManager authenticationManager;

    public void check () {
                UsernamePasswordAuthenticationToken auth = UsernamePasswordAuthenticationToken.unauthenticated("user","123456");
        Authentication result = authenticationManager.authenticate(auth);

    }
    ```
    会发现，我们刚才的provider永远也走不到。在进一步debug下，我发现，spring security初始化``AuthenticationManager``是分为两步的，这可能也是为什么我们没办法获取spring security默认给我们的authenticateManager 的原因把。其中第一步，是在``org.springframework.security.config.annotation.web.configuration.HttpSecurityConfiguration#httpSecurity``中
    ```java

    @Bean(HTTPSECURITY_BEAN_NAME)
    @Scope("prototype")
    HttpSecurity httpSecurity() throws Exception {
        LazyPasswordEncoder passwordEncoder = new LazyPasswordEncoder(this.context);
        AuthenticationManagerBuilder authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(
                this.objectPostProcessor, passwordEncoder);
                // 这里会初始化第一个authenticationManager，并且作为AuthenticationConfiguration获取到的manager
                // 可以看到这里其实是把这个作为我们后面http的manager的parent manager了
        authenticationBuilder.parentAuthenticationManager(authenticationManager());
        authenticationBuilder.authenticationEventPublisher(getAuthenticationEventPublisher());
        HttpSecurity http = new HttpSecurity(this.objectPostProcessor, authenticationBuilder, createSharedObjects());
        WebAsyncManagerIntegrationFilter webAsyncManagerIntegrationFilter = new WebAsyncManagerIntegrationFilter();
        webAsyncManagerIntegrationFilter.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);
        // @formatter:off
        http
            .csrf(withDefaults())
            .addFilter(webAsyncManagerIntegrationFilter)
            .exceptionHandling(withDefaults())
            .headers(withDefaults())
            .sessionManagement(withDefaults())
            .securityContext(withDefaults())
            .requestCache(withDefaults())
            .anonymous(withDefaults())
            .servletApi(withDefaults())
            .apply(new DefaultLoginPageConfigurer<>());
        http.logout(withDefaults());
        // @formatter:on
        applyCorsIfAvailable(http);
        applyDefaultConfigurers(http);
        return http;
    }
    ```
    这里的是第一个初始化的AuthenticateManager，作为我们后面那一个的parent。然后在我们配置Secuirty的config的时候，会生成第二个AuthenticateManager：
    ```java
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.
        .authenticationProvider(new KAuthenticationProvider())
        ...
        .build();
    }
    ```
    注意，此时我们的Provider给的是当前build出来的第二个AuthenticateManager，是之前那个的child
    然后看ProviderManager中，有如下代码：
    ```java
            // 如果当前的manager没有成功获取身份信息，则去parent中继续获取
            if (result == null && this.parent != null) {
            // Allow the parent to try.
            try {
                parentResult = this.parent.authenticate(authentication);
                result = parentResult;
            }
            catch (ProviderNotFoundException ex) {
                // ignore as we will throw below if no other exception occurred prior to
                // calling parent and the parent
                // may throw ProviderNotFound even though a provider in the child already
                // handled the request
            }
            catch (AuthenticationException ex) {
                parentException = ex;
                lastException = ex;
            }
        }

    ```
    可以看到这里其实是在当前判断失败时会去parent那里再去尝试的。

    到现在为止，好像一切都还很正常。但是如果我要自己定义一个filter去根据user的token获取user的身份，然后再用我们拿到的``AuthenticationManager``去认证的话，就会出现无法认证的情况。因为我们自己获取到的是parent,而我们把provider放到的是child中。直接使用parent的authenticate是无法调用child的方法的。

    ### 复现
    1. 关于 ``AuthenticationManager``无法直接注入,直接把下面的代码放入自己项目即可复现
        ```java
        @Component
        public class SecurityIssue {
            @Autowired
            private AuthenticationManager authenticationManager;
        }

        ```
    2. 无法使用自己的provider
        * 首先定义一个自己的authentication 
        
        ```java
        package com.kz.web.test;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;

import java.util.Collection;

public class MyAuthentication implements Authentication {
    private boolean authState;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public Object getCredentials() {
        return null;
    }

    @Override
    public Object getDetails() {
        return null;
    }

    @Override
    public Object getPrincipal() {
        return null;
    }

    @Override
    public boolean isAuthenticated() {
        return authState;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        this.authState = isAuthenticated;
    }

    @Override
    public String getName() {
        return null;
    }
}
        ```

         * 然后定义自己的provider
        ```java
package com.kz.web.test;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;

public class MyAuthenticateProvider implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // 直接给与验证通过
        authentication.setAuthenticated(true);
        return authentication;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        System.out.println("MyAuthenticateProvider.supports");
        return MyAuthentication.class.isAssignableFrom(authentication);
    }
}

        ```

        * 定义自己的filter
        ```java
package com.kz.web.test;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

public class SimpleFilter extends OncePerRequestFilter {
    AuthenticationManager authenticationManager;
    
    public SimpleFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        System.out.println("SimpleFilter.doFilterInternal");
        MyAuthentication myAuthentication = new MyAuthentication();
        this.authenticationManager.authenticate(myAuthentication);
        filterChain.doFilter(request, response);
    }
}
        ```

        * 定义自己的config
        ```java
        package com.kz.web.test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.logout.LogoutFilter;

@Configuration
public class SimpleConfig {
    @Autowired
    private AuthenticationConfiguration authenticationConfiguration;

    @Bean
    public AuthenticationManager manager() throws Exception {
        return authenticationConfiguration.getAuthenticationManager();
    }


    @Bean
    @Order(0)
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/login").permitAll() // 登录页放行
                        .requestMatchers("/public/**", "/error").permitAll() // 明确放行登录页和公共路径
                        .requestMatchers("/admin/**").hasAuthority("admin")    // 需要 ADMIN 角色
                        .anyRequest().authenticated()                     // 其他所有路径需要认证
                )
                .anonymous(anon -> anon
                        .principal("anonymousUser") // 匿名用户
                )
                .formLogin(form ->
                        form.disable()
                )
                .addFilterAfter(new SimpleFilter(manager()), LogoutFilter.class)
                .authenticationProvider(new MyAuthenticateProvider())
                .logout(logout -> logout
                        .logoutUrl("/logout")          // 登出URL
                        .logoutSuccessUrl("/login?logout") // 登出成功后跳转
                        .permitAll()
                );
        return http.build();
    }
}

        ```

        运行上面的代码会发现自己的provider没有走进去。我下午尝试的时候还出现了循环依赖，AuthenticationManager 自己循环依赖自己。这个问题还没解决。还在看
