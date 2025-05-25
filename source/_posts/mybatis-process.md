---
title: mybatis加载流程梳理
date: 2023-12-18 19:27:22
categories: 
- java
- web
tags:
- java
- mybatis
- web
---
### 关于mybatis

mybatis是一个数据库持久层框架。通过给其配置数据源，让其管理我们与数据库的链接，并且它让我们的代码和sql语句实现了分离。基本使用方法如下，

<!-- more -->

```java
public static void main(String[] args) throws IOException {

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources
                                    .getResourceAsReader("mybatisConfig.xml"));
        SqlSession sqlSession = sqlSessionFactory.openSession();

        WordfsMapper mapper = sqlSession.getMapper(WordfsMapper.class);
        List<Map<String, Object>> maps = mapper.selectAll();
        deal(maps);
    }
```

详细的就先不说了，主要说说大概的步骤。首先配置sqlSessionFactory，然后再通过sqlsessionfactory来获取一个跟数据库的链接session。最后所有的crud都是通过这个session获取到的mapper来完成的。这个mapper是我们定义的<font color='red'>接口</font>，里面全是未实现的抽象方法。例如本例中的selectAll。

### 关于spring

在springboot启动过程中，会默认将主启动类所在及其子包下的所有@Component 作为bean注入到上下文环境中，在项目启动后我们就可以直接使用这些bean了，spring也是一样，只不过需要手动配置xml配置文件。

spring注入bean，我们也都知道是有好几种方法的。例如直接在目标类上加@Component注解，或者在配置类中用@Bean注解注入。但是以上种种情况，我们注入进spring容器上下文中的bean都必须是被实例化的，换句话说，注入的bean起码得是一个对象，不能是一个接口。但是我们的mapper偏偏就是一个接口。那么spring是怎么帮我们注入这个mapper的呢？



### spring集成mybatis

spring集成mapper非常简单。直接在mapper接口上添加@Mapper注解，完事！想用的话直接@Autowire注入即可。

非常简单！！但是其中spring在后端其实为我们做了很多。我们都不知道（致敬默默奉献的spring），那么我们就来看一下spring是怎么帮我们把这个接口注入容器中让我们使用的吧

首先看一下依赖。由于我这里是引入的mybatisplus，所以这里就是mybatisplus的依赖。其实mapper注入跟mybatis的依赖几乎一样

```pom
        <!-- https://mvnrepository.com/artifact/com.baomidou/mybatisplus-spring-boot-starter -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>
```

* 首先先说说第一种注入方式，就是在主启动类上添加@MapperScan注解。我们可以先叫它包扫描注解。先看看这个注解。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(MapperScannerRegistrar.class) //导入了MapperScannerRegistrar这个类
@Repeatable(MapperScans.class)
public @interface MapperScan {

  /**
   * Alias for the {@link #basePackages()} attribute. Allows for more concise annotation declarations e.g.:
   * {@code @MapperScan("org.my.pkg")} instead of {@code @MapperScan(basePackages = "org.my.pkg"})}.
   *
   * @return base package names
   */
  String[] value() default {};

  /**
   * Base packages to scan for MyBatis interfaces. Note that only interfaces with at least one method will be
   * registered; concrete classes will be ignored.
   *
   * @return base package names for scanning mapper interface
   */
  String[] basePackages() default {}; //包路径
```

可以看到这个注解是导入了<font color='blue'>MapperScannerRegistrar</font>这个类的。这个类是做什么的呢？我们就来这个类内部看一看。下面是我截取的该类内部的一段代码。

```java
public class MapperScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {

  /**
   * {@inheritDoc}
   */
  @Override
  public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      //获取MapperScan（包扫描）注解
    AnnotationAttributes mapperScanAttrs = AnnotationAttributes
        .fromMap(importingClassMetadata.getAnnotationAttributes(MapperScan.class.getName()));
    if (mapperScanAttrs != null) {
      registerBeanDefinitions(mapperScanAttrs, registry, generateBaseBeanName(importingClassMetadata, 0));
    }
  }

  void registerBeanDefinitions(AnnotationAttributes annoAttrs, BeanDefinitionRegistry registry, String beanName) {

    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
    builder.addPropertyValue("processPropertyPlaceHolders", true);

    //注入各种属性，代码省略

    List<String> basePackages = new ArrayList<>();
    basePackages.addAll(
        Arrays.stream(annoAttrs.getStringArray("value")).filter(StringUtils::hasText).collect(Collectors.toList()));
//添加扫描的包路径
    basePackages.addAll(Arrays.stream(annoAttrs.getStringArray("basePackages")).filter(StringUtils::hasText)
        .collect(Collectors.toList()));

    basePackages.addAll(Arrays.stream(annoAttrs.getClassArray("basePackageClasses")).map(ClassUtils::getPackageName)
        .collect(Collectors.toList()));

    //懒加载相关，代码省略

    builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(basePackages));

    registry.registerBeanDefinition(beanName, builder.getBeanDefinition());

  }

```

可以看到这个类实现了spring提供的<font color='red'>ImportBeanDefinitionRegistrar</font>接口。这个接口的作用就是在被别的类@Import导入后，会调用registerBeanDefinitions这个方法。

可以看到，MapperScannerRegistrar 这个类中，重写的registerBeanDefinitions这个方法获取了MapperScan这个包扫描注解，并且判断不为空后，进入之后的方法。下面的方法其实就是把一个叫做MapperScannerConfigurer的类<font color='red'>注册</font>进了spring容器中。

所以我们可以把MapperScannerRegistrar 这个暂且叫做mapper的配置类注册器。

* > MapperScannerConfigurer看名字应该像是mapper扫描的配置类。我们也进去看看。

```java
public class MapperScannerConfigurer
    implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {
	//各种属性，getter，setter

  /**
   * {@inheritDoc}
   * 
   * @since 1.0.2
   */
  @Override
  public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    if (this.processPropertyPlaceHolders) {
      processPropertyPlaceHolders();
    }

    ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
    scanner.setAddToConfig(this.addToConfig);
    scanner.setAnnotationClass(this.annotationClass);
    scanner.setMarkerInterface(this.markerInterface);
    scanner.setSqlSessionFactory(this.sqlSessionFactory);
    scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
    scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
    scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
    scanner.setResourceLoader(this.applicationContext);
    scanner.setBeanNameGenerator(this.nameGenerator);
    scanner.setMapperFactoryBeanClass(this.mapperFactoryBeanClass);
    if (StringUtils.hasText(lazyInitialization)) {
      scanner.setLazyInitialization(Boolean.valueOf(lazyInitialization));
    }
    scanner.registerFilters();
    scanner.scan(
        StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));
  }

 //省略别的方法

}
```

可以看到，这个类是实现了BeanDefinitionRegistryPostProcessor这个接口的。这个接口是函数式接口。作用是在类注册进spring后，进行一些操作。这个操作就在postProcessBeanDefinitionRegistry方法中。而我们前面知道了，MapperScannerConfigurer这个mapper扫描配置类刚刚才被注册进spring，所以此时必然会执行该方法。

该方法本质其实就是new了一个ClassPathMapperScanner（路径mapper扫描器），然后执行了scan方法。我们看看这个方法。

```java
public int scan(String... basePackages) {
        int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
        this.doScan(basePackages);
        if (this.includeAnnotationConfig) {
            AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
        }

        return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;
    }
```

可以看到，ClassPathMapperScanner是继承了ClassPathBeanDefinitionScanner类的。我们要看的scan方法，ClassPathMapperScanner并没有重写，用的就是父类的scan方法。可是这里重写了doscan这个方法。所以其实真正的逻辑是下面：

```java
@Override
  public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

    if (beanDefinitions.isEmpty()) {
      LOGGER.warn(() -> "No MyBatis mapper was found in '" + Arrays.toString(basePackages)
          + "' package. Please check your configuration.");
    } else {
      processBeanDefinitions(beanDefinitions);
    }

    return beanDefinitions;
  }
```

调用父类doscan后，获取扫描到的的所有BeanDefinitionHolder。进行了处理。

```java
private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
    GenericBeanDefinition definition;
    for (BeanDefinitionHolder holder : beanDefinitions) {
      definition = (GenericBeanDefinition) holder.getBeanDefinition();
      String beanClassName = definition.getBeanClassName();
      definition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName); 
        //这一步很重要，偷梁换柱，把所有bean内部的实例class全部换为了MapperFactoryBean.class
      definition.setBeanClass(this.mapperFactoryBeanClass);

      definition.getPropertyValues().add("addToConfig", this.addToConfig);
	//注入sqlSessionFactory
      boolean explicitFactoryUsed = false;
      if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {
        definition.getPropertyValues().add("sqlSessionFactory",
            new RuntimeBeanReference(this.sqlSessionFactoryBeanName));
        explicitFactoryUsed = true;
      } else if (this.sqlSessionFactory != null) {
        definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
        explicitFactoryUsed = true;
      }
//注入sqlSessionTemplate（会顶替sqlSessionFactory）
      if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {
        definition.getPropertyValues().add("sqlSessionTemplate",
            new RuntimeBeanReference(this.sqlSessionTemplateBeanName));
        explicitFactoryUsed = true;
      } else if (this.sqlSessionTemplate != null) {
        
        definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);
        explicitFactoryUsed = true;
      }
      definition.setLazyInit(lazyInitialization);
    }
```

可以看到，大概就做了两件事，1.偷梁换柱，把bean对应的mapper的class类统一换为了MapperFactoryBean.class。2. 注入sqlSessionTemplate

至此，mapper在spring容器中注入过程就全部完毕了。可能有人会有疑问了，我要的是mapper，你给我MapperFactoryBean，这能行吗？系统运行起来不得崩溃吗？

下面就该看一下如何从spring中获取对应的mapper了

### spring获取mapper

首先，我们进入MapperFactoryBean这个类看一下。

```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {

  private Class<T> mapperInterface;

  private boolean addToConfig = true;

  public MapperFactoryBean() {
    // intentionally empty
  }

  public MapperFactoryBean(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }

  /**
   * {@inheritDoc}
   */
  @Override
  protected void checkDaoConfig() {
    super.checkDaoConfig();

    notNull(this.mapperInterface, "Property 'mapperInterface' is required");

    Configuration configuration = getSqlSession().getConfiguration();
    if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
      try {
        configuration.addMapper(this.mapperInterface);
      } catch (Exception e) {
        logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", e);
        throw new IllegalArgumentException(e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public T getObject() throws Exception {
    return getSqlSession().getMapper(this.mapperInterface);
  }

//省略一部分代码
}
```

可以看到它是集成了FactoryBean的。这其实就是一个bean的工厂类。只不过目前这个工厂只管mapper。

当我们向spring要mapper时，因为mapper在spring中的实例均为MapperFactoryBean这个工厂，此时就会调用getObject（）这个方法。

看看这个方法内部：

```java
  public SqlSession getSqlSession() {
    return this.sqlSessionTemplate;
  }

@Override
  public T getObject() throws Exception {
    return getSqlSession().getMapper(this.mapperInterface);
  }
```

是不是发现什么了，跟我们原生使用mybatis的逻辑几乎是一样的，获取sqlsession（这里是sqlSessionTemplate），再用这个来getMapper。就获取到我们要的mapper了。而这里的mapperInterface，就是我们注入mapperfactory的时候那一句

```java
definition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName);
```

把每一个bean对应的构造方法传入当前bean的全类名。就是当前的mapper。就是这里的mapperInterface。

而这个sqlSessionTemplate也是我们刚才注入mapperfactory的时候，注入的sqlSessionTemplate。

这样，其实就是我们获取mapper的底实现。发现到了底层，其实跟我们原生使用差不了太多。



<font size='5'>最后，我看网上很多对mybatis自动注入进spring讲解的不够详细，只说了每一个mapper底层其实是mapperfatory工厂，并没有讲明为什么。我这次算是带着大家从源码级别过了一遍mapper的注入流程啦。应该比较详细了。</font>

<font size='5' color = 'gree'>其实mapper注入除了启动类的MapperScan注解，还有Mapper注解是比较常用的。其实这个跟这个原理差不多。这边就先讲到这里。有兴趣的大家可以自己去看一下相关的逻辑。</font>


