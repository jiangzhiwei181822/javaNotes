> tedu.cn

<span style="font-size: 50px">spring cloud 入门手册</span>

# 目录
[TOC]

---
# 十一、ribbon 负载均衡
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![ribbon02](https://note.youdao.com/yws/res/4970/34FD9147DC5A405499F6BAC34E783F0B)

- 修改 sp06-ribbon 项目 

1. 添加 ribbon 起步依赖（可选）
2. RestTemplate 设置 `@LoadBalanced`
3. 访问路径设置为服务id

## 添加 ribbon 起步依赖（可选）
- ==eureka 依赖中已经包含了 ribbon==

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```
## RestTemplate 设置 `@LoadBalanced`
```java
package com.tedu.sp06;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableDiscoveryClient
@SpringBootApplication
public class Sp06RibbonApplication {
	
	@LoadBalanced
	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(Sp06RibbonApplication.class, args);
	}

}
```


## 访问路径设置为服务id
```java
package com.tedu.sp06.consoller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@RestController
public class RibbonController {
	@Autowired
	private RestTemplate rt;
	
	@GetMapping("/item-service/{orderId}")
	public JsonResult<List<Item>> getItems(@PathVariable String orderId) {
		return rt.getForObject("http://item-service/{1}", JsonResult.class, orderId);
	}

	@PostMapping("/item-service/decreaseNumber")
	public JsonResult decreaseNumber(@RequestBody List<Item> items) {
		return rt.postForObject("http://item-service/decreaseNumber", items, JsonResult.class);
	}

	/////////////////////////////////////////
	
	@GetMapping("/user-service/{userId}")
	public JsonResult<User> getUser(@PathVariable Integer userId) {
		return rt.getForObject("http://user-service/{1}", JsonResult.class, userId);
	}

	@GetMapping("/user-service/{userId}/score") 
	public JsonResult addScore(
			@PathVariable Integer userId, Integer score) {
		return rt.getForObject("http://user-service/{1}/score?score={2}", JsonResult.class, userId, score);
	}
	
	/////////////////////////////////////////
	
	@GetMapping("/order-service/{orderId}")
	public JsonResult<Order> getOrder(@PathVariable String orderId) {
		return rt.getForObject("http://order-service/{1}", JsonResult.class, orderId);
	}

	@GetMapping("/order-service")
	public JsonResult addOrder() {
		return rt.getForObject("http://order-service/", JsonResult.class);
	}
}

```

## 访问测试
> - 访问测试，ribbon 会把请求分发到 8001 和 8002 两个服务端口上<br />
> > http://localhost:3001/item-service/34

> ![image](https://note.youdao.com/yws/res/5482/1DD675840453422693205A107EE10819)

> ![image](https://note.youdao.com/yws/res/5484/0F8022F4EC124DD5B991B79A365390DF)

## ribbon 重试

### pom.xml 添加 spring-retry 依赖
```xml
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>
```

### application.yml 配置 ribbon 重试
```yml
spring:
  application:
    name: ribbon
    
server:
  port: 3001
  
eureka:
  client:    
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
ribbon:
  MaxAutoRetriesNextServer: 2
  MaxAutoRetries: 1
  OkToRetryOnAllOperations: true
```

- ConnectionTimeout
- ReadTimeout
- OkToRetryOnAllOperations=true<br />
对连接超时、读取超时都进行重试
- MaxAutoRetriesNextServer<br />
更换实例的次数
- MaxAutoRetries<br />
当前实例重试次数，尝试失败会更换下一个实例

### 主程序设置 RestTemplate 的请求工厂的超时属性
```java
package com.tedu.sp06;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@EnableDiscoveryClient
@SpringBootApplication
public class Sp06RibbonApplication {

	@LoadBalanced
	@Bean
	public RestTemplate getRestTemplate() {
		SimpleClientHttpRequestFactory f = new SimpleClientHttpRequestFactory();
		f.setConnectTimeout(1000);
		f.setReadTimeout(1000);
		return new RestTemplate(f);
		
		//RestTemplate 中默认的 Factory 实例中，两个超时属性默认是 -1，
		//未启用超时，也不会触发重试
		//return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(Sp06RibbonApplication.class, args);
	}

}
```

### item-service 的 ItemController 添加延迟代码，以便测试 ribbon 的重试机制
```java
package com.tedu.sp02.item.controller;

import java.util.List;
import java.util.Random;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.service.ItemService;
import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class ItemController {
	@Autowired
	private ItemService itemService;
	
	@Value("${server.port}")
	private int port;
	
	@GetMapping("/{orderId}")
	public JsonResult<List<Item>> getItems(@PathVariable String orderId) throws Exception {
		log.info("server.port="+port+", orderId="+orderId);

        ///--设置随机延迟
		long t = new Random().nextInt(5000);
		if(Math.random()<0.6) { 
			log.info("item-service-"+port+" - 暂停 "+t);
			Thread.sleep(t);
		}
		///~~
		
		List<Item> items = itemService.getItems(orderId);
		return JsonResult.ok(items).msg("port="+port);
	}
	
	@PostMapping("/decreaseNumber")
	public JsonResult decreaseNumber(@RequestBody List<Item> items) {
		itemService.decreaseNumbers(items);
		return JsonResult.ok();
	}
}
```
## 访问，测试 ribbon 重试机制
> - 通过 ribbon 访问 item-service，当超时，ribbon 会重试请求集群中其他服务器
> > http://localhost:3001/item-service/35
> 
> ![image](https://note.youdao.com/yws/res/5037/BB21396F3A47435C9F4CC0A29A800ED9)

- ==ribbon的重试机制，在 feign 和 zuul 中进一步进行了封装，后续可以使用feign或zuul的重试机制==


---
# 十二、ribbon + hystrix 断路器
<div style="text-align: right"> 

[返回目录](#目录) 

</div>


- https://github.com/Netflix/Hystrix/wiki

> ![hystrix1](https://note.youdao.com/yws/res/4969/C5019D2547FC4D5EAE9B7F243CDD4E99)

## 微服务宕机时，ribbon 无法转发请求

> - ==关闭 item-service==
> 
> ![image](https://note.youdao.com/yws/res/5517/D7350EBCD06449489A53B3A549C8988D)

> ![image](https://note.youdao.com/yws/res/5513/272856268B05499C89548D6283BFAE38)


## 复制 sp06-ribbon 项目，命名为sp07-hystrix

- ==选择 sp06-ribbon 项目，ctrl-c，ctrl-v，复制为sp07-hystrix==
- ==关闭 sp06-ribbon 项目，后续测试使用 sp07-hystrix 项目==

> ![image](https://note.youdao.com/yws/res/5053/BBCB748C04BC4712BB6CCEAD5C860B54)

## 修改 pom.xml，并添加 hystrix 起步依赖
> ![image](https://note.youdao.com/yws/res/5043/DA31B9A01638446D9D33C3B2F883C619)

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

## 修改 application.yml
> ![image](https://note.youdao.com/yws/res/5052/228FF00A228447028DEA21E9A81501B0)

```yml

spring:
  application:
    name: hystrix
    
server:
  port: 3001
  
eureka:
  client:    
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
ribbon:
  MaxAutoRetriesNextServer: 2
  MaxAutoRetries: 1
  OkToRetryOnAllOperations: true
```

## 主程序添加 `@EnableCircuitBreaker` 启用 hystrix 断路器

- ==可以使用 `@SpringCloudApplication` 注解代替三个注解==

```java
package com.tedu.sp06;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

//@EnableCircuitBreaker
//@EnableDiscoveryClient
//@SpringBootApplication

@SpringCloudApplication
public class Sp06RibbonApplication {

	@LoadBalanced
	@Bean
	public RestTemplate getRestTemplate() {
		SimpleClientHttpRequestFactory f = new SimpleClientHttpRequestFactory();
		f.setConnectTimeout(1000);
		f.setReadTimeout(1000);
		return new RestTemplate(f);
		
		//RestTemplate 中默认的 Factory 实例中，两个超时属性默认是 -1，
		//未启用超时，也不会触发重试
		//return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(Sp06RibbonApplication.class, args);
	}

}
```


## RibbonController 中添加降级方法

- 为每个方法添加降级方法，例如 `getItems()` 添加降级方法 `getItemsFB()`
- 添加 `@HystrixCommand` 注解，指定降级方法名

```java
package com.tedu.sp06.consoller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@RestController
public class RibbonController {
	@Autowired
	private RestTemplate rt;
	
	@GetMapping("/item-service/{orderId}")
	@HystrixCommand(fallbackMethod = "getItemsFB")
	public JsonResult<List<Item>> getItems(@PathVariable String orderId) {
		return rt.getForObject("http://item-service/{1}", JsonResult.class, orderId);
	}

	@PostMapping("/item-service/decreaseNumber")
	@HystrixCommand(fallbackMethod = "decreaseNumberFB")
	public JsonResult decreaseNumber(@RequestBody List<Item> items) {
		return rt.postForObject("http://item-service/decreaseNumber", items, JsonResult.class);
	}

	/////////////////////////////////////////
	
	@GetMapping("/user-service/{userId}")
	@HystrixCommand(fallbackMethod = "getUserFB")
	public JsonResult<User> getUser(@PathVariable Integer userId) {
		return rt.getForObject("http://user-service/{1}", JsonResult.class, userId);
	}

	@GetMapping("/user-service/{userId}/score") 
	@HystrixCommand(fallbackMethod = "addScoreFB")
	public JsonResult addScore(@PathVariable Integer userId, Integer score) {
		return rt.getForObject("http://user-service/{1}/score?score={2}", JsonResult.class, userId, score);
	}
	
	/////////////////////////////////////////
	
	@GetMapping("/order-service/{orderId}")
	@HystrixCommand(fallbackMethod = "getOrderFB")
	public JsonResult<Order> getOrder(@PathVariable String orderId) {
		return rt.getForObject("http://order-service/{1}", JsonResult.class, orderId);
	}

	@GetMapping("/order-service")
	@HystrixCommand(fallbackMethod = "addOrderFB")
	public JsonResult addOrder() {
		return rt.getForObject("http://order-service/", JsonResult.class);
	}
	
	/////////////////////////////////////////

	public JsonResult<List<Item>> getItemsFB(@PathVariable String orderId) {
		return JsonResult.err("获取订单商品列表失败");
	}
	public JsonResult decreaseNumberFB(@RequestBody List<Item> items) {
		return JsonResult.err("更新商品库存失败");
	}
	public JsonResult<User> getUserFB(@PathVariable Integer userId) {
		return JsonResult.err("获取用户信息失败");
	}
	public JsonResult addScoreFB(@PathVariable Integer userId, Integer score) {
		return JsonResult.err("增加用户积分失败");
	}
	public JsonResult<Order> getOrderFB(@PathVariable String orderId) {
		return JsonResult.err("获取订单失败");
	}
	public JsonResult addOrderFB() {
		return JsonResult.err("添加订单失败");
	}

}

```


## hystrix 短路超时设置

`hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds`

为了测试 hystrix 短路功能，我们把 hystrix 等待超时设置得非常小（500毫秒）

==此设置一般应大于 ribbon 的重试超时时长，例如 10 秒==

```yml

spring:
  application:
    name: hystrix
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
ribbon:
  MaxAutoRetriesNextServer: 2
  MaxAutoRetries: 1
  OkToRetryOnAllOperations: true
  
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 500
            
```



## 启动项目进行测试
> ![image](https://note.youdao.com/yws/res/5035/B66A1B246D2849879FEAA904B6B8D19A)

> - 通过 hystrix 服务，访问可能==超时失败==的 item-service
> > http://localhost:3001/item-service/35
> 
> - 通过 hystrix 服务，访问==未启动==的 user-service
> > http://localhost:3001/user-service/7
>
> - 可以看到，如果 item-service 请求超时，hystrix 会立即执行降级方法
> - 访问 user-service，由于该服务未启动，hystrix也会立即执行降级方法

> ![image](https://note.youdao.com/yws/res/5029/FEB0636AE3F9495D860AC9919FA41B93)

---
# 十三、hystrix dashboard 断路器仪表盘
<div style="text-align: right"> 

[返回目录](#目录) 

</div>


> ![dashboard0](https://note.youdao.com/yws/res/4966/6EBF7408A4A04016ABAFAB948897952C)

hystrix 对请求的熔断和断路处理，可以产生监控信息，hystrix dashboard可以实时的进行监控

## sp07-hystrix 项目添加 actuator，并暴露 hystrix 监控端点
### pom.xml 添加 actuator 依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
### 调整 application.yml 配置，并暴露 hystrix 监控端点

```yml
spring:
  application:
    name: hystrix
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
ribbon:
  MaxAutoRetriesNextServer: 1
  MaxAutoRetries: 1
  OkToRetryOnAllOperations: true
  
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000

management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

### 访问 actuator 路径，查看监控端点
> - http://localhost:3001/actuator
> 
> ![image](https://note.youdao.com/yws/res/5044/26E6E6784CD147608F832EDA2288DB84)

## 新建 sp08-hystrix-dashboard 项目

> ![image](https://note.youdao.com/yws/res/4987/4AEA18FBEB6348549B168E3733899C8D)

> ![image](https://note.youdao.com/yws/res/5001/31DD0157A670412E8BAA9CE8C8B7D973)

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.tedu</groupId>
	<artifactId>sp08-hystrix-dashboard</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp08-hystrix-dashboard</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>


```

### application.yml

```yml
spring:
  application:
    name: hystrix-dashboard
    
server:
  port: 4001

eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```


### 主程序添加 `@EnableHystrixDashboard` 和 `@EnableDiscoveryClient`

```java
package com.tedu.sp08;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@EnableDiscoveryClient
@EnableHystrixDashboard
@SpringBootApplication
public class Sp08HystrixDashboardApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp08HystrixDashboardApplication.class, args);
	}

}

```




## 启动，并访问测试
> ![image](https://note.youdao.com/yws/res/5045/CB1496A826F74591949CBF83B1E7D6EE)


### 访问 hystrix dashboard
> - http://localhost:4001/hystrix

> ![image](https://note.youdao.com/yws/res/4956/C691E027DAD9487D8239B21058666797)

### 填入 hystrix 的监控端点，开启监控

> - http://localhost:3001/actuator/hystrix.stream
> 
> ![image](https://note.youdao.com/yws/res/5020/E5C2370F1F2F48EEB2C09B914162D21C)


> - ==通过 hystrix 访问服务多次，观察监控信息==
> > http://localhost:3001/item-service/35
> 
> > http://localhost:3001/user-service/7
> > http://localhost:3001/user-service/7/score?score=100
> 
> > http://localhost:3001/order-service/123abc
> > http://localhost:3001/order-service/

> ![image](https://note.youdao.com/yws/res/4960/A7EB1D28FBEA4C7FAEE5D302F5793423)

---
## hystrix 熔断
整个链路达到一定的阈值，默认情况下，==10秒内产生超过20次请求==，则符合第一个条件。
满足第一个条件的情况下，如果请求的==错误百分比==大于阈值，则会打开断路器，==默认为50%==。
Hystrix的逻辑，先判断是否满足第一个条件，再判断第二个条件，如果==两个条件都满足，则会开启断路器==

### 使用 apache 的并发访问测试工具 ab
http://httpd.apache.org/docs/current/platform/windows.html#down
> ![image](https://note.youdao.com/yws/res/4974/5BEDB7BC278E493AADDD7E3E75489A5A)

- ==用 ab 工具，以并发50次，来发送20000个请求==

```
ab -n 20000 -c 50 http://localhost:3001/item-service/35
```

> - ==断路器状态为 Open，所有请求会被短路，直接降级执行 fallback 方法==

> ![image](https://note.youdao.com/yws/res/4965/37FB5934101E466D8095AC95C6F4156D)

## hystrix 配置
> https://github.com/Netflix/Hystrix/wiki/Configuration

- `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds` <br />
请求超时时间，超时后触发失败降级

- `hystrix.command.default.circuitBreaker.requestVolumeThreshold` <br />
10秒内请求数量，默认20，如果没有达到该数量，即使请求全部失败，也不会触发断路器打开

- `hystrix.command.default.circuitBreaker.errorThresholdPercentage` <br />
失败请求百分比，达到该比例则触发断路器打开

- `hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds` <br />
断路器打开多长时间后，再次允许尝试访问（半开），仍失败则继续保持打开状态，如成功访问则关闭断路器，默认 5000






---
# 十四、feign 整合ribbon+hystrix
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

微服务应用中，ribbon 和 hystrix 总是同时出现，feign 整合了两者，并提供了**声明式消费者**客户端

> - ==用 feign 代替 hystrix+ribbon==

> ![feign-1](https://note.youdao.com/yws/res/4971/75247D03F569422388CC80E4FACCA472)

## 新建 sp09-feign 项目

> ![image](https://note.youdao.com/yws/res/4977/7273F1C8620E41EB8E8081DB1516F201)

> ![image](https://note.youdao.com/yws/res/5000/AF8EDF8923A24702A74CEECDADE3D48A)

## pom.xml

- ==需要添加 sp01-commons 依赖==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.tedu</groupId>
	<artifactId>sp09-feign</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp09-feign</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.tedu</groupId>
			<artifactId>sp01-commons</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```
## application.yml

```yml
spring:
  application:
    name: feign
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

## 主程序添加 `@EnableDiscoveryClient` 和 `@EnableFeignClients`

```java
package com.tedu.sp09;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class Sp09FeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp09FeignApplication.class, args);
	}

}

```

## java 源文件

> ![image](https://note.youdao.com/yws/res/5002/32B1C2978E2C436BB2379EE1E56E60DA)

### ItemFeignService

```java
package com.tedu.sp09.service;

import java.util.List;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

import com.tedu.sp01.pojo.Item;
import com.tedu.web.util.JsonResult;

@FeignClient("item-service")
public interface ItemFeignService {
	@GetMapping("/{orderId}")
	JsonResult<List<Item>> getItems(@PathVariable String orderId);

	@PostMapping("/decreaseNumber")
	JsonResult decreaseNumber(@RequestBody List<Item> items);
}

```

### UserFeignService

- ==注意`@RequestParam`不能省略==

```java
package com.tedu.sp09.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestParam;

import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@FeignClient("user-service")
public interface UserFeignService {
	@GetMapping("/{userId}")
	JsonResult<User> getUser(@PathVariable Integer userId);

	@GetMapping("/{userId}/score") 
	JsonResult addScore(@PathVariable Integer userId, @RequestParam Integer score);
}

```

### OrderFeignService

```java
package com.tedu.sp09.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import com.tedu.sp01.pojo.Order;
import com.tedu.web.util.JsonResult;

@FeignClient("order-service")
public interface OrderFeignService {
	@GetMapping("/{orderId}")
	JsonResult<Order> getOrder(@PathVariable String orderId);

	@GetMapping("/")
	JsonResult addOrder();

}
```

### FeignController

```java
package com.tedu.sp09.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.pojo.User;
import com.tedu.sp09.service.ItemFeignService;
import com.tedu.sp09.service.OrderFeignService;
import com.tedu.sp09.service.UserFeignService;
import com.tedu.web.util.JsonResult;

@RestController
public class FeignController {
	@Autowired
	private ItemFeignService itemServcie;
	@Autowired
	private UserFeignService userServcie;
	@Autowired
	private OrderFeignService orderServcie;
	
	@GetMapping("/item-service/{orderId}")
	public JsonResult<List<Item>> getItems(@PathVariable String orderId) {
		return itemServcie.getItems(orderId);
	}

	@PostMapping("/item-service/decreaseNumber")
	public JsonResult decreaseNumber(@RequestBody List<Item> items) {
		return itemServcie.decreaseNumber(items);
	}

	/////////////////////////////////////////
	
	@GetMapping("/user-service/{userId}")
	public JsonResult<User> getUser(@PathVariable Integer userId) {
		return userServcie.getUser(userId);
	}

	@GetMapping("/user-service/{userId}/score") 
	public JsonResult addScore(@PathVariable Integer userId, Integer score) {
		return userServcie.addScore(userId, score);
	}
	
	/////////////////////////////////////////
	
	@GetMapping("/order-service/{orderId}")
	public JsonResult<Order> getOrder(@PathVariable String orderId) {
		return orderServcie.getOrder(orderId);
	}

	@GetMapping("/order-service")
	public JsonResult addOrder() {
		return orderServcie.addOrder();
	}
}
```

## 启动服务，并访问测试

> ![image](https://note.youdao.com/yws/res/5033/09BDA2238D4C421E86A8885AC9C1EBE5)

> > http://eureka1:2001
> 
> > http://localhost:3001/item-service/35<br />
> > http://localhost:3001/item-service/decreaseNumber<br />
> > 使用postman，POST发送以下格式数据：<br />
> > ==[{"id":1, "name":"abc", "number":23},{"id":2, "name":"def", "number":11}]==
> 
> > http://localhost:3001/user-service/7<br />
> > http://localhost:3001/user-service/7/score?score=100
> 
> > http://localhost:3001/order-service/123abc<br />
> > http://localhost:3001/order-service/
