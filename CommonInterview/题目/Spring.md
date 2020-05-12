1. Spring模块
   
   1. Spring Core： 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖注入功能。
   
   2. Spring Aspects ：该模块为与AspectJ的集成提供支持。
   
   3. Spring AOP ：提供了面向切面的编程实现。
   
   4. Spring JDBC : Java数据库连接。
   
   5. Spring JMS ：Java消息服务。
   
   6. Spring ORM : 用于支持Hibernate等ORM工具。
   
   7. Spring Web : 为创建Web应用程序提供支持。
   
   8. Spring Test : 提供了对 JUnit 和 TestNG 测试的支持。

2. @RestController vs @Controller
   
   单独使用 @Controller 不加 @ResponseBody的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。
   
   @RestController只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。
 
   @Controller + @ResponseBody= @RestController（Spring 4 之后新加的注解）。
   
   @ResponseBody 注解的作用是将 Controller 的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到HTTP 响应(Response)对象的 body 中，通常用来返回 JSON 或者 XML 数据，返回 JSON 数据的情况比较多。   
  
3. Spring IoC
   
   IoC（Inverse of Control:控制反转）是一种设计思想，就是 将原本在程序中手动创建对象的控制权，交由Spring框架来管理。 IoC 在其他语言中也有应用，并非 Spirng 特有。 IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。
   
   将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。
   
   IOC初始化过程：
   读取XML-> Resource->解析BeanDefinition->注册BeanFactory

4. AOP
   
   AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。
   
   Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用Cglib ，这时候Spring AOP会使用 Cglib 生成一个被代理对象的子类来作为代理   
   
   当然你也可以使用 AspectJ ,Spring AOP 已经集成了AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。
   
   使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

5. Spring AOP 和 AspectJ AOP 有什么区别？   

   Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。 Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。
   
   Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，
   
   如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

6. Spring 中的 bean 的作用域有哪些?
   1. singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
   
   2. prototype : 每次请求都会创建一个新的 bean 实例。
   
   3. request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
   
   4. session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
   
   5. global-session：全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话   
   
7. Spring 中的单例 bean 的线程安全问题了解吗？
   
   大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。
   
   常见的有两种解决办法：
   
   1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
   
   2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

8. Spring 中的 bean 生命周期?
   
   Bean 容器找到配置文件中 Spring Bean 的定义。
   
   Bean 容器利用 Java Reflection API 创建一个Bean的实例。
   
   如果涉及到一些属性值 利用 set()方法设置一些属性值。
   
   如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入Bean的名字。
   
   如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
   
   如果Bean实现了 BeanFactoryAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoade r对象的实例。
   
   与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。
   
   如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessBeforeInitialization() 方法
   
   如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
   
   如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
   
   如果有和加载这个 Bean的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessAfterInitialization() 方法
   
   当要销毁 Bean 的时候，如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
   
   当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。    
   
   ----
   
   1）BeanFactoryPostProcessor的postProcessorBeanFactory()方法：若某个IoC容器内添加了实现了BeanFactoryPostProcessor接口的实现类Bean，那么在该容器中实例化任何其他Bean之前可以回调该Bean中的postPrcessorBeanFactory()方法来对Bean的配置元数据进行更改，比如从XML配置文件中获取到的配置信息。
   
   （2）Bean的实例化：Bean的实例化是使用反射实现的。
   
   （3）Bean属性注入：Bean实例化完成后，利用反射技术实现属性及依赖Bean的注入。
   
   （4）BeanNameAware的setBeanName()方法：如果某个Bean实现了BeanNameAware接口，那么Spring将会将Bean实例的ID传递给setBeanName()方法，在Bean类中新增一个beanName字段，并实现setBeanName()方法。
   
   （5）BeanFactoryAware的setBeanFactory()方法：如果某个Bean实现了BeanFactoryAware接口，那么Spring将会将创建Bean的BeanFactory传递给setBeanFactory()方法，在Bean类中新增了一个beanFactory字段用来保存BeanFactory的值，并实现setBeanFactory()方法。
   
   （6）ApplicationContextAware的setApplicationContext()方法：如果某个Bean实现了ApplicationContextAware接口，那么Spring将会将该Bean所在的上下文环境ApplicationContext传递给setApplicationContext()方法，在Bean类中新增一个ApplicationContext字段用来保存ApplicationContext的值，并实现setApplicationContext()方法。
   
   （7）BeanPostProcessor预初始化方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后，执行初始化之前会调用BeanPostProcessor中的postProcessBeforeInitialization()方法执行预初始化处理。
   
   （8）InitializingBean的afterPropertiesSet()方法：如果Bean实现了InitializingBean接口，那么Bean在实例化完成后将会执行接口中的afterPropertiesSet()方法来进行初始化。
   
   （9）自定义的inti-method指定的方法：如果配置文件中使用init-method属性指定了初始化方法，那么Bean在实例化完成后将会调用该属性指定的初始化方法进行Bean的初始化。
   
   （10）BeanPostProcessor初始化后方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后并且完成初始化调用后执行该接口中的postProcessorAfterInitialization()方法进行初始化后处理。
   
   （11）使用Bean：此时有关Bean的所有准备工作均已完成，Bean可以被程序使用了，它们将会一直驻留在应用上下文中，直到该上下文环境被销毁。
   
   （12）DisposableBean的destory()方法：如果Bean实现了DisposableBean接口，Spring将会在Bean实例销毁之前调用该接口的destory()方法，来完成一些销毁之前的处理工作。
   
   （13）自定义的destory-method指定的方法：如果在配置文件中使用destory-method指定了销毁方法，那么在Bean实例销毁之前会调用该指定的方法完成一些销毁之前的处理工作。
   
   注意：
   
   1、BeanFactoryPostProcessor接口与BeanPostProcessor接口的作用范围是整个上下文环境中，使用方法是单独新增一个类来实现这些接口，那么在处理其他Bean的某些时刻就会回调响应的接口中的方法。
   
   2、BeanNameAware、BeanFactoryAware、ApplicationContextAware的作用范围的Bean范围，即仅仅对实现了该接口的指定Bean有效，所有其使用方法是在要使用该功能的Bean自己来实现该接口。
   
   3、第8点与第9点所述的两个初始化方法作用是一样的，我们完全可以使用其中的一种即可，一般情况我们使用第9点所述的方式，尽量少的去来Bean中实现某些接口，保持其独立性，低耦合性，尽量不要与Spring代码耦合在一起。第12和第13也是如此。  

9. SpringMVC 工作原理了解吗?
   
   1. 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
   
   2. DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
   
   3. 解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
   
   4. HandlerAdapter 会根据 Handler来调用真正的处理器开处理请求，并处理相应的业务逻辑。
   
   5. 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
   
   6. ViewResolver 会根据逻辑 View 查找实际的 View。
   
   7. DispaterServlet 把返回的 Model 传给 View（视图渲染）。
   
   9. 把 View 返回给请求者（浏览器）   

10. Spring 框架中用到了哪些设计模式？
    
    工厂设计模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
    
    代理设计模式 : Spring AOP 功能的实现。
    
    单例设计模式 : Spring 中的 Bean 默认都是单例的。
    
    模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
    
    包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
    
    观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
    
    适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。

11. @Component 和 @Bean 的区别是什么？
    
    1. 作用对象不同: @Component 注解作用于类，而@Bean注解作用于方法。
    
    2. @Component通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。@Bean 注解通常是我们在标有该注解的方法中定义产生这个 bean,@Bean告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
    
    3. @Bean 注解比 Component 注解的自定义性更强，而且很多地方我们只能通过 @Bean 注解来注册bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 @Bean来实现。       
    
12. 将一个类声明为Spring的 bean 的注解有哪些?
    
    我们一般使用 @Autowired 注解自动装配 bean，要想把类标识成可用于 @Autowired 注解自动装配的 bean 的类,采用以下注解可实现：
    
    @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个Bean不知道属于哪个层，可以使用@Component 注解标注。
    
    @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
    
    @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
    
    @Controller : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

13. Spring 管理事务的方式有几种？
    
    1. 编程式事务，在代码中硬编码。(不推荐使用)
    
    2. 声明式事务，在配置文件中配置（推荐使用）    
    
    声明式事务又分为两种：
    
    1. 基于XML的声明式事务
    
    2. 基于注解的声明式事务  

14. Spring 事务中的隔离级别有哪几种?
    TransactionDefinition 接口中定义了五个表示隔离级别的常量：
    
    1. TransactionDefinition.ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
    
    2. TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
    
    3. TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
    
    4. TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
    
    5. TransactionDefinition.ISOLATION_SERIALIZABLE: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。       
 
15. Spring 事务中哪几种事务传播行为?
    
    支持当前事务的情况：
    
    TransactionDefinition.PROPAGATION_REQUIRED： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
    
    TransactionDefinition.PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
    
    TransactionDefinition.PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）
    
    不支持当前事务的情况：
    
    TransactionDefinition.PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
    
    TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
    
    TransactionDefinition.PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。
    
    其他情况：
    
    TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

16. @Transactional(rollbackFor = Exception.class)注解了解吗？
    
    Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。
    
    当@Transactional注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。
    
    在@Transactional注解中如果不配置rollbackFor属性,那么事物只会在遇到RuntimeException的时候才会回滚,加上rollbackFor=Exception.class,可以让事物在遇到非运行时异常时也回滚。

17. 如何使用JPA在数据库中非持久化一个字段？
    
    @Transient    

18. 注解生效原理

    注解的实现：  
    1. 接口使用@interface定义。
    2. 通过继承以下注解，实现功能：
    元注解@Target,@Retention,@Documented,@Inherited   
     
    ```
       * 元注解@Target,@Retention,@Documented,@Inherited * 
       * @Target 表示该注解用于什么地方，可能的 ElemenetType 参数包括： 
       * ElemenetType.CONSTRUCTOR 构造器声明 
       * ElemenetType.FIELD 域声明（包括 enum 实例） 
       * ElemenetType.LOCAL_VARIABLE 局部变量声明 
       * ElemenetType.METHOD 方法声明 
       * ElemenetType.PACKAGE 包声明 
       * ElemenetType.PARAMETER 参数声明 
       * ElemenetType.TYPE 类，接口（包括注解类型）或enum声明 
       * 
       * @Retention 表示在什么级别保存该注解信息。可选的 RetentionPolicy 参数包括： 
       * RetentionPolicy.SOURCE 注解将被编译器丢弃 
       * RetentionPolicy.CLASS 注解在class文件中可用，但会被VM丢弃 
       * RetentionPolicy.RUNTIME VM将在运行期也保留注释，因此可以通过反射机制读取注解的信息。 
       * 
       * @Documented 将此注解包含在 javadoc 中 
       * 
       * @Inherited 允许子类继承父类中的注解
       
       
       @Target(ElementType.METHOD) 
       @Retention(RetentionPolicy.RUNTIME) 
       @Documented 
       @Inherited 
    ```           
    @Controller处理过程：
    
    Controller类使用继承@Component注解的方法，将其以单例的形式放入spring容器，如果仔细看的话会发现每个注解里面都有一个默认的value()方法，它的作用是为当前的注解声明一个名字，一般默认为类名，然后spring会通过配置文件中的<context:component-scan>的配置，进行如下操作：
    
    1. 使用asm技术扫描.class文件，并将包含@Component及元注解为@Component的注解@Controller、@Service、@Repository或者其他自定义的的bean注册到beanFactory中，
    
    2. 然后spring在注册处理器
    
    3. 实例化处理器，然后将其放到beanPostFactory中，然后我们就可以在类中进行使用了。
    
    4. 创建bean时，会自动调用相应的处理器进行处理。
    
    标签分为两种：
    1. 类级别的注解：如@Component、@Repository、@Controller、@Service以及JavaEE6的@ManagedBean和@Named注解，都是添加在类上面的类级别注解。
       Spring容器根据注解的过滤规则扫描读取注解Bean定义类，并将其注册到Spring IoC容器中。
    
    2. 类内部的注解：如@Autowire、@Value、@Resource以及EJB和WebService相关的注解等，都是添加在类内部的字段或者方法上的类内部注解。
       SpringIoC容器通过Bean后置注解处理器解析Bean内部的注解。
    
    BeanPostProcessor也称为Bean后置处理器，它是Spring中定义的接口，在Spring容器的创建过程中（具体为Bean初始化前后）会回调BeanPostProcessor中定义的两个方法。   
    postProcessBeforeInitialization()方法会在每一个bean对象的初始化方法调用之前回调
    postProcessAfterInitialization()方法会在每个bean对象的初始化方法调用之后被回调
    
 19. aop场景及原理，实现   
     场景一： 记录日志
     场景二： 监控方法运行时间 （监控性能）
     场景三： 权限控制
     场景四： 缓存优化 （第一次调用查询数据库，将查询结果放入内存对象， 第二次调用， 直接从内存对象返回，不需要查询数据库 ）
     场景五： 事务管理 （调用方法前开启事务， 调用方法后提交关闭事务 ）
     
     AOP 思想： 基于代理思想，对原来目标对象，创建代理对象，在不修改原对象代码情况下，通过代理对象，调用增强功能的代码，从而对原有业务方法进行增强 ！
 
     Spring AOP是基于动态代理机制实现的，通过动态代理机制生成目标对象的代理对象，当外部调用目标对象的相关方法时，Spring注入的其实是代理对象Proxy，通过调用代理对象的方法执行AOP增强处理，然后回调目标对象的方法。
 20. autowire和resource注入的区别
     对比项 	    @Autowire	              @Resource
     装配方式	优先按类型	              优先按名称
     作用范围 	字段、构造器 、setter方法	  字段、setter方法
     属性	    required	              name、type
     注解来源	Spring注解	              JDK
     注意	    作用范围在字段上，均无需在写setter方法
     
     1. @Autowire
        默认按照类型(by-type)装配，默认情况下要求依赖对象必须存在。
        
        ①如果允许依赖对象为null，需设置required属性为false
        ②如果使用按照名称(by-name)装配，需结合@Qualifier注解使用
     2. @Resource
        默认按照名称(by-name)装配，名称可以通过name属性指定。
     
        ①若没有指定name
     
          当在字段上注解时，默认取name=字段名称装配。
          当在setter方法上注解时，默认取name=属性名称装配。
        ②当按照名称(by-name）装配未匹配时，按照类型(by-type)装配。
     
          当显示指定name属性后，只能按照名称(by-name)装配。
          
        注意： @Resoure装配顺序
        
        1. 若指定name属性，则按照名称(by-name)装配，未找到则抛异常；
        2. 若指定type属性，则按照类型(by-type)装配，未找到或者找到多个则抛异常；
        3. 若同时指定name和type属性，则找到唯一匹配的bean装配，未找到则抛异常；
        4. 若既未指定name属性，又未指定type属性，则按照名称(by-name)装配；如果未找到，则按照类型(by-type)装配


 
 21. mybatis的错误在什么时候报出，一二级缓存
     一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。  一级缓存的作用域是同一个SqlSession，在第一个sqlSession执行相同的sql语句后结果放在内存中，第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存也就不存在了。Mybatis默认开启一级缓存。 
     
     本地缓存不能被关闭，可以调用clearCache()来清空本地缓存，或者改变缓存的作用域

     二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession去操作数据库得到数据会存在二级缓存区域，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。
     
     二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。
     
     Mybatis默认没有开启二级缓存需要在setting全局参数中配置开启二级缓存。  
     
     如果缓存中有数据就不用从数据库中获取，大大提高系统性能。
   
     原理:   
     1. SqlSession 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
     2. Executor MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
     3. StatementHandler 封装了JDBC Statement操作，负责对JDBCstatement的操作，如设置参数、将Statement结果集转换成List集合。
     4. ParameterHandler 负责对用户传递的参数转换成JDBC Statement 所需要的参数
     5. ResultSetHandler *负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
     6. TypeHandler 负责java数据类型和jdbc数据类型之间的映射和转换
     7. MappedStatement MappedStatement维护了一条<select|update|delete|insert>节点的封
     8. SqlSource 负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
     9. BoundSql 表示动态生成的SQL语句以及相应的参数信息
     10. Configuration MyBatis所有的配置信息都维持在Configuration对象之中

 
 22. spring aop:jdk proxy,cglib,私有方法可以拦截吗？内部调用呢
     JDK动态代理
     1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
     
     2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
     
     3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
     
     4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。
     
     GCLIB代理
     1. cglib（Code Generation Library）是一个强大的,高性能,高质量的Code生成类库。它可以在运行期扩展Java类与实现Java接口。
     
     2. cglib封装了asm，可以在运行期动态生成新的class。
     
     拦截问题：
     aop拦截内部调用：
     spring的aop无法拦截内部方法调用，动态代理无法拦截，这是jdk基于接口代理所引发的问题
     而实际使用 cglib 则全程都是用的是代理 bean
     
     原因：通过上图的AOP基本原理，我们知道真正执行methodA()的是目标对象，那么methodA()中调用methodB()就是目标对象的方法而不是代理对象的，也就自然不会执行AOP的增强逻辑。
     处理：A中调用B：不要直接用this（因为this是目标对象，自然无法实现代理类的增强方法@before等）,而是先去尝试获取代理类：UserServiceImpl service = AopContext.currentProxy() != null ? (UserService)AopContext.currentProxy() : this;
     
     私有方法：
     jdk是代理接口，私有方法必然不会存在在接口里，所以就不会被拦截到； 
     cglib是子类，private的方法照样不会出现在子类里，也不能被拦截。 
     解决：
     自己写一个代理类，用java反射机制可以获取private修饰的方法， fields[j].setAccessible(true);  注意这句，设置成true以访问私有属性。
 
 23. cglib和jdk代理的区别
     1. JDK动态代理只能对实现了接口的类生成代理，而不能针对类 
     
     2. CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法因为是继承，所以该类或方法最好不要声明成final。利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
 
 24. hibernate隐式回收原理
 
 25. 动态代理和静态代理
     静态：由程序员创建代理类或特定工具自动生成源代码再对其编译。在程序运行前代理类的.class文件就已经存在了。
     
     动态：在程序运行时运用反射机制动态创建而成。