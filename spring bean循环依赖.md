# bean的生命周期

Spring Bean的生命周期是Spring面试热点问题。这个问题即考察对Spring的微观了解，又考察对Spring的宏观认识，想要答好并不容易！本文希望能够从源码角度入手，帮助面试者彻底搞定Spring Bean的生命周期。



## 面试版

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果Bean实现了 `BeanFactoryAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoade` r对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)

### 只有四个！

是的，Spring Bean的生命周期只有这四个阶段。把这四个阶段和每个阶段对应的扩展点糅合在一起虽然没有问题，但是这样非常凌乱，难以记忆。要彻底搞清楚Spring的生命周期，首先要把这四个阶段牢牢记住。实例化和属性赋值对应构造方法和setter方法的注入，初始化和销毁是用户能自定义扩展的两个阶段。在这四步之间穿插的各种扩展点，稍后会讲。

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

实例化 -> 属性赋值 -> 初始化 -> 销毁

主要逻辑都在doCreate()方法中，逻辑很清晰，就是顺序调用以下三个方法，这三个方法与三个生命周期阶段一一对应，非常重要，在后续扩展接口分析中也会涉及。

1. createBeanInstance() -> 实例化
2. populateBean() -> 属性赋值
3. initializeBean() -> 初始化

源码如下，能证明实例化，属性赋值和初始化这三个生命周期的存在。关于本文的Spring源码都将忽略无关部分，便于理解：



```dart
// 忽略了无关代码
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // Instantiate the bean.
   BeanWrapper instanceWrapper = null;
   if (instanceWrapper == null) {
       // 实例化阶段！
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }

   // Initialize the bean instance.
   Object exposedObject = bean;
   try {
       // 属性赋值阶段！
      populateBean(beanName, mbd, instanceWrapper);
       // 初始化阶段！
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }

   
   }
```

至于销毁，是在容器关闭时调用的，详见`ConfigurableApplicationContext#close()`

### 常用扩展点

Spring生命周期相关的常用扩展点非常多，所以问题不是不知道，而是记不住或者记不牢。其实记不住的根本原因还是不够了解，这里通过源码+分类的方式帮大家记忆。

#### 第一大类：影响多个Bean的接口

实现了这些接口的Bean会切入到多个Bean的生命周期中。正因为如此，这些接口的功能非常强大，Spring内部扩展也经常使用这些接口，例如自动注入以及AOP的实现都和他们有关。

- BeanPostProcessor
- InstantiationAwareBeanPostProcessor

这两兄弟可能是Spring扩展中**最重要**的两个接口！InstantiationAwareBeanPostProcessor作用于**实例化**阶段的前后，BeanPostProcessor作用于**初始化**阶段的前后。正好和第一、第三个生命周期阶段对应。通过图能更好理解：

![img](https:////upload-images.jianshu.io/upload_images/4558491-dc3eebbd1d6c65f4.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

未命名文件 (1).png



InstantiationAwareBeanPostProcessor实际上继承了BeanPostProcessor接口，严格意义上来看他们不是两兄弟，而是两父子。但是从生命周期角度我们重点关注其特有的对实例化阶段的影响，图中省略了从BeanPostProcessor继承的方法。



```dart
InstantiationAwareBeanPostProcessor extends BeanPostProcessor
```

##### InstantiationAwareBeanPostProcessor源码分析：

- postProcessBeforeInstantiation调用点，忽略无关代码：



```tsx
@Override
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
            throws BeanCreationException {

        try {
            // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
            // postProcessBeforeInstantiation方法调用点，这里就不跟进了，
            // 有兴趣的同学可以自己看下，就是for循环调用所有的InstantiationAwareBeanPostProcessor
            Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
            if (bean != null) {
                return bean;
            }
        }
        
        try {   
            // 上文提到的doCreateBean方法，可以看到
            // postProcessBeforeInstantiation方法在创建Bean之前调用
            Object beanInstance = doCreateBean(beanName, mbdToUse, args);
            if (logger.isTraceEnabled()) {
                logger.trace("Finished creating instance of bean '" + beanName + "'");
            }
            return beanInstance;
        }
        
    }
```

可以看到，postProcessBeforeInstantiation在doCreateBean之前调用，也就是在bean实例化之前调用的，英文源码注释解释道该方法的返回值会替换原本的Bean作为代理，这也是Aop等功能实现的关键点。

- postProcessAfterInstantiation调用点，忽略无关代码：



```tsx
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {

   // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
   // state of the bean before properties are set. This can be used, for example,
   // to support styles of field injection.
   boolean continueWithPropertyPopulation = true;
    // InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation()
    // 方法作为属性赋值的前置检查条件，在属性赋值之前执行，能够影响是否进行属性赋值！
   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
         if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
               continueWithPropertyPopulation = false;
               break;
            }
         }
      }
   }

   // 忽略后续的属性赋值操作代码
}
```

可以看到该方法在属性赋值方法内，但是在真正执行赋值操作之前。其返回值为boolean，返回false时可以阻断属性赋值阶段（`continueWithPropertyPopulation = false;`）。

关于BeanPostProcessor执行阶段的源码穿插在下文Aware接口的调用时机分析中，因为部分Aware功能的就是通过他实现的!只需要先记住BeanPostProcessor在初始化前后调用就可以了。

#### 第二大类：只调用一次的接口

这一大类接口的特点是功能丰富，常用于用户自定义扩展。
 第二大类中又可以分为两类：

1. Aware类型的接口
2. 生命周期接口

##### 无所不知的Aware

Aware类型的接口的作用就是让我们能够拿到Spring容器中的一些资源。基本都能够见名知意，Aware之前的名字就是可以拿到什么资源，例如`BeanNameAware`可以拿到BeanName，以此类推。调用时机需要注意：所有的Aware方法都是在初始化阶段之前调用的！
 Aware接口众多，这里同样通过分类的方式帮助大家记忆。
 Aware接口具体可以分为两组，至于为什么这么分，详见下面的源码分析。如下排列顺序同样也是Aware接口的执行顺序，能够见名知意的接口不再解释。

```
Aware Group1
```

1. BeanNameAware
2. BeanClassLoaderAware
3. BeanFactoryAware

```
Aware Group2
```

1. EnvironmentAware
2. EmbeddedValueResolverAware 这个知道的人可能不多，实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用，非常方便。
3. ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware) 这几个接口可能让人有点懵，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口，如下：



```kotlin
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
        MessageSource, ApplicationEventPublisher, ResourcePatternResolver {}
```

这里涉及到另一道面试题，ApplicationContext和BeanFactory的区别，可以从ApplicationContext继承的这几个接口入手，除去BeanFactory相关的两个接口就是ApplicationContext独有的功能，这里不详细说明。

##### Aware调用时机源码分析

详情如下，忽略了部分无关代码。代码位置就是我们上文提到的initializeBean方法详情，这也说明了Aware都是在初始化阶段之前调用的！



```dart
    // 见名知意，初始化阶段调用的方法
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {

        // 这里调用的是Group1中的三个Bean开头的Aware
        invokeAwareMethods(beanName, bean);

        Object wrappedBean = bean;
        
        // 这里调用的是Group2中的几个Aware，
        // 而实质上这里就是前面所说的BeanPostProcessor的调用点！
        // 也就是说与Group1中的Aware不同，这里是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        // 下文即将介绍的InitializingBean调用点
        invokeInitMethods(beanName, wrappedBean, mbd);
        // BeanPostProcessor的另一个调用点
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

        return wrappedBean;
    }
```

可以看到并不是所有的Aware接口都使用同样的方式调用。Bean××Aware都是在代码中直接调用的，而ApplicationContext相关的Aware都是通过BeanPostProcessor#postProcessBeforeInitialization()实现的。感兴趣的可以自己看一下ApplicationContextAwareProcessor这个类的源码，就是判断当前创建的Bean是否实现了相关的Aware方法，如果实现了会调用回调方法将资源传递给Bean。
 至于Spring为什么这么实现，应该没什么特殊的考量。也许和Spring的版本升级有关。基于对修改关闭，对扩展开放的原则，Spring对一些新的Aware采用了扩展的方式添加。

BeanPostProcessor的调用时机也能在这里体现，包围住invokeInitMethods方法，也就说明了在初始化阶段的前后执行。

关于Aware接口的执行顺序，其实只需要记住第一组在第二组执行之前就行了。每组中各个Aware方法的调用顺序其实没有必要记，有需要的时候点进源码一看便知。

##### 简单的两个生命周期接口

至于剩下的两个生命周期接口就很简单了，实例化和属性赋值都是Spring帮助我们做的，能够自己实现的有初始化和销毁两个生命周期阶段。

1. InitializingBean 对应生命周期的初始化阶段，在上面源码的`invokeInitMethods(beanName, wrappedBean, mbd);`方法中调用。
    有一点需要注意，因为Aware方法都是执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展Spring的常用方式。
    除了实现InitializingBean接口之外还能通过注解或者xml配置的方式指定初始化方法，至于这几种定义方式的调用顺序其实没有必要记。因为这几个方法对应的都是同一个生命周期，只是实现方式不同，我们一般只采用其中一种方式。
2. DisposableBean 类似于InitializingBean，对应生命周期的销毁阶段，以ConfigurableApplicationContext#close()方法作为入口，实现是通过循环取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 。感兴趣的可以自行跟一下源码。

### 扩展阅读: BeanPostProcessor 注册时机与执行顺序

##### 注册时机

我们知道BeanPostProcessor也会注册为Bean，那么Spring是如何保证BeanPostProcessor在我们的业务Bean之前初始化完成呢？
 请看我们熟悉的refresh()方法的源码，省略部分无关代码：



```java
@Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {

            try {
                // Allows post-processing of the bean factory in context subclasses.
                postProcessBeanFactory(beanFactory);

                // Invoke factory processors registered as beans in the context.
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation.
                // 所有BeanPostProcesser初始化的调用点
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
                // 所有单例非懒加载Bean的调用点
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                finishRefresh();
            }

    }
```

可以看出，Spring是先执行registerBeanPostProcessors()进行BeanPostProcessors的注册，然后再执行finishBeanFactoryInitialization初始化我们的单例非懒加载的Bean。

##### 执行顺序

BeanPostProcessor有很多个，而且每个BeanPostProcessor都影响多个Bean，其执行顺序至关重要，必须能够控制其执行顺序才行。关于执行顺序这里需要引入两个排序相关的接口：PriorityOrdered、Ordered

- PriorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序
- Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序
- 都没有实现是三等公民，最后执行

在以下源码中，可以很清晰的看到Spring注册各种类型BeanPostProcessor的逻辑，根据实现不同排序接口进行分组。优先级高的先加入，优先级低的后加入。



```kotlin
// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
// 首先，加入实现了PriorityOrdered接口的BeanPostProcessors，顺便根据PriorityOrdered排了序
            String[] postProcessorNames =
                    beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
                if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
            currentRegistryProcessors.clear();

            // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
// 然后，加入实现了Ordered接口的BeanPostProcessors，顺便根据Ordered排了序
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
                if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                }
            }
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
            currentRegistryProcessors.clear();

            // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
// 最后加入其他常规的BeanPostProcessors
            boolean reiterate = true;
            while (reiterate) {
                reiterate = false;
                postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
                for (String ppName : postProcessorNames) {
                    if (!processedBeans.contains(ppName)) {
                        currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                        processedBeans.add(ppName);
                        reiterate = true;
                    }
                }
                sortPostProcessors(currentRegistryProcessors, beanFactory);
                registryProcessors.addAll(currentRegistryProcessors);
                invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
                currentRegistryProcessors.clear();
            }
```

根据排序接口返回值排序，默认升序排序，返回值越低优先级越高。



```dart
    /**
     * Useful constant for the highest precedence value.
     * @see java.lang.Integer#MIN_VALUE
     */
    int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

    /**
     * Useful constant for the lowest precedence value.
     * @see java.lang.Integer#MAX_VALUE
     */
    int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
```

PriorityOrdered、Ordered接口作为Spring整个框架通用的排序接口，在Spring中应用广泛，也是非常重要的接口。

### 总结

Spring Bean的生命周期分为`四个阶段`和`多个扩展点`。扩展点又可以分为`影响多个Bean`和`影响单个Bean`。整理如下：
 四个阶段

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction

多个扩展点

- 影响多个Bean
  - BeanPostProcessor
  - InstantiationAwareBeanPostProcessor
- 影响单个Bean
  - Aware
    - Aware Group1
      - BeanNameAware
      - BeanClassLoaderAware
      - BeanFactoryAware
    - Aware Group2
      - EnvironmentAware
      - EmbeddedValueResolverAware
      - ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
  - 生命周期
    - InitializingBean
    - DisposableBean







# Spring中的循环依赖

Spring中的循环依赖一直是Spring中一个很重要的话题，一方面是因为源码中为了解决循环依赖做了很多处理，另外一方面是因为面试的时候，如果问到Spring中比较高阶的问题，那么循环依赖必定逃不掉。

如果你回答得好，那么这就是你的必杀技，反正，那就是面试官的必杀技，这也是取这个标题的原因，当然，本文的目的是为了让你在之后的所有面试中能多一个必杀技，专门用来绝杀面试官！

本文的核心思想就是，

当面试官问：

“请讲一讲Spring中的循环依赖。”的时候，

我们到底该怎么回答？

主要分下面几点

1. 什么是循环依赖？
2. 什么情况下循环依赖可以被处理？
3. Spring是如何解决的循环依赖？

同时本文希望纠正几个目前业界内经常出现的几个关于循环依赖的错误的说法

1. 只有在setter方式注入的情况下，循环依赖才能解决（**错**）
2. 三级缓存的目的是为了提高效率（**错**）

OK，铺垫已经做完了，接下来我们开始正文

# 什么是循环依赖？

从字面上来理解就是A依赖B的同时B也依赖了A，就像下面这样

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


体现到代码层次就是这个样子

```
@Component
public class A {
    // A中注入了B
 @Autowired
 private B b;
}

@Component
public class B {
    // B中也注入了A
 @Autowired
 private A a;
}
```

当然，这是最常见的一种循环依赖，比较特殊的还有

```
// 自己依赖自己
@Component
public class A {
    // A中注入了A
 @Autowired
 private A a;
}
```

虽然体现形式不一样，但是实际上都是同一个问题----->循环依赖

# 什么情况下循环依赖可以被处理？

在回答这个问题之前首先要明确一点，Spring解决循环依赖是有前置条件的

1. 出现循环依赖的Bean必须要是单例
2. 依赖注入的方式不能全是构造器注入的方式（很多博客上说，只能解决setter方法的循环依赖，这是错误的）

其中第一点应该很好理解，第二点：不能全是构造器注入是什么意思呢？我们还是用代码说话

```
@Component
public class A {
// @Autowired
// private B b;
 public A(B b) {

 }
}


@Component
public class B {

// @Autowired
// private A a;

 public B(A a){

 }
}
```

在上面的例子中，A中注入B的方式是通过构造器，B中注入A的方式也是通过构造器，这个时候循环依赖是无法被解决，如果你的项目中有两个这样相互依赖的Bean，在启动时就会报出以下错误：

```
Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

为了测试循环依赖的解决情况跟注入方式的关系，我们做如下四种情况的测试

| 依赖情况               | 依赖注入方式                                       | 循环依赖是否被解决 |
| :--------------------- | :------------------------------------------------- | :----------------- |
| AB相互依赖（循环依赖） | 均采用setter方法注入                               | 是                 |
| AB相互依赖（循环依赖） | 均采用构造器注入                                   | 否                 |
| AB相互依赖（循环依赖） | A中注入B的方式为setter方法，B中注入A的方式为构造器 | 是                 |
| AB相互依赖（循环依赖） | B中注入A的方式为setter方法，A中注入B的方式为构造器 | 否                 |

具体的测试代码很简单，我就不放了。从上面的测试结果我们可以看到，不是只有在setter方法注入的情况下循环依赖才能被解决，即使存在构造器注入的场景下，循环依赖依然被可以被正常处理掉。

那么到底是为什么呢？Spring到底是怎么处理的循环依赖呢？不要急，我们接着往下看

# Spring是如何解决的循环依赖？

关于循环依赖的解决方式应该要分两种情况来讨论

1. 简单的循环依赖（没有AOP）
2. 结合了AOP的循环依赖

## **简单的循环依赖（没有AOP）**

我们先来分析一个最简单的例子，就是上面提到的那个demo

```
@Component
public class A {
    // A中注入了B
 @Autowired
 private B b;
}

@Component
public class B {
    // B中也注入了A
 @Autowired
 private A a;
}
```

通过上文我们已经知道了这种情况下的循环依赖是能够被解决的，那么具体的流程是什么呢？我们一步步分析

首先，我们要知道**Spring在创建Bean的时候默认是按照自然排序来进行创建的，所以第一步Spring会去创建A**。

与此同时，我们应该知道，Spring在创建Bean的过程中分为三步

1. 实例化，对应方法：`AbstractAutowireCapableBeanFactory`中的`createBeanInstance`方法
2. 属性注入，对应方法：`AbstractAutowireCapableBeanFactory`的`populateBean`方法
3. 初始化，对应方法：`AbstractAutowireCapableBeanFactory`的`initializeBean`

这些方法在之前源码分析的文章中都做过详细的解读了，如果你之前没看过我的文章，那么你只需要知道

1. 实例化，简单理解就是new了一个对象
2. 属性注入，为实例化中new出来的对象填充属性
3. 初始化，执行aware接口中的方法，初始化方法，完成`AOP`代理

基于上面的知识，我们开始解读整个循环依赖处理的过程，整个流程应该是以A的创建为起点，前文也说了，第一步就是创建A嘛！

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


创建A的过程实际上就是调用`getBean`方法，这个方法有两层含义

1. 创建一个新的Bean
2. 从缓存中获取到已经被创建的对象

我们现在分析的是第一层含义，因为这个时候缓存中还没有A嘛！

### 调用getSingleton(beanName)

首先调用`getSingleton(a)`方法，这个方法又会调用`getSingleton(beanName, true)`，在上图中我省略了这一步

```
public Object getSingleton(String beanName) {
    return getSingleton(beanName, true);
}
```

`getSingleton(beanName, true)`这个方法实际上就是到缓存中尝试去获取Bean，整个缓存分为三级

1. `singletonObjects`，一级缓存，存储的是所有创建好了的单例Bean
2. `earlySingletonObjects`，完成实例化，但是还未进行属性注入及初始化的对象
3. `singletonFactories`，提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

因为A是第一次被创建，所以不管哪个缓存中必然都是没有的，因此会进入`getSingleton`的另外一个重载方法`getSingleton(beanName, singletonFactory)`。

### 调用getSingleton(beanName, singletonFactory)

这个方法就是用来创建Bean的，其源码如下：

```
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {

            // ....
            // 省略异常处理及日志
            // ....

            // 在单例对象创建前先做一个标记
            // 将beanName放入到singletonsCurrentlyInCreation这个集合中
            // 标志着这个单例Bean正在创建
            // 如果同一个单例Bean多次被创建，这里会抛出异常
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                // 上游传入的lambda在这里会被执行，调用createBean方法创建一个Bean后返回
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            // ...
            // 省略catch异常处理
            // ...
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                // 创建完成后将对应的beanName从singletonsCurrentlyInCreation移除
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                // 添加到一级缓存singletonObjects中
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

上面的代码我们主要抓住一点，通过`createBean`方法返回的Bean最终被放到了一级缓存，也就是单例池中。

那么到这里我们可以得出一个结论：**一级缓存中存储的是已经完全创建好了的单例Bean**

### 调用addSingletonFactory方法

如下图所示：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


在完成Bean的实例化后，属性注入之前Spring将Bean包装成一个工厂添加进了三级缓存中，对应源码如下：

```
// 这里传入的参数也是一个lambda表达式，() -> getEarlyBeanReference(beanName, mbd, bean)
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            // 添加到三级缓存中
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这里只是添加了一个工厂，通过这个工厂（`ObjectFactory`）的`getObject`方法可以得到一个对象，而这个对象实际上就是通过`getEarlyBeanReference`这个方法创建的。那么，什么时候会去调用这个工厂的`getObject`方法呢？这个时候就要到创建B的流程了。

当A完成了实例化并添加进了三级缓存后，就要开始为A进行属性注入了，在注入时发现A依赖了B，那么这个时候Spring又会去`getBean(b)`，然后反射调用setter方法完成属性注入。

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


因为B需要注入A，所以在创建B的时候，又会去调用`getBean(a)`，这个时候就又回到之前的流程了，但是不同的是，之前的`getBean`是为了创建Bean，而此时再调用`getBean`不是为了创建了，而是要从缓存中获取，因为之前A在实例化后已经将其放入了三级缓存`singletonFactories`中，所以此时`getBean(a)`的流程就是这样子了

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


从这里我们可以看出，注入到B中的A是通过`getEarlyBeanReference`方法提前暴露出去的一个对象，还不是一个完整的Bean，那么`getEarlyBeanReference`到底干了啥了，我们看下它的源码

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

它实际上就是调用了后置处理器的`getEarlyBeanReference`，而真正实现了这个方法的后置处理器只有一个，就是通过`@EnableAspectJAutoProxy`注解导入的`AnnotationAwareAspectJAutoProxyCreator`。**也就是说如果在不考虑****`AOP`的情况下，上面的代码等价于：**

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    return exposedObject;
}
```

**也就是说这个工厂啥都没干，直接将实例化阶段创建的对象返回了！所以说在不考虑****`AOP`的情况下三级缓存有用嘛？讲道理，真的没什么用**，我直接将这个对象放到二级缓存中不是一点问题都没有吗？如果你说它提高了效率，那你告诉我提高的效率在哪?

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


那么三级缓存到底有什么作用呢？不要急，我们先把整个流程走完，在下文结合`AOP`分析循环依赖的时候你就能体会到三级缓存的作用！

到这里不知道小伙伴们会不会有疑问，B中提前注入了一个没有经过初始化的A类型对象不会有问题吗？

答：不会

这个时候我们需要将整个创建A这个Bean的流程走完，如下图：

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


从上图中我们可以看到，虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以这是没有问题的。

## **结合了AOP的循环依赖**

之前我们已经说过了，在普通的循环依赖的情况下，三级缓存没有任何作用。三级缓存实际上跟Spring中的`AOP`相关，我们再来看一看`getEarlyBeanReference`的代码：

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

如果在开启`AOP`的情况下，那么就是调用到`AnnotationAwareAspectJAutoProxyCreator`的`getEarlyBeanReference`方法，对应的源码如下：

```
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    // 如果需要代理，返回一个代理对象，不需要代理，直接返回当前传入的这个bean对象
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```

回到上面的例子，我们对A进行了`AOP`代理的话，那么此时`getEarlyBeanReference`将返回一个代理后的对象，而不是实例化阶段创建的对象，这样就意味着B中注入的A将是一个代理对象而不是A的实例化阶段创建后的对象。![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

看到这个图你可能会产生下面这些疑问

1. 在给B注入的时候为什么要注入一个代理对象？

答：当我们对A进行了`AOP`代理时，说明我们希望从容器中获取到的就是A代理后的对象而不是A本身，因此把A当作依赖进行注入时也要注入它的代理对象

1. 明明初始化的时候是A对象，那么Spring是在哪里将代理对象放入到容器中的呢？

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


在完成初始化后，Spring又调用了一次`getSingleton`方法，这一次传入的参数又不一样了，false可以理解为禁用三级缓存，前面图中已经提到过了，在为B中注入A时已经将三级缓存中的工厂取出，并从工厂中获取到了一个对象放入到了二级缓存中，所以这里的这个`getSingleton`方法做的时间就是从二级缓存中获取到这个代理后的A对象。`exposedObject == bean`可以认为是必定成立的，除非你非要在初始化阶段的后置处理器中替换掉正常流程中的Bean，例如增加一个后置处理器：

```
@Component
public class MyPostProcessor implements BeanPostProcessor {
 @Override
 public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  if (beanName.equals("a")) {
   return new A();
  }
  return bean;
 }
}
```

不过，请不要做这种骚操作，徒增烦恼！

1. 初始化的时候是对A对象本身进行初始化，而容器中以及注入到B中的都是代理对象，这样不会有问题吗？

答：不会，这是因为不管是`cglib`代理还是`jdk`动态代理生成的代理类，内部都持有一个目标类的引用，当调用代理对象的方法时，实际会去调用目标对象的方法，A完成初始化相当于代理对象自身也完成了初始化

1. 三级缓存为什么要使用工厂而不是直接使用引用？换而言之，为什么需要这个三级缓存，直接通过二级缓存暴露一个引用不行吗？

答：**这个工厂的目的在于延迟对实例化阶段生成的对象的代理，只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象**

我们思考一种简单的情况，就以单独创建A为例，假设AB之间现在没有依赖关系，但是A被代理了，这个时候当A完成实例化后还是会进入下面这段代码：

```
// A是单例的，mbd.isSingleton()条件满足
// allowCircularReferences：这个变量代表是否允许循环依赖，默认是开启的，条件也满足
// isSingletonCurrentlyInCreation：正在在创建A，也满足
// 所以earlySingletonExposure=true
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
// 还是会进入到这段代码中
if (earlySingletonExposure) {
 // 还是会通过三级缓存提前暴露一个工厂对象
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

看到了吧，即使没有循环依赖，也会将其添加到三级缓存中，而且是不得不添加到三级缓存中，因为到目前为止Spring也不能确定这个Bean有没有跟别的Bean出现循环依赖。

假设我们在这里直接使用二级缓存的话，那么意味着所有的Bean在这一步都要完成`AOP`代理。这样做有必要吗？

不仅没有必要，而且违背了Spring在结合`AOP`跟Bean的生命周期的设计！Spring结合`AOP`跟Bean的生命周期本身就是通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来完成的，在这个后置处理的`postProcessAfterInitialization`方法中对初始化后的Bean完成`AOP`代理。如果出现了循环依赖，那没有办法，只有给Bean先创建代理，但是没有出现循环依赖的情况下，设计之初就是让Bean在生命周期的最后一步完成代理而不是在实例化后就立马完成代理。

## **三级缓存真的提高了效率了吗？**

现在我们已经知道了三级缓存的真正作用，但是这个答案可能还无法说服你，所以我们再最后总结分析一波，三级缓存真的提高了效率了吗？分为两点讨论：

1. 没有进行`AOP`的Bean间的循环依赖

从上文分析可以看出，这种情况下三级缓存根本没用！所以不会存在什么提高了效率的说法

1. 进行了`AOP`的Bean间的循环依赖

就以我们上的A、B为例，其中A被`AOP`代理，我们先分析下使用了三级缓存的情况下，A、B的创建流程

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


假设不使用三级缓存，直接在二级缓存中

![img](spring%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.assets/640)


上面两个流程的唯一区别在于为A对象创建代理的时机不同，在使用了三级缓存的情况下为A创建代理的时机是在B中需要注入A的时候，而不使用三级缓存的话在A实例化后就需要马上为A创建代理然后放入到二级缓存中去。

对于整个A、B的创建过程而言，消耗的时间是一样的。

综上，不管是哪种情况，三级缓存提高了效率这种说法都是错误的！

# 总结

面试官：”Spring是如何解决的循环依赖？“

答：Spring通过三级缓存解决了循环依赖，其中一级缓存为单例池（`singletonObjects`）,二级缓存为早期曝光对象`earlySingletonObjects`，三级缓存为早期曝光对象工厂（`singletonFactories`）。

当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。

当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取：

第一步，先获取到三级缓存中的工厂；

第二步，调用对象工工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。

当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。至此，循环依赖结束！

面试官：”为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？“

答：如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。

# 一道思考题