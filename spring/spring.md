# asdf

BeanFactory -> ApplicationContext（添加了AOP，事件处理等）



**Bean** : In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans。Bean不一定是POJO，可能是业务对象

**ApplicationContext** : The `org.springframework.context.ApplicationContext` interface <span style='color:red;font-weight:bold'>represents the Spring IoC container</span> and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata.

Container的任务是，通过配置文件，找到需要初始化的对象并初始化，然后将其装配起来。

可以通过context.getBean 方法，获取对应的Bean，但是应该尽量避免这样使用，而是通过注入的方式管理依赖



建议通过构造函数注入，但是有时候会发生循环引用问题，这时候可以调整某个注入为setter注入



Beans that are singleton-scoped and set to be pre-instantiated (the default) are created when the container is created 默认情况下，Bean是一个在container创建时候就初始化完成的单例对象



Setter injection

```xml
<!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
```



Constructor Injection

```xml
 <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
```



对于不是直接作为属性的依赖方式，可以使用depends-on来标识。比如静态函数中需要另外一个类的被加载



Bean Scope

对于prototype类型的Bean，Container是不负责它的销毁的。因此客户端需要自己小心维护对象的生存周期





Environment = properties + profiles



# AOP

**切面Aspect**: 是一个业务的概念，比如要打日志，要开启事务，要计时，要鉴权等等。Aspect一般是一个类；

**切点PointCut**：是指切面如何和业务的代码关联起来，也就是说哪个业务代码要触发这个切面。一般切点是业务的一个方法调用（Spring的AOP只支持方法，AspectJ支持field访问）。 (the method serving as the pointcut signature must have a `void` return type).



Aspect的执行顺序： 如果是不同的Aspect，可以用继承Order类来定义顺序。对于同一个Aspect中，是不确定的













