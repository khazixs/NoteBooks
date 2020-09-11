## 生命周期原理

Spring Bean的完整生命周期从创建Spring容器开始，直到最终Spring容器销毁Bean，这其中包含了一系列关键点。即程序员可以进行扩展的函数的执行时机。



1. 首先第一步容器启动，根据相关配置扫描到所有需要Spring容器管理的对象，执行invokeBeanFactoryPostProcessors(beanFactory)方法将class变成对应的实现了beanDefinition接口的对象

2. 然后将所有的beanDefinition对象放入到实现由BeanFactory接口的工厂类DefaultListableBeanFactory中的beanDefinitionMap中，key为beanName，并将所有的key都存入一个list中方便之后的遍历

3. 通过上一步中的list遍历beanDefinitionMap依次获取所有的beanDefinition对象并对其进行处理

4. 验证beanDefinition中包含的各种信息，如是否是抽象类，是否是单例，是否是懒加载等等，来决定是否对该对象的bean进行实例化

5. 通过了验证之后，会由beanDefinition中的class信息获得对象的Clazz对象

6. 根据clazz获取所有的构造方法，并且推断要使用的构造方法（推断构造方法与该对象的注入模型有关，如果没有设置要使用的注入模型，那么就使用默认的构造函数）

7. 通过反射由构造对象Constructor实例化该对象，此时还仅仅只是对象，还不是bean。

   —————————————————————————————实例化完成—————————————————————————————————

8. 合并beanDefinition（这一步实际中使用的较少，可以暂时忽略）

9. 提前暴露一个bean工厂对象，这里其实就是以beanName为key，以一个能生成代理对象的lambda表达式作为value，存放到三级缓存（一个Map对象，singletonFactories）中，这里其实就是在为循环依赖提供便利。

10. 填充属性——自动注入，这里是具体执行的是一个populateBean的方法，该方法中会执行getSingleton这个方法，而getSingleton这个方法会依次检查一二三级缓存，看需要的bean是否存在。

11. 这一步会执行部分aware接口，通过判断目标bean的对象上是否有实现Aware来执行，如BeanNameAware接口的setBeanName方法、BeanFactoryAware接口的setBeanFactory方法

12. 判断是否实现了BeanPostProcessor接口，执行其postProcessorBeforeInitialization方法。

13. 执行由注解配置（@PostConstructor）的spring生命周期回调方法

14. 执行接口版的生命周期回调方法（InitializingBean接口的afterPropertiesSet方法）和xml版（initmethod方法）的生命周期回调方法

15. beanPostProcessor的前置方法（AOP那些增强方法）,生成bean的代理对象

16. 将代理对象放到一级缓存单例池singletonObjects中以供使用.

    —————————————————————————————初始化完成—————————————————————————————————

17. 这是应用程序可以使用单例池中的bean对象。

    —————————————————————————————使用完成—————————————————————————————————

18. 调用DiposiableBean的destroy方法。

19. 调用xml中配置的destroy-method方法。

    —————————————————————————————销毁完成—————————————————————————————————

至此一个bean的所有生命周期全部结束。



ps：

单例池——singletonObjects——Map<String,Object> ——ConcurrentHashMap<>()

singletonsCurrentlyInCreation——正在创建的单例bean的名字集合

执行finishBeanFactoryInitialization(beanFactory)方法对所有bean进行了实例化和初始化。具体是其中的preInstantiateSingletons()方法

beforeSingletonCreation(beanName);

doCreateBean——实例化bean



## bean生命周期扩展点的解释

​	![img](https://images2015.cnblogs.com/blog/592104/201611/592104-20161127142806690-1797767117.jpg)

![img](https://images0.cnblogs.com/i/580631/201405/181453414212066.png)

![img](https://images0.cnblogs.com/i/580631/201405/181454040628981.png)

解说：

1. **BeanFactoryPostProcessor**的**postProcessorBeanFactory()**方法：若某个IoC容器内添加了实现了BeanFactoryPostProcessor接口的实现类Bean，那么在该容器中实例化任何其他Bean之前可以回调该Bean中的postPrcessorBeanFactory()方法来对Bean的配置元数据进行更改，比如从XML配置文件中获取到的配置信息。
2. **InstantiationAwareBeanPostProcessor**接口的**postProcessorBeforeInitialization()**用于修改所有bean的上下文信息。
3. Bean的实例化：Bean的实例化是使用反射实现的。
4. **InstantiationAwareBeanPostProcessor**接口的**postPropertyValues()**用于修改属性值，但是只对XML配置有效。
5. Bean属性注入：Bean实例化完成后，利用反射技术实现属性及依赖Bean的注入。
6. **BeanNameAware**的**setBeanName()**方法：如果某个Bean实现了BeanNameAware接口，那么Spring将会将Bean实例的ID传递给setBeanName()方法，在Bean类中新增一个beanName字段，并实现setBeanName()方法。
7. **BeanFactoryAware**的**setBeanFactory()**方法：如果某个Bean实现了BeanFactoryAware接口，那么Spring将会将创建Bean的BeanFactory传递给setBeanFactory()方法，在Bean类中新增了一个beanFactory字段用来保存BeanFactory的值，并实现setBeanFactory()方法。
8. **ApplicationContextAware**的**setApplicationContext()**方法：如果某个Bean实现了ApplicationContextAware接口，那么Spring将会将该Bean所在的上下文环境ApplicationContext传递给setApplicationContext()方法，在Bean类中新增一个ApplicationContext字段用来保存ApplicationContext的值，并实现setApplicationContext()方法。
9. **BeanPostProcessor**预初始化方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后，执行初始化之前会调用BeanPostProcessor中的**postProcessBeforeInitialization()**方法执行预初始化处理。**针对Spring上下文中的所有bean。**
10. **InitializingBean**的**afterPropertiesSet()**方法：如果Bean实现了InitializingBean接口，那么Bean在实例化完成后将会执行接口中的**afterPropertiesSet()**方法来进行初始化。**针对某个具体的bean**。
11. 自定义的inti-method指定的方法：如果配置文件中使用**init-method**属性指定了初始化方法，那么Bean在实例化完成后将会调用该属性指定的初始化方法进行Bean的初始化。
12. **BeanPostProcessor**初始化后方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后并且完成初始化调用后执行该接口中的**postProcessorAfterInitialization()**方法进行初始化后处理。**针对Spring上下文中的所有bean。**
13. 使用Bean：此时有关Bean的所有准备工作均已完成，Bean可以被程序使用了，它们将会一直驻留在应用上下文中，直到该上下文环境被销毁。
14. **DisposableBean**的**destory()**方法：如果Bean实现了DisposableBean接口，Spring将会在Bean实例销毁之前调用该接口的**destory()**方法，来完成一些销毁之前的处理工作。
15. 自定义的destory-method指定的方法：如果在配置文件中使用**destory-method**指定了销毁方法，那么在Bean实例销毁之前会调用该指定的方法完成一些销毁之前的处理工作。

*注意：*

***1、BeanFactoryPostProcessor接口与BeanPostProcessor接口的作用范围是整个上下文环境中，使用方法是单独新增一个类来实现这些接口，那么在处理其他Bean的某些时刻就会回调响应的接口中的方法。***

***2、BeanNameAware、BeanFactoryAware、ApplicationContextAware的作用范围的Bean范围，即仅仅对实现了该接口的指定Bean有效，所有其使用方法是在要使用该功能的Bean自己来实现该接口。***

***3、第10点与第11点所述的两个初始化方法作用是一样的，我们完全可以使用其中的一种即可，一般情况我们使用第11点所述的方式，尽量少的去来Bean中实现某些接口，保持其独立性，低耦合性，尽量不要与Spring代码耦合在一起。第14和第15也是如此。***

## 循环依赖的原理

循环依赖总结一下（假设A,B之间循环依赖）：
一级缓存singletonObject，也就是常说的单例池，是个Map
二级缓存earlySingletonObjects，也就是提前一点的单例池，哈哈，字面翻译,也是Map
三级缓存singletonFactories，这个Map有点特殊，因为这个Map的value存放的是一个lambda表达式
1、单例池不能存放原始对象，只能存放经过完整生命周期的对象，也就是java bean
2、A，B在注入属性都会执行一个addSingletonFactory方法，这个方法里面三级缓存就出现了，三级缓存put了key为beanName，value为一个lambda表达式
3、其实最容易绕晕的地方是，当B注入属性A的时候，执行populateBean注入一个bean的属性的时候会执行getSingleton这个方法，一定要记得！！populateBean方法体中没有直接调用getSingleton这个方法，但一定要记得，执行了这个方法
4、getSingleton这个方法，会依次到一级缓存，二级缓存，三级缓存中get(beanName)，很显然当B注入A属性的时候，一级，二级里面都没有内容，只有三级有，这时会执行lambda表达式，lambda表达式的作用就是生成代理对象！！然后把生成的代理对象存入二级缓存，并返回这个代理对象，B就会得到这个代理对象A，B就会认为这个代理对象A就是A的最后的bean对象，因此也就完成了对A的属性注入这步操作，接着依次执行B后续的操作，最后就完成了B的生命周期，B就成功变成了bean对象，B也就完成了使命
5、当B完成使命之后，A就会继续注入B，这时就会注入属性成功了，接下来开始执行AOP操作，因为上一步中A已经生成了代理对象A，也就是相当于完成了AOP，所以B就不执行AOP操作了，此时A就会执行最后一步操作，将代理对象A放入到单例池中去，这时A就会执行方法getSingleton，从二级缓存中获得了代理对象A，最后将其存入单例池，也就是一级缓存！好了，现在A和B都放入了单例池，圆满结束！！！！

## BeanFactory和FactoryBean的区别？

## 这些扩展点都有什么实际应用？

