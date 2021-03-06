# Spring源码深度分析2--从容器中获取bean对象
# 1 加载bean对象
**`MessageService message = (MessageService) bf.getBean("messageService");`**
**AbstractBeanFactory.java**
```java
protected <T> T doGetBean(String name, Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException {
        //提取对应的beanname
        final String beanName = this.transformedBeanName(name);
        //检查缓存中或者实例工厂中是否有对应的实例
        //尝试从缓存中获取或者SingletonFactories中的ObjectFactory中获取***********
        Object sharedInstance = this.getSingleton(beanName);
        Object bean;
        if (sharedInstance != null && args == null) {
            if (this.logger.isDebugEnabled()) {
                if (this.isSingletonCurrentlyInCreation(beanName)) {
                    this.logger.debug("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
                } else {
                    this.logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
                }
            }
            
            //返回对应的实例，有时候存储在诸如 BeanFactory 的情况并不是直接返回实例本身而是返回指定方法返回的实例
            //***********
            bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {//缓存中没有******
            //只有在单例情况下才会尝试解决循环依赖的问题，如果是prototype情况下，抛出异常
            if (this.isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }
            
            BeanFactory parentBeanFactory = this.getParentBeanFactory();
    //如果 beanDefinitionMap 也就是在所有已经加载的类中不包括 beanName 则尝试从parentBeanFactory中检测
            if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                String nameToLookup = this.originalBeanName(name);
                if (args != null) {
                    return parentBeanFactory.getBean(nameToLookup, args);
                }

                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
            //如果不是仅仅做类型检查则是创建bean ， 这里要进行记录
            if (!typeCheckOnly) {
                this.markBeanAsCreated(beanName);
            }

            try {
               //将存储XML 配置文件的 GernericBeanDefinition 转换为RootBeanDefinition，如果指定beanname
               //是子bean的话同时会合并父类的相关属性
                final RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                this.checkMergedBeanDefinition(mbd, beanName, args);
                String[] dependsOn = mbd.getDependsOn();
                String[] var11;
                //若存在依赖则需要递归实例化依赖的 bean
                if (dependsOn != null) {
                    var11 = dependsOn;
                    int var12 = dependsOn.length;

                    for(int var13 = 0; var13 < var12; ++var13) {
                        String dep = var11[var13];
                        if (this.isDependent(beanName, dep)) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                        }
                        //缓存依赖调用
                        this.registerDependentBean(dep, beanName);
                        this.getBean(dep);
                    }
                }
                //实例化依赖的bean后就可以实例化mbd本身了（RootBeanDefinition mbd）
                if (mbd.isSingleton()) {
                    //缓存中没有bean就需要从头开始bean的加载过程***********
                    sharedInstance = this.getSingleton(beanName, new ObjectFactory<Object>() {
                        public Object getObject() throws BeansException {
                            try {
                                return AbstractBeanFactory.this.createBean(beanName, mbd, args);
                            } catch (BeansException var2) {
                                AbstractBeanFactory.this.destroySingleton(beanName);
                                throw var2;
                            }
                        }
                    });
                    bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                } else if (mbd.isPrototype()) {
                    var11 = null;

                    Object prototypeInstance;
                    try {
                        this.beforePrototypeCreation(beanName);
                        prototypeInstance = this.createBean(beanName, mbd, args);
                    } finally {
                        this.afterPrototypeCreation(beanName);
                    }

                    bean = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                } else {
                    String scopeName = mbd.getScope();
                    Scope scope = (Scope)this.scopes.get(scopeName);
                    if (scope == null) {
                        throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                    }

                    try {
                        Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                            public Object getObject() throws BeansException {
                                AbstractBeanFactory.this.beforePrototypeCreation(beanName);

                                Object var1;
                                try {
                                    var1 = AbstractBeanFactory.this.createBean(beanName, mbd, args);
                                } finally {
                                    AbstractBeanFactory.this.afterPrototypeCreation(beanName);
                                }

                                return var1;
                            }
                        });
                        bean = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                    } catch (IllegalStateException var21) {
                        throw new BeanCreationException(beanName, "Scope '" + scopeName + "' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton", var21);
                    }
                }
            } catch (BeansException var23) {
                this.cleanupAfterBeanCreationFailure(beanName);
                throw var23;
            }
        }
        //检查需要的类型是否有符合bean 的实际类型
        if (requiredType != null && bean != null && !requiredType.isInstance(bean)) {
            try {
                return this.getTypeConverter().convertIfNecessary(bean, requiredType);
            } catch (TypeMismatchException var22) {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Failed to convert bean '" + name + "' to required type '" + ClassUtils.getQualifiedName(requiredType) + "'", var22);
                }

                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
        } else {
            return bean;
        }
    }
```
对于加载过程中所涉及的步骤大致如下 ：
1. 转换对应 beanName
传入的参数可能是别名，也可能是 FactoryBean ，所以需要进行一系列的解析
2. 尝试从缓存中加载单例
单例在 Spring 的同一个容器内只会被创建一次，后续再获取bean ，就直接从单例缓存中获取了 。 当然这里也只是尝试加载，首先尝试从缓存中加载，如果加载不成功则再次尝试从singletonFactories 中加载（创建单例 bean 的时候会存在依赖注入的情况）
3. **bean 的实例化**
如果从缓存中得到了 bean 的原始状态，则需要对 bean 进行实例化。
缓存中记录的只是最原始的 bean 状态， 井不一定是我们最终想要的 bean 。
4. 原型模式的依赖检查
只有在单例情况下才会尝试解决循环依赖
5. **检查 parentBeanFactory**
检测如果当前加载的 XML 配置文件中不包含 beanName 所对应的配置，就只能到 parentBeanFactory 去尝试
下了，然后再去递归的调用 getBean 方法 。
6. **将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition**
从 XML 配置文件中读取到的 bean 信息是存储在 GernericBeanDefinition 中的 ，但是所有的 bean 后续处理都是针对于 RootBeanDefinition 的 ，所以这里需要进行一个转换，转换的同时如果父类 bean 不为空的话，则会一并合并父类的属性。
7. 寻找依赖
8. **针对不同的 scope 进行 bean 的创建**
9. 类型转换
将返回的 bean 转换为 requiredType 所指定的类型

## 3.1 先了解一下FactoryBean 的使用
一般情况下， Spring 通过反射机制利用 bean 的 class 属性指定实现类来实例化 bean ，某些情况下比较复杂，用户可以通过实现FactoryBean 的工厂类接口定制实例化 bean 的逻辑。
```java
public interface FactoryBean<T> {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```
当配置文件中＜bean＞的 class 属性配置的`实现类是 FactoryBean `时，通过 getBean()方法返可的不 是 FactoryBean 本身，而是`FactoryBean#getObject()方法所返回的对象`，相当FactoryBean#getObject()代理了 getBean()方法;这样就避免了每设置一个＜bean＞都需要添加多个\<property>标签

## 3.2 缓存中获取单例 bean
```java
//DefaultSingletonBeanRegistry.java
 public Object getSingleton(String beanName) {
         //参数 true 标识允许早期依赖
        return this.getSingleton(beanName, true);
    }

    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        //检查缓存中是否存在实例
        Object singletonObject = this.singletonObjects.get(beanName);
        //如果为空，则锁定全局变量singletonObjects进行处理
        if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
            Map var4 = this.singletonObjects;
            synchronized(this.singletonObjects) {
                //如果此bean正在加载则不处理
                singletonObject = this.earlySingletonObjects.get(beanName);
                
                if (singletonObject == null && allowEarlyReference) {
                    //当某些方法需要提前初始化的时候则会调用addsingletonFactory 方法将对应的              
                    //ObjectFactory 初始化策略存储在singletonFactories 
                    ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        //调用预先设定的getObject()
                        singletonObject = singletonFactory.getObject();
                        //记录在缓存中，earlysingletonObjects和singletonFactories互斥
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }

        return singletonObject != NULL_OBJECT ? singletonObject : null;
    }
```
这个方法首先尝试从`singletonObjects` 里面获取实例，如果获取不到再从`earlySingletonObjects `里面获取，如果还获取不到，再尝试从`singletonFactories`里面获取beanName对应的` ObjectFactory`,然后调用这个`ObjectFactory`的getObject 来创建bean，并放到`earlySingletonObjects` 里面去，并且从`singletonFacotories`里面 remove掉这个ObjectFactory，而对于后续的所有内存操作都只为了循环依赖检测时候使用，也就是在allowEarlyReference为true的情况下才 会使用。
> 存储 bean 的不同的 map:
> `singletonObjects`:用于保存BeanName和创建bean实例之间的关系，bean name-->bean instance。
    `singletonFactories`:用于保存BeanName和创建bean的工厂之间的关系，bean name-> ObjectFactory。
    `earlySingletonObjects`:也是保存BeanName和创建bean实例之间的关系，与 singletonObjects的不同之处在于，`当一个单例bean被放到这里面后，那么当bean还 在创建过程中，就可以通过getBean方法获取到了，其目的是用来检测循环引用`。
   `registeredSingletons`:用来保存当前所有已注册的bean。

## 3.3 从 bean的实例中获取对象
**无论是从缓存中获取到的bean还是通过不同的scope策略加载的bean都只是最原始的bean 状态，并不一定是我们最终想要的bean**.举个例子，假如我们需要对工厂bean进行处理，那 么这里得到的其实是工厂bean的初始状态，但是我们真正需要的是工厂bean中定义的 factory-method 方法中返回的bean，而`getObjectForBeanlinstance`方法就是完成这个工作，继续处理获取到的原始bean。
```java
protected Object getObjectForBeanInstance(Object beanInstance, String name, String beanName, RootBeanDefinition mbd) {
        //如果指定的name是工厂相关（以&为前缀）且beanInstance又不是FactoryBean类型则验证不通过
        if (BeanFactoryUtils.isFactoryDereference(name) && !(beanInstance instanceof FactoryBean)) {
            throw new BeanIsNotAFactoryException(this.transformedBeanName(name), beanInstance.getClass());
         //现在我们有了个bean的实例，这个实例可能会是正常的bean 或者是FactoryBean 
         //如果是FactoryBean我们使用它创建实例，但是如果用户想要直接获取工厂实例而不是工厂的 
         //getobject方法对应的实例那么传人的name应该加入前缀&
        } else if (beanInstance instanceof FactoryBean && !BeanFactoryUtils.isFactoryDereference(name)) {
            //加载FactoryBean
            Object object = null;
            if (mbd == null) {
                //尝试从缓存中加载bean
                object = this.getCachedObjectForFactoryBean(beanName);
            }

            if (object == null) {
                //到这里已经明确知道beanInstance一定是FactoryBean类型
                FactoryBean<?> factory = (FactoryBean)beanInstance;
                //containsBeanDefinition 检测beanDefinitionMap中
                //也就是在所有已经加载的类中检测是否定义beanName
                if (mbd == null && this.containsBeanDefinition(beanName)) {
                    //将存储XML配置文件的GernericBeanDefinition 转换为RootBeanDefinition，如 
                    //果指定BeanName是子Bean的话同时会合并父类的相关属性
                    mbd = this.getMergedLocalBeanDefinition(beanName);
                }
                //是否是用户定义的而不是应用程序本身定义的
                boolean synthetic = mbd != null && mbd.isSynthetic();
                //解析bean的工作委托***********
                object = this.getObjectFromFactoryBean(factory, beanName, !synthetic);
            }

            return object;
        } else {
            return beanInstance;
        }
    }
```
1. 对FactoryBean正确性的验证。
2. 对非FactoryBean不做任何处理。
3. 对bean进行转换。
4. 将从Factory中解析bean的工作委托给`getObjectFromFactoryBean`。
```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
        //判断是单例的
        if (factory.isSingleton() && this.containsSingleton(beanName)) {
            synchronized(this.getSingletonMutex()) {
                //缓存中有就直接取出
                Object object = this.factoryBeanObjectCache.get(beanName);
                if (object == null) {
                    //委托任务**********doGetObjectFromFactoryBean(factory, beanName)
                    object = this.doGetObjectFromFactoryBean(factory, beanName);
                    Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
                    if (alreadyThere != null) {
                        object = alreadyThere;
                    } else {
                        if (object != null && shouldPostProcess) {
                            try {
                                //调用后置处理器
                                object = this.postProcessObjectFromFactoryBean(object, beanName);
                            } catch (Throwable var9) {
                                throw new BeanCreationException(beanName, "Post-processing of FactoryBean's singleton object failed", var9);
                            }
                        }

                        this.factoryBeanObjectCache.put(beanName, object != null ? object : NULL_OBJECT);
                    }
                }

                return object != NULL_OBJECT ? object : null;
            }
        } else {
           //不是单例的************
            Object object = this.doGetObjectFromFactoryBean(factory, beanName);
            if (object != null && shouldPostProcess) {
                try {
                    object = this.postProcessObjectFromFactoryBean(object, beanName);
                } catch (Throwable var11) {
                    throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", var11);
                }
            }

            return object;
        }
    }
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, String beanName) throws BeanCreationException {
        Object object;
        try {
            //需要权限验证
            if (System.getSecurityManager() != null) {
                AccessControlContext acc = this.getAccessControlContext();

                try {
                    object = AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
                        public Object run() throws Exception {
                            //直接调用getObject()
                            return factory.getObject();
                        }
                    }, acc);
                } catch (PrivilegedActionException var6) {
                    throw var6.getException();
                }
            } else {
                object = factory.getObject();
            }
        } catch (FactoryBeanNotInitializedException var7) {
            throw new BeanCurrentlyInCreationException(beanName, var7.toString());
        } catch (Throwable var8) {
            throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", var8);
        }

        if (object == null && this.isSingletonCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName, "FactoryBean which is currently in creation returned null from getObject");
        } else {
            return object;
        }
    }
```
上面我们已经讲述了了FactoryBean的调用方法，**如果bean声明为FactoryBean类型，则当 提取bean时提取的并不是FactoryBean，而是FactoryBean 中对应的getObject 方法返回的bean**， 而 `doGetObjectFromFactoryBean`正是实现这个功能的。但是，我们看到在上面的方法中除了调 用`object =factory.getObject()`得到我们想要的结果后并没有直接返回，而是接下来又做了些后 处理的操作，这个又是做什么用的呢?于是我们跟踪进入`AbstractAutowireCapableBeanFactory` 类的`postProcessObjectFromFactoryBean`方法：`尽可能保证所有bean初始化后都会调用注册的 BeanPostProcessor的postProcessAfterlnitialization 方法进行处理`

## 3.4 获取单例
如果缓存中不存在已经加载的单例 bean就需要从头开始 bean 的加载过程了，而 Spring 中使用 getSingleton 的重载方法实现 bean 的加载过程 。
```java
//AbstractBeanFactory.java
sharedInstance = this.getSingleton(beanName, new ObjectFactory<Object>() {
                        public Object getObject() throws BeansException {
                            try {
                                //核心部分是createBean(beanName, mbd, args)***
                                return AbstractBeanFactory.this.createBean(beanName, mbd, args);
                            } catch (BeansException var2) {
                                AbstractBeanFactory.this.destroySingleton(beanName);
                                throw var2;
                            }
                        }
                    });
```
```java
//DefaultSingletonBeanRegistry.java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        Map var3 = this.singletonObjects;
        //全局变量需要同步
        synchronized(this.singletonObjects) {
            //首先检查对应的bean是否已经加载过，因为单例是复用的
            Object singletonObject = this.singletonObjects.get(beanName);
            //如果为空才可以开始进行bean初始化
            if (singletonObject == null) {
                if (this.singletonsCurrentlyInDestruction) {
                    throw new BeanCreationNotAllowedException(beanName, "Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)");
                }

                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
                }

                //**********记录加载状态，便于检测循环依赖
                this.beforeSingletonCreation(beanName);
                boolean newSingleton = false;
                boolean recordSuppressedExceptions = this.suppressedExceptions == null;
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = new LinkedHashSet();
                }

                try {
                    //初始化bean********
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                } catch (IllegalStateException var16) {
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        throw var16;
                    }
                } catch (BeanCreationException var17) {
                    BeanCreationException ex = var17;
                    if (recordSuppressedExceptions) {
                        Iterator var8 = this.suppressedExceptions.iterator();

                        while(var8.hasNext()) {
                            Exception suppressedException = (Exception)var8.next();
                            ex.addRelatedCause(suppressedException);
                        }
                    }

                    throw ex;
                } finally {
                    if (recordSuppressedExceptions) {
                        this.suppressedExceptions = null;
                    }
                    //***********移除正在加载状态记录
                    this.afterSingletonCreation(beanName);
                }

                if (newSingleton) {
                    //加入缓存
                    this.addSingleton(beanName, singletonObject);
                }
            }

            return singletonObject != NULL_OBJECT ? singletonObject : null;
        }
    }
```
上述代码中其实是使用了**回调方法**，使得程序可以**在单例创建的前后做一些准备及处理操 作**，而真正的获取单例bean的方法其实并不是在此方法中实现的，其实现逻辑是在ObjectFactory 类型的实例singletonFactory中实现的。而这些准备及处理操作包括如下内容。
    1.检查缓存是否已经加载过。
    2.若没有加载，则记录beanName的正在加载状态。
    3.加载单例前记录加载状态`beforeSingletonCreation(beanName)`。
    4.通过调用参数传入的ObjectFactory的个体Object方法实例化bean，`createBean(beanName, mbd, args)`才是核心。
    5.加载单例后的处理方法调用。
    6.将结果记录至缓存并删除加载bean过程中所记录的各种辅助状态。
    7.返回处理结果
    
## 1.5 准备创建bean
>Spring 代码中的一些规律 ： 一个真正干活的函数其实是以 do 开头的，比如 doGetObjectFromFactoryBean ；而给我们错觉的函数，比如 getObjectFromFactoryBean ，其实只是从全局角度去做些统筹的工作 。

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Creating instance of bean '" + beanName + "'");
        }

        RootBeanDefinition mbdToUse = mbd;
        //解析class
        Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
            mbdToUse = new RootBeanDefinition(mbd);
            mbdToUse.setBeanClass(resolvedClass);
        }
        //验证及准备覆盖的方法********
        try {
            mbdToUse.prepareMethodOverrides();
        } catch (BeanDefinitionValidationException var7) {
            throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var7);
        }

        Object beanInstance;
        try {
            //给BeanPostProcessors一个机会来返回代理来替代真正的实例*********
            beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
            if (beanInstance != null) {
                //这里相当于返回了代理对象
                return beanInstance;
            }
        } catch (Throwable var8) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var8);
        }
        //*************如果没有进行代理
        beanInstance = this.doCreateBean(beanName, mbdToUse, args);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Finished creating instance of bean '" + beanName + "'");
        }

        return beanInstance;
    }
```
1. 根据设置的class 属性或者根据className来解析Class。
2. 对override属性进行标记及验证。在Spring配置中是存在lookup-method和replace-method的，而这两个配置的加 载其实就是将配置统一存放在BeanDefinition中的methodOverrides属性里，而这个函数的操作 其实也就是针对于这两个配置的。
3. 应用初始化前的后处理器，解析指定bean是否存在初始化前的短路操作。
4. 创建bean。

### 处理override属性
在Spring配 置中存在lookup-method和replace-method两个配置功能，面这两个配置的加载其实就是将配置 统一存放在BeanDefinition 中的 methodOverrides属性里，这两个功能实现原理其实是在bean实 例化的时候如果检测到存在methodOverrides 属性，会动态地为当前bean生成代理并使用对应 的拦截器为bean做增强处理
### 实例化的前置处理
```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
        Object bean = null;  
        //如果尚未被解析
        if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
            if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
                Class<?> targetType = this.determineTargetType(beanName, mbd);
                if (targetType != null) {
                    //实例化前的后置处理器应用****
                    bean = this.applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                    if (bean != null) {
                     //实例化后的后置处理器应用****
                        bean = this.applyBeanPostProcessorsAfterInitialization(bean, beanName);
                    }
                }
            }

            mbd.beforeInstantiationResolved = bean != null;
        }

        return bean;
    }
```
- 实例化前的后置处理器应用
bean的实例化前调用，也就是将AbsractBeanDefinition转换为BeanWrapper前的处理。**给子类一个修改BeanDefinition的机会**，也就是说当程序经过这个方法后，bean可能已经不是我 们认为的bean了，而是或许成为了一个经过处理的代理bean，可能是通过cglib生成的，也可能是通过其他技术生成的。即**在bean的实例化前 会调用后处理器的方法进行处理。**
- 实例化后的后置处理器应用
Spring中的规则是在bean的初始化后尽 可能保证将注册的后处理器的 postProcessAfterlnitialization 方法应用到该bean中，因为如果返 回的bean不为空，那么便不会再次经历普通bean的创建过程，所以只能在这里应用后处理器 的postProcessAfterlnitialization 方法。

## 1.6 循环依赖
**循环依赖**就是循环引用，就是两个或多个bean相互之间的持有对方，比如CircleA引用 CircleB，CircleB引用CircleC，CircleC引用CircleA，则它们最终反映一个环。此处不是循 环调用，循环调用是方法之间的环调用，如图所示。
![](_v_images/20191117142152372_4015.png)
### Spring如何解决循环依赖
Spring 容器循环依赖包括构造器循环依赖和 setter 循环依赖
在 Spring 中将循环依赖的处理分成了 3 种情况 ：
- **1.构造器循环依赖 **
表示通过构造器注入构成的循环依赖，此依赖是**无法解决**的，只能抛出BeanCurrentlyInCreationException 异常表示循环依赖。
如在创建TestA类时，构造器需要TestB类，那将去创建TestB，在创建TestB类时又发现 需要TestC类，则又去创建TestC，最终在创建TestC时发现又需要TestA，从而形成一个环， 没办法创建。
Spring容器将每一个正在创建的bean标识符放在一个**“当前创建bean池”**中，`bean标识 符在创建过程中将一直保持在这个池中，因此如果在创建bean过程中发现自己已经在“当前 创建bean池”里时，将抛出BeanCurrentlyInCreationException异常表示循环依赖`；而对于创建 完毕的bean 将从“当前创建bean池”中清除掉。

- **2.setter循环依赖** 
表示通过setter注入方式构成的循环依赖。
对于setter注入造成的依赖是通过Spring 容器 提前暴露刚完成构造器注入但未完成其他步骤（如setter注入）的bean来完成的，而且**只能解 决单例作用域的bean循环依赖**。通过提前暴露一个单例工厂方法，从而使其他bean 能引用到 该bean，如下代码所示： 
```java
addSingletonFactory(beanName,new ObjectFactory(){ 
    public Object getobject()throws BeansException{ 
        return getEarlyBeanReference(beanName,mbd,bean); 
    }
}
```
具体步骤如下。
1. Spring容器创建单例“testA”bean，首先根据无参构造器创建bean，并暴露一个 “ObjectFactory”用于返回一个提前暴露一个创建中的bean，并将“testA”标识符放到“当前 创建bean池”，然后进行setter注入“testB”。
2. Spring容器创建单例“testB”bean，首先根据无参构造器创建bean，并暴露一个 “ObjectFactory”用于返回一个提前暴露一个创建中的bean，并将“testB”标识符放到“当前 创建bean池”，然后进行 setter注入“circle”。
3. Spring容器创建单例“testC”bean，首先根据无参构造器创建bean，并暴露一个 “ObjectFactory”用于返回一个提前暴露一个创建中的bean，并将“testC”标识符放到“当前 创建bean池”，然后进行seter注入‘“testA”。**进行注入‘“testA”时由于提前暴露了“ObjectFactory” 工厂，从而使用它返回提前暴露一个创建中的bean。**
4. 最后在依赖注入“testB”和“testA”，完成setter注入。

- **3.prototype 范围的依赖处理**
对“prototype”作用域bean，Spring容器无法完成依赖注入，因为Spring容器不进行缓 存“prototype”作用域的bean，因此无法提前暴露一个创建中的bean。

## 1.7 创建bean
当经历 过`resolveBeforelnstantiation `方法后，程序有两个选择，如果创建了代理或者说重写了` InstantiationAware BeanPostProcessor#postProcessBeforelnstantiation`方法并在方法`postProcessBeforeInstantiation`中改变了bean，则直接返回就可以了，否则需要进行常规bean的创建。而 这常规bean的创建就是在`doCreateBean()`中完成的。
```java
//AbstractAutowireCapableBeanFactory.java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
        BeanWrapper instanceWrapper = null;
        //如果是单例需要先清除缓存
        if (mbd.isSingleton()) {
            instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
        }

        if (instanceWrapper == null) {
            //根据指定bean使用对应的策略创建新的实例，如：工厂方法、构造函数自动注入、简单初始化
            //*********将BeanDefinition 转换为BeanWrapper
            instanceWrapper = this.createBeanInstance(beanName, mbd, args);
        }

        final Object bean = instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null;
        Class<?> beanType = instanceWrapper != null ? instanceWrapper.getWrappedClass() : null;
        mbd.resolvedTargetType = beanType;
        Object var7 = mbd.postProcessingLock;
        synchronized(mbd.postProcessingLock) {
            if (!mbd.postProcessed) {
                try {
                    //应用MergedBeanDefinitionPostProcessors，Autowired注解
                    this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                } catch (Throwable var17) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Post-processing of merged bean definition failed", var17);
                }

                mbd.postProcessed = true;
            }
        }
        //是否需要提早曝光：单例&允许循环依赖&当前bean正在创建中，检测循环依赖 
        boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
        if (earlySingletonExposure) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
            }
            //为避免后期循环依赖，可以在bean初始化完成前将创建实例的objectFactory加入工厂
            this.addSingletonFactory(beanName, new ObjectFactory<Object>() {
                public Object getObject() throws BeansException {
                    //对bean再一次依赖引用，主要应用SmartInstantiationAwareBeanPostProcessor 
                    //其中我们熟知的AOP就是在这里将advice动态织入bean中，若没有则直接返回bean，不做任何处理
                    return AbstractAutowireCapableBeanFactory.this.getEarlyBeanReference(beanName, mbd, bean);
                }
            });
        }

        Object exposedObject = bean;

        try {
            //对bean进行填充，将各个属性值注入，其中，可能存在依赖于其他bean的属性，则会递归初始依赖 bean 
            this.populateBean(beanName, mbd, instanceWrapper);
            if (exposedObject != null) {
                //调用初始化方法，比如init-method
                exposedObject = this.initializeBean(beanName, exposedObject, mbd);
            }
        } catch (Throwable var18) {
            if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
                throw (BeanCreationException)var18;
            }

            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var18);
        }

        if (earlySingletonExposure) {
            Object earlySingletonReference = this.getSingleton(beanName, false);
            //earlysingletonReference 只有在检测到有循环依赖的情况下才会不为空
            if (earlySingletonReference != null) {
                //如果exposedObject没有在初始化方法中被改变，也就是没有被增强
                if (exposedObject == bean) {
                    exposedObject = earlySingletonReference;
                } else if (!this.allowRawInjectionDespiteWrapping && this.hasDependentBean(beanName)) {
                    String[] dependentBeans = this.getDependentBeans(beanName);
                    Set<String> actualDependentBeans = new LinkedHashSet(dependentBeans.length);
                    String[] var12 = dependentBeans;
                    int var13 = dependentBeans.length;

                    for(int var14 = 0; var14 < var13; ++var14) {
                        String dependentBean = var12[var14];
                        //检测依赖
                        if (!this.removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                            actualDependentBeans.add(dependentBean);
                        }
                    }
                    /*因为bean创建后其所依赖的bean一定是已经创建的，
                     *actualDependentBeans 不为空则表示当前bean创建后其依赖的bean却没有没全部创建完，
                     *也就是说存在循环依赖
                     */
                    if (!actualDependentBeans.isEmpty()) {
                        throw new BeanCurrentlyInCreationException(beanName, "Bean with name '" + beanName + "' has been injected into other beans [" + StringUtils.collectionToCommaDelimitedString(actualDependentBeans) + "] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
                    }
                }
            }
        }

        try {
            //根据scope注册bean********
            this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
            return exposedObject;
        } catch (BeanDefinitionValidationException var16) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
        }
    }
```
**思路概要：**
1. 如果是单例则需要首先清除缓存。
2. 实例化bean，将BeanDefinition转换为Bean Wrapper。
    转换是一个复杂的过程，但是我们可以尝试概括大致的功能，如下所示。
    如果存在工厂方法则使用工厂方法进行初始化。
    一个类有多个构造函数，每个构造函数都有不同的参数，所以需要根据参数锁定构造 函数并进行初始化。
    如果既不存在工厂方法也不存在带有参数的构造函数，则使用默认的构造函数进行 bean的实例化。
3. MergedBeanDefinitionPostProcessor的应用。
    bean合并后的处理，Autowired注解正是通过此方法实现诸如类型的预解析。
4. 依赖处理。
    在Spring中会有循环依赖的情况，例如，当A中含有B的属性，而B中又含有A的属性 时就会构成一个循环依赖，此时如果A和B都是单例，那么在Spring中的处理方式就是当创 建B的时候，涉及自动注入A的步骤时，并不是直接去再次创建A，而是通过放入缓存中的 ObjectFactory来创建实例，这样就解决了循环依赖的问题。
5. 属性填充。将所有属性填充至bean的实例中。
6. 循环依赖检查。

### 1.创建bean的实例
```java
//AbstractAutowireCapableBeanFactory.java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
        //解析class
        Class<?> beanClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
        } else if (mbd.getFactoryMethodName() != null) {
        //如果工厂方法不为空则使用工厂方法初始化策略**********
            return this.instantiateUsingFactoryMethod(beanName, mbd, args);
        } else {
            boolean resolved = false;
            boolean autowireNecessary = false;
            if (args == null) {
                Object var7 = mbd.constructorArgumentLock;
                synchronized(mbd.constructorArgumentLock) {
                    //根据参数锁定对应的构造函数或对应的工厂方法
                    if (mbd.resolvedConstructorOrFactoryMethod != null) {
                        resolved = true;
                        autowireNecessary = mbd.constructorArgumentsResolved;
                    }
                }
            }
            //如果已经解析过则使用解析好的构造函数方法不需要再次锁定
            if (resolved) {
                return autowireNecessary ? this.autowireConstructor(beanName, mbd, (Constructor[])null, (Object[])null) : this.instantiateBean(beanName, mbd);
            } else {
                //根据参数解析构造函数
                Constructor<?>[] ctors = this.determineConstructorsFromBeanPostProcessors(beanClass, beanName);
                //返回默认构造函数构造 还是 构造函数自动注入
                return ctors == null && mbd.getResolvedAutowireMode() != 3 && !mbd.hasConstructorArgumentValues() && ObjectUtils.isEmpty(args) ? this.instantiateBean(beanName, mbd) : this.autowireConstructor(beanName, mbd, ctors, args);
            }
        }
    }
```
逻辑概要：
1. 如果在RootBeanDefinition中存在factoryMethodName属性，或者说在配置文件中配置 了factory-method，那么Spring 会尝试使用instantiateUsingFactoryMethod(beanName,mbd,args)方法根据RootBeanDefinition中的配置生成bean的实例。
2. 解析构造函数并进行构造函数的实例化。
   因为一个bean对应的类中可能会有多个构造 函数，而每个构造函数的参数不同，Spring在根据参数及类型去判断最终会使用哪个构造函数 进行实例化。
   但是，判断的过程是个比较消耗性能的步骤，所以采用缓存机制，如果已经解析过则 不需要重复解析而是直接从RootBeanDefinition 中的属性resolvedConstructorOrFactoryMethod缓存 的值去取，否则需要再次解析，并将解析的结果添加至RootBeanDefinition中的属性 resolvedConstructorOrFactoryMethod中。

1. **autowireConstructor**

2. **instantiateBean**

3. **实例化策略**
程序中，首先判断如果beanDefinition.getMethodOverrides(）为空也就是 用户没有使用replace或者lookup的配置方法，**那么直接使用反射的方式**，简单快捷，但是如果使 用了这两个特性，在直接使用反射的方式创建实例就不妥了，因为需要将这两个配置提供的功能切 人进去，所以就必须要**使用动态代理的方式将包含两个特性所对应的逻辑的拦截增强器设置进去**， 这样才可以保证在调用方法的时候会被相应的拦截器增强，返回值为包含拦截器的代理实例。

### 2.记录创建 bean 的 ObjectFactory
主要是解决循环依赖的
```java
//doCerate()函数中：
```

### 3.属性注入


### 4.初始化bean

### 5.注册DisposableBean