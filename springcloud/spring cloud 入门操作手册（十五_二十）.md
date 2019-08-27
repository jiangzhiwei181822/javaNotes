> tedu.cn

<span style="font-size: 50px">spring cloud 入门手册</span>

# 目录
[TOC]

---
# 十五、feign + ribbon 负载均衡和重试
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> - 无需额外配置，feign ==默认已启用==了 ribbon ==负载均衡==和==重试==机制。可以通过配置对参数进行调整

## application.yml 配置 ribbon 超时和重试

> - ribbon.xxx 全局配置
> - item-service.ribbon.xxx 对特定服务实例的配置

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
      
ribbon:
  ConnectTimeout: 1000
  ReadTimeout: 1000
  
item-service:
  ribbon:
    ConnectTimeout: 500
    ReadTimeout: 1000
    MaxAutoRetriesNextServer: 2
    MaxAutoRetries: 1
```

## 启动服务，访问测试

> http://localhost:3001/item-service/35


---
# 十六、feign + hystrix 降级
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

## feign 启用 hystrix

feign 默认没有启用 hystrix，添加配置，启用 hystrix
- `feign.hystrix.enabled=true`

### application.yml 添加配置

```yml
feign:
  hystrix:
    enabled: true
```

启用 hystrix 后，访问服务
> http://localhost:3001/item-service/35

==默认1秒==会快速失败

> ![image](https://note.youdao.com/yws/res/5012/2EAF2FB091FB4BD0B9369A15AB4E4E22)

### 可以添加配置，加大 hystrix 超时时间

```yml
feign:
  hystrix:
    enabled: true
    
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000
```



## feign + hystrix 降级

### 降级类

#### ItemFeignServiceFB

```java
package com.tedu.sp09.service;

import java.util.List;
import org.springframework.stereotype.Component;
import com.tedu.sp01.pojo.Item;
import com.tedu.web.util.JsonResult;

@Component
public class ItemFeignServiceFB implements ItemFeignService {

	@Override
	public JsonResult<List<Item>> getItems(String orderId) {
		return JsonResult.err("无法获取订单商品列表");
	}

	@Override
	public JsonResult decreaseNumber(List<Item> items) {
		return JsonResult.err("无法修改商品库存");
	}

}

```

#### UserFeignServiceFB

```java
package com.tedu.sp09.service;

import org.springframework.stereotype.Component;
import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@Component
public class UserFeignServiceFB implements UserFeignService {

	@Override
	public JsonResult<User> getUser(Integer userId) {
		return JsonResult.err("无法获取用户信息");
	}

	@Override
	public JsonResult addScore(Integer userId, Integer score) {
		return JsonResult.err("无法增加用户积分");
	}

}

```

#### OrderFeignServiceFB

```java
package com.tedu.sp09.service;

import org.springframework.stereotype.Component;
import com.tedu.sp01.pojo.Order;
import com.tedu.web.util.JsonResult;

@Component
public class OrderFeignServiceFB implements OrderFeignService {

	@Override
	public JsonResult<Order> getOrder(String orderId) {
		return JsonResult.err("无法获取商品订单");
	}

	@Override
	public JsonResult addOrder() {
		return JsonResult.err("无法保存订单");
	}

}

```

### feign service 接口中指定降级类

#### ItemFeignService

```java
...
@FeignClient(name="item-service", fallback = ItemFeignServiceFB.class)
public interface ItemFeignService {
...
```

#### UserFeignService

```java
...
@FeignClient(name="user-service", fallback = UserFeignServiceFB.class)
public interface UserFeignService {
...
```

#### OrderFeignService

```java
...
@FeignClient(name="order-service",fallback = OrderFeignServiceFB.class)
public interface OrderFeignService {
...
```

### 启动服务，访问测试

> http://localhost:3001/item-service/35

> ![image](https://note.youdao.com/yws/res/5048/F40B817F606345D3BA153AB512BF2261)








---
# 十七、feign + hystrix 监控和熔断测试
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![feign](https://note.youdao.com/yws/res/4967/11D0E51FECAF4CD18889CC11F66ACC8A)

## 修改sp09-feign项目<br />pom.xml 添加 hystrix 起步依赖

- ==feign 没有包含完整的 hystrix 依赖==

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>
		spring-cloud-starter-netflix-hystrix
	</artifactId>
</dependency>
```

## 主程序添加 `@EnableCircuitBreaker`

```java
package com.tedu.sp09;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableCircuitBreaker
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class Sp09FeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp09FeignApplication.class, args);
	}

}

```

## sp09-feign 配置 actuator，暴露 `hystrix.stream` 监控端点

### actuator 依赖

确认已经添加了 actuator 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### application.yml 暴露 `hystrix.stream` 端点

```yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

## 启动服务，查看监控端点

> ![image](https://note.youdao.com/yws/res/5039/FC014DB10AED409E81BB31AE15339402)

## hystrix dashboard 
启动 hystrix dashboard 服务，填入 feign 监控路径，开启监控
> 访问 http://localhost:4001/hystrix
> - 填入 feign 监控路径<br />
> http://localhost:3001/actuator/hystrix.stream


> ![image](https://note.youdao.com/yws/res/4983/6835C744590E445C8D4C143C2ED2ABBB)

## 熔断测试

- ==用 ab 工具，以并发50次，来发送20000个请求==

```
ab -n 20000 -c 50 http://localhost:3001/item-service/35
```

> - ==断路器状态为 Open，所有请求会被短路，直接降级执行 fallback 方法==

> ![image](https://note.youdao.com/yws/res/4965/37FB5934101E466D8095AC95C6F4156D)


---
# 十八、order service 调用商品库存服务和用户服务
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![order2](https://note.youdao.com/yws/res/5650/13565DD868B343D682C83D723D174DDA)

修改 sp04-orderservice 项目，添加 feign，调用 item service 和 user service

1. pom.xml
2. application.yml
3. 主程序
4. ItemFeignService
5. UserFeignService
6. ItemFeignServiceFB
7. UserFeignServiceFB
8. OrderController


## pom.xml

- 添加以下依赖：
- actuator
- feign
- hystrix

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
	<artifactId>sp04-orderservice</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp04-orderservice</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
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
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>
                spring-cloud-starter-netflix-eureka-client
            </artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
        	<groupId>org.springframework.cloud</groupId>
        	<artifactId>spring-cloud-starter-openfeign</artifactId>
        	<version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
        	<groupId>org.springframework.cloud</groupId>
        	<artifactId>
        		spring-cloud-starter-netflix-hystrix
        	</artifactId>
        	<version>2.1.1.RELEASE</version>
        </dependency>
	</dependencies>

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

- ribbon 重试和 hystrix 超时，这里没有设置，采用了默认值

```yml

spring:
  application:
    name: order-service
    
server:
  port: 8201
  
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
feign:
  hystrix:
    enabled: true
    
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
        
---
spring:
  profiles: order1
  
server:
  port: 8201
  
---
spring:
  profiles: order2
  
server:
  port: 8202
  
```

## 主程序

```java
package com.tedu.sp04;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

//@EnableDiscoveryClient
//@SpringBootApplication

@EnableFeignClients
@SpringCloudApplication
public class Sp04OrderserviceApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp04OrderserviceApplication.class, args);
	}

}

```

## ItemFeignService

```java
package com.tedu.sp04.order.feignclient;

import java.util.List;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

import com.tedu.sp01.pojo.Item;
import com.tedu.web.util.JsonResult;

@FeignClient(name="item-service", fallback = ItemFeignServiceFB.class)
public interface ItemFeignService {
	@GetMapping("/{orderId}")
	JsonResult<List<Item>> getItems(@PathVariable String orderId);

	@PostMapping("/decreaseNumber")
	JsonResult decreaseNumber(@RequestBody List<Item> items);
}

```

## UserFeignService

```java
package com.tedu.sp04.order.feignclient;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestParam;

import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@FeignClient(name="user-service", fallback = UserFeignServiceFB.class)
public interface UserFeignService {
	@GetMapping("/{userId}")
	JsonResult<User> getUser(@PathVariable Integer userId);

	@GetMapping("/{userId}/score") 
	JsonResult addScore(@PathVariable Integer userId, @RequestParam Integer score);
}
```

## ItemFeignServiceFB

- 获取商品列表的降级方法，==模拟==使用缓存数据

```java
package com.tedu.sp04.order.feignclient;

import java.util.Arrays;
import java.util.List;

import org.springframework.stereotype.Component;

import com.tedu.sp01.pojo.Item;
import com.tedu.web.util.JsonResult;

@Component
public class ItemFeignServiceFB implements ItemFeignService {

	@Override
	public JsonResult<List<Item>> getItems(String orderId) {
		if(Math.random()<0.5) {
			return JsonResult.ok().data(
			
				Arrays.asList(new Item[] {
						new Item(1,"缓存aaa",2),
						new Item(2,"缓存bbb",1),
						new Item(3,"缓存ccc",3),
						new Item(4,"缓存ddd",1),
						new Item(5,"缓存eee",5)
				})
			
			);
		}
		return JsonResult.err("无法获取订单商品列表");
	}

	@Override
	public JsonResult decreaseNumber(List<Item> items) {
		return JsonResult.err("无法修改商品库存");
	}

}
```


## UserFeignServiceFB

- 获取用户信息的降级方法，==模拟==使用缓存数据

```java
package com.tedu.sp04.order.feignclient;

import org.springframework.stereotype.Component;

import com.tedu.sp01.pojo.User;
import com.tedu.web.util.JsonResult;

@Component
public class UserFeignServiceFB implements UserFeignService {

	@Override
	public JsonResult<User> getUser(Integer userId) {
		if(Math.random()<0.4) {
			return JsonResult.ok(new User(7, "缓存name"+userId, "缓存pwd"+userId));
		}
		return JsonResult.err("无法获取用户信息");
	}

	@Override
	public JsonResult addScore(Integer userId, Integer score) {
		return JsonResult.err("无法增加用户积分");
	}

}
```

## OrderServiceImpl

```java
package com.tedu.sp04.order.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.pojo.User;
import com.tedu.sp01.service.OrderService;
import com.tedu.sp04.order.feignclient.ItemFeignService;
import com.tedu.sp04.order.feignclient.UserFeignService;
import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class OrderServiceImpl implements OrderService {
	
	@Autowired
	private ItemFeignService itemService;
	@Autowired
	private UserFeignService userService;

	@Override
	public Order getOrder(String orderId) {
		//调用user-service获取用户信息
		JsonResult<User> user = userService.getUser(7);
		
		//调用item-service获取商品信息
		JsonResult<List<Item>> items = itemService.getItems(orderId);
		
		
		Order order = new Order();
		order.setId(orderId);
		order.setUser(user.getData());
		order.setItems(items.getData());
		return order;
	}

	@Override
	public void addOrder(Order order) {
		//调用item-service减少商品库存
		itemService.decreaseNumber(order.getItems());
		
		//TODO: 调用user-service增加用户积分
		userService.addScore(7, 100);
		
		log.info("保存订单："+order);
	}

}
```

## order-service 配置启动参数

- ==--spring.profiles.active=order1==
- ==--spring.profiles.active=order2==

> ![image](https://note.youdao.com/yws/res/5051/610E41D3797849BF949A52C29620E398)

## 启动服务，访问测试

> ![image](https://note.youdao.com/yws/res/5024/2E9B33C56EBC456BBD9693A55C093259)
> 
> - 根据orderid，获取订单<br />
> http://localhost:8201/123abc
> - 保存订单<br />
> http://localhost:8201/
> - 通过 feign 访问 order service<br />
> http://localhost:3001/order-service/123abc<br/>
> http://localhost:3001/order-service/

## hystrix dashboard 监控 order service 断路器

- 访问 http://localhost:4001/hystrix ，填入 order service 的断路器监控路径，启动监控
> - http://localhost:8201/actuator/hystrix.stream 
> - http://localhost:8202/actuator/hystrix.stream 

> ![image](https://note.youdao.com/yws/res/4980/C8A1893A3F5D409B96B4E6241C8E15CF)

---
# 十九、hystrix + turbine 集群聚合监控
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![turbine2](https://note.youdao.com/yws/res/5659/CEFE9832A4AB4A44B817A8F4736A350E)

hystrix dashboard 一次只能监控一个服务实例，使用 turbine 可以==汇集监控信息==，将聚合后的信息提供给 hystrix dashboard 来集中展示和监控

## 新建 sp10-turbine 项目

> ![image](https://note.youdao.com/yws/res/4975/EECFFCACBB99417D90BD28144BF98AE9)

> ![image](https://note.youdao.com/yws/res/5006/D5321EEE179D4C5D950F030E74D62613)

## pom.xml
```pom.xml
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
	<artifactId>sp10-turbine</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp10-turbine</name>
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
			<artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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

## application.yml
```yml

spring:
  application:
    name: turbin
    
server:
  port: 5001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
turbine:
  app-config: order-service, feign
  cluster-name-expression: new String("default")
```

## 主程序
```java
package com.tedu.sp10;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.turbine.EnableTurbine;

@EnableTurbine
@EnableDiscoveryClient
@SpringBootApplication
public class Sp10TurbineApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp10TurbineApplication.class, args);
	}

}
```

## 访问测试

> - turbine 监控路径 <br />
> http://localhost:5001/turbine.stream

> - 在 hystrix dashboard 中填入turbine 监控路径，开启监控 <br />
> http://localhost:4001/hystrix

> - ==turbine聚合了feign服务和order-service服务集群的hystrix监控信息==
> 
> ![image](https://note.youdao.com/yws/res/4959/F664292F08FB4EEABD9D57E1862D4560)













---
# 二十、zuul API网关
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![zuul2](https://note.youdao.com/yws/res/4962/9960F0FAD678452ABC9882EFB809B0D8)


zuul API 网关，为微服务应用提供统一的对外访问接口。
zuul 还提供过滤器，对所有微服务提供统一的请求校验。

## 新建 sp11-zuul 项目

> ![image](https://note.youdao.com/yws/res/4981/B13D7BD3924044E5ACFDC2300ADD646A)

> ![image](https://note.youdao.com/yws/res/5009/97817223F0354E70ADCCE904D570A92B)

## pom.xml

- ==需要添加 sp01-commons 依赖==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.tedu</groupId>
	<artifactId>sp11-zuul</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp11-zuul</name>
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
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.retry</groupId>
			<artifactId>spring-retry</artifactId>
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

## applicatoin.yml

> - ==zuul 路由配置可以省略，缺省以服务 id 作为访问路径==

```yml
spring:
  application:
    name: zuul
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka

zuul:
  routes:
    item-service: /item-service/**
    user-service: /user-service/**
    order-service: /order-service/**
```

## 主程序
```java
package com.tedu.sp11;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class Sp11ZuulApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp11ZuulApplication.class, args);
	}

}
```

## 启动服务，访问测试

> ![image](https://note.youdao.com/yws/res/5669/F1FF91B081044F67939340514B99DACA)

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



## zuul + ribbon 负载均衡

zuul 已经集成了 ribbon，默认==已经实现了负载均衡==

## zuul + ribbon 重试

### pom.xml 添加 spring-retry 依赖

> - ==需要 spring-retry 依赖==
```xml
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>
```

### 配置 zuul 开启重试，并配置 ribbon 重试参数

```yml
spring:
  application:
    name: zuul
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka

zuul:
  retryable: true

#  routes:
#    item-service: /item-service/**
#    user-service: /user-service/**
#    order-service: /order-service/**
    
ribbon:
  ConnectTimeout: 1000
  ReadTimeout: 1000
  MaxAutoRetriesNextServer: 1
  MaxAutoRetries: 1
```

## zuul + hystrix 降级

### 创建降级类

- ==getRoute() 方法中指定应用此降级类的服务id，星号或null值可以通配所有服务==

### ItemServiceFallback

```java
package com.tedu.sp11;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class ItemServiceFallback implements FallbackProvider {
	@Override
	public String getRoute() {
		return "item-service"; //"*"; //null;
	}

	@Override
	public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return response();
	}

	private ClientHttpResponse response() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }
            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }
            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
            	log.info("fallback body");
            	String s = JsonResult.err().msg("后台服务错误").toString();
                return new ByteArrayInputStream(s.getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```

#### OrderServiceFallback

```java
package com.tedu.sp11;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class OrderServiceFallback implements FallbackProvider {
	@Override
	public String getRoute() {
		return "order-service"; //"*"; //null;
	}

	@Override
	public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return response();
	}

	private ClientHttpResponse response() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }
            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }
            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
            	log.info("fallback body");
            	String s = JsonResult.err().msg("后台服务错误").toString();
                return new ByteArrayInputStream(s.getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}

```

### zuul + hystrix 短路

降低 hystrix 超时时间，以便测试降级

```yml
spring:
  application:
    name: zuul
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka

zuul:
  retryable: true
    
ribbon:
  ConnectTimeout: 1000
  ReadTimeout: 2000
  MaxAutoRetriesNextServer: 1
  MaxAutoRetries: 1
  
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 500
```

> ![image](https://note.youdao.com/yws/res/5050/568B0985308D47B1962674FDF8A5D7C4)

## zuul + hystrix dashboard 监控

### 暴露 hystrix.stream 监控端点

- ==zuul 已经包含 actuator 依赖==

```yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```
> - 查看暴露的监控端点 <br />
> http://localhost:3001/actuator
>
> http://localhost:3001/actuator/hystrix.stream

### 开启监控

启动 sp08-hystrix-dashboard，填入 zuul 的监控端点路径，开启监控

> http://localhost:4001/hystrix
> 
> 填入监控端点：<br /> http://localhost:3001/actuator/hystrix.stream

> ![image](https://note.youdao.com/yws/res/4978/04933436F25E4894BFD49B2D2951313A)

### zuul + turbine 聚合监控

==修改 turbine 项目，聚合 zuul 服务实例==

```yml

spring:
  application:
    name: turbin
    
server:
  port: 5001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
turbine:
  app-config: order-service, zuul
  cluster-name-expression: new String("default")
```

> - 监控端点<br />
> http://localhost:5001/turbine.stream


> ![image](https://note.youdao.com/yws/res/4958/EDDC40227E3B4177847310B5251F2E64)
