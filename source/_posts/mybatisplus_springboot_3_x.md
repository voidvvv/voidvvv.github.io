---
title: springboot 3.4.4 引入mybatisplus时产生的版本冲突问题
date: 2025-06-03T22:25:48+08:00
categories:
- springboot3
- mybatis
---


**Invalid value type for attribute 'factoryBeanObjectType': java.lang.String**
[项目地址](https://github.com/voidvvv/kz_utils)
<!--more-->

这个错误是使用版本不正确导致的。
我的项目使用的springboot版本是3.4.4比较新的3.x：
```xml
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.4.4</version>

```
mybatis使用的版本是
```xml
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.5.5</version>
```
此时会出现版本依赖错误，该版本中的mybatis-spring依赖版本是2.1.2，这个是无法兼容我们的springboot版本的。需要我们手动排除后手动指定：
```xml
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.5.5</version>
			<exclusions>
				<exclusion>
					<groupId>org.mybatis</groupId>
					<artifactId>mybatis-spring</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>3.0.3</version>
		</dependency>

```

