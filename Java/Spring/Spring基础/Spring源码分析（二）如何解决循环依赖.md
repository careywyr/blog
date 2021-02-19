# Spring源码分析（二）如何解决循环依赖

在上一篇Spring源码分析中，我们跳过了一部分关于Spring解决循环依赖部分的代码，为了填上这个坑，我这里另开一文来好好讨论下这个问题。

首先解释下什么是循环依赖，其实很简单，就是有两个类它们互相都依赖了对方，如下所示:

```java
@Component
public class AService {

    @Autowired
    private BService bService;
}
```

```java
@Component
public class BService {
    
    @Autowired
    private AService aService;
}
```

AService和BService显然两者都在内部依赖了对方，单拎出来看仿佛看到了多线程中常见的死锁代码，但很显然Spring解决了这个问题，不然我们也不可能正常的使用它了。

所谓创建Bean实际上就是调用**getBean()** 方法，这个方法可以在**AbstractBeanFactory**这个类里面找到，这个方法一开始会调用**getSingleton()**方法。

```java
// Eagerly check singleton cache for manually registered singletons.
Object sharedInstance = getSingleton(beanName);
```

这个方法的实现长得很有意思，有着一堆if语句。

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            synchronized(this.singletonObjects) {
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                           	// 从三级缓存里取出放到二级缓存中
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }

    return singletonObject;
}
```

但这一坨if很好理解，就是一层层的去获取这个bean，首先从**singletonObjects**中获取，这里面存放的是**已经完全创建好的单例Bean**；如果取不到，那么就往下走，去**earlySingletonObjects**里面取，这个是**早期曝光的对象**；如果还是没有，那么再去第三级缓存**singletonFactories**里面获取,它是提前暴露的对象工厂，这里会从三级缓存里取出后放到二级缓存中。那么总的来说，Spring去获取一个bean的时候，其实并不是直接就从容器里面取，而是先从缓存里找，而且缓存一共有**三级**。那么从这个方法返回的并不一定是我们需要的bean，后面会调用**getObjectForBeanInstance()**方法去得到实例化后的bean，这里就不多说了。

但如果缓存里面的确是取不到bean呢？那么说明这个bean的确还未创建，需要去创建一个bean，这样我们就会去到前一篇生命周期中的创建bean的方法了。回顾下流程：实例化--属性注入--初始化--销毁。那么我们回到文章开头的例子，有ServiceA和ServiceB两个类。一般来说，Spring是按照自然顺序去创建bean，那么第一个要创建的是ServiceA。显然一开始缓存里是没有的，我们会来到创建bean的方法。首先进行实例化阶段，我们会来到第一个跟解决循环依赖有关的代码，在实例化阶段的代码中就可以找到。

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

首先看看第一行，**earlySingletonExposure**这个变量它会是什么值？

它是有一个条件表达式返回的，一个个来看，首先，mbd.isSingleton()。我们知道Spring默认的Bean的作用域都是单例的，因此这里正常来说都是返回true没问题。第二个，**this.allowCircularReference**，这个变量是标记是否允许循环引用，默认也是true。第三个，调用了一个方法,**isSingletonCurrentlyInCreation(beanName)**，进入该代码可以看出它是返回当前的bean是不是正常创建，显然也是true。因此这个**earlySingletonExposure**返回的就是true。

接下来就进入了if语句的实现里面了，也就是**addSingletonFactory()**这个方法。看到里面的代码中出现**singletonFactories**这个变量是不是很熟悉？翻到上面的**getSingleton()**就知道了，其实就是三级缓存，所以这个方法的作用是**通过三级缓存提前暴露一个工厂对象**。

```java
/**
 * Add the given singleton factory for building the specified singleton
 * if necessary.
 * <p>To be called for eager registration of singletons, e.g. to be able to
 * resolve circular references.
 * @param beanName the name of the bean
 * @param singletonFactory the factory for the singleton object
 */
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
   Assert.notNull(singletonFactory, "Singleton factory must not be null");
   synchronized (this.singletonObjects) {
      if (!this.singletonObjects.containsKey(beanName)) {
         this.singletonFactories.put(beanName, singletonFactory);
         this.earlySingletonObjects.remove(beanName);
         this.registeredSingletons.add(beanName);
      }
   }
}
```

接下来，回忆下上一章节说的实例化之后的步骤，就是属性注入了。这就意味着ServiceA需要将ServiceB注入进去，那么显然又要调用**getBean()**方法去获取ServiceB。ServiceB还没有创建，则也会进入这个**createBean()**方法，同样也会来到这一步依赖注入。ServiceB中依赖了ServiceA，则会调用**getBean()**去获取ServiceA。此时的获取ServiceA可就不是再创建Bean了，而是从缓存中获取。这个缓存就是上面**getSingleton()**这个方法里面我们看到的**singletonFactory**。那么这个singletonFactory哪里来的，就是这个**addSingletonFactory()**方法的第二个参数，即**getEarlyBeanReference()**方法。

```java
/**
 * Obtain a reference for early access to the specified bean,
 * typically for the purpose of resolving a circular reference.
 * @param beanName the name of the bean (for error handling purposes)
 * @param mbd the merged bean definition for the bean
 * @param bean the raw bean instance
 * @return the object to expose as bean reference
 */
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
   Object exposedObject = bean;
   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
         exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
      }
   }
   return exposedObject;
}
```

查看**bp.getEarlyBeanReference(exposedObject, beanName)**的实现，发现有两个，一个是spring-beans下的**SmartInstantiationAwareBeanPostProcessor**，一个是spring-aop下的**AbstractAutoProxyCreator**。我们在未使用AOP的情况下，取的还是第一种实现。

```java
default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
   return bean;
}
```

那么令人惊讶的是，这方法直接返回了bean，也就是说如果不考虑AOP的话，这个方法啥都没干，就是把实例化创建的对象直接返回了。如果考虑AOP的话调用的是另一个实现:

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
   Object cacheKey = getCacheKey(bean.getClass(), beanName);
   this.earlyProxyReferences.put(cacheKey, bean);
   return wrapIfNecessary(bean, beanName, cacheKey);
}
```

可以看出，如果使用了AOP的话，这个方法返回的实际上是bean的代理，并不是它本身。那么通过这部分我们可以认为，在没有使用AOP的情况下，三级缓存是没有什么用的，所谓三级缓存实际上只是跟Spring的AOP有关的。

好了我们现在是处于创建B的过程，但由于B依赖A，所以调用了获取A的方法，则A从三级缓存进入了二级缓存，得到了A的代理对象。当然我们不需要担心注入B的是A的代理对象会带来什么问题，**因为生成代理类的内部都是持有一个目标类的引用，当调用代理对象的方法的时候，实际上是会调用目标对象的方法的**，所以所以代理对象是没影响的。当然这里也反应了我们实际上从容器中要获取的对象实际上是代理对象而不是其本身。

那么我们再回到创建A的逻辑往下走，能看到后面实际上又调用了一次**getSingleton()**方法。传入的**allowEarlyReference**为false。

```java
if (earlySingletonExposure) {
   Object earlySingletonReference = getSingleton(beanName, false);
   if (earlySingletonReference != null) {
      if (exposedObject == bean) {
         exposedObject = earlySingletonReference;
      }
      ...
   }
}
```

翻看上面的**getSingleton()**代码可以看出，allowEarlyReference为false就相当于禁用三级缓存，代码只会执行到通过二级缓存get。

```java
singletonObject = this.earlySingletonObjects.get(beanName);
```

因为在前面我们在创建往B中注入A的时候已经从三级缓存取出来放到二级缓存中了，所以这里A可以通过二级缓存去取。再往下就是生命周期后面的代码了，就不再继续了。

那么现在就会有个疑问，**我们为什么非要三级缓存，直接用二级缓存似乎就足够了？**

看看上面**getEarlyBeanReference()**这个方法所在的类，它是SpringAOP自动代理的关键类，它实现了**SmartInstantiationAwareBeanPostProcessor**，也就是说它也是个后置处理器BeanPostProcessor，它有着自定义的初始化后的方法。

```java
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      if (this.earlyProxyReferences.remove(cacheKey) != bean) {
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```

很明显它这里是**earlyProxyReferences**缓存中找不到当前的bean的话就会去创建代理。也就是说SpringAOP希望在Bean初始化后进行创建代理。如果我们只使用二级缓存，也就是在这个地方

```java
 addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

直接调用**getEarlyBeanReference()**并将得到的早期引用放入二级缓存。这就意味着无论bean之间是否存在互相依赖，只要创建bean走到这一步都得去创建代理对象了。然而Spring并不想这么做，不信自己可以动手debug一下，如果ServiceA和ServiceB之间没有依赖关系的话，**getEarlyBeanReference()**这个方法压根就不会执行。总的来说就是，**如果不使用三级缓存直接使用二级缓存的话，会导致所有的Bean在实例化后就要完成AOP代理，这是没有必要的。**

最后我们重新梳理下流程，记得Spring创建Bean的时候是按照自然顺序的，所以A在前B在后：

![循环依赖创建Bean的流程](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/1613740800708.png)

我们首先进行A的创建，但由于依赖了B，所以开始创建B，同样的，对B进行属性注入的时候会要用到A，那么就会通过**getBean()**去获取A，A在实例化阶段会提前将对象放入三级缓存中，如果没有使用AOP，那么本质上就是这个bean本身，否则是AOP代理后的代理对象。三级缓存**singletonFactories**会将其存放进去。那么通过**getBean()**方法获取A的时候，核心其实在于**getSingleton()**方法， 它会将其从三级缓存中取出，然后放到二级缓存中去。而最终B创建结束回到A初始化的时候，会再次调用一次**getSingleton()**方法，此时入参的**allowEarlyReference**为false，因此是去二级缓存中取，得到真正需要的bean或代理对象，最后A创建结束，流程结束。

所以Spring解决循环依赖的原理大致就讲完了，但根据上述的结论，我们可以思考一个问题，什么情况的循环依赖是无法解决的？

根据上面的流程图，我们知道，要解决循环依赖首先一个大前提是**bean必须是单例**的，基于这个前提我们才值得继续讨论这个问题。然后根据上述总结，可以知道，每个bean都是要进行实例化的，也就是要执行构造器。所以能不能解决循环依赖问题其实跟依赖注入的方式有关。

依赖注入的方式有setter注入，构造器注入和Field方式。

Filed方式就是我们平时用的最多的，属性上加个@Autowired或者@Resource之类的注解，这个对解决循环依赖无影响；

如果A和B都是通过setter注入，显然对于执行构造器没有影响，所以不影响解决循环依赖；

如果A和B互相通过构造器注入，那么执行构造器的时候也就是实例化的时候，A在自己还没放入缓存的时候就去创建B了，那么B也是拿不到A的，因此会出错；

如果A中注入B的方式为setter，B中注入A为构造器，由于A先实例化，执行构造器，并创建缓存，都没有问题，继续属性注入，依赖了B然后走创建B的流程，获取A也可以从缓存里面能取到，流程一路通畅。

如果A中注入B的方式为构造器，B中注入A为setter，那么这个时候A先进入实例化方法，发现需要B，那么就会去创建B，而A还没放入三级缓存里，B再创建的时候去获取A就会获取失败。

好了，以上就是关于Spring解决循环依赖问题的所有内容，这个问题的答案我是很久之前就知道了，但真的只是知道答案，这次是自己看源码加debug一点点看才知道为啥是这个答案，虽然还做不到彻底学的通透，但的确能对这个问题的理解的更为深刻一点，再接再厉吧。

