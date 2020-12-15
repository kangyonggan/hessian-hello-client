## Hessian是什么
[Hessian](http://hessian.caucho.com/)是一个轻量的基于二进制协议的RPC框架。

## 环境及目的
本篇文章主要是想尝试在SpringBoot中使用Hessian进行最简单的通信。

- SpringBoot 2.0.9
- Hessian 4.0.60

## 服务端
服务端提供一个`IUserService`给客户端使用。

### 依赖
`pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.60</version>
</dependency>
```

### 配置
```java
@Autowired
private IUserService userService;

@Bean("/userService")
public HessianServiceExporter userService() {
    HessianServiceExporter exporter = new HessianServiceExporter();
    exporter.setService(userService);
    exporter.setServiceInterface(IUserService.class);
    return exporter;
}
```

其中`IUserService`只是一个普通服务，详情移步：[https://github.com/kangyonggan/hessian-hello.git](https://github.com/kangyonggan/hessian-hello.git)。

启动服务端项目已备使用：`java -jar hessian-hello-1.0-SNAPSHOT.jar`

## 客户端
依赖服务放提供的jar，并且调用服务方接口。

### 依赖
`pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- 服务方接口，会间接依赖spring-boot-web和hessian -->
<dependency>
    <groupId>com.kangyonggan</groupId>
    <artifactId>hessian-hello</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### 配置
```java
@Bean
public HessianProxyFactoryBean userService() {
    HessianProxyFactoryBean factoryBean = new HessianProxyFactoryBean();
    factoryBean.setServiceUrl("http://localhost:8080/userService");
    factoryBean.setServiceInterface(IUserService.class);
    return factoryBean;
}
```

注意：`url`中的`/userService`一定要与服务提供的路径一致。

### 测试
```java
package com.kangyonggan.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @author kyg
 */
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserServiceTest {

    @Autowired
    private IUserService userService;

    @Test
    public void getUserById() {
        System.out.println(userService.getUserById(9L));
    }

}
```


输出：

```
User{id=9, name='name-9'}
```

客户端源码：[https://github.com/kangyonggan/hessian-hello-client.git](https://github.com/kangyonggan/hessian-hello-client.git)。

## 遗留问题
1. hessian像不像其他rpc框架一样支持集群、负载均衡、熔断、服务降级等？
2. hessian与k8s结合会擦出什么火花？
3. 在分布式微服务中如何优雅的使用hessian做微服务架构？
4. hessian缺点以及性能如何？
5. hessian适用什么样的项目？

