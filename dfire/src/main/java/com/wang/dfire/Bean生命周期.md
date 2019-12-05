

获取Bean，这个时候InstantiationAwareBeanPostProcessor有机会通过postProcessBeforeInstantiation方法（执行后执行applyBeanPostProcessorsAfterInitialization方法）直接返回一个Bean对象，不走下面的流程。

* Bean 实例化(构造方法)

* 将实例的引用放入第三级缓存singletonFactories中

* Bean属性配置

  * 所有的InstantiationAwareBeanPostProcessor执行postProcessAfterInstantiation

  * 所有的 InstantiationAwareBeanPostProcessor执行postProcessProperties,postProcessPropertyValues
    * AutowiredAnnotationBeanPostProcessor 这里会去找Bean的@AutoWired和@Refrence的Bean(比如A依赖了B)并且去加载
      * 如果此依赖Bean（比如B）反过来依赖了原Bean（比如A）。首先去singletonObjects中寻找A，没有找到。那么会从第三级缓存singletonFactories将A取出来，放入earlySingletonObjects中.并且B依赖此引用A.等待B加载完成,A再依赖B
    * AutowiredAnnotationBeanPostProcessor  这里会去找到@Value注解的值，装配上

* Bean 初始
  * 实现了BeanNameAware 执行setBeanName
  * 实现了BeanClassLoaderAware执行setBeanClassLoader
  * 实现了BeanFactoryAware 执行setBeanFactory
  * 调用所有的BeanPostProcessor的postProcessorBeforeInitialization
    * 实现了ApplicationContextAware执行setApplicationContext，这个逻辑写在ApplicationContextAwareProcessor中，其实也是一个BeanPostProcessor，与setBeanName等不同
  * 实现了InitializingBean执行afterPropertiesSet
  * 执行配置的init-method
  * 调用所有的BeanPostProcessor的postProcessorAfterInitialization
* 加入singletonObjects，并且从earlySingletonObjects，singletonFactories中去除自身引用

* Bean销毁



InstantiationAwareBeanPostProcessor 继承了 BeanPostProcessor，在Bean实例化的时候起作用。由于也是BeanPostProcessor，理论上在Bean初始化的时候也能起作用。

三级缓存,均为HashMap：首先从singletonObjects取，其次从earlySingletonObjects中取，取不到再去singletonFactories中取。从singletonFactories取之后放入earlySingletonObjects中。

spring借此机制完成了Bean循环依赖的解决。但是如果是通过构造方法的循环依赖,AB的构造方法中均依赖了对方，那么循环依赖不能解决。
