> tedu.cn

<span style="font-size: 50px">spring cloud 入门手册</span>

# 目录
[TOC]

---
# 五、order service 订单服务
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

1. 新建项目
2. 配置依赖 pom.xml
3. 配置 application.yml
4. ~~配置主程序~~
5. 编写代码


## 新建 spring boot 起步项目
> ![image](https://note.youdao.com/yws/res/4985/B207D713C9294E378D14AA5F7C3DCC6E)

## 选择依赖项
- 只选择 web

> ![image](https://note.youdao.com/yws/res/5008/8F524AA83345448E8AD4374AF841C43D)

## pom.xml
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

## applicatoin.yml

```yml
spring:
  application:
    name: order-service
    
server:
  port: 8201
```

## 主程序

- 默认代码，不需要修改

```java
package com.tedu.sp04;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Sp04OrderserviceApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp04OrderserviceApplication.class, args);
	}

}
```

## java 源文件

> ![image](https://note.youdao.com/yws/res/5011/8B32212ABBF543929DF6D5ED49351EE2)

### OrderServiceImpl

```java
package com.tedu.sp04.order.service;

import org.springframework.stereotype.Service;

import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.service.OrderService;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class OrderServiceImpl implements OrderService {

	@Override
	public Order getOrder(String orderId) {
		//TODO: 调用user-service获取用户信息
		//TODO: 调用item-service获取商品信息
		Order order = new Order();
		order.setId(orderId);
		return order;
	}

	@Override
	public void addOrder(Order order) {
		//TODO: 调用item-service减少商品库存
		//TODO: 调用user-service增加用户积分
		log.info("保存订单："+order);
	}

}
```
### OrderController

```java
package com.tedu.sp04.order.controller;

import java.util.Arrays;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.pojo.Order;
import com.tedu.sp01.pojo.User;
import com.tedu.sp01.service.OrderService;
import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class OrderController {
	@Autowired
	private OrderService orderService;
	
	@GetMapping("/{orderId}")
	public JsonResult<Order> getOrder(@PathVariable String orderId) {
		log.info("get order, id="+orderId);
		
		Order order = orderService.getOrder(orderId);
		return JsonResult.ok(order);
	}
	
	@GetMapping("/")
	public JsonResult addOrder() {
		//模拟post提交的数据
		Order order = new Order();
		order.setId("123abc");
		order.setUser(new User(7,null,null));
		order.setItems(Arrays.asList(new Item[] {
				new Item(1,"aaa",2),
				new Item(2,"bbb",1),
				new Item(3,"ccc",3),
				new Item(4,"ddd",1),
				new Item(5,"eee",5),
		}));
		orderService.addOrder(order);
		return JsonResult.ok();
	}
}
```

---
# 六、service 访问测试
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> - ==item-service==
> > 根据orderid，查询商品<br />
> > http://localhost:8001/35
> 
> > 减少商品库存<br />
> > http://localhost:8001/decreaseNumber
> > 
> > 使用postman，POST发送以下格式数据：<br />
> > ==[{"id":1, "name":"abc", "number":23},{"id":2, "name":"def", "number":11}]==
> 
> - ==user-service==
> > 根据userid查询用户信息<br />
> > http://localhost:8101/7
> > 
> > 根据userid，为用户增加积分<br />
> > http://localhost:8101/7/score?score=100
> 
> - ==order-service==
> > 根据orderid，获取订单<br />
> > http://localhost:8201/123abc
> 
> > 保存订单，观察控制台日志输出<br />
> > http://localhost:8201/



---
# 七、eureka 注册与发现
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![eureka-01](https://note.youdao.com/yws/res/4998/05FE6D20899144758392491F236978D9)

1. 创建eureka项目
2. 配置依赖 pom.xml
3. 配置 application.yml
4. 主程序启用 eureka 服务器
5. 启动，访问测试

## 创建 eureka server 项目：sp05-eureka
> ![image](https://note.youdao.com/yws/res/4976/CC26FF6D4B324951AF472EB1846AAA06)
> ![image](https://note.youdao.com/yws/res/5013/6D425557E5074B1CA6B73607CD42C5A7)

## pom.xml
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
	<artifactId>sp05-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp05-eureka</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
    name: eureka-server
    
server:
  port: 2001
  
eureka:
  server:
    enable-self-preservation: false
  instance:
    hostname: eureka1
  client:
    register-with-eureka: false
    fetch-registry: false
  
```

- `eureka.server.enable-self-preservation`<br />
eureka 的自我保护状态：心跳失败的比例，在15分钟内是否低于85%,如果出现了低于的情况，Eureka Server会将当前的实例注册信息保护起来，同时提示一个警告，一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据。也就是不会注销任何微服务


- `eureka.client.register-with-eureka=false`<br />
不向自身注册

- `eureka.client.fetch-registry=false`<br />
不从自身拉去注册信息

- `eureka.instance.lease-expiration-duration-in-seconds`<br />
最后一次心跳后，间隔多久认定微服务不可用，默认90

## 主程序
- 添加 @EnableEurekaServer
```java
package com.tedu.sp05;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class Sp05EurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp05EurekaApplication.class, args);
	}

}
```

## 修改 hosts 文件，添加 eureka 域名映射

C:\Windows\System32\drivers\etc\hosts

添加内容：

```
127.0.0.1       eureka1
127.0.0.1       eureka2
```

## 启动，并访问测试
- http://eureka1:2001

> ![image](https://note.youdao.com/yws/res/4961/D437EC7C24EA4E2D9182F5F4BDA5D200)































---
# 八、service provider 服务提供者
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![eureka-01](https://note.youdao.com/yws/res/4998/05FE6D20899144758392491F236978D9)

- 修改 item-service、user-service、order-service，把微服务注册到 eureka 服务器

1. pom.xml 添加eureka依赖
2. application.yml 添加eureka注册配置
3. 主程序启用eureka客户端
4. 启动服务，在eureka中查看注册信息

## pom.xml 添加eureka依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>
		spring-cloud-starter-netflix-eureka-client
	</artifactId>
	<version>2.1.1.RELEASE</version>
</dependency>
```

## application.yml 添加 eureka注册配置
```yml
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka
```

- `eureka.instance.lease-renewal-interval-in-seconds`<br />
心跳间隔时间，默认 30 秒



- `eureka.client.registry-fetch-interval-seconds`<br />
拉取注册信息间隔时间，默认 30 秒

## 主程序启用服务注册发现客户端
```java
添加 @EnableDiscoveryClient 注解
```

## 启动，并访问 eureka 查看注册信息

> ![image](https://note.youdao.com/yws/res/5409/6DC289C3303E44ADA677F8E420EA2693)

- http://eureka1:2001

> ![image](https://note.youdao.com/yws/res/4988/0EC08475571F436F9328F77EEC97C9D7)






























---
# 九、ribbon 服务消费者
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![ribbon01](https://note.youdao.com/yws/res/4972/E2CC42F21E6A48A7910A7E6A216908BB)

ribbon 提供了负载均衡和重试功能

1. 新建 ribbon 项目
2. pom.xml
3. application.yml
4. 主程序
5. controller
6. 启动，并访问测试


## 新建 sp06-ribbon 项目
> ![image](https://note.youdao.com/yws/res/4986/3BFC93EE7BCC4F609375623E1FE54E17)
> ![image](https://note.youdao.com/yws/res/5014/6314DFCFF1454BA594AC5BC66AE2F071)

## pom.xml
- ==eureka-client 中已经包含 ribbon 依赖==

- ==需要添加 sp01-commons 依赖==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.tedu</groupId>
	<artifactId>springcloud-012-ribbon</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springcloud-012-ribbon</name>
	<description>Demo project for Spring Cloud</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.tedu</groupId>
			<artifactId>user-service-commons</artifactId>
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
    name: ribbon
    
server:
  port: 3001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka
```

## 主程序

- ==创建 RestTemplate 实例==

```java
package com.tedu.sp06;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableDiscoveryClient
@SpringBootApplication
public class Sp06RibbonApplication {
	
	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(Sp06RibbonApplication.class, args);
	}

}

```

## RibbonController
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
		return rt.getForObject("http://localhost:8001/{1}", JsonResult.class, orderId);
	}

	@PostMapping("/item-service/decreaseNumber")
	public JsonResult decreaseNumber(@RequestBody List<Item> items) {
		return rt.postForObject("http://localhost:8001/decreaseNumber", items, JsonResult.class);
	}

	/////////////////////////////////////////
	
	@GetMapping("/user-service/{userId}")
	public JsonResult<User> getUser(@PathVariable Integer userId) {
		return rt.getForObject("http://localhost:8101/{1}", JsonResult.class, userId);
	}

	@GetMapping("/user-service/{userId}/score") 
	public JsonResult addScore(
			@PathVariable Integer userId, Integer score) {
		return rt.getForObject("http://localhost:8101/{1}/score?score={2}", JsonResult.class, userId, score);
	}
	
	/////////////////////////////////////////
	
	@GetMapping("/order-service/{orderId}")
	public JsonResult<Order> getOrder(@PathVariable String orderId) {
		return rt.getForObject("http://localhost:8201/{1}", JsonResult.class, orderId);
	}

	@GetMapping("/order-service")
	public JsonResult addOrder() {
		return rt.getForObject("http://localhost:8201/", JsonResult.class);
	}
}

```

## 启动服务，并访问测试
> ![image](https://note.youdao.com/yws/res/5047/D12A530486F4444A946C6AF976B4D3EF)

> > http://eureka1:2001
> 
> > http://localhost:3001/item-service/35
> > http://localhost:3001/item-service/decreaseNumber<br />
> > 使用postman，POST发送以下格式数据：<br />
> > ==[{"id":1, "name":"abc", "number":23},{"id":2, "name":"def", "number":11}]==
> 
> > http://localhost:3001/user-service/7
> > http://localhost:3001/user-service/7/score?score=100
> 
> > http://localhost:3001/order-service/123abc
> > http://localhost:3001/order-service/

















---
# 十、eureka 和 “服务提供者”的高可用
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![ribbon02](https://note.youdao.com/yws/res/4970/34FD9147DC5A405499F6BAC34E783F0B)


## eureka 高可用



### application.yml
```yml
spring:
  application:
    name: eureka-server
    
#server:
#  port: 2001

eureka:
  server:
    enable-self-preservation: false
#  instance:
#    hostname: eureka1
#  client:
#    register-with-eureka: false
#    fetch-registry: false

---
spring:
  profiles: eureka1

server:
  port: 2001
  
eureka:
  instance:
    hostname: eureka1
  client:
    service-url: 
      defaultZone: http://eureka2:2002/eureka

---
spring:
  profiles: eureka2

server:
  port: 2002
  
eureka:
  instance:
    hostname: eureka2
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka
    
```
### STS 配置启动参数 `--spring.profiles.active`

- ==eureka1 启动参数：==
```
--spring.profiles.active=eureka1
```


> ![image](https://note.youdao.com/yws/res/4989/41D1A302ACBB49FBBCC9C1FFA6206B41)

> ![image](https://note.youdao.com/yws/res/4995/B6247D4415C74EBDA0FA752B05296169)

- ==eureka2 启动参数：==
```
--spring.profiles.active=eureka2
```

> ![image](https://note.youdao.com/yws/res/4997/D61DCE37F3EC42B494096BD58CF52498)

> ![image](https://note.youdao.com/yws/res/4994/6595D74C250A445294618C75457D7189)

> ![image](https://note.youdao.com/yws/res/5046/61AF441DE04546D899A1E9DA82B79F95)

### 访问 eureka 服务器，查看注册信息
- http://eureka1:2001/

> ![image](https://note.youdao.com/yws/res/4999/F09D1114F56149EF9B6CFBFA2A5D4469)

- http://eureka2:2002/

> ![image](https://note.youdao.com/yws/res/5004/A4BA995758024F979D41FBA7B190516E)

### eureka客户端注册时，向两个服务器注册

修改以下微服务
- sp02-itemservice
- sp03-userservice
- sp04-orderservice
- sp06-ribbon

```yml
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

当一个 eureka 服务宕机时，仍可以连接另一个 eureka 服务

## item-service 高可用
### application.yml
```yml

spring:
  application:
    name: item-service
    
#server:
#  port: 8001
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka

---
spring:
  profiles: item1
  
server:
  port: 8001
---
spring:
  profiles: item2

server:
  port: 8002

```



### 配置启动参数

- ==item-service-8001==
```
--spring.profiles.active=item1
```

> ![image](https://note.youdao.com/yws/res/4993/24E0AC80DEF94478B59E7C056206CF6B)

> ![image](https://note.youdao.com/yws/res/4992/8D2C71AD1A5E44E591B7CAB2070500F1)

> ![image](https://note.youdao.com/yws/res/5003/B4E4BE067E6C45238D2FFC4B5092CE5F)

- ==item-service-8002==
```
--spring.profiles.active=item2
```

> ![image](https://note.youdao.com/yws/res/4990/891AD11F831749CBB7D08ED19D0154E5)

> ![image](https://note.youdao.com/yws/res/5034/DE939D842F71431BA2ABF46B9870F537)

### 启动测试

- 访问 eureka 查看 item-service 注册信息

> ![image](https://note.youdao.com/yws/res/4979/5E1BA1A055E243439F46BA4791B809FB)

> - 访问两个端口测试<br />
> > http://localhost:8001/35
> > http://localhost:8002/35
