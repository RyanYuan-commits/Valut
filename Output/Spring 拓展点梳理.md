---
type: permanent
---
---

**关键词**: Spring

---

## 1	BeanFactoryPostProcessor 及其子接口

### 1.1	BeanFactoryPostProcessor

```java
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

允许 Bean 操控 Bean Factory, 来添加, 修改或删除 Bean Definition. Spring 调用 postProcessBeanFactory 方法时, 所有通过标准流程加载的 Bean Definition 加载完成, 但尚未实例化任何 Bean. 

Dubbo 中处理 @DubboReference 时, 借助 BeanFactoryPostProcessor 的能力, 构造新的 Bean 并将其存放到 ConfigurableListableBeanFactory 中, ConfigurableListableBeanFactory 的继承关系为:

![[ConfigurableListableBeanFactory类图.png]]

ConfigurableListableBeanFactory 并没有实现 BeanDefinitionRegistry 接口, 所以其只具备修改现有 BeanDefinition 的能力, 无法创建新的 BeanDefinition.

### 1.2	BeanDefinitionRegistryPostProcessor

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

    void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;

}
```

BeanDefinitionRegistryPostProcessor 是 BeanFactoryPostProcessor 的子接口, 其基础上拓展了创建新 BeanDefinition 的能力.

### 1.3	执行流程

执行位置: PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors:

```java
public static void invokeBeanFactoryPostProcessors(
        ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    Set<String> processedBeans = new HashSet<>();

    if (beanFactory instanceof BeanDefinitionRegistry registry) {
        List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
        List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

        for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor registryProcessor) {
                registryProcessor.postProcessBeanDefinitionRegistry(registry);
                registryProcessors.add(registryProcessor);
            }
            else {
                regularPostProcessors.add(postProcessor);
            }
        }

        List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

        // ... 执行 BeanDefinitionRegistryPostProcessor 的 postProcessBeanDefinitionRegistry 方法

        invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
        invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }

    // ......
}
```

这个方法负责调用已注册的 BeanFactoryPostProcessor Bean, 分阶段处理, 优先执行实现了 BeanDefinitionRegistryPostProcessor 接口的类, 确保后续 BeanFactoryPostProcessor 处理时 BeanDefinition 是全量的; 

## 2	Aware 接口及其子接口

### 2.1	ApplicationContextAware

```java
public interface ApplicationContextAware extends Aware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;  
    
}
```

实现 ApplicationContextAware 能够使 Bean 感知到 ApplicationContext 实例; 

setApplicationContext 方法在 populateBean 之后，但在 init 回调 (例如 InitializingBean#afterPropertiesSet() 或自定义 init-method) 之前调用.

App最常用的 Aware 接口, 可以让 Bean 感知到外部的 ApplicationContext, 从而可以使用上下文来动态获取 Bean, 发布或监听自定义事件, 访问 Spring 基础设施等; 

但是, Bean 的属性填充和初始化是在 BeanFactory 中进行的, 而 ApplicaitonContext 和 BeanFactory 是依赖关系, 所以在 BeanFactory 无法感知外部的 ApplicaitonContext 对象. 所以, 需要有一个 Bean 在进入 BeanFactory 的领域之前保留下对 ApplicationContext 的引用, 这个 Bean 的类型是 ApplicationContextAwareProcessor.

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor{}
```

ApplicationContextAwareProcessor 实现了 BeanPostProcessor 接口, 能够在 Bean 初始化之前对 Bean 进行修改, ApplicationContext 就是在这里被注入的.

```java
private void invokeAwareInterfaces(Object bean) {  
    if (bean instanceof EnvironmentAware) {  
       ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());  
    }  
    if (bean instanceof EmbeddedValueResolverAware) {  
       ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);  
    }  
    if (bean instanceof ResourceLoaderAware) {  
       ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);  
    }  
    if (bean instanceof ApplicationEventPublisherAware) {  
       ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);  
    }  
    if (bean instanceof MessageSourceAware) {  
       ((MessageSourceAware) bean).setMessageSource(this.applicationContext);  
    }  
    if (bean instanceof ApplicationStartupAware) {  
       ((ApplicationStartupAware) bean).setApplicationStartup(this.applicationContext.getApplicationStartup());  
    }  
    if (bean instanceof ApplicationContextAware) {  
       ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);  
    }  
}
```

在 ApplicationContextAwareProcessor 的 invokeAwareInterfaces 中, 还调用了其他 Aware 接口的 set 方法:

- EnvironmentAware: 应用运行环境感知类, 运行环境一般指的是配置文件 profiles 和 各种属性 properties, 实现这个接口能让 Bean 获取到当前的 Environment 对象.
	
- EmbeddedValueResolverAware: 注入字符串值解析器, 解析占位符和表达式，如 ${property}.
	
- ResourceLoaderAware: 注入资源加载器, 加载 classpath 或文件系统中的资源.
	
- ApplicationEventPublisherAware: 注入事件发布器, 如果想要发布事件但是又不想依赖于整个 ApplicationContext 的时候, 可以实现这个接口.
	
- MessageSourceAware: 注入消息源, 支持国际化消息解析.

### 2.2	BeanFactoryAware

```java
public interface BeanFactoryAware extends Aware {  
    void setBeanFactory(BeanFactory beanFactory) throws BeansException;  
}
```

向 bean 实例提供其所属的工厂. 在填充普通 bean 属性之后, 但在初始化回调之前调用, 适用于需要直接与底层的 BeanFactory 交互的场景, 进行更低级别的 bean 管理, 比如动态获取原型作用域的 bean 实例。

setBeanFactory 在 AbstractAutowireCapableBeanFactory#initializeBean 中被调用: 

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
  
    // ......
}

private void invokeAwareMethods(String beanName, Object bean) {  
    if (bean instanceof Aware) {  
       if (bean instanceof BeanNameAware) {  
          // ......
       }  
       if (bean instanceof BeanClassLoaderAware) {  
          // ...
       }  
       if (bean instanceof BeanFactoryAware) {  
          ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);  
       }  
    }  
}
```

在这里还能看到其他的 Aware:

- BeanNameAware: 让当前 Bean 感知到自己的 Bean Name, Bean Name 是 Bean Factory 中实际使用的 bean 名称，可能与最初指定的名称不同, 例如内部 bean 名称可能会通过添加 "#..." 后缀来确保唯一性。如需提取原始名称(无后缀), 可使用BeanFactoryUtils#originalBeanName(String) 方法.
	
- BeanClassLoaderAware: 让当前 Bean 感知到加载自己的类加载器.

## 3	BeanPostProcessor

```java
public interface BeanPostProcessor {  
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
       return bean;  
    }  
  
    @Nullable  
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
       return bean;  
    }  
}
```

允许在 Bean 初始化前后对 Bean 进行修改, 此时的 Bean 已经完成了实例化. 用于对已经实例化好的 Bean 进行属性修改, 动态代理等; 比如 ApplicationContextAwareProcessor 就是 BeanPostProcessor 的实现类.

BeanPostProcessor 在 AbstractAutowireCapableBeanFactory#initializeBean 中被调用:

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {  
    // ......
  
    Object wrappedBean = bean;  
    if (mbd == null || !mbd.isSynthetic()) {  
       wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);  
    }  
  
    try {  
       invokeInitMethods(beanName, wrappedBean, mbd);  
    }  
    catch (Throwable ex) {  
       // ......
    }  
    if (mbd == null || !mbd.isSynthetic()) {  
       wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);  
    }  
  
    return wrappedBean;  
}
```

applyBeanPostProcessorsBeforeInitialization 在实例化方法 invokeInitMethods 之前调用, 而 applyBeanPostProcessorsAfterInitialization 在其之后调用.

这两个方法都是通过 getBeanPostProcessors() 方法获取所有的 BeanPostProcessor 遍历执行.

---

# 📚 参考内容

