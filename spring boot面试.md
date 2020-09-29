---
typora-copy-images-to:media
---

# 注解大全

使用注解的优势：

   1.采用纯java代码，不在需要配置繁杂的xml文件

   2.在配置中也可享受面向对象带来的好处

   3.类型安全对重构可以提供良好的支持

   4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能

 

**一、注解详解（配备了完善的释义****）------(可采用ctrl+F 来进行搜索哦~~~~)**

@SpringBootApplication：申明让spring boot自动给程序进行必要的配置，这个配置等同于：

@Configuration ，@EnableAutoConfiguration 和 @ComponentScan 三个配置。

@ResponseBody：表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析为跳转路径，加上@esponsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上@Responsebody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用。

@Controller：用于定义控制器类，在spring项目中由控制器负责将用户发来的URL请求转发到对应的服务接口（service层），一般这个注解在类中，通常方法需要配合注解@RequestMapping。

@RestController：用于标注控制层组件(如struts中的action)，@ResponseBody和@Controller的合集。

@RequestMapping：提供路由信息，负责URL到Controller中的具体函数的映射。

@EnableAutoConfiguration：SpringBoot自动配置（auto-configuration）：尝试根据你添加的jar依赖自动配置你的Spring应用。例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库”。你可以将@EnableAutoConfiguration或者@SpringBootApplication注解添加到一个@Configuration类上来选择自动配置。如果发现应用了你不想要的特定自动配置类，你可以使用@EnableAutoConfiguration注解的排除属性来禁用它们。

@ComponentScan：表示将该类自动发现扫描组件。个人理解相当于，如果扫描到有@Component、@Controller、@Service等这些注解的类，并注册为Bean，可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。如果没有配置的话，Spring Boot会扫描启动类所在包下以及子包下的使用了@Service,@Repository等注解的类。

@Configuration：相当于传统的xml配置文件，如果有些第三方库需要用到xml文件，建议仍然通过@Configuration类作为项目的配置主类——可以使用@ImportResource注解加载xml配置文件。

@Import：用来导入其他配置类。

@ImportResource：用来加载xml配置文件。

@Autowired：自动导入依赖的bean

@Service：一般用于修饰service层的组件

@Repository：使用@Repository注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，同时也不需要为它们提供XML配置项。

@Bean：用@Bean标注方法等价于XML中配置的bean。

@Value：注入Spring boot application.properties配置的属性的值。示例代码：

@Inject：等价于默认的@Autowired，只是没有required属性；

@Component：泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

@Bean:相当于XML中的,放在方法的上面，而不是类，意思是产生一个bean,并交给spring管理。

@AutoWired：自动导入依赖的bean。byType方式。把配置好的Bean拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。当加上（required=false）时，就算找不到bean也不报错。

@Qualifier：当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者，具体使用方式如下：

@Resource(name=”name”,type=”type”)：没有括号内内容的话，默认byName。与@Autowired干类似的事。

**二、注解列表如下**

@SpringBootApplication：包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中

@ComponentScan：让spring Boot扫描到Configuration类并把它加入到程序上下文。

@Configuration ：等同于spring的XML配置文件；使用Java代码可以检查类型安全。

@EnableAutoConfiguration ：自动配置。

@ComponentScan ：组件扫描，可自动发现和装配一些Bean。

@Component可配合CommandLineRunner使用，在程序启动后执行一些基础任务。

@RestController：注解是@Controller和@ResponseBody的合集,表示这是个控制器bean,并且是将函数的返回值直 接填入HTTP响应体中,是REST风格的控制器。

@Autowired：自动导入。

@PathVariable：获取参数。

@JsonBackReference：解决嵌套外链问题。

@RepositoryRestResourcepublic：配合spring-boot-starter-data-rest使用。

**三、JPA注解**

@Entity：@Table(name=”“)：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@MappedSuperClass:用在确定是父类的entity上。父类的属性子类可以继承。

@NoRepositoryBean:一般用作父类的repository，有这个注解，spring不会去实例化该repository。

@Column：如果字段名与列名相同，则可以省略。

@Id：表示该属性为主键。

@GeneratedValue(strategy = GenerationType.SEQUENCE,generator = “repair_seq”)：表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。

@SequenceGeneretor(name = “repair_seq”, sequenceName = “seq_repair”, allocationSize = 1)：name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

@Transient：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式

@JsonIgnore：作用是json序列化时将[Java ](http://lib.csdn.net/base/java)bean中的一些属性忽略掉,序列化和反序列化都受影响。

@JoinColumn（name=”loginId”）:一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

@OneToOne、@OneToMany、@ManyToOne：对应[hibernate](http://lib.csdn.net/base/javaee)配置文件中的一对一，一对多，多对一。

**四、springMVC相关注解**

@RequestMapping：@RequestMapping(“/path”)表示该控制器处理所有“/path”的UR L请求。RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。该注解有六个属性：
params:指定request中必须包含某些参数值是，才让该方法处理。
headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
value:指定请求的实际地址，指定的地址可以是URI Template 模式
method:指定请求的method类型， GET、POST、PUT、DELETE等
consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

@RequestParam：用在方法的参数前面。
@RequestParam
String a =request.getParameter(“a”)。

@PathVariable:路径变量。如

参数与大括号里的名字一样要相同。

**五、全局异常处理**

@ControllerAdvice：包含@Component。可以被扫描到。统一处理异常。

@ExceptionHandler（Exception.class）：用在方法上面表示遇到这个异常就执行以下方法。

------

**六、项目中具体配置解析和使用环境**

***\**\*@MappedSuperclass：\*\***

1.@MappedSuperclass 注解使用在父类上面，是用来标识父类的

2.@MappedSuperclass 标识的类表示其不能映射到数据库表，因为其不是一个完整的实体类，但是它所拥有的属性能够映射在其子类对用的数据库表中

3.@MappedSuperclass 标识的类不能再有@Entity或@Table注解

**@Column：**

1.当实体的属性与其映射的数据库表的列不同名时需要使用@Column标注说明，该属性通常置于实体的属性声明语句之前，还可与 @Id 标注一起使用。

2.@Column 标注的常用属性是name，用于设置映射数据库表的列名。此外，该标注还包含其它多个属性，如：unique、nullable、length、precision等。具体如下：

 1 name属性：name属性定义了被标注字段在数据库表中所对应字段的名称

 2 unique属性：unique属性表示该字段是否为唯一标识，默认为false，如果表中有一个字段需要唯一标识，则既可以使用该标记，也可以使用@Table注解中的@UniqueConstraint

 3 nullable属性：nullable属性表示该字段是否可以为null值，默认为true

 4 insertable属性：insertable属性表示在使用”INSERT”语句插入数据时，是否需要插入该字段的值

 5 updateable属性：updateable属性表示在使用”UPDATE”语句插入数据时，是否需要更新该字段的值

 6 insertable和updateable属性：一般多用于只读的属性，例如主键和外键等，这些字段通常是自动生成的

 7 columnDefinition属性：columnDefinition属性表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用，如果数据库中表已经建好，该属性没有必要使用

 8 table属性：table属性定义了包含当前字段的表名

 9 length属性：length属性表示字段的长度，当字段的类型为varchar时，该属性才有效，默认为255个字符

 10 precision属性和scale属性：precision属性和scale属性一起表示精度，当字段类型为double时，precision表示数值的总长度，scale表示小数点所占的位数

 

  具体如下：
  1.double类型将在数据库中映射为double类型，precision和scale属性无效
  2.double类型若在columnDefinition属性中指定数字类型为decimal并指定精度，则最终以columnDefinition为准
  3.BigDecimal类型在数据库中映射为decimal类型，precision和scale属性有效
  4.precision和scale属性只在BigDecimal类型中有效

3.@Column 标注的columnDefinition属性: 表示该字段在数据库中的实际类型.通常 ORM 框架可以根据属性类型自动判断数据库中字段的类型,但是对于Date类型仍无法确定数据库中字段类型究竟是DATE,TIME还是TIMESTAMP.此外,String的默认映射类型为VARCHAR,如果要将 String 类型映射到特定数据库的 BLOB 或TEXT字段类型.

4.@Column标注也可置于属性的getter方法之前

@Getter和@Setter（Lombok）

@Setter：注解在属性上；为属性提供 setting 方法 @Getter：注解在属性上；为属性提供 getting 方法

 1 @Data：注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法
 2 
 3 @Setter：注解在属性上；为属性提供 setting 方法
 4 
 5 @Getter：注解在属性上；为属性提供 getting 方法
 6 
 7 @Log4j2 ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象，和@Log4j注解类似
 8 
 9 @NoArgsConstructor：注解在类上；为类提供一个无参的构造方法
 10 
 11 @AllArgsConstructor：注解在类上；为类提供一个全参的构造方法
 12 
 13 @EqualsAndHashCode:默认情况下，会使用所有非瞬态(non-transient)和非静态(non-static)字段来生成equals和hascode方法，也可以指定具体使用哪些属性。
 14 
 15 @toString:生成toString方法，默认情况下，会输出类名、所有属性，属性会按照顺序输出，以逗号分割。
 16 
 17 @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor
 18 无参构造器、部分参数构造器、全参构造器，当我们需要重载多个构造器的时候，只能自己手写了
 19 
 20 @NonNull：注解在属性上，如果注解了，就必须不能为Null
 21 
 22 @val:注解在属性上，如果注解了，就是设置为final类型，可查看源码的注释知道

 

当你在执行各种持久化方法的时候，实体的状态会随之改变，状态的改变会引发不同的生命周期事件。这些事件可以使用不同的注释符来指示发生时的回调函数。

@javax.persistence.PostLoad：加载后。

@javax.persistence.PrePersist：持久化前。

@javax.persistence.PostPersist：持久化后。

@javax.persistence.PreUpdate：更新前。

@javax.persistence.PostUpdate：更新后。

@javax.persistence.PreRemove：删除前。

@javax.persistence.PostRemove：删除后。

**1）数据库查询**

@PostLoad事件在下列情况下触发：

执行EntityManager.find()或getreference()方法载入一个实体后。

执行JPQL查询后。

EntityManager.refresh()方法被调用后。

**2）数据库插入**

@PrePersist和@PostPersist事件在实体对象插入到数据库的过程中发生：

@PrePersist事件在调用persist()方法后立刻发生，此时的数据还没有真正插入进数据库。

@PostPersist事件在数据已经插入进数据库后发生。

**3）数据库更新**

@PreUpdate和@PostUpdate事件的触发由更新实体引起：

@PreUpdate事件在实体的状态同步到数据库之前触发，此时的数据还没有真正更新到数据库。

@PostUpdate事件在实体的状态同步到数据库之后触发，同步在事务提交时发生。

**4）数据库删除**

@PreRemove和@PostRemove事件的触发由删除实体引起：

@PreRemove事件在实体从数据库删除之前触发，即在调用remove()方法删除时发生，此时的数据还没有真正从数据库中删除。

@PostRemove事件在实体从数据库中删除后触发。

# 核心注解原理

今天跟大家来探讨下SpringBoot的核心注解@SpringBootApplication以及run方法，理解下springBoot为什么不需要XML，达到零配置

首先我们先来看段代码

```
@SpringBootApplication
public class StartEurekaApplication
{
    public static void main(String[] args)

    {



        SpringApplication.run(StartEurekaApplication.class, args);



    }



}
```

我们点进@SpringBootApplication来看

```
@Target(ElementType.TYPE)



@Retention(RetentionPolicy.RUNTIME)



@Documented



@Inherited



@SpringBootConfiguration



@EnableAutoConfiguration



@ComponentScan(excludeFilters = {



      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),



      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })



public @interface SpringBootApplication {



 



}
```

上面的元注解我们在这里不在做解释，相信大家在开发当中肯定知道，我们要来说@SpringBootConfiguration @EnableAutoConfiguration 这两个注解,到这里我们知道 SpringBootApplication注解里除了元注解，我们可以看到又是@SpringBootConfiguration，@EnableAutoConfiguration，@ComponentScan的组合注解，官网上也有详细说明，那我们现在把目光投向这三个注解。

首先我们先来看 @SpringBootConfiguration，那我们点进来看

```
@Target(ElementType.TYPE)



@Retention(RetentionPolicy.RUNTIME)



@Documented



@Configuration



public @interface SpringBootConfiguration {



 



}
```

我们可以看到这个注解除了元注解以外，就只有一个@Configuration，那也就是说这个注解相当于@Configuration，所以这两个注解作用是一样的，那他是干嘛的呢，相信很多人都知道，它是让我们能够去注册一些额外的Bean，并且导入一些额外的配置。那@Configuration还有一个作用就是把该类变成一个配置类，不需要额外的XML进行配置。所以@SpringBootConfiguration就相当于@Configuration。

那我们继续来看下一个@EnableAutoConfiguration，这个注解官网说是 让Spring自动去进行一些配置，那我们点进来看

```
@Target(ElementType.TYPE)



@Retention(RetentionPolicy.RUNTIME)



@Documented



@Inherited



@AutoConfigurationPackage



@Import(EnableAutoConfigurationImportSelector.class)



public @interface EnableAutoConfiguration {



}
```

可以看到它是由 @AutoConfigurationPackage，@Import(EnableAutoConfigurationImportSelector.class)这两个而组成的，我们先说@AutoConfigurationPackage，他是说：让包中的类以及子包中的类能够被自动扫描到spring容器中。 我们来看@Import(EnableAutoConfigurationImportSelector.class)这个是核心，之前我们说自动配置，那他到底帮我们配置了什么，怎么配置的？

就和@Import(EnableAutoConfigurationImportSelector.class)息息相关，程序中默认使用的类就自动帮我们找到。我们来看EnableAutoConfigurationImportSelector.class

```
public class EnableAutoConfigurationImportSelector



      extends AutoConfigurationImportSelector {



 



   @Override



   protected boolean isEnabled(AnnotationMetadata metadata) {



      if (getClass().equals(EnableAutoConfigurationImportSelector.class)) {



         return getEnvironment().getProperty(



               EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class,



               true);



      }



      return true;



   }



 



}
```

可以看到他继承了AutoConfigurationImportSelector我们继续来看AutoConfigurationImportSelector，这个类有一个方法

```
public String[] selectImports(AnnotationMetadata annotationMetadata) {



   if (!isEnabled(annotationMetadata)) {



      return NO_IMPORTS;



   }



   try {



      AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader



            .loadMetadata(this.beanClassLoader);



      AnnotationAttributes attributes = getAttributes(annotationMetadata);



      List<String> configurations = getCandidateConfigurations(annotationMetadata,



            attributes);



      configurations = removeDuplicates(configurations);



      configurations = sort(configurations, autoConfigurationMetadata);



      Set<String> exclusions = getExclusions(annotationMetadata, attributes);



      checkExcludedClasses(configurations, exclusions);



      configurations.removeAll(exclusions);



      configurations = filter(configurations, autoConfigurationMetadata);



      fireAutoConfigurationImportEvents(configurations, exclusions);



      return configurations.toArray(new String[configurations.size()]);



   }



   catch (IOException ex) {



      throw new IllegalStateException(ex);



   }



}
```

这个类会帮你扫描那些类自动去添加到程序当中。我们可以看到getCandidateConfigurations()这个方法，他的作用就是引入系统已经加载好的一些类，到底是那些类呢，我们点进去看一下

```
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,



      AnnotationAttributes attributes) {



   List<String> configurations = SpringFactoriesLoader.loadFactoryNames(



         getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());



   Assert.notEmpty(configurations,



         "No auto configuration classes found in META-INF/spring.factories. If you "



               + "are using a custom packaging, make sure that file is correct.");



   return configurations;



}
```

这个类回去寻找的一个目录为META-INF/spring.factories，也就是说他帮你加载让你去使用也就是在这个META-INF/spring.factories目录装配的，他在哪里？

![深入SpringBoot核心注解原理](media/java6-1558274425.jpg)

我们点进spring.factories来看

![深入SpringBoot核心注解原理](media/java2-1558274425.jpg)

我们可以发现帮我们配置了很多类的全路径，比如你想整合activemq，或者说Servlet

![深入SpringBoot核心注解原理](media/java8-1558274426.jpg)

可以看到他都已经帮我们引入了进来，我看随便拿几个来看

```
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,



org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,



org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,



org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,
```

比如我们经常用的security，可以看到已经帮你配置好，所以我们的EnableAutoConfiguration主要作用就是让你自动去配置，但并不是所有都是创建好的，是根据你程序去进行决定。 那我们继续来看

```
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, 
classes = TypeExcludeFilter.class), @Filter(type = FilterType.CUSTOM, 

classes = AutoConfigurationExcludeFilter.class) })
```

这个注解大家应该都不陌生，扫描包，放入spring容器，那他在springboot当中做了什么策略呢？我们可以点跟烟去思考，帮我们做了一个排除策略，他在这里结合SpringBootConfiguration去使用，为什么是排除，因为不可能一上来全部加载，因为内存有限。

那么我们来总结下@SpringbootApplication：就是说，他已经把很多东西准备好，具体是否使用取决于我们的程序或者说配置，那我们到底用不用？那我们继续来看一行代码

```
public static void main(String[] args)
{
    SpringApplication.run(StartEurekaApplication.class, args);
}
```

那们来看下在执行run方法到底有没有用到哪些自动配置的东西，比如说内置的Tomcat，那我们来找找内置Tomcat，我们点进run

```
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return new SpringApplication(sources).run(args);
    }
```

然后他调用又一个run方法，我们点进来看

```
public ConfigurableApplicationContext run(String... args) {
   //计时器
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
  ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   //监听器
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);



      Banner printedBanner = printBanner(environment);



      //准备上下文



      context = createApplicationContext();



      analyzers = new FailureAnalyzers(context);



         //预刷新context



      prepareContext(context, environment, listeners, applicationArguments,



            printedBanner);



     //刷新context



      refreshContext(context);



     //刷新之后的context



      afterRefresh(context, applicationArguments);



      listeners.finished(context, null);



      stopWatch.stop();



      if (this.logStartupInfo) {



         new StartupInfoLogger(this.mainApplicationClass)



               .logStarted(getApplicationLog(), stopWatch);



      }



      return context;



   }



   catch (Throwable ex) {



      handleRunFailure(context, listeners, analyzers, ex);



      throw new IllegalStateException(ex);



   }



}
```

那我们关注的就是 refreshContext(context); 刷新context，我们点进来看

```
private void refreshContext(ConfigurableApplicationContext context) {



   refresh(context);



   if (this.registerShutdownHook) {



      try {



         context.registerShutdownHook();



      }



      catch (AccessControlException ex) {



         // Not allowed in some environments.



      }



   }



}
```

我们继续点进refresh(context);

```
protected void refresh(ApplicationContext applicationContext) {



   Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);



   ((AbstractApplicationContext) applicationContext).refresh();



}
```

他会调用 ((AbstractApplicationContext) applicationContext).refresh();方法，我们点进来看

```
public void refresh() throws BeansException, IllegalStateException {



   synchronized (this.startupShutdownMonitor) {



      // Prepare this context for refreshing.



      prepareRefresh();



 



      // Tell the subclass to refresh the internal bean factory.



      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();



 



      // Prepare the bean factory for use in this context.



      prepareBeanFactory(beanFactory);



 



      try {



         // Allows post-processing of the bean factory in context subclasses.



         postProcessBeanFactory(beanFactory);



 



         // Invoke factory processors registered as beans in the context.



         invokeBeanFactoryPostProcessors(beanFactory);



 



         // Register bean processors that intercept bean creation.



         registerBeanPostProcessors(beanFactory);



 



         // Initialize message source for this context.



         initMessageSource();



 



         // Initialize event multicaster for this context.



         initApplicationEventMulticaster();



 



         // Initialize other special beans in specific context subclasses.



         onRefresh();



 



         // Check for listener beans and register them.



         registerListeners();



 



         // Instantiate all remaining (non-lazy-init) singletons.



         finishBeanFactoryInitialization(beanFactory);



 



         // Last step: publish corresponding event.



         finishRefresh();



      }



 



      catch (BeansException ex) {



         if (logger.isWarnEnabled()) {



            logger.warn("Exception encountered during context initialization - " +



                  "cancelling refresh attempt: " + ex);



         }



 



         // Destroy already created singletons to avoid dangling resources.



         destroyBeans();



 



         // Reset 'active' flag.



         cancelRefresh(ex);



 



         // Propagate exception to caller.



         throw ex;



      }



 



      finally {



         // Reset common introspection caches in Spring's core, since we



         // might not ever need metadata for singleton beans anymore...



         resetCommonCaches();



      }



   }



}
```

这点代码似曾相识啊 没错，就是一个spring的bean的加载过程我在，解析springIOC加载过程的时候介绍过这里面的方法，如果你看过Spring源码的话 ，应该知道这些方法都是做什么的。现在我们不关心其他的，我们来看一个方法叫做 onRefresh();方法

```
protected void onRefresh() throws BeansException {



   // For subclasses: do nothing by default.



}
```

他在这里并没有实现，但是我们找他的其他实现，我们来找

![深入SpringBoot核心注解原理](media/java1-1558274426.jpg)

我们既然要找Tomcat那就肯定跟web有关，我们可以看到有个ServletWebServerApplicationContext

```
@Override



protected void onRefresh() {



   super.onRefresh();



   try {



      createWebServer();



   }



   catch (Throwable ex) {



      throw new ApplicationContextException("Unable to start web server", ex);



   }



}
```

我们可以看到有一个createWebServer();方法他是创建web容器的，而Tomcat不就是web容器，那他是怎么创建的呢，我们继续看

```
private void createWebServer() {



   WebServer webServer = this.webServer;



   ServletContext servletContext = getServletContext();



   if (webServer == null && servletContext == null) {



      ServletWebServerFactory factory = getWebServerFactory();



      this.webServer = factory.getWebServer(getSelfInitializer());



   }



   else if (servletContext != null) {



      try {



         getSelfInitializer().onStartup(servletContext);



      }



      catch (ServletException ex) {



         throw new ApplicationContextException("Cannot initialize servlet context",



               ex);



      }



   }



   initPropertySources();



}
```

factory.getWebServer(getSelfInitializer());他是通过工厂的方式创建的

```
public interface ServletWebServerFactory {



 



   WebServer getWebServer(ServletContextInitializer... initializers);



 



}
```

可以看到 它是一个接口，为什么会是接口。因为我们不止是Tomcat一种web容器。

![深入SpringBoot核心注解原理](media/java5-1558274426.jpg)

我们看到还有Jetty，那我们来看TomcatServletWebServerFactory

```
@Override



public WebServer getWebServer(ServletContextInitializer... initializers) {



   Tomcat tomcat = new Tomcat();



   File baseDir = (this.baseDirectory != null) ? this.baseDirectory



         : createTempDir("tomcat");



   tomcat.setBaseDir(baseDir.getAbsolutePath());



   Connector connector = new Connector(this.protocol);



   tomcat.getService().addConnector(connector);



   customizeConnector(connector);



   tomcat.setConnector(connector);



   tomcat.getHost().setAutoDeploy(false);



   configureEngine(tomcat.getEngine());



   for (Connector additionalConnector : this.additionalTomcatConnectors) {



      tomcat.getService().addConnector(additionalConnector);



   }



   prepareContext(tomcat.getHost(), initializers);



   return getTomcatWebServer(tomcat);



}
```

那这块代码，就是我们要寻找的内置Tomcat，在这个过程当中，我们可以看到创建Tomcat的一个流程。因为run方法里面加载的东西很多，所以今天就浅谈到这里。如果不明白的话， 我们在用另一种方式来理解下，

大家要应该都知道stater举点例子

```
<dependency>



    <groupId>org.springframework.boot</groupId>



    <artifactId>spring-boot-starter-data-redis</artifactId>



</dependency>



<dependency>



    <groupId>org.springframework.boot</groupId>



    <artifactId>spring-boot-starter-freemarker</artifactId>



</dependency>
```

所以我们不防不定义一个stater来理解下，我们做一个需求，就是定制化不同的人跟大家说你们好，我们来看

```
<parent>



    <groupId>org.springframework.boot</groupId>



    <artifactId>spring-boot-starter-parent</artifactId>



    <version>2.1.4.RELEASE</version>



    <relativePath/>



</parent>



<groupId>com.zgw</groupId>



<artifactId>gw-spring-boot-srater</artifactId>



<version>1.0-SNAPSHOT</version>



 



<dependencies>



    <dependency>



        <groupId>org.springframework.boot</groupId>



        <artifactId>spring-boot-autoconfigure</artifactId>



    </dependency>



</dependencies>
```

我们先来看maven配置写入版本号，如果自定义一个stater的话必须依赖spring-boot-autoconfigure这个包,我们先看下项目目录

![深入SpringBoot核心注解原理](media/java5-1558274426-1.jpg)

```
public class GwServiceImpl  implements GwService{



    @Autowired



    GwProperties properties;



 



    @Override



    public void Hello()



    {



        String name=properties.getName();



        System.out.println(name+"说:你们好啊");



    }



}
```

我们做的就是通过配置文件来定制name这个是具体实现

```
@Component



@ConfigurationProperties(prefix = "spring.gwname")



public class GwProperties {



 



    String name="zgw";



 



    public String getName() {



        return name;



    }



 



    public void setName(String name) {



        this.name = name;



    }



}
```

这个类可以通过@ConfigurationProperties读取配置文件

```
@Configuration



@ConditionalOnClass(GwService.class)  //扫描类



@EnableConfigurationProperties(GwProperties.class) //让配置类生效



public class GwAutoConfiguration {



 



    /**



    * 功能描述 托管给spring



    * @author zgw



    * @return



    */



    @Bean



    @ConditionalOnMissingBean



    public GwService gwService()



    {



        return new GwServiceImpl();



    }



}
```

这个为配置类，为什么这么写因为，spring-boot的stater都是这么写的，我们可以参照他仿写stater，以达到自动配置的目的，然后我们在通过spring.factories也来进行配置

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.gw.GwAutoConfiguration
```

然后这样一个简单的stater就完成了，然后可以进行maven的打包，在其他项目引入就可以使用，在这里列出代码地址

> https://github.com/zgw1469039806/gwspringbootsrater