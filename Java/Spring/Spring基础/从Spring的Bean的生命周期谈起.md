# 从Spring Bean的生命周期谈起

Spring Bean的生命周期真的是面试的时候关于Spring的最高频的考点之一了，笔者曾经被这个问题问懵了不止一次，一直记不住那一大串的步骤，但实际上尝试去死记硬背那些步骤的我是错误的，表面上看只是背诵一个流程，实际上，这个流程牵扯到的知识点可是很多而且很有意思的。

下面这个图我想很多人应该都看过相同的或者相似的：

![Spring Bean的生命周期](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/0b9e6e25-19c1-4b12-98bd-9fef01c426d1-924724.jpg)

看起来还是挺长的对吧，但是我们其实可以把它划分成下面几个大的步骤：

1. 实例化Bean
2. 设置对象属性，依赖注入
3. 注入Aware接口
4. 处理BeanPostProcessor，自定义的处理（前置处理和后置处理）
5. InitializingBean接口与init-method配置：执行我们自己定义的初始化方法使用
6. destroy：bean的销毁 

接下来，我会根据这个步骤，一步步的讲解相关的知识点，从生命周期出发，将Spring中重要的类或接口来一一说明。

## 一、实例化Bean

这第一步，容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入，可以理解成new xx()。但要注意的是，对于BeanFactory容器和ApplicationContext容器它们的实例化是有区别的。

- 对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean()进行实例化。
- 对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。

### BeanFactory和ApplicationContext

那么既然提到了BeanFactory和ApplicationContext，我就对这两个接口也简单介绍一下。它们分别位于**org.springframework.beans**和**org.springframework.context**两个包下，而这两个包，正是Spring IoC容器的基础。BeanFactory提供了框架最基础的一些功能，包括IOC的配置机制，提供Bean的各种定义，建立Bean之间的依赖关系等。ApplicationContext继承了BeanFactory，因此它拥有BeanFactory的所有功能，但它本身还继承了一些其他的接口，比如消息资源配置，事件发布等。

BeanFactory是Spring框架的基础设施，面向Spring。ApplicationContext面向Spring框架的开发者。所以我们平常开发的过程中其实也可以发现到，我们更多的会用上的是ApplicationContext相关的工具，而不是BeanFactory。

### Bean的实例化前后

除此之外，在实例化这个步骤的前后，实际上还有隐藏任务，牵扯到的接口叫做**InstantiationAwareBeanPostProcessor**。它继承了**BeanPostProcessor**。可能有的人不知道BeanPostProcessor这个接口，那么我就先介绍下这个。

**BeanPostProcessor**也是位于**org.springframework.beans**下，它定义了一系列的回调方法用来让使用者可以自定义Bean实例化或者依赖解析等的逻辑。它本身只有两个方法: postProcessBeforeInitialization和postProcessAfterInitialization。如方法名所说，定义Bean的**初始化**前后的逻辑。这个接口在整个生命周期中都有着关联。

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

因此不要弄混了这里我说的是**初始化**前后，而生命周期的第一步是实例化。这里用到的**InstantiationAwareBeanPostProcessor**它有自己的对于实例化逻辑处理的两个方法。postProcessBeforeInstantiation和postProcessAfterInstantiation。

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

再然后是一段跟一个面试题紧密相关的代码

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

