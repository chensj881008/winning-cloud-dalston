# Spring Cloud

## 一、常用注解

### 1.1 Spring 注解

#### `@SpringBootTest`

​	@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)

​	为Web 测试环境的端口为随机端口的配置

#### `@ConfigurationProperties`

​	设置属性的信息 如`prefix`就是前缀

#### `@PathVariable`

​	可以获取`RESTful`风格的`Uri`路径上的参数。

#### `@EnableEurekaServer`

​	开启Eureka Server

#### `@EnableEurekaClient`

开启Eureka Client

#### `@EnableDiscoveryClient`

开启客户端注册

> spring cloud中discovery service有许多种实现（eureka、consul、zookeeper等等），
> `@EnableDiscoveryClient`基于`spring-cloud-commons`, 
> `@EnableEurekaClient`基于`spring-cloud-netflix`。
> 其实用更简单的话来说，就是如果选用的注册中心是`eureka`，那么就推荐`@EnableEurekaClient`，
> 如果是其他的注册中心，那么推荐使用`@EnableDiscoveryClient`。

#### `@Import`

​	`@Import(EurekaServerInitializerConfiguration.class)`

* 将标记的class注入到spring IOC容器中

* 只能注解在类上，以及唯一的参数value上可以配置3种类型的值Configuration，ImportSelector，ImportBeanDefinitionRegistrar

  * **基于Configuration也就是直接填对应的class数组**

    ```java 
    @Import({Square.class,Circular.class})
    ```

  * **基于自定义ImportSelector的使用**

    ```java
    /**
     * 定义一个我自己的ImportSelector
     *
     * @author zhangqh
     * @date 2018年5月1日
     */
    public class MyImportSelector implements  ImportSelector{
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            return new String[]{"com.zhang.bean.Triangle"};
        }
    }
    
    // 使用@Import
    @Import({Square.class,Circular.class,MyImportSelector.class})
    ```

  * **基于ImportBeanDefinitionRegistrar的使用**

      ```java
      /**
       * 定义一个自定的ImportBeanDefinitionRegistrar
       *
       * @author zhangqh
       * @date 2018年5月1日
       */
      public class MyImportBeanDefinitionRegistrar  implements ImportBeanDefinitionRegistrar{
          public void registerBeanDefinitions(
                  AnnotationMetadata importingClassMetadata,
                  BeanDefinitionRegistry registry) {
              // new一个RootBeanDefinition
              RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Rectangle.class);
              // 注册一个名字叫rectangle的bean
              registry.registerBeanDefinition("rectangle", rootBeanDefinition);
          }
      }

      // 使用@Import
      @Import({Square.class,Circular.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
      ```

#### `@Conditional`

`@Conditional`是Spring4新提供的注解，它的作用是按照一定的条件进行判断，满足条件给容器注册bean。

```java
//此注解可以标注在类和方法上
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME) 
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

从代码中可以看到，需要传入一个Class数组，并且需要继承Condition接口：

```java
public interface Condition {
    boolean matches(ConditionContext var1, AnnotatedTypeMetadata var2);
}
```

Condition是个接口，需要实现matches方法，返回true则注入bean，false则不注入。

#### `@ConditionalOnMissingBean`

（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）

该注解表示，如果存在它修饰的类的bean，则不需要再创建这个bean

如果当前容器中已经有bean了，就不注入备用bean，如果没有，则注入备用bean

#### `@ConditionalOnClass`

（某个class位于类路径上，才会实例化一个Bean）

​	该注解的参数对应的类必须存在，否则不解析该注解修饰的配置类

注解@ConditionalOnClass和@Bean,可以仅当某些类存在于 classpath 上时候才创建某个Bean

常规使用代码如下：

```java
 // 仅当类 java.util.HashMap 存在于 classpath 上时才创建一个bean : beanA
 // 注意这里使用了 @ConditionalOnClass 的属性value，
 @Bean
 @ConditionalOnClass(value={java.util.HashMap.class})
 public A beanA(){}
 // 仅当类 com.sample.Dummy 存在于 classpath 上时才创建一个bean : beanB
 // 注意这里使用了 @ConditionalOnClass 的属性 name，
 @Bean
 @ConditionalOnClass(name="com.sample.Dummy")
 public B beanB(){}
```

`name`or `value`

- `name` : 不确定指定类在`classpath` 上
- `value` : 确定指定类在 `classpath` 上

#### @ConditionalOnBean

​	仅仅在当前上下文中存在某个对象时，才会实例化一个Bean

#### @ConditionalOnExpression

​	当表达式为true的时候，才会实例化一个Bean

#### @ConditionalOnMissingClass

​	某个class类路径上不存在的时候，才会实例化一个Bean）

#### @ConditionalOnNotWebApplication

（不是web应用）

#### `@Autowired` 

用来注入已有的bean,required`默认为true，当为false的时候，表示忽略当前要注入的bean，如果有直接注入，没有跳过，不会报错


### 1.2 java元注解

java中元注解有四个： @Retention @Target @Document @Inherited；

```java
 @Retention：注解的保留位置　　　　　　　　　
       @Retention(RetentionPolicy.SOURCE)   //注解仅存在于源码中，在class字节码文件中不包含
　　    @Retention(RetentionPolicy.CLASS)     // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得，
　　　	 @Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到
@Target:注解的作用目标　
        @Target(ElementType.TYPE)   //接口、类、枚举、注解
        @Target(ElementType.FIELD) //字段、枚举的常量
        @Target(ElementType.METHOD) //方法
        @Target(ElementType.PARAMETER) //方法参数
        @Target(ElementType.CONSTRUCTOR)  //构造函数
        @Target(ElementType.LOCAL_VARIABLE)//局部变量
        @Target(ElementType.ANNOTATION_TYPE)//注解
        @Target(ElementType.PACKAGE) ///包   
@Document：说明该注解将被包含在javadoc中
@Inherited：说明子类可以继承父类中的该注解
```

## 二、坑点

### 1、`eureka client` 使用如下配置时，启动后会自动关闭

在spring cloud `Finchley.RELEASE`、`Greenwich.RELEASE`都有问题

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
必须添加如下的引用，才不会启动后关闭
```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### mysql

问题：The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone....

解决方案:

```bash
show variables like '%time_zone%';
set global time_zone='+8:00';
```

或者在my.ini中增加[mysqld]下面的配置

```bash
default-time-zone='+08:00'
```

### Feign @PathVariable
  使用Feign的时候,如果参数中带有@PathVariable形式的参数,则要用value=""标明对应的参数,否则会抛出IllegalStateException异常
  ```java
        @PutMapping("/disuseable/{sn}")
        ApiResponse disUseAble(@PathVariable String sn);   // wrong
  ```
   修改为

   ```java
      @PutMapping("/disuseable/{sn}")
      ApiResponse disUseAble(@PathVariable(value="sn") String sn);  // right
   ```



## 三、Eureka 治理中心

### 3.1 Eureka概念

#### （1）Register-服务注册

​	当Eureka Client向Eureka Server注册时，Eureka Client 提供自身的元数据，比如 IP 地址、端口、运行状况H1标的 Uri 主页地址等信息。

#### （2）Renew-服务续约

​	Eureka CLIent 在默认的情况下会每隔 30 秒发送一次心跳来进行服务续约。通过服务续约来告知Eureka Server该Eureka Client 仍然可用，没有出现故障。正常情况下，如果Eureka Server在90 秒内没有收到 ureka Client 的心跳， Eureka Server 会将 Eureka Client 实例从注册列表中删除。注意：官网建议不要更改服务续约的间隔时间。

#### （3）Fetch Registries-获取服务注册列表信息

​	Eureka Client从Eureka Server 获取服务注册表信息，井将其缓存在本地。Eureka Client会
使用服务注册列表信息查找其他服务的信息，从而进行远程调用。该注册列表信息定时（每30 秒） 更新一次，每次返回注册列表信息可能与 Eureka Client 的缓存信息不同，Eureka Client会自己处理这些信息。如果由于某种原因导致注册列表信息不能及时匹配，Eureka Client 会重新获取整个注册表信息。 Eureka Server 缓存了所有的服务注册列表信息，并将整个注册列表以及每个应用程序 信息进行了压缩，压缩内容和没有压缩的内容完全相同。 Eureka Client和Eureka Server 可以使用 JSON和XML 数据格式进行通信。在默认的情况下， Eureka Client使用JSON 格式的方式来获取服务注册列表的信息。

####  （4）Cancel-服务下线

​	Eureka Client 在程序关闭时可以向 Eureka Server 发送下线请求。发送请求后，该客户端的
实例信息将从Eureka Server 的服务注册列表中删除。该下线请求不会自动完成，需要在程序关闭时调用以下代码：

```java
DiscoveryManager.getinstance().shutdownComponent();
```

#### （5）Eviction-服务剔除

在默认情况下，当 Eureka Client 90 秒没有向 Eureka Server 发送服务续约（即心跳〉时，Eureka Server 会将该服务实例从服务注册列表删除，即服务剔除。

### 3.2 Eureka 架构

![Eureka高可用架构](https://github.com/Netflix/eureka/raw/master/images/eureka_architecture.png)

​	在这个架构中有两个角色 ，即 Eureka Server和Eureka Client。而Eureka Client 又分为Applicaton Service和Application Client 即服务提供者和服务消费者。每个区域有一个Eureka 集群，并且每个区域至少有一个 Eureka Server 可以处理区域故障，以防服务器瘫痪。

​	Eureka Client向Eureka Server注册， 将自己的客户端信息提交给 Eureka Server。然后，Eureka Client通过向 Eureka Server 发送心跳（每 30 次）来续约服务。如果某个客户端不能持续续约，那Eureka Server 判定该客户端不可用，该不可用的客户端将在大约 90 秒后从Eureka Server服务注册列表中删除。服务注册列表信息和服务续约信息会被复制到集群中的每个Eureka Server节点。来自任何区域的Eureka Client 都可以获取整个系统的服务注册列表信息，根据这些注 列表信息，Application Client 可以远程调用Applicaton Service 来消费服务。

### 3.3 服务注册

#### 3.3.1 Eureka Client

​	服务注册，即Eureka Client向Eureka Server 提交自己的服务信息，包括 IP 地址、端口、Serviceld 等信息。如果 Eureka Client 置文件中没有配置 Serviceld ，则默认为配置文件中配置的服务名 ，即`$ {spring application.name ｝`的值。

​	当Eureka Client 启动时，会将自身 的服务信息发送到 Eureka Server 这个过程其实非常简单，现在来从源码的角度分析服务注册的过程。在Maven的依赖包下，找到`eureka-client-1.9.8.jar`包。在`com.netflix.discovery`包下有`DiscoveryClient`类，该类包含了
Eureka Client向Eureka Server注册和续约的相关方法。其中，`DiscoveryClient`实现了`EurekaClient`
并且它是单例模式(`@Singleton`)，而 `EurekaClient`继承了`LookupService`接口之间的关系:

![1554910343477](C:\Users\chensj\AppData\Roaming\Typora\typora-user-images\1554910343477.png)

在`DiscoveryClient`类中有 个服务注册的方法`register()`， 该方法Http请求`Eureka Server`注册，代码如下：

```java
boolean register() throws Throwable {
        logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
        EurekaHttpResponse<Void> httpResponse;
        try {
            httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
        } catch (Exception e) {
            logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
            throw e;
        }
        if (logger.isInfoEnabled()) {
            logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
        }
        return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
    }

```

这个方法的调用则是来至于`InstanceInfoReplicator`,该类实现了`Runnable`接口，在`run()`方法中调用了`DiscoveryClient.register()`

```java
  @Override
    public void run() {
        try {
            // 刷新 应用实例信息
            discoveryClient.refreshInstanceInfo();
            // 判断 应用实例信息 是否数据不一致
            Long dirtyTimestamp = instanceInfo.isDirtyWithTime();
            if (dirtyTimestamp != null) {
                // TODO 发起注册
                discoveryClient.register();
                // 设置 应用实例信息 数据一致
                instanceInfo.unsetIsDirty(dirtyTimestamp);
            }
        } catch (Throwable t) {
            logger.warn("There was a problem with the instance info replicator", t);
        } finally {
            // 提交任务，并设置该任务的 Future
            Future next = scheduler.schedule(this, replicationIntervalSeconds, TimeUnit.SECONDS);
            scheduledPeriodicRef.set(next);
        }
    }
```

而`InstancelnfoReplicator`类是在`DiscoveryClient`初始化过程中使用的， 其中有一个`initScheduledTasks()` 方法，该方法主要开启了 取服务注册列表的信息。如果需要向Eureka Server注册，则开启注册，同时开启定时任务Eureka Server服务续约。

```java
private void initScheduledTasks() {
        // 获取服务列表，任务调度获取注册列表代码
        if (clientConfig.shouldFetchRegistry()) {
            // registry cache refresh timer
            int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
            int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "cacheRefresh",
                            scheduler,
                            cacheRefreshExecutor,
                            registryFetchIntervalSeconds,
                            TimeUnit.SECONDS,
                            expBackOffBound,
                            new CacheRefreshThread()
                    ),
                    registryFetchIntervalSeconds, TimeUnit.SECONDS);
        }
		// 向Eureka Server注册服务
        if (clientConfig.shouldRegisterWithEureka()) {
            int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
            int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();
            logger.info("Starting heartbeat executor: " + "renew interval is: {}", renewalIntervalInSecs);

            // Heartbeat timer
            // 心跳检测，并续约
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "heartbeat",
                            scheduler,
                            heartbeatExecutor,
                            renewalIntervalInSecs,
                            TimeUnit.SECONDS,
                            expBackOffBound,
                            new HeartbeatThread()
                    ),
                    renewalIntervalInSecs, TimeUnit.SECONDS);

            // InstanceInfo replicator
            instanceInfoReplicator = new InstanceInfoReplicator(
                    this,
                    instanceInfo,
                    clientConfig.getInstanceInfoReplicationIntervalSeconds(),
                    2); // burstSize

            statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
                @Override
                public String getId() {
                    return "statusChangeListener";
                }

                @Override
                public void notify(StatusChangeEvent statusChangeEvent) {
                    if (InstanceStatus.DOWN == statusChangeEvent.getStatus() ||
                            InstanceStatus.DOWN == statusChangeEvent.getPreviousStatus()) {
                        // log at warn level if DOWN was involved
                        logger.warn("Saw local status change event {}", statusChangeEvent);
                    } else {
                        logger.info("Saw local status change event {}", statusChangeEvent);
                    }
                    // 更新客户端信息，并发送到服务端
                    instanceInfoReplicator.onDemandUpdate();
                }
            };

            if (clientConfig.shouldOnDemandUpdateStatusChange()) {
                applicationInfoManager.registerStatusChangeListener(statusChangeListener);
            }

           
 // 开启 应用实例信息复制器 TODO 并在指定延迟时间后执行一次注册
            instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }
    }
```

#### 3.3.2 Eureka Server

​		再来跟踪 Eureka server端的代码，在Maven的 org.springframework.cloud:spring-cloud-netflix-eureka-server:2.1.0.RELEASE包下，在org.springframework.cloud.netflix.eureka.server会发现有一 个`EurekaServerBootstrap`的类，将监听`ServletContextEvent`事件，所以` EurekaServerBootstrap`类在程序启动时具有最先初始化的权利，初始化的时候将会调用如下方法：

```java
public void contextInitialized(ServletContextEvent event) {
        try {
            // 初始化 Eureka-Server 配置环境
            initEurekaEnvironment();

            // 初始化 Eureka-Server 上下文
            initEurekaServerContext();

            ServletContext sc = event.getServletContext();
            sc.setAttribute(EurekaServerContext.class.getName(), serverContext);
        } catch (Throwable e) {
            logger.error("Cannot bootstrap eureka server :", e);
            throw new RuntimeException("Cannot bootstrap eureka server :", e);
        }
    }
```

​		在`initEurekaServerContext()`中，会有创建`PeerAwareInstanceRegistry`(应用实例信息的注册表)实例，这个类就是保持客户端注册信息的地方，同时也会创建`PeerEurekaNodes`实例，保存注册中心集群的地方，用于后续将注册信息推送到同等节点时候使用。

```java
 // 【2.2.5】创建 应用实例信息的注册表
        PeerAwareInstanceRegistry registry;
        // AWS 相关，跳过
        if (isAws(applicationInfoManager.getInfo())) {
            registry = new AwsInstanceRegistry(
                    eurekaServerConfig,
                    eurekaClient.getEurekaClientConfig(),
                    serverCodecs,
                    eurekaClient
            );
            awsBinder = new AwsBinderDelegate(eurekaServerConfig, eurekaClient.getEurekaClientConfig(), registry, applicationInfoManager);
            awsBinder.start();
        } else {
            registry = new PeerAwareInstanceRegistryImpl(
                    eurekaServerConfig,
                    eurekaClient.getEurekaClientConfig(),
                    serverCodecs,
                    eurekaClient
            );
        }

        // 【2.2.6】创建 Eureka-Server 集群节点集合
        PeerEurekaNodes peerEurekaNodes = getPeerEurekaNodes(
                registry,
                eurekaServerConfig,
                eurekaClient.getEurekaClientConfig(),
                serverCodecs,
                applicationInfoManager
        );
```

​		通过代码跟踪发现，执行注册的是`com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#register`,代码如下：

```java
 public void register(final InstanceInfo info, final boolean isReplication) {
        // 租约过期时间
        int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
        if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
            leaseDuration = info.getLeaseInfo().getDurationInSecs();
        }
        // 注册应用实例信息
        super.register(info, leaseDuration, isReplication);
        // Eureka-Server peer node 复制
        replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
    }
```

​		这里会调用父类`register()`，也就是`com.netflix.eureka.registry.AbstractInstanceRegistry#register`，这个时候就可以知道注册列表信息是保存在一个Map中，在完成client注册后，再讲client推送到各个同等节点中去`replicateToPeers`。

​		使用方法调用方查看，会将代码追踪到`com.netflix.eureka.resources.ApplicationResource#addInstance`,这个接口是一个处理Http协议的Post请求方法，这里面会调用`PeerAwareInstanceRegistryImpl#register`

```java
 @POST
    @Consumes({"application/json", "application/xml"})
    public Response addInstance(InstanceInfo info,
                                @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
        // 校验参数是否合法
        logger.debug("Registering instance {} (replication={})", info.getId(), isReplication);
        // validate that the instanceinfo contains all the necessary required fields
        if (isBlank(info.getId())) {
            return Response.status(400).entity("Missing instanceId").build();
        } else if (isBlank(info.getHostName())) {
            return Response.status(400).entity("Missing hostname").build();
        } else if (isBlank(info.getIPAddr())) {
            return Response.status(400).entity("Missing ip address").build();
        } else if (isBlank(info.getAppName())) {
            return Response.status(400).entity("Missing appName").build();
        } else if (!appName.equals(info.getAppName())) {
            return Response.status(400).entity("Mismatched appName, expecting " + appName + " but was " + info.getAppName()).build();
        } else if (info.getDataCenterInfo() == null) {
            return Response.status(400).entity("Missing dataCenterInfo").build();
        } else if (info.getDataCenterInfo().getName() == null) {
            return Response.status(400).entity("Missing dataCenterInfo Name").build();
        }

        // AWS 相关，跳过
        // handle cases where clients may be registering with bad DataCenterInfo with missing data
        DataCenterInfo dataCenterInfo = info.getDataCenterInfo();
        if (dataCenterInfo instanceof UniqueIdentifier) {
            String dataCenterInfoId = ((UniqueIdentifier) dataCenterInfo).getId();
            if (isBlank(dataCenterInfoId)) {
                boolean experimental = "true".equalsIgnoreCase(serverConfig.getExperimental("registration.validation.dataCenterInfoId"));
                if (experimental) {
                    String entity = "DataCenterInfo of type " + dataCenterInfo.getClass() + " must contain a valid id";
                    return Response.status(400).entity(entity).build();
                } else if (dataCenterInfo instanceof AmazonInfo) {
                    AmazonInfo amazonInfo = (AmazonInfo) dataCenterInfo;
                    String effectiveId = amazonInfo.get(AmazonInfo.MetaDataKey.instanceId);
                    if (effectiveId == null) {
                        amazonInfo.getMetadata().put(AmazonInfo.MetaDataKey.instanceId.getName(), info.getId());
                    }
                } else {
                    logger.warn("Registering DataCenterInfo of type {} without an appropriate id", dataCenterInfo.getClass());
                }
            }
        }

        // 注册应用实例信息 @see com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl.register
        registry.register(info, "true".equals(isReplication));

        // 返回 204 成功
        // 204 to be backwards compatible
        return Response.status(204).build();
    }
```

### 3.4 服务续约

#### 3.4.1 Eureka Client

同样是在在`com.netflix.discovery`包下有`DiscoveryClient`类中，查看`DiscoveryClient#renew`方法

```java
 boolean renew() {
        EurekaHttpResponse<InstanceInfo> httpResponse;
        try {
            httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
            logger.debug("{} - Heartbeat status: {}", PREFIX + appPathIdentifier, httpResponse.getStatusCode());
            if (httpResponse.getStatusCode() == 404) {
                REREGISTER_COUNTER.increment();
                logger.info("{} - Re-registering apps/{}", PREFIX + appPathIdentifier, instanceInfo.getAppName());
                long timestamp = instanceInfo.setIsDirtyWithTime();
                // 发起注册
                boolean success = register();
                if (success) {
                    instanceInfo.unsetIsDirty(timestamp);
                }
                return success;
            }
            return httpResponse.getStatusCode() == 200;
        } catch (Throwable e) {
            logger.error("{} - was unable to send heartbeat!", PREFIX + appPathIdentifier, e);
            return false;
        }
    }
```

查看这个方法的调用，来至于`com.netflix.discovery.DiscoveryClient.HeartbeatThread`的`run`方法，这个是在DiscoveryClient初始化(initScheduledTasks)的时候使用的，用于创建心跳检测使用的定时器的时候，会启动这个线程，就会按照时间来启动任务：

```java
// TODO 心跳检测，并续约
scheduler.schedule(
    //监督任务，在执行超时时调度子任务
    new TimedSupervisorTask(
        "heartbeat",
        scheduler,
        heartbeatExecutor,
        renewalIntervalInSecs,
        TimeUnit.SECONDS,
        expBackOffBound,
        new HeartbeatThread()
    ),
    renewalIntervalInSecs, TimeUnit.SECONDS);

// HeartbeatThread.class
private class HeartbeatThread implements Runnable {
    public void run() {
        // 续约
        if (renew()) {
            lastSuccessfulHeartbeatTimestamp = System.currentTimeMillis();
        }
    }
}

```

#### 3.4.2 Eureka Server

​	同样是在`com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl`中，方法为`renew`

```java
 public boolean renew(final String appName, final String id, final boolean isReplication) {
     // 续租
     if (super.renew(appName, id, isReplication)) {
         // Eureka-Server 复制
         replicateToPeers(Action.Heartbeat, appName, id, null, null, isReplication);
         return true;
     }
     return false;
 }
```

继续向上跟踪方法调用方法，就会到`com.netflix.eureka.resources.InstanceResource#renewLease`,这是一个处理http协议的put请求，用于接收客户端发送的put请求

```java
 @PUT
    public Response renewLease(
            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication,
            @QueryParam("overriddenstatus") String overriddenStatus,
            @QueryParam("status") String status,
            @QueryParam("lastDirtyTimestamp") String lastDirtyTimestamp) {
        boolean isFromReplicaNode = "true".equals(isReplication);
        // 续租
        boolean isSuccess = registry.renew(app.getName(), id, isFromReplicaNode);

        // 续租失败
        // Not found in the registry, immediately ask for a register
        if (!isSuccess) {
            logger.warn("Not Found (Renew): {} - {}", app.getName(), id);
            return Response.status(Status.NOT_FOUND).build();
        }

        // 比较 InstanceInfo 的 lastDirtyTimestamp 属性
        // Check if we need to sync based on dirty time stamp, the client
        // instance might have changed some value
        Response response = null;
        if (lastDirtyTimestamp != null && serverConfig.shouldSyncWhenTimestampDiffers()) {
            response = this.validateDirtyTimestamp(Long.valueOf(lastDirtyTimestamp), isFromReplicaNode);
            // Store the overridden status since the validation found out the node that replicates wins
            if (response.getStatus() == Response.Status.NOT_FOUND.getStatusCode()
                    && (overriddenStatus != null)
                    && !(InstanceStatus.UNKNOWN.name().equals(overriddenStatus))
                    && isFromReplicaNode) {
                registry.storeOverriddenStatusIfRequired(app.getAppName(), id, InstanceStatus.valueOf(overriddenStatus));
            }
        } else { // 成功
            response = Response.ok().build();
        }
        logger.debug("Found (Renew): {} - {}; reply status={}" + app.getName(), id, response.getStatus());
        return response;
    }
```

### 3.5 续约时间与剔除时间设置

​	续约时间：表示客户端多长时间发送一次请求到服务端，表示当前客户端依然可以使用

```yaml
eureka:
  instance:
    # 续约的时间间隔，默认为30s,不建议修改
    lease-renewal-interval-in-seconds: 30
```

​	剔除时间: 表示服务单多久未接收到客户端发出的续约请求或者心跳检测，服务端将会把这个应用实例从可用实例剔除。

```yaml
eureka:
  instance:
    # 剔除多久未接受到心跳的实例的时间参数，不建议修改
    lease-expiration-duration-in-seconds: 90
```

### 3.6 为什么Eureka Client获取服务实例这么慢

#### 3.6.1 Eureka Client注册延迟

​		从上面注册的代码中可以看出，注册使用的是如下的代码

```java
// 开启 应用实例信息复制器 TODO 并在指定延迟时间后执行一次注册
instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
```

`InstanceInfoReplicator`的`start`方法中会传入一个时间参数，促使注册的流程延迟执行

```java
 public void start(int initialDelayMs) {
        if (started.compareAndSet(false, true)) {
            // 设置 应用实例信息 数据不一致
            instanceInfo.setIsDirty();  // for initial register
            // 提交任务，并设置该任务的 Future
            // 延迟initialDelayMs后，运行且仅仅运行一次InstanceInfoReplicator的run()
            Future next = scheduler.schedule(this, initialDelayMs, TimeUnit.SECONDS);
            scheduledPeriodicRef.set(next);
        }
    }

```

而这个时间参数是来自于`com.netflix.discovery.DiscoveryClient`创建的时候，使用的`EurekaClientConfig`中获取的，通过跟踪代码发现，这个方法来自于`com.netflix.discovery.DefaultEurekaClientConfig#getInitialInstanceInfoReplicationIntervalSeconds`，延迟了40s

```java
public int getInitialInstanceInfoReplicationIntervalSeconds() {
        return configInstance.getIntProperty(
                namespace + INITIAL_REGISTRATION_REPLICATION_DELAY_KEY, 40).get();
    }
```

#### 3.6.2 Eureka Server 的响应缓存

​	Eureka Server 维护每 30 更新一次响应缓存，可通过更改配置`eureka.server.response-cache-update-interval-ms`来修改。所以即使是刚刚注册的实例，也不会立即出现在服务注册列表中。

#### 3.6.3  Eureka Client 缓存

​	Eureka Client 保留注册表信息的缓存。该缓存每30秒更新一次(如前所述).因此,Eureka Client刷新本地缓存并发现其他新注册的实例可能需要30 秒。

#### 3.6.4 LoadBalancer 缓存

​	Ribbon的负载平衡器从本地的 Eureka Client 获取服务注册列表信息。 Ribbon本身还维护了缓存，以避免每个请求都需要从 Eureka Client获取服务注册列表。此缓存每30秒刷新一次(可由 ribbon.ServerListRefreshlnterval设置），所以可能至少需要30秒的时间才能使用新注册的实例。
​	综上因素，一个新注册的实例，默认延迟 40 秒向服务注册中心注册，所以不能马上被Eureka Server 发现。另外，刚注册的 Eureka Client 不能立即被其他服务调用，原因是调用方由于各种缓存没有及时获取到最新的服务注册列表信息。

### 3.7 Eureka自我保护

​		当有一个新的 Eureka Server 出现时，它尝试从相邻 Peer节点获取所有服务实例注册表信息。如果从相邻 Peer 点获取信息时出现了故障 Eureka Server 会尝试其他的 Peer 节点。如果Eureka Server能够成功获取所有的服务实例信息，则根据配置信息设置服务续约的阀值。在任何时间，如果 Eureka Server接收到的服务续约低于为该值配置的百分比（默认为 15 分钟内低于 85% ）,则服务器开启自我保护模式，即不再剔除注册列表的信息。
​		这样做的好处在于，如果是 Eureka Server 自身的网络问题而导致Eureka Client无法续约，Eureka Client 注册列表信息不再被删除，也就是Eureka Client 还可以被其他服务消费。
​	在默认情况下， Eureka Server 的自我保护模式是开启的，如果需要关闭，则在配置文件添加以下代码：

```yam
eureka:
  server:
    # 自我保护
    enable-self-preservation: false
```

### 3.8 高可用注册中心

#### 3.8.1 两个集群
```yaml
# peer1 
server:
  # 端口
  port: 8000
eureka:
  instance:
    hostname: peer1
  client:
    # 本身为注册中心，不需要去检索服务信息
    register-with-eureka: false
    # 本身为注册中心，是否需要在注册中心注册，默认true  集群设置为true
    fetch-registry: true
    # 注册地址
    service-url:
      # 默认访问地址
      defaultZone: http://peer2:8001/eureka
spring:
  application:
    name: eureka-server
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/java

```

```yaml
# peer2 
server:
  # 端口
  port: 8000
eureka:
  instance:
    hostname: peer1
  client:
    # 本身为注册中心，不需要去检索服务信息
    register-with-eureka: false
    # 本身为注册中心，是否需要在注册中心注册，默认true  集群设置为true
    fetch-registry: true
    # 注册地址
    service-url:
      # 默认访问地址
      defaultZone: http://peer1:8000/eureka
spring:
  application:
    name: eureka-server
```



#### 3.8.2 多个集群

##### *appliction.yml*

```yml
eureka:
  client:
    service-url:
      # 默认访问地址
      defaultZone: http://peer1:8000/eureka,http://peer2:8001/eureka,http://peer3:8002/eureka
```

##### *application-peerX.yml*

```yaml
server:
  # 端口
  port: 8000
eureka:
  instance:
    hostname: peerX
  client:
    # 本身为注册中心，不需要去检索服务信息
    register-with-eureka: false
    # 本身为注册中心，是否需要在注册中心注册，默认true  集群设置为true
    fetch-registry: true
spring:
  application:
    name: eureka-server-peerX
```

修改上面的`peerX`到指定参数即可

## 四、Ribbon 负载均衡

​	 Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的，包括后续的Feign，它也是基于Ribbon实现的工具。所以，对Spring Cloud Ribbon的理解和使用，对于我们使用Spring Cloud来构建微服务非常重要。

### 4.1 RestTemplate

​	`RestTemplate`是`Spring Resources`中一个访问 第三方RESTful API接口的网络请求框架。RestTemplate 的设计原则和其他Spring Template（例如 JdbcTemplate、JmsTemplate ）类似，都是为执行复杂任务提供了具有默认行为的简单方法。
​	`RestTemplate`是用来消费`REST`服务的，所以`RestTemplate`主要方法都与REST的HTTP协议的一些方法紧密相连，例如HEAD、GET、POST、PUT、DELETE、OPTIONS 等方法，这些方法在`RestTemplate`类对应的方法为 `headForHeaders()`、 `getForObject()`、`postForObject()`、`put()`和`delete()`等。

测试使用代码：

```java
// 配置类
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory){
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory(){
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(15000);
        factory.setReadTimeout(5000);
        return factory;
    }
}
//Controller
 @Autowired
    private RestTemplate restTemplate;
    /**
     * RestTemplate 请求
     */
    @PostMapping(value = "/hello")
    public Map<String,Object> hello(){
        Map<String,Object> resultMap = new HashMap<>();
        resultMap.put("data",restTemplate.getForObject("https://www.baidu.com",String.class));
        resultMap.put("status", Constants.SUCCESS_STATUS);
        return resultMap;
    }
```

### 4.2 Ribbon

​		负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式。一种是**独立进程单元**，通过**负载均衡策略**，将**请求转发**到不同的执行单元上，例如**Ngnix** 。另一种是将**负载均衡逻辑以代码的形式**封装到服务消费者的**客户端**上，服务消费者客户端维护了一份服务提供的信息列表，有了信息列表，通过**负载均衡策略**将**请求分摊**给多个服务提供者，从而达到负载均衡的目的。

​		`Ribbon Netflix`公司开源的一个负载均衡的组件，它属上述的第二种方式，是将负载均衡逻辑封装在客户端中，并且运行在客户端的进程里。 `Ribbon`是一个经过了云端测试的IPC库，可以很好地控制 HTTP和TCP客户端的负载均衡行为。

​		在`Spring Cloud `构建的微服务系统中， `Ribbon `作为服务消费者的负载均衡器，有两种使用方式，一 种是和` RestTemplate`相结合，另一种是和`Feign`相结合。`Feign`已经默认集成了`Ribbon`。

​		Ribbon有很多子模块，但很多模块没有用于生产环境，目前 Netflix 公司用于生产环境的 Ribbon
子模块如下。

* `ribbon-loadbalance` ：可以独立使用或与其他模块 起使用的负载均衡器 API
* `ribbon-eureka` ：Ribbon结合Eureka客户端的API，为负载均衡器提供动态服务注册列表信息。
* `ribbon-core`: Ribbon的核心API

### 4.3 RestTemplate+Ribbon

#### 4.3.1 配置类

```java
@Configuration
public class RestTemplateConfig {

    /**
     * 实例化RestTemplate，注册bean
     * 使用@LoadBalanced就会结合Ribbon进行负载均衡
     * @return
     */
    @Bean
    @LoadBalanced
    public RestTemplate initRestTemplate(){
        return  new RestTemplate();
    }
}

```

#### 4.3.2 测试服务

```java
@Service
public class RibbonService {

    @Autowired
    private RestTemplate restTemplate;

    /**
     * 请求http://eureka-client/hi/{username}
     * @param username
     * @return
     */
    public String hi(String username){
        return restTemplate.getForObject("http://eureka-client/hi/"+username,String.class);
    }
}

```

#### 4.3.3 控制层

```java
@RestController
public class RestClientController {

    @Autowired
    private RibbonService ribbonService;

    @PostMapping(value = "/hi/{username}")
    public Map hi(@PathVariable String username){
        Map<String,Object> resultMap = new HashMap<>();
        resultMap.put("data",ribbonService.hi(username));
        resultMap.put("status", Constants.SUCCESS);
        return resultMap;
    }

    @Autowired
    private LoadBalancerClient loadBalancer;

    /**
     * 获取实例信息，从结果来看，将会是从每个客户端走一次
     * @return
     */
    @PostMapping(value = "testRibbon")
    public String testRibbon(){
        ServiceInstance instance = loadBalancer.choose("eureka-client");
        return instance.getHost()+":"+instance.getPort();
    }
}
```

> 上面的例子就是实现了负载均衡

### 4.4 LoadBalancerClient

​		在上面的工程的基础上，复制一份，修改application.yml文件为如下内容：

```yaml
server:
  port: 8769
# 定义Ribbon客户端将访问的服务，服务的InstanceId是`stores`
stores:
  ribbon:
    listOfServers: example.com,google.com
# 不注册
ribbon:
  eureka:
    enabled: false
```

控制层代码：

```java
@RestController
public class RibbonController {

    @Autowired
    private LoadBalancerClient loadBalancer;

    /**
     * 这个方法可以知道：
     *  在Ribbon 中的负载均衡客户端为LoadBalancerClient 。在Spring Cloud项目中，
     *  负载均衡器 Ribbon 会默认从Eureka Client 的服务注册列表中获取服务的信息，并缓存。
     *  根据缓存的服务注册列表信息，可以通过LoadBalancerClient来选择不同的服务实例，
     *  从而实现负载均衡。如果禁止Ribbon Eureka获取注册列表信息，则需要自己去维护一份服
     *  务注册列表信息。 根据自己维护服务注册列表的信息，Ribbon可以实现负载均衡。
     * @return
     */
    @PostMapping(value = "/testRibbon")
    public String testRibbon(){
        // 从配置文件中获取stores的服务端信息
        ServiceInstance instance = loadBalancer.choose("stores");
        return instance.getHost()+":"+instance.getPort();
    }
}
```

结果：

```bash
example.com:80
google.com:80
```

结论：

     	在Ribbon中的负载均衡客户端为LoadBalancerClient 。在Spring Cloud项目中，负载均衡器 Ribbon 会默认从Eureka Client 的服务注册列表中获取服务的信息，并缓存。根据缓存的服务注册列表信息，可以通过LoadBalancerClient来选择不同的服务实例，从而实现负载均衡。如果禁止Ribbon Eureka获取注册列表信息，则需要自己去维护一份服务注册列表信息。 根据自己维护服务注册列表的信息，Ribbon可以实现负载均衡。
### 4.5 Ribbon 源码

#### 4.5.1 LoadBalancerClient

​	首先，跟踪`LoadBalancerClient`的源码，它是一个接口类，继承了`ServiceinstanceChooser`，它的实现类为 `RibbonLoadBalanceClient`，它们之间的关系如下：

![LoadBalancerClient](https://upload-images.jianshu.io/upload_images/15186572-e0d91cb242cef494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/287/format/webp)

​	在`LoadBalancerClient`中有三个方法，两个`execute`方法，均用来执行请求，一个`reconstructURI`方法，用于重构Url，代码如下：

```java
<T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException;
<T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException;
URI reconstructURI(ServiceInstance instance, URI original);
```



> LoadBalancerClient 是由`spring-cloud-netflix`提供的

​	`ServiceinstanceChooser`接口中只有一个方法用于根据serviceId获取ServiceInstance，就是通过服务名来选择服务实例，代码如下：

```java
public interface ServiceInstanceChooser {
    ServiceInstance choose(String serviceId);
}
```

​		LoadBalancerClient的实现类为 RibbonLoadBalancerClient，RibbonLoadBalancerClient是一个非常重要的类。最终的负载均衡的请求由它来执行。下面是部分RibbonLoadBalancerClient的源码:

```java
@Override
public ServiceInstance choose(String serviceId) {
    Server server = getServer(serviceId);
    if (server == null) {
        return null;
    }
    return new RibbonServer(serviceId, server, isSecure(server, serviceId),
                            serverIntrospector(serviceId).getMetadata(server));
}

protected Server getServer(String serviceId) {
		return getServer(getLoadBalancer(serviceId));
	}

	protected Server getServer(ILoadBalancer loadBalancer) {
		if (loadBalancer == null) {
			return null;
		}
		return loadBalancer.chooseServer("default"); // TODO: better handling of key
	}

	protected ILoadBalancer getLoadBalancer(String serviceId) {
		return this.clientFactory.getLoadBalancer(serviceId);
	}
```

​		从上面代码中可以看出，choose方法是用来选择具体的服务实例，该方法通过getServer()方法获取去获取实例，从上面的代码可以看出来，最终是交由**`ILoadBalancer`**类去选择实例。

`ILoadBalancer`归属于`ribbon-loadbalancer`的jar下面，并且是一个接口，接口具体定义的信息如下：

```java
public interface ILoadBalancer {

	/**
	 *  服务器的初始列表。可以添加一个服务列表
	 *  此API还可用于添加其他同一逻辑服务器（主机：端口）
	 *
	 * @param newServers new servers to add
	 */
	public void addServers(List<Server> newServers);

	/**
	 * 用于根据key去获取Server
	 * 从负载均衡器中选择服务器
	 */
	public Server chooseServer(Object key);

	/**
	 * 标记服务器关闭
	 */
	public void markServerDown(Server server);

	/**
	 * 获取当前的服务器列表
	 */
	@Deprecated
	public List<Server> getServerList(boolean availableOnly);

	/**
	 * 获取可用的Server集合
	 * @return Only the servers that are up and reachable.
     */
    public List<Server> getReachableServers();

    /**
     * 获取所有的服务器集合
     */
	public List<Server> getAllServers();
}
```

​	`ILoadBalancer`的子类为：`BaseLoadBalancer`，`BaseLoadBalancer`的默认实现类是`DynamicServerListLoadBalancer`,类之间关系图如下：

![ILoadBalancer继承类图](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=1264848754,2417907410&fm=173&app=25&f=JPEG?w=600&h=334&s=2BD282455FA0B76854500C0B0000E0C3)

#### 4.5.2 DynamicServerListLoadBalancer

​		查看`DynamicServerListLoadBalancer`类的源码，`DynamicServerListLoadBalancer`需要配置IClientConfig、IRule、IPing、ServerList和ServerListFilter参数，查看父级BaseLoadBalancer的构造方法，还需要配置ILoadBalancer。查看BaseLoadBalancer
源码，在默认的情况下，实现如下配置。

* IClientConfig ribbonClientConfig: DefaultClientConfiglmpl

* IRule ribbonRule: RoundRobinRule

* IPing ribbonPing: DummyPing

* ServerList ribbonServerList: ConfigurationBasedServerList

* ServerListFilter ribbonServerListFilter: ZonePreferenceServerListFilter

* ILoadBalancer ribbonLoadBalancer: ZoneAwareLoadBalancer

  

##### 4.5.2.1 IClientConfig 

​		IClientConfig用于配置负载均衡的客户端，IClientConfig的默认实现类为DefaultClientConfiglmpl

##### 4.5.2.2 IRule

​		IRule用于配置负载均衡的策略，IRule 有三个方法，其中choose()是根据key来获取erver实例的， setLoadBalancer()和 getLoadBalancer()是用来设置和获取 ILoadBalancer 的，它的源码如下：

``` java
public interface IRule{
   public Server choose(Object key); 
    public void setLoadBalancer(ILoadBalancer lb);
    public ILoadBalancer getLoadBalancer();    
}
```

​		IRule 有很多默认的实现类，这些实现类根据不同的算法和逻辑来处理负载均衡的策略。IRule 默认实现类有7种。在大多数情况下，这些默认的实现类是可以满足需求的，如果有特殊的谛求 可以自己实现。Rule 其实现类之间的关系如下图所示。

![IRule](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=3152572671,101778655&fm=173&app=25&f=JPEG?w=640&h=309&s=23D2036E4FE0A5680AE99C0C000070C2)

* IRule
  这是所有负载均衡策略的父接口，里边的核心方法就是choose方法，用来选择一个服务实例。

* AbstractLoadBalancerRule
  AbstractLoadBalancerRule是一个抽象类，里边主要定义了一个ILoadBalancer，就是我们上文所说的负载均衡器，负载均衡器的功能我们在上文已经说的很详细了，这里就不再赘述，这里定义它的目的主要是辅助负责均衡策略选取合适的服务端实例。

* RandomRule
  看名字就知道，这种负载均衡策略就是随机选择一个服务实例，看源码我们知道，在RandomRule的无参构造方法中初始化了一个Random对象，然后在它重写的choose方法又调用了choose(ILoadBalancer lb, Object key)这个重载的choose方法，在这个重载的choose方法中，每次利用random对象生成一个不大于服务实例总数的随机数，并将该数作为下标所以获取一个服务实例。

* RoundRobinRule
  RoundRobinRule这种负载均衡策略叫做线性负载均衡策略，也就是我们在上文所说的BaseLoadBalancer负载均衡器中默认采用的负载均衡策略。这个类的choose(ILoadBalancer lb, Object key)函数整体逻辑是这样的：开启一个计数器count，在while循环中遍历服务清单，获取清单之前先通过incrementAndGetModulo方法获取一个下标，这个下标是一个不断自增长的数先加1然后和服务清单总数取模之后获取到的（所以这个下标从来不会越界），拿着下标再去服务清单列表中取服务，每次循环计数器都会加1，如果连续10次都没有取到服务，则会报一个警告No available alive servers after 10 tries from load balancer: XXXX。

* RetryRule
  看名字就知道这种负载均衡策略带有重试功能。首先RetryRule中又定义了一个subRule，它的实现类是RoundRobinRule，然后在RetryRule的choose(ILoadBalancer lb, Object key)方法中，每次还是采用RoundRobinRule中的choose规则来选择一个服务实例，如果选到的实例正常就返回，如果选择的服务实例为null或者已经失效，则在失效时间deadline之前不断的进行重试（重试时获取服务的策略还是RoundRobinRule中定义的策略），如果超过了deadline还是没取到则会返回一个null。

* WeightedResponseTimeRule
  WeightedResponseTimeRule是RoundRobinRule的一个子类，在WeightedResponseTimeRule中对RoundRobinRule的功能进行了扩展，WeightedResponseTimeRule中会根据每一个实例的运行情况来给计算出该实例的一个权重，然后在挑选实例的时候则根据权重进行挑选，这样能够实现更优的实例调用。WeightedResponseTimeRule中有一个名叫DynamicServerWeightTask的定时任务，默认情况下每隔30秒会计算一次各个服务实例的权重，权重的计算规则也很简单，如果一个服务的平均响应时间越短则权重越大，那么该服务实例被选中执行任务的概率也就越大。

* ClientConfigEnabledRoundRobinRule
  ClientConfigEnabledRoundRobinRule选择策略的实现很简单，内部定义了RoundRobinRule，choose方法还是采用了RoundRobinRule的choose方法，所以它的选择策略和RoundRobinRule的选择策略一致，不赘述。

* BestAvailableRule
  BestAvailableRule继承自ClientConfigEnabledRoundRobinRule，它在ClientConfigEnabledRoundRobinRule的基础上主要增加了根据loadBalancerStats中保存的服务实例的状态信息来过滤掉失效的服务实例的功能，然后顺便找出并发请求最小的服务实例来使用。然而loadBalancerStats有可能为null，如果loadBalancerStats为null，则BestAvailableRule将采用它的父类即ClientConfigEnabledRoundRobinRule的服务选取策略（线性轮询）。

* PredicateBasedRule
  PredicateBasedRule是ClientConfigEnabledRoundRobinRule的一个子类，它先通过内部定义的一个过滤器过滤出一部分服务实例清单，然后再采用线性轮询的方式从过滤出来的结果中选取一个服务实例。

* ZoneAvoidanceRule

  ZoneAvoidanceRule是PredicateBasedRule的一个实现类，只不过这里多一个过滤条件，ZoneAvoidanceRule中的过滤条件是以ZoneAvoidancePredicate为主过滤条件和以AvailabilityPredicate为次过滤条件组成的一个叫做CompositePredicate的组合过滤条件，过滤成功之后，继续采用线性轮询的方式从过滤结果中选择一个出来。

  

##### 4.5.2.3 IPing 

  IPing 用于向server发送"ping"，来判断server是否有响应，从而判断server是否可用，它有一个isAlive()方法，源代码如下：

```java
public interface IPing {
    public boolean isAlive(Server server);
}
```

IPing类的实现类有PingUrl、PingConstant、NoOpPing、DummyPing和NIWSDiscoveryPing，各个之间的关系如下图：

![IPing类图](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=1644883936,4220581473&fm=173&app=25&f=JPEG?w=640&h=239&s=23D2836FBD83BB605CFC551B000070C1)

* PingUrl：真实地去ping某个URL，判断其是否可用。
* PingConstant：固定返回某服务是否可用，默认返回true，即可用。
* NoOpPing：不去ping，直接返回true，即可用。
* DummyPing：直接返回true，并实现了initWithNiewsConfig()方法。
* NIWSDiscoveryPing：根据DiscoveryEnableServer的InstanceInfo的InstanceStatus去判断，如果为InstanceStatus.UP，则可用，否则不可用。

##### 4.5.2.4 ServerList

​	ServerList是定义获取所有server的注册列表信息的接口，源代码如下：

```java
public interface ServerList<T extends Server> {

    public List<T> getInitialListOfServers();
   
    public List<T> getUpdatedListOfServers();   
}
```

##### 4.5.2.5 ServerListFilter

​		ServerListFilter接口定义了可根据配置去过滤或者特性动态的获取符合条件的server列表的方法，代码如下：

```java
public interface ServerListFilter<T extends Server> {

    public List<T> getFilteredListOfServers(List<T> servers);

}

```

​		阅读DynamicServerListLoadBalancer的源码，DynamicServerListLoadBalancer的构造函数中有一个initWithNiewsConfig()方法。在该方法中经过一系列的初始化配置，最终执行了restOfInit()方法。DynamicServerListLoadBalancer的部分源码如下：

```java
 public DynamicServerListLoadBalancer(IClientConfig clientConfig) {
        initWithNiwsConfig(clientConfig);
}

 @Override
public void initWithNiwsConfig(IClientConfig clientConfig) {
        try {
            // 代码省略
            restOfInit(clientConfig);
        } catch (Exception e) {
             // 代码省略
        }
}
void restOfInit(IClientConfig clientConfig) {
        boolean primeConnection = this.isEnablePrimingConnections();
        // turn this off to avoid duplicated asynchronous priming done in BaseLoadBalancer.setServerList()
        this.setEnablePrimingConnections(false);
        enableAndInitLearnNewServersFeature();
		// 用来获取所有的ServerList
        updateListOfServers();
        if (primeConnection && this.getPrimeConnections() != null) {
            this.getPrimeConnections()
                    .primeConnections(getReachableServers());
        }
        this.setEnablePrimingConnections(primeConnection);
        LOGGER.info("DynamicServerListLoadBalancer for client {} initialized: {}", clientConfig.getClientName(), this.toString());
}
```

​		进一步跟踪updateListOfServers()方法的源码，最终由serverListImpl.getUpdatedListOfServers()获取所有的服务列表，代码如下：

```java
 public void updateListOfServers() {
        List<T> servers = new ArrayList<T>();
        if (serverListImpl != null) {
            servers = serverListImpl.getUpdatedListOfServers();
            LOGGER.debug("List of Servers for {} obtained from Discovery client: {}",
                    getIdentifier(), servers);

            if (filter != null) {
                servers = filter.getFilteredListOfServers(servers);
                LOGGER.debug("Filtered List of Servers for {} obtained from Discovery client: {}",
                        getIdentifier(), servers);
            }
        }
        updateAllServerList(servers);
    }
```

​		而serverListImpl是ServerList接口的具体实现类。跟踪源码，ServerList的实现类为DiscoveryEnableNIEWSServerList，这个类在ribbon-eureka.jar的com.netflix.niews.loadbalancer包下。其中，DiscoveryEnableNIEWSServerList有getInitialListOfServers()和getUpdatedListOfServers()的方法，具体代码如下:

```java
@Override
public List<DiscoveryEnabledServer> getInitialListOfServers(){
    return obtainServersViaDiscovery();
}

@Override
public List<DiscoveryEnabledServer> getUpdatedListOfServers(){
    return obtainServersViaDiscovery();
}
```

​		继续跟踪源码，obtainServersViaDiscovery()方法是根据eurekaClientProvider.get()方法来获取EurekaClient的，在根据EurekaClient来获取服务注册表列信息，代码如下：

```java
 private List<DiscoveryEnabledServer> obtainServersViaDiscovery() {
        List<DiscoveryEnabledServer> serverList = new ArrayList<DiscoveryEnabledServer>();
        // 获取客户端
        if (eurekaClientProvider == null || eurekaClientProvider.get() == null) {
            logger.warn("EurekaClient has not been initialized yet, returning an empty list");
            return new ArrayList<DiscoveryEnabledServer>();
        }
        EurekaClient eurekaClient = eurekaClientProvider.get();
        if (vipAddresses!=null){
            for (String vipAddress : vipAddresses.split(",")) {
                // if targetRegion is null, it will be interpreted as the same region of client
                // 如果targetRegion为null，它认为客户端在同一区域
                List<InstanceInfo> listOfInstanceInfo = eurekaClient.getInstancesByVipAddress(vipAddress, isSecure, targetRegion);
                // 遍历
                for (InstanceInfo ii : listOfInstanceInfo) {
                    if (ii.getStatus().equals(InstanceStatus.UP)) {

                        if(shouldUseOverridePort){
                            if(logger.isDebugEnabled()){
                                logger.debug("Overriding port on client name: " + clientName + " to " + overridePort);
                            }

                            // copy is necessary since the InstanceInfo builder just uses the original reference,
                            // and we don't want to corrupt the global eureka copy of the object which may be
                            // used by other clients in our system
                            // 客户端信息复制
                            InstanceInfo copy = new InstanceInfo(ii);

                            if(isSecure){
                                ii = new InstanceInfo.Builder(copy).setSecurePort(overridePort).build();
                            }else{
                                ii = new InstanceInfo.Builder(copy).setPort(overridePort).build();
                            }
                        }

                        DiscoveryEnabledServer des = new DiscoveryEnabledServer(ii, isSecure, shouldUseIpAddr);
                        des.setZone(DiscoveryClient.getZone(ii));
                        serverList.add(des);
                    }
                }
                if (serverList.size()>0 && prioritizeVipAddressBasedServers){
                    // 如果当前的vipAddress有服务器，我们不使用后续的基于vipAddress的服务器
                    break; // if the current vipAddress has servers, we dont use subsequent vipAddress based servers
                }
            }
        }
        return serverList;
    }
```

​			其中，eurekaClientProvider的实现类是LegacyEurekaClientProvider，LegacyEurekaClientProvider是一个获取eurekaClient实例的类，其中代码如下：

```java
class LegacyEurekaClientProvider implements Provider<EurekaClient> {

    private volatile EurekaClient eurekaClient;

    @Override
    public synchronized EurekaClient get() {
        if (eurekaClient == null) {
            eurekaClient = DiscoveryManager.getInstance().getDiscoveryClient();
        }

        return eurekaClient;
    }
}
```

​		EurekaClient的实现类为DiscoveryClient，DiscoveryClient具有服务注册、获取服务注册列表等功能。由此可见，负载均衡是从Eureka Client获取服务列表信息的，并根据IRule的策略去路由，根据IPing去判断服务的可用性。

##### 	4.5.2.6 何时获取服务列表

​		上面的操作遗留一个问题，负载均衡器没个多长时间从Eureka Client获取注册信息呢？在BaseLoadBalancer类的源码中，在BaseLoadBalancer的构造方法开启了一个PingTask任务，代码如下：

```java
public BaseLoadBalancer() {
    this.name = DEFAULT_NAME;
    this.ping = null;
    setRule(DEFAULT_RULE);
    //启动ping命令
    setupPingTask();
    lbStats = new LoadBalancerStats(DEFAULT_NAME);
}
```

​		在setupPingTask()的具体代码逻辑里，开启了ShutdownEnabledTimer的PingTask任务，在默认情况下，变量pingIntervalSeconds的值为10，即每10秒向Eureka Client发送一次心跳“ping”。

```java
void setupPingTask() {
    if (canSkipPing()) {
        return;
    }
    if (lbTimer != null) {
        lbTimer.cancel();
    }
    lbTimer = new ShutdownEnabledTimer("NFLoadBalancer-PingTimer-" + name,
                                       true);
    // 每10s ping 一次
    lbTimer.schedule(new PingTask(), 0, pingIntervalSeconds * 1000);
    forceQuickPing();
}
```

​	PingTask源码：PingTask创建了一个Pinger对象，并执行了runPinger()方法。

```java
class PingTask extends TimerTask {
    public void run() {
        try {
            new Pinger(pingStrategy).runPinger();
        } catch (Exception e) {
            logger.error("LoadBalancer [{}]: Error pinging", name, e);
        }
    }
}
```

Pinger源码：

```java
public void runPinger() throws Exception {
            // 判断当前是否有运行的
            if (!pingInProgress.compareAndSet(false, true)) {
                return; // Ping in progress - nothing to do
            }
            Server[] allServers = null;
            boolean[] results = null;
            Lock allLock = null;
            Lock upLock = null;
            try {
                /*
                 * 除非正在进行addServer操作，否则readLock应该是空闲的
                 */
                allLock = allServerLock.readLock();
                allLock.lock();
                allServers = allServerList.toArray(new Server[allServerList.size()]);
                allLock.unlock();

                int numCandidates = allServers.length;
                // 获取服务的可用性
                results = pingerStrategy.pingServers(ping, allServers);
				// 设置一致与不一致的list，用于拉取信息
                final List<Server> newUpList = new ArrayList<Server>();
                final List<Server> changedServers = new ArrayList<Server>();
                // 遍历当前存储所有的客户端，检查isAlive状态，分别放入新运行的、状态改变的两个List中
                for (int i = 0; i < numCandidates; i++) {
                    boolean isAlive = results[i];
                    Server svr = allServers[i];
                    boolean oldIsAlive = svr.isAlive();

                    svr.setAlive(isAlive);
                    // 判断IsAlive
                    if (oldIsAlive != isAlive) {
                        changedServers.add(svr);
                        logger.debug("LoadBalancer [{}]:  Server [{}] status changed to {}",
                    		name, svr.getId(), (isAlive ? "ALIVE" : "DEAD"));
                    }

                    if (isAlive) {
                        newUpList.add(svr);
                    }
                }
                upLock = upServerLock.writeLock();
                upLock.lock();
                upServerList = newUpList;
                upLock.unlock();
                // isAlive不同的，则通知ServerStatusChangeListener服务注册
                // 列表信息发生了改变，进行更新或者重新拉取
                notifyServerStatusChangeListener(changedServers);
            } finally {
                pingInProgress.set(false);
            }
        }
    }
```

​		查看Pinger的runPinger()方法，最终根据pingerStrategy.pingServers(ping, allServers)来获取服务的可用性，如果该返回结果与之前相同，则不向Eureka Client获取注册列表；如果不同，则通知ServerStatusChangeListener服务注册列表信息发生了改变，进行更新或者重新拉取。

​		由此可见，LoadBalancerClient是在初始化时向Eureka获取服务注册列表信息，并且每10秒向EurekaClient发送“ping”，来判断服务的可用性。如果服务的可用性发生了改变或者服务的数量和之前不一致，则更新或者重新拉取。LoadBalancerClient有了这些服务注册列表信息，就可以根据具体IRule的策略来进行负载均衡。

#### 4.5.3 RestTemplate如何实现负载均衡

​	上面的代码只是解释了Ribbon是如何进行负载均衡的，那么RestTemplate是怎么实现负载均衡的？相对来说，只是添加了一个`@LoadBalanced`的注解。

##### 4.5.3.1 @LoadBalanced 源码

```java
@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface LoadBalanced {
}
```

##### 4.5.3.2 @LoadBalanced 的使用者

​		在`@LoadBalanced`源码处点击，即可查看到使用者，发现`LoadBalancerAutoConfiguration`类（`LoadBalancer`的自动配置类）使用到了该注解，`LoadBalancerAutoConfiguration`类的代码如下：

```java
@Configuration
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
@EnableConfigurationProperties(LoadBalancerRetryProperties.class)
public class LoadBalancerAutoConfiguration {
	// 维护了一个被@LoadBalanced修饰的RestTemplate对象的List
	@LoadBalanced
	@Autowired(required = false)
	private List<RestTemplate> restTemplates = Collections.emptyList();
	
    // 其他Bean初始化完成后，获取所有RestTemplateCustomizer的Bean
	@Bean
	public SmartInitializingSingleton loadBalancedRestTemplateInitializer(
			final List<RestTemplateCustomizer> customizers) {
		return new SmartInitializingSingleton() {
            // 单例初始化完成后
			@Override
			public void afterSingletonsInstantiated() {
                // 处理RestTemplate
				for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
					for (RestTemplateCustomizer customizer : customizers) {
                        // 给RestTemplate增加拦截器LoadBalancerInterceptor
                        // 代码在61行
						customizer.customize(restTemplate);
					}
				}
			}
		};
	}

	@Autowired(required = false)
	private List<LoadBalancerRequestTransformer> transformers = Collections.emptyList();

	@Bean
	@ConditionalOnMissingBean
	public LoadBalancerRequestFactory loadBalancerRequestFactory(
			LoadBalancerClient loadBalancerClient) {
		return new LoadBalancerRequestFactory(loadBalancerClient, transformers);
	}
    // 不使用RetryTemplate 执行如下配置
	@Configuration
	@ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
	static class LoadBalancerInterceptorConfig {
		@Bean
		public LoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient,
				LoadBalancerRequestFactory requestFactory) {
			return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
		}
		// RestTemplateCustomizer 注册的时候需要获取LoadBalancerInterceptor，拦截器
        // 将拦截器通过RestTemplateCustomizer.customize()方法设置给RestTemplate
		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final LoadBalancerInterceptor loadBalancerInterceptor) {
			return new RestTemplateCustomizer() {
				@Override
				public void customize(RestTemplate restTemplate) {
					List<ClientHttpRequestInterceptor> list = new ArrayList<>(
							restTemplate.getInterceptors());
					list.add(loadBalancerInterceptor);
                    // 为RestTemplate添加LoadBalancerInterceptor
					restTemplate.setInterceptors(list);
				}
			};
		}
	}
	// 使用RetryTemplate 执行如下配置
	@Configuration
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryAutoConfiguration {
		@Bean
		public RetryTemplate retryTemplate() {
			RetryTemplate template =  new RetryTemplate();
			template.setThrowLastExceptionOnExhausted(true);
			return template;
		}

		@Bean
		@ConditionalOnMissingBean
		public LoadBalancedRetryPolicyFactory loadBalancedRetryPolicyFactory() {
			return new LoadBalancedRetryPolicyFactory.NeverRetryFactory();
		}
	}

	@Configuration
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryInterceptorAutoConfiguration {
		@Bean
		@ConditionalOnMissingBean
		public RetryLoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient, LoadBalancerRetryProperties properties,
				LoadBalancedRetryPolicyFactory lbRetryPolicyFactory,
				LoadBalancerRequestFactory requestFactory) {
			return new RetryLoadBalancerInterceptor(loadBalancerClient, properties,
					lbRetryPolicyFactory, requestFactory);
		}

		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final RetryLoadBalancerInterceptor loadBalancerInterceptor) {
			return new RestTemplateCustomizer() {
				@Override
				public void customize(RestTemplate restTemplate) {
					List<ClientHttpRequestInterceptor> list = new ArrayList<>(
							restTemplate.getInterceptors());
					list.add(loadBalancerInterceptor);
					restTemplate.setInterceptors(list);
				}
			};
		}
	}
}
```

​		从上面的代码可以看出，使用的是LoadBalancerInterceptor拦截器，进入这个拦截器，查看一下方法

```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {

	private LoadBalancerClient loadBalancer;
	private LoadBalancerRequestFactory requestFactory;

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer, LoadBalancerRequestFactory requestFactory) {
		this.loadBalancer = loadBalancer;
		this.requestFactory = requestFactory;
	}

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
		// for backwards compatibility
		this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
	}

	@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
        // 负载均衡
		return this.loadBalancer.execute(serviceName, requestFactory.createRequest(request, body, execution));
	}
}
```

​		RestTemplate的负载均衡是在LoadBalancerInterceptor中完成的，通过`loadBalancer.execute()`方法。

### 4.6总结

​		综上所述 Ribbon的负载均衡主要是通过`LoadBalancerClient`来实现的，而 `LoadBalancerClient`具体交给了`ILoadBalancer`来处理，默认使用`DynamicServerListLoadBalancer`，`ILoadBalancer`通过配置`IRule`和`IPing` ，向`EurekaClient`获取注册列表的信息，默认10秒向EurekaClient发送一次“ping ”, 进而检查是否需要更新服务的注册列表信息。最后 ，在得到服务注册列表信息后，`ILoadBalancer`根据`IRule`的策略进行负载均衡。
​		而`RestTemplate` 加上`＠LoadBalance`注解后，在远程调度时能够负载均衡，主要是维护了一个被`＠LoadBalance` 注解的RestTemplate的列表，并给该列表中的RestTemplate对象添加了拦截器。在拦截器的方法中 ，将远程调度方法交给了`Ribbon`负载均衡器`LoadBalancerClient`去处理，从而达到了负载均衡的目的。



## 五、Feign客户端

### 5.1 Feign使用

#### 5.1.1 pom.xml

​	在eureka-client的基础上增加feign客户端信息

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.6</version>
        </dependency>
    </dependencies>
```

#### 5.1.2 FeignClientConfig配置

​	FeignClient访问配置

```java
@Configuration
public class FeignClientConfig {
    /**
     * Feign 在远程调用失败后会进行重试。
     * @return Retryer
     */
    @Bean
    public Retryer feignRetryer(){
        return new Retryer.Default(100,SECONDS.toMillis(1),5);
    }
}
```

#### 5.1.3 EurekaFeignClient配置

​	FeignClient访问方法

```java
@FeignClient(value = Constants.DEFAULT_EUREKA_CLIENT_NAME,
             configuration = FeignClientConfig.class)
public interface EurekaFeignClient {
    /**
     * feign请求
     * @param username 用户名称
     * @return
     */
    @GetMapping(value = "/hi/{username}")
    String sayHiFromFeignEurekaClient(@PathVariable(value="username") String username);
}
```

### 5.2@FeignClient源码

```java
@Target(ElementType.TYPE) // 表明注解的作用目标在接口上
@Retention(RetentionPolicy.RUNTIME)//注解会在class字节码文件中存在，在运行时可以通过反射获取到
@Documented //说明该注解将被包含在javadoc中
// 用于创建声明式API接口，该接口是RESTful风格的
// Feign 被设计成插拔式的，可以注入其他组件和Feign一起使用
// 最典型的是如果Ribbon可用,Feign会和Ribbon结合进行负载均衡。
public @interface FeignClient {
	/**
	 * 具有可选协议前缀的服务的名称
	 * 是被调用的服务的 Serviceld
	 */
	@AliasFor("name")
	String value() default "";
	/**
	 * 带有可选协议前缀的服务标识。 {@link #value（）value}的同义词。
	 * @deprecated use {@link #name() name} instead
	 */
	@Deprecated
	String serviceId() default "";
	/**
	 * 带有可选协议前缀的服务标识. {@link #value() value}的同义词。
	 * 是被调用的服务的 Serviceld
	 */
	@AliasFor("value")
	String name() default "";
	/**
	 * 设置feign client的<code>@Qualifier</code>值。
	 */
	String qualifier() default "";
	/**
	 * 绝对URL或可解析的主机名（协议是可选的）
	 */
	String url() default "";
	/**
	 * 是否应该解码404s而不是抛出FeignExceptions
	 * 404 是被解码，还是抛异常。
	 */
	boolean decode404() default false;
	/**
	 * 指明 FeignClient 配置类，默认的配置类为FeignClientsConfiguration
	 * 这个类注入默认的Decoder、Encoder和Constract等配置的Bean
	 * feign client的自定义<code>@Configuration</code>。
	 * 可以包含构成客户端的部分的override <code>@Bean</code>定义，
	 * 例如 
	 * {@link feign.codec.Decoder}，{@link feign.codec.Encoder}，
	 * {@link feign.Contract}。  
	 * @see FeignClientsConfiguration 默认值
	 *
	 * @see FeignClientsConfiguration for the defaults
	 */
	Class<?>[] configuration() default {};
	/**
	 *  配置熔断器的处理类
	 *  指定Feign客户端界面的后备类。回退类必须实现由此批注注释的接口，并且是有效的spring bean。
	 */
	Class<?> fallback() default void.class;
	/**
	 *  为指定的Feign客户端界面定义回退工厂。 
	 * fallback工厂必须生成实现由{@link FeignClient}注释的接口的回退类的实例。
	 * 后备工厂必须是有效的spring bean。
	 *
	 * @see feign.hystrix.FallbackFactory for details.
	 */
	Class<?> fallbackFactory() default void.class;
	/**
	 * 所有方法级映射都使用的路径前缀。可以与<code>@RibbonClient</code>一起使用。
	 */
	String path() default "";
	/**
	 * 是否将feign proxy 标记为主bean。默认为true。
	 */
	boolean primary() default true;
}
```

### 5.3 FeignClient配置

#### 5.3.1 [FeignClient配置类源码](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.3.6.RELEASE/single/spring-cloud-netflix.html#spring-cloud-feign-overriding-defaults)

> A central concept in Spring Cloud’s Feign support is that of the named client.
> Spring Cloud的Feign支持的核心概念是指定客户端的概念。
> Each feign client is part of an ensemble of components that work together to contact a remote server on demand,
> 每个客户端都是组件集合的一部分，它们一起工作并且按需联系远程服务器，
> and the ensemble has a name that you give it as an application developer using the `@FeignClient` annotation.
> 这样的集合都有一个名称，应用程序开发人员可以使用`@FeignClient`注解来提供名称。
> Spring Cloud creates a new ensemble as an `ApplicationContext` on demand for each named client using `FeignClientsConfiguration`.
> Spring Cloud 根据ApplicationContext需要为每个命名客户端创建一个基于`FeignClientsConfiguration`创建新的集合
> This contains (amongst other things) an `feign.Decoder`, a `feign.Encoder`, and a `feign.Contract`.
> 这些包含`feign.Decoder`、 `feign.Encoder`和 `feign.Contract`

​	`FeignClient`默认的配置类是`FeignClientsConfiguration`,这个类在`spring-cloud-netflix.core`的jar包下。打开这个类，可以发现这个类注入了很多`Feign`相关的配置`Bean`,包括`FeignRetryer`、`FeignLoggerFactory`和`FormattingConversionService`等。Spring Cloud Netiflix默认为如下的bean提供默认的注入Bean类型:

* `Decoder`feignDecoder :( `ResponseEntityDecoder`包装一个`SpringDecoder`）
* `Encoder` feignEncoder： `SpringEncoder`
* `Logger` feignLogger： `Slf4jLogger`
* `Contract` feignContract： `SpringMvcContract`
* `Feign.Builder` feignBuilder： `HystrixFeign.Builder`
* `Client`feignClient：如果启用了Ribbon，则为`LoadBalancerFeignClient`，否则使用默认的feignClient。

`FeignClientsConfiguration`的配置类部分代码如下，`@ConditionalOnMissingBean`注解表示，如果没有注入该类的 Bean 会默认注入一个 Bean

```java
@Bean
@ConditionalOnMissingBean
public Decoder feignDecoder() {
	return new ResponseEntityDecoder(new SpringDecoder(this.messageConverters));
}

@Bean
@ConditionalOnMissingBean
public Encoder feignEncoder() {
	return new SpringEncoder(this.messageConverters);
}

@Bean
@ConditionalOnMissingBean
public Contract feignContract(ConversionService feignConversionService) {
	return new SpringMvcContract(this.parameterProcessors, feignConversionService);
}
```

#### 5.3.2 [重写配置](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.3.6.RELEASE/single/spring-cloud-netflix.html#spring-cloud-feign-overriding-defaults)

​		重写`FeignClientsConfiguration`类中的Bean覆盖掉默认的配置Bean ,从而达到自定义配置的目的。例如Feign默认的配置在请求失败后，重试次数为`Retryer.NEVER_RETRY`，即不重试。现在希望在请求失败后能够重试，这时需要写个配置FeignConfig类，在该类中注`Retryer`的Bean覆盖掉默认的`Retryer`的Bean，并将FeignConfig类定为FeignClient配置类FeignConfig类。

```java
@Configuration
public class FeignClientConfig {
    /**
     * Feign 在远程调用失败后会进行重试。
     * @return Retryer
     */
    @Bean
    public Retryer feignRetryer(){
        return new Retryer.Default(100,SECONDS.toMillis(1),5);
    }
}
```

​	通过覆盖了默认的`Retryer`的Bean更改了该`FeignClient`请求失败重试的策略，重试问隔为 100 毫秒，最大重试时间为1秒，重试次数为5次。

### 5.4 Feign源码

​		`Feign`是一个伪Java Http客户端，`Feign`不做任何的请求处理。`Feign`通过处理注解生成`Request`，从而实现简化`HTTP API`开发的目的，即开发人员可以使用注解的方式定制`Request API`模板，在发送`Http Request`请求之前，`feign`通过处理注解的方式替换掉`Request `模板中的参数，这种实现方式显得更为直接、可理解。

`@EnableFeignClients`源码

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({FeignClientsRegistrar.class})
public @interface EnableFeignClients {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<?>[] defaultConfiguration() default {};

    Class<?>[] clients() default {};
}
```

​		从`@EnableFeignClients`源码中可以看出，使用这个注解的时候，会将`FeignClientsRegistrar`这个类注入到IOC容器中，查看`FeignClientsRegistrar`的源码，发现这个类是用来动态加载Bean的(`ImportBeanDefinitionRegistrar`)，注册方法是`registerBeanDefinitions`，在这个方法中就会有FeignClient默认配置加载、FeignClient的扫描与注册到IOC。

```java
	@Override
	public void registerBeanDefinitions(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
        // 默认配置FeignClientsConfiguration
		registerDefaultConfiguration(metadata, registry);
        // 包扫描
		registerFeignClients(metadata, registry);
	}
```

​	程序启动时，首先在启动配置上检查是否有`@EnableFeignClients`注解，如果有该注解，则开启包扫描，扫描被`@FeignClient`注解接口。代码如下：

#### 5.4.1  默认配置FeignClientsConfiguration处理

```java
private void registerDefaultConfiguration(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
	// 获取使用EnableFeignClients注解的类		
     Map<String, Object> defaultAttrs = metadata
            .getAnnotationAttributes(EnableFeignClients.class.getName(), true);
	// 如果有该注解 
	if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
            String name;
            if (metadata.hasEnclosingClass()) {
                name = "default." + metadata.getEnclosingClassName();
            }else {
                name = "default." + metadata.getClassName();
            }
            //  注册FeignClient默认的FeignConfig
            registerClientConfiguration(registry, name,defaultAttrs.get("defaultConfiguration"));
		}
}
```

#### 5.4.2 包扫描

​		程序启动后通过包扫描,当类有@FeignClient注解，将注解的信息取出，连同类名一起取出，赋给BeanDefinitionBuilder，然后根据BeanDefinitionBuilder得到beanDefinition，最后beanDefinition式注入到ioc容器中，下面是获取需要扫描那些包的代码：

```java
public void registerFeignClients(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
        // 获取Scanner
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
        // 设置Loader
		scanner.setResourceLoader(this.resourceLoader);

		Set<String> basePackages;
        // 获取使用EnableFeignClients注解的类	
		Map<String, Object> attrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName());
        // 指定注解过滤器 过滤使用@FeignClient的类
		AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
				FeignClient.class);
		final Class<?>[] clients = attrs == null ? null
				: (Class<?>[]) attrs.get("clients");
        // 无客户端的时候
		if (clients == null || clients.length == 0) {
            // 设置过滤器
			scanner.addIncludeFilter(annotationTypeFilter);
            // 指定基础包 getBasePackages处理数据
			basePackages = getBasePackages(metadata);
		}
		else { // 有客户端的时候
			final Set<String> clientClasses = new HashSet<>();
			basePackages = new HashSet<>();
			for (Class<?> clazz : clients) {
				basePackages.add(ClassUtils.getPackageName(clazz));
				clientClasses.add(clazz.getCanonicalName());
			}
			AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
				@Override
				protected boolean match(ClassMetadata metadata) {
					String cleaned = metadata.getClassName().replaceAll("\\$", ".");
					return clientClasses.contains(cleaned);
				}
			};
			scanner.addIncludeFilter(
					new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
		}
		// 遍历基础包
		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner
					.findCandidateComponents(basePackage);
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					// verify annotated class is an interface
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(annotationMetadata.isInterface(),
							"@FeignClient can only be specified on an interface");

					Map<String, Object> attributes = annotationMetadata
							.getAnnotationAttributes(
									FeignClient.class.getCanonicalName());

					String name = getClientName(attributes);
                    // 注册FeignClient对应的FeignConfig到IOC容器中
					registerClientConfiguration(registry, name,
							attributes.get("configuration"));
					// 注册FeignClient到IOC容器中
					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
	}
```

#### 5.4.2 Feign Client 对应的FeignConfig注册

registerClientConfiguration方法  注册FeignClient对应的FeignConfig，注册到IOC容器中

```java
private void registerClientConfiguration(BeanDefinitionRegistry registry, Object name,
			Object configuration) {
		BeanDefinitionBuilder builder = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientSpecification.class);
		builder.addConstructorArgValue(name);
		builder.addConstructorArgValue(configuration);
        // 注册配置文件
		registry.registerBeanDefinition(
				name + "." + FeignClientSpecification.class.getSimpleName(),
				builder.getBeanDefinition());
	}
```

#### 5.4.3 FeignClient注册

registerFeignClient方法，将取出的BeanDefinitionRegistry赋给BeanDefinitionBuilder，然后根据BeanDefinitionBuilder得到beanDefinition，最后beanDefinition式注入到ioc容器中

```java
private void registerFeignClient(BeanDefinitionRegistry registry,
			AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
		String className = annotationMetadata.getClassName();
		BeanDefinitionBuilder definition = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientFactoryBean.class);
		validate(attributes);
		definition.addPropertyValue("url", getUrl(attributes));
		definition.addPropertyValue("path", getPath(attributes));
		String name = getName(attributes);
		definition.addPropertyValue("name", name);
		definition.addPropertyValue("type", className);
		definition.addPropertyValue("decode404", attributes.get("decode404"));
		definition.addPropertyValue("fallback", attributes.get("fallback"));
		definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

		String alias = name + "FeignClient";
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

		boolean primary = (Boolean)attributes.get("primary"); // has a default, won't be null

		beanDefinition.setPrimary(primary);

		String qualifier = getQualifier(attributes);
		if (StringUtils.hasText(qualifier)) {
			alias = qualifier;
		}

		BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,new String[] { alias });
        //beanDefinition式注入到ioc容器中
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}
```

#### 5.4.4 Feign Client使用

在Feign Client使用的过程存在以下两个线：

* Client(接口) - Feign(抽象类) - ReflectiveFeign(实现类)。 
* InvocationHandlerFactory(接口) - SynchronousMethodHandler(实现类) 

​		注入bean之后，通过jdk的代理，当请求Feign Client的方法时会被拦截，代码在`ReflectiveFeign`类，代码如下：

```java
public <T> T newInstance(Target<T> target) {
    Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
    Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
    List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();
 
    for (Method method : target.type().getMethods()) {
      if (method.getDeclaringClass() == Object.class) {
        continue;
      } else if(Util.isDefault(method)) {
        DefaultMethodHandler handler = new DefaultMethodHandler(method);
        defaultMethodHandlers.add(handler);
        methodToHandler.put(method, handler);
      } else {
        methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
      }
    }
    InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class<?>[]{target.type()}, handler);
 
    for(DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
      defaultMethodHandler.bindTo(proxy);
    }
    return proxy;
  }
```

#### 5.4.5 生成RequestTemplate

​		在SynchronousMethodHandler类进行拦截处理，当被FeignClient的方法被拦截会根据参数生成RequestTemplate对象，该对象就是http请求的模板，代码如下：

```java
@Override
  public Object invoke(Object[] argv) throws Throwable {
    RequestTemplate template = buildTemplateFromArgs.create(argv);
    Retryer retryer = this.retryer.clone();
    while (true) {
      try {
        return executeAndDecode(template);
      } catch (RetryableException e) {
        retryer.continueOrPropagate(e);
        if (logLevel != Logger.Level.NONE) {
          logger.logRetry(metadata.configKey(), logLevel);
        }
        continue;
      }
    }
  }
```

#### 5.4.5 使用client请求获取响应

​		其中有个executeAndDecode()方法，该方法是通RequestTemplate生成Request请求对象，然后根据用HTTP client获取response。即通过Http Client通过Http请求来获取响应

```java
 Object executeAndDecode(RequestTemplate template) throws Throwable {
    Request request = targetRequest(template);

    if (logLevel != Logger.Level.NONE) {
      logger.logRequest(metadata.configKey(), logLevel, request);
    }

    Response response;
    long start = System.nanoTime();
    try {
      // 获取响应
      response = client.execute(request, options);
      // ensure the request is set. TODO: remove in Feign 10
      response.toBuilder().request(request).build();
    } catch (IOException e) {
      if (logLevel != Logger.Level.NONE) {
        logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime(start));
      }
      throw errorExecuting(request, e);
    }
     //... 省略代码
  }
```

### 5.5 Feign 使用HttpClient和OkHttp

​		从上面的代码中可以看出，`Client`是最终发送`Request`和接收`Response`的，在Feign中`Client`是一个接口，它有一个默认实现：`Client.Default`。`Client.Default`是通过`HttpURLConnection`来实现网络请求的，Feign的`Client`接口还支持`HttpClient`和`OkHttp`来实现网络请求。















总到来说，Feign的源码实现的过程如下：

- 首先通过@EnableFeignCleints注解开启FeignCleint

- 根据Feign的规则实现接口，并加@FeignCleint注解

- 程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。

- 当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate

  - Client(接口) - Feign(抽象类) - ReflectiveFeign(实现类)。 
  - InvocationHandlerFactory(接口) - SynchronousMethodHandler(实现类) 

- RequesTemplate在生成Request

- Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp

- 最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

  