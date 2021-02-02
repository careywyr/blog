# Spring源码分析（一） 从Spring Bean的生命周期谈起

Spring Bean的生命周期真的是面试的时候关于Spring的最高频的考点之一了，笔者曾经被这个问题问懵了不止一次，一直记不住那一大串的步骤，但实际上尝试去死记硬背那些步骤的我是错误的，表面上看只是背诵一个流程，实际上，这个流程牵扯到的知识点可是很多而且很有意思的。

下面这个图我想很多人应该都看过相同的或者相似的：

![Spring Bean的生命周期](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/0b9e6e25-19c1-4b12-98bd-9fef01c426d1-924724.jpg)

看起来还是挺长的对吧，但是我们其实可以把它划分成下面四个大的步骤：

1. 实例化Bean
2. 设置对象属性，依赖注入
3. 初始化
4. destroy：bean的销毁 

接下来，我会根据这个步骤，一步步的讲解相关的知识点，从生命周期出发，将Spring中重要的类或接口来一一说明。

## 一、实例化Bean

这第一步，容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入，可以理解成new xx()。但要注意的是，对于BeanFactory容器和ApplicationContext容器它们的实例化是有区别的。

- 对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean()进行实例化。
- 对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。

那么既然提到了BeanFactory和ApplicationContext，我就对这两个接口也简单介绍一下。它们分别位于**org.springframework.beans**和**org.springframework.context**两个包下，而这两个包，正是Spring IoC容器的基础。BeanFactory提供了框架最基础的一些功能，包括IOC的配置机制，提供Bean的各种定义，建立Bean之间的依赖关系等。ApplicationContext继承了BeanFactory，因此它拥有BeanFactory的所有功能，但它本身还继承了一些其他的接口，比如消息资源配置，事件发布等。

BeanFactory是Spring框架的基础设施，面向Spring。ApplicationContext面向Spring框架的开发者。所以我们平常开发的过程中其实也可以发现到，我们更多的会用上的是ApplicationContext相关的工具，而不是BeanFactory。

除此之外，在实例化这个步骤的前后，实际上还有隐藏任务，牵扯到的接口叫做**InstantiationAwareBeanPostProcessor**。它继承了**BeanPostProcessor**。可能有的人不知道BeanPostProcessor这个接口，那么我就先介绍下这个。

**BeanPostProcessor**也是位于**org.springframework.beans**下，又叫做**后置处理器**，它定义了一系列的回调方法用来让使用者可以自定义Bean实例化或者依赖解析等的逻辑。它本身只有两个方法: postProcessBeforeInitialization和postProcessAfterInitialization。如方法名所说，定义Bean的**初始化**前后的逻辑。这个接口在整个生命周期中都有着关联。

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

不要弄混了**BeanPostProcessor**中的方法我说的是**初始化**前后，而生命周期的第一步是实例化。这里用到的**InstantiationAwareBeanPostProcessor**它有自己的对于实例化逻辑处理的两个方法。postProcessBeforeInstantiation和postProcessAfterInstantiation。

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        return null;
    }

    default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        return true;
    }

    @Nullable
    default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        return null;
    }
}
```

通过查找调用postProcessBeforeInstantiation这个方法的地方，会追溯到创建Bean的关键方法。就是**AbstractAutowireCapableBeanFactory**下的createBean()。

```java
/**
 * Central method of this class: creates a bean instance,
 * populates the bean instance, applies post-processors, etc.
 * @see #doCreateBean
 */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
      throws BeanCreationException {
	// 里面的代码我下面挑出重点的步骤一步步的贴出来
}
```

通过查看源码对于此方法的注释就知道它有多重要了。它可以创建Bean的实例，对其进行属性填充，调用BeanPostProcesser等。第一步我们注意到的代码应该是

```java
Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
```

它实际上是比我们上面生命周期的图示还要之前的一步，就是获取**BeanDefinition**对象中的信息来对Bean class进行解析。所谓**BeanDefinition**，它就是对于Bean的一个描述。它包含了Bean的一些基础信息，如Bean的name、Bean的作用域、和其他Bean的关系等等。解析完之后再进行到下一步。

```java
// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
if (bean != null) {
  return bean;
}
```

这个注释其实也给出了它的意义了，它给予BeanPostProcessor们一个机会，可以返回一个代理而不是目标bean的实例。但我们翻阅到上面的**InstantiationAwareBeanPostProcessor**对于此方法的实现，会发现它返回的是null，所以默认的逻辑这里是不会return的。所以我们会走到下一步，doCreateBean。

```java
Object beanInstance = doCreateBean(beanName, mbdToUse, args);
if (logger.isTraceEnabled()) {
   logger.trace("Finished creating instance of bean '" + beanName + "'");
}
return beanInstance;
```

进入到这个方法，它的第一行注释也是简单粗暴，就是来实例化Bean的。

```java
// Instantiate the bean.
BeanWrapper instanceWrapper = null;
if (mbd.isSingleton()) {
   instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
}
if (instanceWrapper == null) {
   instanceWrapper = createBeanInstance(beanName, mbd, args);
}
```

但可以发现，createBeanInstance()这个方法返回的是一个**BeanWrapper**对象。**BeanWrapper**是对Bean的一个包装，它可以设置获取被包装的对象，获取被包装bean的属性描述器等。我们平时开发是不会直接使用这个类的。通过createBeanInstance()这个方法的注释我们也能明白，它的作用是对于指定的bean生成一个新的实例，这里可以使用合适的实例化策略，比如工厂方法，构造器注入或者简单实例化等。

```java
	/**
	 * Create a new instance for the specified bean, using an appropriate instantiation strategy:
	 * factory method, constructor autowiring, or simple instantiation.
	 */
	protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    //代码我也不贴了
  }
```

接下来这段代码，注释也有说明，就是将调用各种postProcessers来进行属性的合并，这里会进行一些注解的扫描。

```java
// Allow post-processors to modify the merged bean definition.
synchronized (mbd.postProcessingLock) {
   if (!mbd.postProcessed) {
      try {
         applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
      }
      catch (Throwable ex) {
         throw new BeanCreationException(mbd.getResourceDescription(), beanName,
               "Post-processing of merged bean definition failed", ex);
      }
      mbd.postProcessed = true;
   }
}
```

再然后是一段跟一个面试题紧密相关的代码，那就是Spring如何解决循环依赖。因为本文的重点是讲bean的生命周期，本来准备一起写完的，但写着写着发现要说的太多了，后面我会另外单独写文章来讲述如何解决循环依赖的问题。

```java
// Eagerly cache singletons to be able to resolve circular references
// even when triggered by lifecycle interfaces like BeanFactoryAware.
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
      isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
   if (logger.isTraceEnabled()) {
      logger.trace("Eagerly caching bean '" + beanName +
            "' to allow for resolving potential circular references");
   }
   addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

## 二、设置对象属性，依赖注入

结束了实例化之后，我们继续在createBean这个方法上继续往下走，到了**populateBean**这一步。

```java
// Initialize the bean instance.
Object exposedObject = bean;
try {
   populateBean(beanName, mbd, instanceWrapper);
   exposedObject = initializeBean(beanName, exposedObject, mbd);
}
catch (Throwable ex) {
   if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
      throw (BeanCreationException) ex;
   }
   else {
      throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
   }
}
```

对于这个方法，我们已开始注意到的就是我之前说过在实例化前后都是存在着隐藏任务，上面提到了实例化之前的postProcessBeforeInstantiation，那么如今已经实例化了，现在出现了实例化之后的方法postProcessAfterInstantiation，这里也是给InstantiationAwareBeanPostProcessors一个机会去自定义属性被赋值后的方式。在后面代码中回去执行这个后置处理器的方法。

```java
// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
// state of the bean before properties are set. This can be used, for example,
// to support styles of field injection.
if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
   for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
      if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
         return;
      }
   }
}
```

继续往下，我们能看到一个熟悉的单词，**Autowire**，不过这里是获取当前注入的方式，根据byName还是byType来获得要注入的属性。这里我也不深入说明了。

```java
PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

int resolvedAutowireMode = mbd.getResolvedAutowireMode();
if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
   MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
   // Add property values based on autowire by name if applicable.
   if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
      autowireByName(beanName, mbd, bw, newPvs);
   }
   // Add property values based on autowire by type if applicable.
   if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
      autowireByType(beanName, mbd, bw, newPvs);
   }
   pvs = newPvs;
}
```

然后就是populateBean的第一步我就提到的方法，如果存在后置处理器，那么会进行属性的校验和处理，同时第二个布尔变量是用来判断是否进行依赖校验的。

```java
boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

PropertyDescriptor[] filteredPds = null;
if (hasInstAwareBpps) {
   if (pvs == null) {
      pvs = mbd.getPropertyValues();
   }
   for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
      PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
      if (pvsToUse == null) {
         if (filteredPds == null) {
            filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
         }
         pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
         if (pvsToUse == null) {
            return;
         }
      }
      pvs = pvsToUse;
   }
}
if (needsDepCheck) {
   if (filteredPds == null) {
      filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
   }
   checkDependencies(beanName, mbd, filteredPds, pvs);
}
```

最后一步，就是将前面这些步骤得到的属性进行注入了。

```java
if (pvs != null) {
   applyPropertyValues(beanName, mbd, bw, pvs);
}
```

## 三、初始化

**populateBean()**方法结束后开始进入initializeBean()方法，正如方法名所表达的，就是bean的**初始化**。注意初始化和实例化不是一个意思，在生命周期中bean是先进行实例化再进行初始化的。

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareMethods(beanName, bean);
         return null;
      }, getAccessControlContext());
   }
   else {
      invokeAwareMethods(beanName, bean);
   }

   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   try {
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
   }
   if (mbd == null || !mbd.isSynthetic()) {
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }

   return wrappedBean;
}
```

首先第一步，先不管那些安全机制的代码，我们要关注的是invokeAwareMethods(beanName, bean);这行代码，这里就是调用所有的Aware类型的接口。Spring中Aware类型的接口可以让我们获取容器中的一些资源。

```java
private void invokeAwareMethods(String beanName, Object bean) {
   if (bean instanceof Aware) {
      if (bean instanceof BeanNameAware) {
         ((BeanNameAware) bean).setBeanName(beanName);
      }
      if (bean instanceof BeanClassLoaderAware) {
         ClassLoader bcl = getBeanClassLoader();
         if (bcl != null) {
            ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
         }
      }
      if (bean instanceof BeanFactoryAware) {
         ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
      }
   }
}
```

比如这里面出现的有三种Aware类型的接口: BeanNameAware, BeanClassLoaderAware,BeanFactoryAware。它们根据bean实现了某个接口来完成不同的工作。

- 如果实现了BeanNameAware接口，则通过setBeanName()方法将beanName设置给这个bean
- 如果实现了BeanClassLoaderAware接口，则通过setBeanClassLoader()方法，将ClassLoader传入
- 如果实现了BeanFactoryAware接口，则通过setBeanFactory()方法，将BeanFactory容器实例传入

结束完Aware接口的调用，继续往下走，到了applyBeanPostProcessorsBeforeInitialization()这个方法（因为if判断是这个bean是空或者这个bean是否是synthetic的，就是是否是应用自己定义的，显然我们自己的bean不是，所以if条件后半部分会是true）。这个方法就是处理我之前有提到过的BeanPostProcessor的postProcessBeforeInitialization()方法。**如果Bean实现了BeanPostProcessor接口，那么就会执行这个方法**。

之后，流程来到了invokeInitMethods()这一步，这一步就是执行bean的自定义初始化方法。如果bean实现了**InitializingBean**接口，就需要重写**afterPropertiesSet()**方法，则invokeInitMethods的时候会调用这个重写后的**afterPropertiesSet()**方法，如果bean自定义了init方法，则会调用指定的init方法（可通过xml中配置bean的init-method属性实现），这两个方法的先后顺序是: **afterPropertiesSet在前，自定义init在后**。

继续下去就是applyBeanPostProcessorsAfterInitialization()方法了，和前面的applyBeanPostProcessorsBeforeInitialization()方法前后呼应，这个就不必多说了。

## 四、销毁

后面的一段代码还是跟处理循环依赖有关系，就先不多说，直接到最后一步，registerDisposableBeanIfNecessary().

销毁这一步其实就是可以让我们自己自定义bean的销毁方法，用到的关键的接口是**DisposableBean**，它和**InitializingBean**接口类似，只是一个初始化阶段的init，一个是结束阶段的destroy。

```java
/**
 * Add the given bean to the list of disposable beans in this factory,
 * registering its DisposableBean interface and/or the given destroy method
 * to be called on factory shutdown (if applicable). Only applies to singletons.
 * @param beanName the name of the bean
 * @param bean the bean instance
 * @param mbd the bean definition for the bean
 * @see RootBeanDefinition#isSingleton
 * @see RootBeanDefinition#getDependsOn
 * @see #registerDisposableBean
 * @see #registerDependentBean
 */
protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
   AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
   if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
      if (mbd.isSingleton()) {
         // Register a DisposableBean implementation that performs all destruction
         // work for the given bean: DestructionAwareBeanPostProcessors,
         // DisposableBean interface, custom destroy method.
         registerDisposableBean(beanName, new DisposableBeanAdapter(
               bean, beanName, mbd, getBeanPostProcessorCache().destructionAware, acc));
      }
      else {
         // A bean with a custom scope...
         Scope scope = this.scopes.get(mbd.getScope());
         if (scope == null) {
            throw new IllegalStateException("No Scope registered for scope name '" + mbd.getScope() + "'");
         }
         scope.registerDestructionCallback(beanName, new DisposableBeanAdapter(
               bean, beanName, mbd, getBeanPostProcessorCache().destructionAware, acc));
      }
   }
}
```

## 代码示例

上面一大段一大段的源码可能看着让人有点难受，不如直接写个demo轻松一下。因为只是表现一下生命周期，那么我们的依赖就很简单，spring-context和spring-beans即可。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

我们先定义一个自定义的后置处理器，为了表现初始化前后和实例化前后的流程，实现**InstantiationAwareBeanPostProcessor**接口，并重写postProcessBeforeInstantiation()/postProcessAfterInstantiation()/postProcessBeforeInitialization()/postProcessAfterInitialization()这四个方法。

```java
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        System.out.println(beanName + "实例化前");
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "实例化后");
        return false;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化前");
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化后");
        return null;
    }

}
```

再定义一个“常规”的Bean，但我们也为了反应初始化和销毁的几个操作，让他实现了InitializingBean, DisposableBean, BeanNameAware三个接口。并实现对应的方法，Aware接口这里我就只实现一个了，只是为了表现注入Aware接口那个步骤。

```java
public class DemoBean implements InitializingBean, DisposableBean, BeanNameAware {

    public DemoBean() {
        System.out.println("demoBean实例化");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("demoBean afterPropertiesSet");
    }

    public void init() {
        System.out.println("demoBean init");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("demoBean destroy");
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("BeanNameAware setBeanName: "+ s);
    }
}
```

配置文件很简单，就两个bean。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans>
        <bean name="myInstantiationAwareBeanPostProcessor" class="cn.leafw.spring.beans.MyInstantiationAwareBeanPostProcessor" />
        <bean name="demoBean" class="cn.leafw.spring.beans.DemoBean" init-method="init">
        </bean>
    </beans>
</beans>
```

测试类也很简单粗暴，就单纯启动和结束。

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:application.xml");
        applicationContext.registerShutdownHook();
    }
}
```

点击运行，接下来就是意料之中的结果输出:

![测试结果](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/Spring%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%B5%8B%E8%AF%95.png)

因为属性注入不方便在这里面进行测试，但大家知道是在实例化和初始化之间就好。所以其实总的来说，Spring的生命周期理解起来不算很难，但大家对生命周期里面关键的接口或者类要有印象，ApplicationContext和BeanFactory就不用多说，**BeanPostProcessor**是一定要知道的，Spring中的很多功能的实现都跟其有关。比如@Autowired注解的原理，数据校验Validate的原理，都是有对应的处理器。

这篇博客写了我挺久的时间，就是越写发现自己对Spring的了解越浅薄，本以为啃完了Spring文档的我应该是能轻松看懂源码，但实际上还是有点吃力的，但看懂了之后真的神清气爽，后面还会继续在这个坑里多填一点东西，学到的越多发现不会的越多，一起进步吧。