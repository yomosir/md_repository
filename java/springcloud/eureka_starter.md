# Eureka Starter

## Introduce

用于服务治理功能，主要为构建服务注册中心，服务注册与服务发现

## 搭建服务注册中心

- 服务注册中心引入的依赖是：

```java
    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
```

可以选择通过springboot initializer选择此依赖

- 创建完项目需要通过`@EnableEurekaServer`注解启用Eureka Server