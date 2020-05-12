1. 什么是循环依赖？
   
   循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。
   
   Spring中循环依赖场景有： 
   （1）构造器的循环依赖 
   （2）field属性的循环依赖。

2. Spring怎么解决循环依赖
    
   Spring的循环依赖的理论依据其实是基于Java的引用传递，当我们获取到对象的引用时，对象的field或则属性是可以延后设置的(但是构造器必须是在获取引用之前)。   
   
   Spring的单例对象的初始化主要分为三步：
   
   （1）createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象
   
   （2）populateBean：填充属性，这一步主要是多bean的依赖属性进行填充
   
   （3）initializeBean：调用spring xml中的init 方法。
   
   那么我们要解决循环引用也应该从初始化过程着手，对于单例来说，在Spring容器整个生命周期内，有且只有一个对象，所以很容易想到这个对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。
   
   这三级缓存分别指： 
   singletonFactories ： 单例对象工厂的cache 
   earlySingletonObjects ：提前暴光的单例对象的Cache 
   singletonObjects：单例对象的cache
   
   让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。
   A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，
   尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，
   B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。
   
   知道了这个原理时候，肯定就知道为啥Spring不能解决“A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象”这类问题了！因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。

#### SpringMVC
1. SpringMVC的整个处理流程
   
   1. 用户发送请求至前端控制器DispatcherServlet
   2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
   3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
   4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器
   5. 执行处理器(Controller，也叫后端控制器)。
   6. Controller执行完成返回ModelAndView
   7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet
   8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
   9. ViewReslover解析后返回具体View
   10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。
   11. DispatcherServlet响应用户

#### Spring问题
1. Spring AOP中自我调用的问题
   这里先说一下AOP拦截不到自我调用方法的原因：假设我们有一个类是ServiceA，这个类中有一个A方法，A方法中又调用了B方法。
   当我们使用AOP进行拦截的时候，首先会创建一个ServiceA的代理类，其实在我们的系统中是存在两个ServiceA的对象的，一个是目标ServiceA对象，一个是生成的代理ServiceA对象，如果在代理类的A方法中调用代理类的B方法，这个时候AOP拦截是可以生效的，
   但是如果在代理类的A方法中调用目标类的B方法，这个时候AOP拦截是不生效的，大多数情况下我们都是在代理类的A方法中直接调用目标类的B方法。
   首先调用的是AOP代理对象而不是目标对象，首先执行事务切面，事务切面内部通过TransactionInterceptor环绕增强进行事务的增强，即进入目标方法之前开启事务，退出目标方法时提交/回滚事务。目标对象内部的自我调用将无法实施切面中的增强。 此处的this指向目标对象，因此调用this.b()将不会执行b事务切面，即不会执行事务增强，因此b方法的事务定义“@Transactional(propagation = Propagation.REQUIRES_NEW)”将不会实施
 
2. Spring的事务传播行为
   
   [事务传播](https://segmentfault.com/a/1190000013341344?utm_source=tag-newest)  