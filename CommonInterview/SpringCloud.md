 1. 服务注册与发现Eureka使用
    启动一个服务注册中心，只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加
    eureka是一个高可用的组件，它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下erureka server也是一个eureka client ,必须要指定一个 server
    当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。
 
 2. 服务消费者
    
    Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign
    
    ribbon是一个负载均衡客户端，可以很好的控制htt和tcp的一些行为。Feign默认集成了ribbon。
    首先Ribbon会从 Eureka Client里获取到对应的服务注册表，也就知道了所有的服务都部署在了哪些机器上，在监听哪些端口号。
    
    简单轮询负载均衡（RoundRobin）： 以轮询的方式依次将请求调度不同的服务器，即每次调度执行i = (i + 1) mod n，并选出第i台服务器。
    
    随机负载均衡 （Random）： 随机选择状态为UP的Server
    
    加权响应时间负载均衡 （WeightedResponseTime）： 根据相应时间分配一个weight，相应时间越长，weight越小，被选中的可能性越低。
    
    区域感知轮询负载均衡（ZoneAvoidanceRule） ：复合判断server所在区域的性能和server的可用性选择server
    
    然后Ribbon就可以使用默认的Round Robin算法，从中选择一台机器
    
    Feign就会针对这台机器，构造并发起请求。
    
    @LoadBalanced注解表明这个restRemplate开启负载均衡的功能。
    通过@EnableDiscoveryClient向服务中心注册
   
    rest+ribbon 
    
    Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。Feign的一个关键机制就是使用了动态代理
    采用的是基于接口的注解。
    Feign 整合了ribbon，具有负载均衡的能力
    整合了Hystrix，具有熔断的能力
    
    首先，如果你对某个接口定义了@FeignClient注解，Feign就会针对这个接口创建一个动态代理
    
    接着你要是调用那个接口，本质就是会调用 Feign创建的动态代理，这是核心中的核心
    
    Feign的动态代理会根据你在接口上的@RequestMapping等注解，来动态构造出你要请求的服务的地址
    
    最后针对这个地址，发起请求、解析响应
    
    @EnableHystrix注解开启Hystrix
    Hystrix会搞很多个小小的线程池，比如订单服务请求库存服务是一个线程池，请求仓储服务是一个线程池，请求积分服务是一个线程池。每个线程池里的线程就仅仅用于请求那个服务。
    
3. Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。      

   在入口applicaton类加上注解@EnableZuulProxy，开启zuul的功能
   
   zuul不仅只是路由，并且还能过滤，做一些安全验证。ZuulFilter
   filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
   pre：路由之前
   routing：路由之时
   post： 路由之后
   error：发送错误调用
   filterOrder：过滤的顺序
   shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
   run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。

4. 服务链路追踪组件zipkin


最后再来总结一下，上述几个Spring Cloud核心组件，在微服务架构中，分别扮演的角色：

 
5. 总结
   Eureka：各个服务启动时，Eureka Client都会将服务注册到Eureka Server，并且Eureka Client还可以反过来从Eureka Server拉取注册表，从而知道其他服务在哪里

   Ribbon：服务间发起请求的时候，基于Ribbon做负载均衡，从一个服务的多台机器中选择一台

   Feign：基于Feign的动态代理机制，根据注解和选择的机器，拼接请求URL地址，发起请求

   Hystrix：发起请求是通过Hystrix的线程池来走的，不同的服务走不同的线程池，实现了不同服务调用的隔离，避免了服务雪崩的问题

   Zuul：如果前端、移动端要调用后端系统，统一从Zuul网关进入，由Zuul网关转发请求给对应的服务