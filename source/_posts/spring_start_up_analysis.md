---
title: springboot bean启动流程bean加载源码级分析
date: 2025-09-02T11:26:21+08:00
tags:
- spring
categories:
- web
---

之前面试被问了三级缓存，非常经典的问题，我回答了解决循环依赖，动态代理的问题，但是被深入问道如何解决的，并没有回答的很好。由此发现自己对于spring的源码研究还是不够透彻（之前研究过，现在忘了很多），所以我今天来从源码角度来分析以及整理一下spring bean加载的整个流程。
本次分析基于 ``springboot 3.5.3``版本

<!-- more -->
```xml
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.3</version>
```

## 启动以及scan
在springboot启动后，本质上是会``new`` 一个 ``SpringApplication``对象，来执行其run方法。这里会创建我们的基础对象，prepare当前的context，以及refreshContext.

```java
	public ConfigurableApplicationContext run(String... args) {
		Startup startup = Startup.create();
		if (this.properties.isRegisterShutdownHook()) {
			SpringApplication.shutdownHook.enableShutdownHookAddition();
		}
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		ConfigurableApplicationContext context = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			startup.started();
			if (this.properties.isLogStartupInfo()) {
				new StartupInfoLogger(this.mainApplicationClass, environment).logStarted(getApplicationLog(), startup);
			}
			listeners.started(context, startup.timeTakenToStarted());
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			throw handleRunFailure(context, ex, listeners);
		}
		try {
			if (context.isRunning()) {
				listeners.ready(context, startup.ready());
			}
		}
		catch (Throwable ex) {
			throw handleRunFailure(context, ex, null);
		}
		return context;
	}
```
refresh过程中，会有一步： ``invokeBeanFactoryPostProcessors``,这一步中，会有一步``invokeBeanDefinitionRegistryPostProcessors``, 这一步会将所有的beandefinition加入到我们的registry中。这里其实是遍历所有的``BeanDefinitionRegistryPostProcessor``的``postProcessBeanDefinitionRegistry``方法。
在springboot中，主要应用到的是``ConfigurationClassPostProcessor``，这个类实现了``BeanDefinitionRegistryPostProcessor``接口，该方法的实现底层是使用``ClassPathBeanDefinitionScanner``来扫描我们当前所有config类下的所有注解component的类以及@bean的对象，将其声明为beandefinition并放入我们的bean registry中。
随后，就是遍历所有的bean definition，生成对应的object并放入我们的spring context中。

## 生成bean对象
这个流程就是我们实际的向spring容器中放入我们的bean对象的过程。
这个过程的入口在``org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization`` 中，这里，会完成我们beanfactory的所有基础bean的初始化。
![2025-09-02T122730](2025-09-02T122730.png)
向下走，来到我们的``beanFactory.preInstantiateSingletons();``方法
![2025-09-02T122843](2025-09-02T122843.png)
可以看到,这里就是会循环遍历我们所有加载的bean definition，来依次生成我们实际的bean对象
![2025-09-02T123004](2025-09-02T123004.png)
在方法内部，我们可以看到，加载一个bean会判断两部分，
* 一部分是当前bean是否异步加载，如果是，那么会使用springboot内置的线程池来生成一个CompletableFuture对象，该对象的内部逻辑就是在后台线程生成bean实例。

### 懒加载  
还有一部分是判断当前bean是否是懒加载，如果不是，那么就会实例化当前bean，**如果是懒加载，那么初始化会跳过这个bean的实例化。**，在后续bean的实例化的过程中，依赖了这个bean才会将其实例化。

### 非懒加载
实际调用的是``org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean``方法获取我们的bean对象。
该方法首先会调用我们的``org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String)``
方法，该方法就是我们三级缓存所使用的地方。显然，如果是第一个bean的实例化过程，三级缓存中是什么都没有的。

#### 实例化对象
在三级缓存中，我们很可能获取不到我们所需要的bean实例，那么spring接下来就会帮我们调用``getSingleton(java.lang.String)``的重载方法, ``org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, org.springframework.beans.factory.ObjectFactory<?>)``，这个方法第一个参数表示我们需要的bean的名字，第二个参数就是一个factory，这个factory可以生成这个bean的实例。并且该singleton方法调用后，会将从factory中获取到的bean实例放入我们的一级缓存``(singletonObjects)``中**（这里是目前第一次对三级缓存的放入操作）**

spring在此时传入的factory方法实现中的逻辑就是 ``org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])``,该方法实际又调用了``org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean``我们真正的create bean方法。
该方法内部有一个``createBeanInstance``，这个是通过构造函数来构造我们真实的bean对象。其中，spring帮我们选取了指定的(Autowired(required = true))的构造方法，并且当构造方法有参数时，会去spring上下文中来找对应的bean对象(getBean(String))，当懒加载的bean被依赖时，此时就会执行懒加载bean的实例化逻辑.
接下来，如果spring判断我们当前环境是需要提前暴露bean的话：
![2025-09-02T133700](2025-09-02T133700.png)
则会向三级缓存的第三级缓存中``（singletonFactories）``，放入我们的一个beanfactory，其实现是调用了getEarlyBeanReference，这个factory返回的是当前实例化对象的代理对象，这个beanfactory就是获取当前**对象并且进行切面动态代理后的对象!**,这里很重要，放入第三级缓存的factory里面是有动态代理逻辑的。
![2025-09-02T133920](2025-09-02T133920.png)
这里是调用所有的``SmartInstantiationAwareBeanPostProcessor``来对当前bean进行加工，bean postprocessor有可能会对bean进行改变，即动态代理。

#### 填充属性
这一步就是``org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean``方法，里面用到了各种策略模式来给我们当前bean中autowire的别的类来进行注入属性。
这一步会出现循环依赖的问题。
比如我们的AService注入了BService，BService同时又注入了AService。
假设我们前面的流程都是加载AService的，那么我们现在就需要populate AService的属性。那么这一步中，会发现BService需要被加载，因此，spring在这里会进行getBean(String) 来获取BService。
整体流程跟AService几乎一样，实例化，然后再来填充属性，这里会走到``org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#resolveFieldValue``。
在``findAutowireCandidates``中， 其中的``addCandidateEntry``方法，会去check是否在三级缓存的一级缓存中,如果在那么会直接从其中取出来，否则会将当前所需类的class作为查找结果返回。
在``doResolveDependency``方法的最后，有一段将class转换为bean对象的代码：
```java
			if (instanceCandidate instanceof Class) {
				instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
			}
```
在方法内我们可以看到就是调用了``getBean(String)``
![2025-09-02T204418](2025-09-02T204418.png)
``getBean(String)``出现了很多次了，其逻辑就是getSingleton（三级缓存），若没获取到，则会去尝试使用构造方法来createBean.
当前的流程是BService要找AService，而AService在我们之前分析的流程中，已经实例化完毕，并且已经放入了三级缓存中的第三级缓存（singletonFactory）中，所以在这一步，BService会成功的获取到AService的实例（依据前面的分析，此处获取到的实例为动态代理后的），在三级缓存源码中，此时会将该bean对应的singletonFactory移除，防止多次调用生成多个不同的bean，但又由于我们的AService目前正处在创建阶段，并没有完成，所以将该实例放入earlySingletonBean中，也就收我们所说的**二级缓存**，这么做之后，后续的getSingleton方法获取AService就会从二级缓存处获取到对应的bean对象了。

至此，BService中的AService对象获取完毕，获取到的也是动态代理后的AService对象。BService的populate阶段也已完成。
最后会对BService进行动态代理，生成最终的spring bean对象，放入singletonBean（一级缓存）中，至此，BService注入完毕。

接下来，流程就会回到AService这里，现在我们可以成功拿到BService的对象bean了，我们就可以成功回填BService到AService里面去了。
**但是，请记住，我们之前实例化BService的时候，已经把AService放入了二级缓存中了**，即spring现在的getBean（String）是会拿到这个放入的对象的，在实际开发中，AService有可能依赖了若干个别的Service，这些Service如果又依赖了AService，那么他们中的AService就都是现在二级缓存中的这个AService bean。

但是，AService回填阶段结束以后，还有一个很重要的阶段，就是**postProcessor**，动态代理也是通过这个实现的。因此我们遇到了一个问题，即，在别的service初始化时，我们已经将AService的代理类放入spring了，我们此时又要对AService修改，代理，所以会有可能出现不同版本的AService，因此Spring在这里又做了一个版本判断:
```java
		// 如果允许提前暴露bean实例
		if (earlySingletonExposure) {
			// 获取spring中AService现有的bean
			Object earlySingletonReference = getSingleton(beanName, false);
			// 若spring中现有的bean不为控
			if (earlySingletonReference != null) {
				// 若当前流程中postprocessor处理后的bean对象与原本bean对象为同一个对象，则将该对象设置为spring版本中的对象。
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				// 否则，意味着我们当前流程使用了postprocessor修改了bean，并且spring中也已经有了一个bean。出现了多版本问题
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					// 出现多版本bean
					}
				}
			}
		}

```
这个代码可以防止spring中的bean与注入到别的service循环依赖中的bean版本不一致的问题。
如果想要手动复现这个错误，我们可以实现一个自己的BeanPostProcessor，来修改一个被spring 动态代理的类：
```java
@Component
public class MyPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (AService.class.isAssignableFrom(bean.getClass())) {
			// 若当前对象的类是我们指定的类，则将其修改为我们自定义别的类
            System.out.println("BeforeInitialization : " + beanName);
            return new BigAService((AService) bean);
        }
        return bean;
    }
}
```

至此，AService实例化完毕，AService注入属性完毕，AService posprocessor处理完毕.
bean加载流程基本结束。






