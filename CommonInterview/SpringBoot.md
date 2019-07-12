1. 什么是spring boot
 
   spring boot 致力于简洁，让开发者写更少的配置，程序能够更快的运行和启动。它是下一代javaweb框架，并且它是spring cloud（微服务）的基础。
   Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。
  
2. spring boot优点
  
   + 独立运行
   + 简化配置
   + 自动配置
   + 无代码生成和XML配置
   + 应用监控
   + 上手容易
3. Spring Boot 的配置文件有哪几种格式？它们有什么区别？
   
   1. .properties 
   2. .yml   
   .yml 格式不支持 @PropertySource 注解导入配置
   
4. Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？
   
   启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：
   1. @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。
   2. @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
   3. @ComponentScan：Spring组件扫描。
   
5. 开启 Spring Boot 特性有哪几种方式？
   
   1）继承spring-boot-starter-parent项目
   
   2）导入spring-boot-dependencies项目依赖
   
6. 运行 Spring Boot 有哪几种方式？
   
   1）打包用命令或者放到容器中运行
   
   2）用 Maven/ Gradle 插件运行
   
   3）直接执行 main 方法运行
   
7. 你如何理解 Spring Boot 中的 Starters？
   
   Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成 Spring 及其他技术，而不需要到处找示例代码和依赖包。
   如你想使用 Spring JPA 访问数据库，只要加入 spring-boot-starter-data-jpa 启动器依赖就能使用了。
   
8. 如何在 Spring Boot 启动的时候运行一些特定的代码？
   
   可以实现接口 ApplicationRunner 或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个 run 方法

9. Spring Boot 有哪几种读取配置的方式
   
   Spring Boot 可以通过 @PropertySource,@Value,@Environment, @ConfigurationProperties 来绑定变量

10. Spring Boot 如何定义多套不同环境配置？          
    
    Profile就是Spring Boot可以对不同环境或者指令来读取不同的配置文件。
    
    配置文件spring.profiles.active:@activatedProperties@
    
    在JAVA配置代码中也可以加不同Profile下定义不同的配置文件，@Profile注解只能组合使用@Configuration和@Component注解。
    
    main方法启动方式：
    // 在Eclipse Arguments里面添加
    --spring.profiles.active=prod
    
    插件启动方式：
    spring-boot:run -Drun.profiles=prod
    
    jar运行方式：
    java -jar xx.jar --spring.profiles.active=prod
11. Spring Boot 2.X 有什么新特性？与 1.X 有什么区别？
    
    + 依赖 JDK 版本升级。2.x 至少需要 JDK 8 的支持，2.x 里面的许多方法应用了 JDK 8 的许多高级新特性，所以你要升级到 2.0 版本，先确认你的应用必须兼容 JDK 8。
    + 第三方类库升级
12. springboot与spring的区别

    java在集成spring等框架需要作出大量的配置,开发效率低,繁琐.所以官方提出 spring boot的核心思想:习惯优于配置.可以快速创建开发基于spring框架的项目.或者支持可以不用或很少的spring配置即可.
13. springboot的核心功能与使用优点.
    核心功能:
    1. springboot项目为独立运行的spring项目,java -jar xx.jar即可运行.
    2. 内嵌servlet容器(可以选择内嵌: tomcat ,jetty等服务器.).
    3. 提供了starter的pom 配置 简化了 maven的配置.
    4. 自动配置spring容器中的bean.当不满足实际开发场景,可自定义bean的自动化配置.
    5. 准生产的应用监控(基于: ssh , http , telnet 对服务器运行的项目进行监控.).
    6. springboot无需做出xml配置,也不是通过代码生成来实现(通过条件注解.).
    
    使用优点:
    1. 快速搭建项目,
    2. 与主流框架集成无需配置集成.
    3. 内嵌服务容器.
    4. 具有应用监控.
    5. 开发部署方便,后期与云计算平台集成方便(docket).
14. jpa和Mybatis区别
    
    第一、jpa是对象与对象之间的映射，而mybatis是对象和结果集的映射。
    
    第二、jpa移植性比较好，不用关心用什么数据库，因为mybatis自由写sql语句，所以当项目移植的时候还需要改sql。（及时判断数据库类型，不嫌累么）。
    
    第三、当需要修改字段的时候mybatis改起来特别费事，而jpa就相对简单。
       

    
    
   
    
         