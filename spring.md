# **Spring介绍**
创建Spring的目的就是用来替代更加重量级的企业级Java技术。

简化java的开发
- 基于POJO轻量级和最小侵入式开发
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少模板式代码

# **Spring Bean的生命周期**

![clipboard.png](https://segmentfault.com/img/bVbgwfO)

### **接口方法的分类**
|分类类型|所包含方法|
|--|--|
|Bean自身的方法|配置文件中的init-method跟destroy-method配置的方法、Bean对象自己调用的方法|
|Bean级生命周期的方法|BeanNameAware、BeanFactoryAware、InitializingBean、DisposableBeand等接口的方法|
|容器级生命周期的方法|InstantiationAwareBeanPostProcessor、BeanPostProcessor等后置处理器实现类中重写的方法|

总结：
- 初始化容器级生命周期的接口，并执行相关的方法。
- 实例化Bean、并且填充定义的属性
- 执行Bean级生命周期接口的方法（BeanNameAware、BeanFactoryAware)
- 执行BeanPostProcessor的postProcessorBeforeInitialization方法（BeanPostProcessor前置处理），这里可以对studentBean的属性进行修改。
- 执行InitializingBean的afterPropertiesSet方法
- 执行Bean自身的init-method
- BeanPostProcessor的postProcessorAfterInitializaton方法（BeanPostProcessor后置处理）
- 【初始化成功】
- 【销毁容器】
- 先执行Disposablebean接口的destory方法
- 执行bean自身的destory-method属性配置的销毁方法。

# **IOC**
IoC(Inversion of Control)控制反转，包含了两个方面
- 控制：当前对象的内部成员控制权
- 反转：这种控制不由当前对象管理，由其他第三方容器（类）来管理。
### **DI**
DI(Dependency injection)依赖注入是Ioc思想的实现方式，通过调用类对某一接口实现类的依赖关系由第三方容器（类）注入，以移除调用类对某一接口实现类的依赖。

使用IoC的好处：
- 不用自己组装，拿来就用。
- 享受单例带来的好处，效率高，不浪费空间。
- 便于单与测试，方便切换mock组件。、
- 便于进行AOP操作，对使用者是透明的。
- 统一配置，便于修改。

### **IOC容器的原理**
IOC容器其实就是一个大工厂，它用来管理我们所有的对象以及依赖关系。
- 通过Java的反射技术获取所有类的信息（类成员变量、类名等）
- 接着通过xml配置文件或者注解来描述类于类之间的关系
- 我们就可以通过这些配置信息和反射技术来构建出对应的对象和依赖关系。
Spring IoC容器实现对象的创建和依赖：
- 根据Bean配置信息在容器内部创建Bean定义注册表
- 根据注册表加载、实例化Bean、建立Bean与Bean之间的依赖关系
- 将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用

### **IoC容器装配Bean**
#### **装配Bean方式**
Spring4.0开始IoC容器装配Bean有4种方式
- XML配置文件
- 注解
- JavaConfig
- 基于Groovy DSL配置（少见）
#### **依赖注入的方式**
- 属性注入，通过setter方法注入
- 构造注入
- 工厂注入
- p标签注入
#### **Bean的作用域**
- 单例Singleton
- 多例prototype
- 与web环境有关的Bean作用域
    - request
    - session

# **AOP**
AOP(Aspect Oriented Programming)，即面向切面编程，可以说的OOP面向对象编程的补充和完善，AOP使用一种“横切”的技术，剖解开封装的对象内部，并将那些影响多个类的公共行为封装到一个可重用的模块，并将其命名为“Aspect”，即切面。
### **AOP的核心概念**
- 连接点(join point)：被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截的方法。
- 切入点(pointcut)：对连接点进行拦截的定义
- 通知/增强(advice）：拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类。
- 切面(aspect)：类是对物体特征的抽象，切面就是对横切关注点的抽象。
- 织入(weave)：将切面应用到目标对象并导致代理对象创建的过程，将advice添加到目标类的具体连接点的过程。
- 引入(introduction)：在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或手段。
### **Spring对AOP的支持**
Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也是由IOC容器负责管理。因此AOP代理可以直接使用容器的其他Bean实例作为目标，这种关系可由IOC容器的依赖注入提供。
Spring创建代理的规则：
- 默认使用Java动态代理，这样就可以为任何接口实例创建代理
- 当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理，也可强制使用CGLIB代理。
JDK代理跟CGLIB代理代理选择：
- 如果是单例最好使用CGLIB代理，如果是多例使用JDK代理
- JDK在创建代理对象的使用性能要高于CGLIB，在生成代理对象的时候性能要低于CGLIB。


# **Spring事务**
事务管理对企业应用而言至关重要，它保证每个用户的每一次操作都是可靠的。

在Spring中，事务时通过TransactionDefinition接口来定义的。
### **事务隔离级别**
隔离级别指若干个并发的事务之间的隔离程度。TransactionDefinition接口中定义了五个表示隔离级别的常量：
|常量|隔离级别|
|---|---|
|TransactionDefinition.ISOLATION_DEFAULT|底层数据库的默认级别|
|TransactionDefinition.ISOLATION_READ_UNCOMMITTED|读未提交|
|TransactionDefinition.ISOLATION_READ_COMMITTED|都提交|
|TransactionDefinition.ISOLATION_REPEATABLE|读提交|
|TransactionDefinition.ISOLATION_SERIALIZABLE|序列化|

### **事务传播行为**
所谓事务传播行为是指，如果在开始当前事务之前，一个事物上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：
|常量|传播行为|传播行为|
|---|---|---|
|TransactionDefinition_PROPAGATION_REQUIRED|存在->加入|不存在->新建|
|TransactionDefinition_PROPAGATION_REQUIRED_NEW|存在->新建，挂起|不存在->新建|
|TransactionDefinition_SUPPORTS|存在->加入|不存在->非事务
|TransactionDefinition_NOT_SUPPORTS|存在->挂起|不存在->非事务
|TransactionDefinition_NEVER|存在—>抛异常|不存在->非事务
|TransactionDefinition_MANDATORY|存在->加入|不存在->抛异常
|TransactionDefinition_NESTED|存在->新建嵌套|不存在->新建


# **Spring MVC**
### **执行流程**

![clipboard.png](https://segmentfault.com/img/bVbfumF)

- 用户发送请求到前端控制器DispatcherServlet
- DispatcherServlet收到请求调用处理器映射器HandleMapping。
- 处理器映射器根据请求的url找到具体的处理器，生成处理器执行链HandleExecutionChain（包含处理器对象和处理器拦截器）一并给DispatcherServlet。
- DispatcherSerlvet根据处理器Handle获取处理器适配器HandleAdapter执行，HandleAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作。
- 执行处理器Handle(Controller，也叫页面处理器)。
- Handle执行完返回ModelAndView
- HandleAdapter讲ModelAndView返回给DispatcherServlet
- DispatcherServlet将ModelAndView传给ModelAndView视图解析器
- ModelAndView解析后返回具体View
- DispatcherServlet堆View进行渲染视图（即将模型数据model填充进视图中）
- DispatcherSerlvet响应用户

# **Autowired跟Resource的区别**
- 用途：做bean的注入时使用
- 共同点：
    - 装配bean，字段上，也可写在setter方法上
- 不同点：
    - @Autowired输入Spring注解，@Resource属于java注解
    - @Autowired默认类型匹配，依赖对象必须存在，如果允许null，则设置required属性为false `@Autowired(required=false)`,也可使用名称匹配，配合@Qualifier使用
    - @Resource默认按名称匹配，通过name属性进行指定
```
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}

public class TestServiceImpl {
    // 下面两种@Resource只要使用一种即可
    @Resource(name="userDao")
    private UserDao userDao; // 用于字段上
    
    @Resource(name="userDao")
    public void setUserDao(UserDao userDao) { // 用于属性的setter方法上
        this.userDao = userDao;
    }
}
```
# **BeanFacotry**
BeanFactory定义了IOC容器的基本类型，并提供了IOC容器应遵守的最基本的接口，也就是spring IOC所遵守的最底层和最基本的编程规范。

BeanFactory仅是一个接口，并不是IOC容器的具体实现，具体的实现有：DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等。

```
public interface BeanFactory {
    //FactoryBean前缀
    String FACTORY_BEAN_PREFIX = "&";

    //根据名称获取Bean对象
    Object getBean(String name) throws BeansException;

    ///根据名称、类型获取Bean对象
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    //根据类型获取Bean对象
    <T> T getBean(Class<T> requiredType) throws BeansException;

    //根据名称获取Bean对象,带参数
    Object getBean(String name, Object... args) throws BeansException;

    //根据类型获取Bean对象,带参数
    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

    //是否存在
    boolean containsBean(String name);

    //是否为单例
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

    //是否为原型（多实例）
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

    //名称、类型是否匹配
    boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;

    //获取类型
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    //根据实例的名字获取实例的别名
    String[] getAliases(String name);
}
```

# **FactoryBean**
一个工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。本质上是个工厂Bean。

```
public interface FactoryBean<T>{

    //FactoryBean创建的Bean实例
    T getObject throw Exception;
    
    //返回FactoryBean创建的Bean类型
    Class<?> getObjectType();
    
    //作用域是singleton还是prototype 
    boolean isSingleton();
}
```

# spring boot

## 配置文件优先级
- `src/main/resources/config/application.properties`  >  `src/main/resources/application.properties `
- 相同位置的`application.properties`  >  `application.yml`


## 注解解释
### @Configuration
JavaConfig的配置形式，配置了@Configuation之后，就是一个IoC容器的配置类。

### @ComponentScan
自动扫描并加载符合条件的组件（比如：@Component和@Repository等）或者Bean定义，最终将这些bean定义加载到IoC容器中。

可以通过basePackage等属性细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明的@ComponentScan所在累的package进行扫描。

所以这也是pringBoot的启动一般放在root package下的原因。因为默认不指定basePackages。



## SpringApplicaiton执行流程


# **内容参考**
- [Spring MVC的执行流程及工作原理][2]
- [全面分析 Spring 的编程式事务管理及声明式事务管理][3]


  [2]: https://www.jianshu.com/p/8a20c547e245
  [3]: https://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/index.html
