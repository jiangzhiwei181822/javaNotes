> tedu.cn

<span style="font-size: 50px">spring cloud 入门手册</span>

# 目录
[TOC]

---
# 二十一、zuul 请求过滤
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![image](https://note.youdao.com/yws/res/5695/13D7D2E54486404CA789605619F448E2)

## 定义过滤器，继承 ZuulFilter

在 sp11-zuul 项目中新建过滤器类

```java
package com.tedu.sp11.filter;

import javax.servlet.http.HttpServletRequest;

import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import com.tedu.web.util.JsonResult;

@Component
public class AccessFilter extends ZuulFilter{
	@Override
	public boolean shouldFilter() {
	    //对指定的serviceid过滤，如果要过滤所有服务，直接返回 true
	    
		RequestContext ctx = RequestContext.getCurrentContext();
		String serviceId = (String) ctx.get(FilterConstants.SERVICE_ID_KEY);
		if(serviceId.equals("item-service")) {
			return true;
		}
		return false;
	}

	@Override
	public Object run() throws ZuulException {
		RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest req = ctx.getRequest();
		String at = req.getParameter("token");
		if (at == null) {
			//此设置会阻止请求被路由到后台微服务
			ctx.setSendZuulResponse(false);
			ctx.setResponseStatusCode(200);
			ctx.setResponseBody(JsonResult.err().code(JsonResult.NOT_LOGIN).toString());
		}
		//zuul过滤器返回的数据设计为以后扩展使用，
		//目前该返回值没有被使用
		return null;
	}

	@Override
	public String filterType() {
		return FilterConstants.PRE_TYPE;
	}

	@Override
	public int filterOrder() {
	    //该过滤器顺序要 > 5，才能得到 serviceid
		return FilterConstants.PRE_DECORATION_FILTER_ORDER+1;
	}
}
```

## 访问测试

> - 没有token参数不允许访问<br />
> http://localhost:3001/item-service/35

> - 有token参数可以访问<br />
> http://localhost:3001/item-service/35?token=1234

---
# 二十二、zuul Cookie过滤
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

zuul 会过滤敏感 http 协议头，默认过滤以下协议头：
- Cookie
- Set-Cookie
- Authorization

可以设置 zuul ==不过滤==这些协议头

```yml
zuul:
  sensitive-headers: 
```


---
# 二十三、config 配置中心
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![config](https://note.youdao.com/yws/res/4957/005DAEBF1CDF432BA8DD6D8453CA2B91)

yml 配置文件保存到 git 服务器，例如 ==github.com==  或 ==gitee.com==

微服务启动时，从服务器获取配置文件

## github 上存放配置文件

- github.com

> - ==新建仓库==
> 
> ![image](https://note.youdao.com/yws/res/5042/8A088ABBBD50445A87023D6285F06CC6)
> - ==仓库命名==
> 
> ![image](https://note.youdao.com/yws/res/4973/CB0BE4AAC3D44A58AB188577182E4C8E)
> - ==点击新建文件==
> 
> ![image](https://note.youdao.com/yws/res/5010/A69BCB059A51424A86FA9642C5028A1A)
> - ==填写子目录名，和文件名，并复制粘贴 item-service 配置文件内容==
> 
> ![image](https://note.youdao.com/yws/res/4996/017E78B041B14E4C91AB2BE651206817)
> - ==点击新建文件==
> 
> ![image](https://note.youdao.com/yws/res/5041/9715A15B504A4BA8B70E6CBF705051EC)
> - ==填写文件名，并复制粘贴 user-service 配置文件内容==
> 
> ![image](https://note.youdao.com/yws/res/5019/D6A023721A634414B25792B1D1FF4674)
> - ==继续添加 order-service 的配置文件==
> 
> ![image](https://note.youdao.com/yws/res/5025/D10C914225C34E9DA8E0E1A0D31F958C)
> - ==hystrix dashboard 的配置文件==
> 
> ![image](https://note.youdao.com/yws/res/5022/0E119210F6DA473FAA1D2A88821550D1)
> - ==turbine 的配置文件==
> 
> ![image](https://note.youdao.com/yws/res/5028/FB1F091BC112461E85D03FF912A6EF57)
> - ==zuul 的配置文件==
> 
> ![image](https://note.youdao.com/yws/res/5032/21182037970647A19D0EE1E9DFD7967D)
> - ==共6个文件==
> 
> ![image](https://note.youdao.com/yws/res/5040/5F3E7B62B92D420D817B6C24DCA09E60)

## config 服务器

config 配置中心从 git 下载所有配置文件。

微服务启动时，从 config 配置中心获取配置信息。

### 新建 sp12-config 项目

> ![image](https://note.youdao.com/yws/res/5005/070904708DC840DC9175AEA617D71F6C)

> ![image](https://note.youdao.com/yws/res/5026/53B9B34440D84EAA8FE3F6B31E9E844C)

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
	<artifactId>sp12-config</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp12-config</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
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
    name: config-server
  
  cloud:
    config:
      server:
        git:
          uri: https://github.com/你的个人路径/sp-config
          searchPaths: config
          #username: your-username
          #password: your-password
    
server:
  port: 6001
    
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

### 主程序添加 `@EnableConfigServer` 和 `@EnableDiscoveryClient`

```java
package com.tedu.sp12;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@EnableDiscoveryClient
@SpringBootApplication
public class Sp12ConfigApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp12ConfigApplication.class, args);
	}

}
```

### 启动，访问测试

> http://localhost:6001/item-service/dev<br />
> http://localhost:6001/zuul/dev<br />
> ...

## config 客户端
修改以下项目，从配置中心获取配置信息
- sp02-itemservice
- sp03-userservice
- sp04-orderservice
- sp08-hystrix-dashboard
- sp10-turbine
- sp11-zuul

### pom.xml 添加 config 客户端依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 在六个项目中添加 bootstrap.yml

bootstrap.yml，引导配置文件，先于 application.yml 加载

- ==item-service==
```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: item-service
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```
- ==user-service==

```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: user-service
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

- ==order-service==

```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: order-service
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

- ==hystrix-dashboard==

```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: hystrix-dashboard
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

- ==turbine==

```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: turbine
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

- ==zuul==

```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: zuul
      profile: dev
      
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
```

## 启动服务，观察从配置中心获取配置信息的日志

> ![image](https://note.youdao.com/yws/res/5016/A03BA23C12F44552AF673582D779AA7F)

> ![image](https://note.youdao.com/yws/res/5021/0E6D50D3283A4A91AADE5487473DCE84)



## 配置刷新

spring cloud 允许运行时==动态刷新配置==，可以重新加载==本地配置文件== application.yml，或者==从配置中心获取==新的配置信息

以 `user-service` 为例演示配置刷新

### pom.xml

user-service 的 pom.xml 中添加 actuator 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### yml 配置文件中暴露 refresh 端点

修改 github 仓库中的 user-service-dev.yml

```yml
sp:
  user-service:
    users: "[{\"id\":7, \"username\":\"abc\",\"password\":\"123\"},{\"id\":8, \"username\":\"def\",\"password\":\"456\"},{\"id\":9, \"username\":\"ghi\",\"password\":\"789\"}]"

spring:
  application:
    name: user-service
    
server:
  port: 8101
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka1:2002/eureka

management:
  endpoints:
    web:
      exposure:
        include: refresh
```

> - 查看暴露的刷新端点
> 
> ![image](https://note.youdao.com/yws/res/5038/866A3AE5BE2D4CCBBEA0E1FF27BDF462)


### UserServiceImpl 添加 `@RefreshScope` 注解

- ==只允许对添加了 `@RefreshScope` 或 `@ConfigurationProperties` 注解的 Bean 刷新配置==，可以将更新的配置数据注入到 Bean 中

```java
package com.tedu.sp03.user.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.stereotype.Service;

import com.fasterxml.jackson.core.type.TypeReference;
import com.tedu.sp01.pojo.User;
import com.tedu.sp01.service.UserService;
import com.tedu.web.util.JsonUtil;

import lombok.extern.slf4j.Slf4j;

@RefreshScope
@Slf4j
@Service
public class UserServiceImpl implements UserService {
	@Value("${sp.user-service.users}")
	private String userJson;
	
	@Override
	public User getUser(Integer id) {
		log.info("users json string : "+userJson);
		List<User> list = JsonUtil.from(userJson, new TypeReference<List<User>>() {});
		for (User u : list) {
			if (u.getId().equals(id)) {
				return u;
			}
		}
		
		return new User(id, "name-"+id, "pwd-"+id);
	}

	@Override
	public void addScore(Integer id, Integer score) {
		//TODO 这里增加积分
		log.info("user "+id+" - 增加积分 "+score);
	}

}
```

## 先启动 user-service，再修改 yml 文件，添加新的用户数据

```yml
sp:
  user-service:
    users: "[{\"id\":7, \"username\":\"abc\",\"password\":\"123\"},{\"id\":8, \"username\":\"def\",\"password\":\"456\"},{\"id\":9, \"username\":\"ghi\",\"password\":\"789\"},{\"id\":99, \"username\":\"aaa\",\"password\":\"111\"}]"
```

## 访问刷新端点刷新配置

> - 使用 postman 向刷新端点发送 post 请求
> 
> ![image](https://note.youdao.com/yws/res/4991/08C49D7B8165452F9A5B92ED0D445841)

## 访问 user-service，查看动态更新的新用户数据

> - http://localhost:8101/99
> 
> ![image](https://note.youdao.com/yws/res/5049/8B515D66907A4ABE90C7D7D26F9694C8)

---
# 二十四、config bus + rabbitmq 消息总线配置刷新
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

> ![springcloud-bus2](https://note.youdao.com/yws/res/4968/AAA7295102B1454FB1BEC784F9FC83C5)

> post 请求消息总线刷新端点，服务器会向 rabbitmq 发布刷新消息，接收到消息的微服务会向配置服务器请求刷新配置信息

## rabbitmq 安装笔记

> https://note.youdao.com/ynoteshare1/index.html?id=03f6700cfa3865e1c490cdb618975b69&type=note

## 需要动态更新配置的微服务，添加 spring cloud bus 依赖，并添加 rabbitmq 连接信息

修改以下微服务
- item-service
- user-service
- order-service
- turbine
- hystrix-dashboard
- zuul

### pom.xml 添加 spring cloud bus 依赖

此处也可以使用 STS 编辑起步依赖，分别添加 bus、rabbitmq 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
```

### 配置文件中添加 rabbitmq 连接信息

- ==连接信息请修改成你的连接信息==

```yml
spring:
  ......
  rabbitmq:
    host: 192.168.64.140
    port: 5672
    username: admin
    password: admin
```

## config-server 添加 spring cloud bus 依赖、配置rabbitmq连接信息，并暴露 bus-refresh 监控端点

### pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
```

### application.yml

```yml

spring:
  application:
    name: config-server
  
  cloud:
    config:
      server:
        git:
          uri: https://github.com/你的名字路径/sp-config
          searchPaths: config
          #username: your-username
          #password: your-password
    
  rabbitmq:
    host: 192.168.64.140
    port: 5672
    username: admin
    password: admin

    
server:
  port: 6001
    
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```

> - 查看刷新端点<br />
> http://localhost:6001/actuator
> 
> ![image](https://note.youdao.com/yws/res/5056/92E4B0BECA4F4485A5908E67E822ACAF)


## 启动服务，请求刷新端点发布刷新消息
> ![image](https://note.youdao.com/yws/res/5017/FA98247835D84AAC8B9CF9BB05BC4EBB)

> - postman 向 bus-refresh 刷新端点发送 post 请求<br />
> http://localhost:6001/actuator/bus-refresh

> ![image](https://note.youdao.com/yws/res/5059/B29D171F7C2D4D3F983E92C8DF438ADE)

> - 如果刷新指定的微服务，可按下面格式访问：<br />
> http://localhost:8082/bus-refresh/user-service:8101

## config 本地文系统

可以把配置文件保存在配置中心服务的 resources 目录下，直接访问本地文件

## 把配置文件保存到 sp12-config 项目的 resources/config 目录下

### 从 github 下载配置文件

> ![image](https://note.youdao.com/yws/res/5093/074864A085C14B1C830C9114CF10F89A)

> - ==解压缩到 sp12-config 的 src/main/resource 目录下==
> 
> ![image](https://note.youdao.com/yws/res/5099/C262AF3B25BE449584AF6EAC04BA6065)


## 修改 application.yml 激活 native profile，并指定配置文件目录

- ==本地路径默认：`[classpath:/, classpath:/config, file:./, file:./config]`==

```yml

spring:
  application:
    name: config-server
  profiles:
    active: native
  
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config

#        git:
#          uri: https://github.com/你的用户路径/sp-config
#          searchPaths: config
#          username: your-username
#          password: your-password
        
    
  rabbitmq:
    host: 192.168.64.140
    port: 5672
    username: admin
    password: admin

    
server:
  port: 6001
    
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:2001/eureka, http://eureka2:2002/eureka
      
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```

---
# 二十五、sleuth 链路跟踪
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

随着系统规模越来越大，微服务之间调用关系变得错综复杂，一条调用链路中可能调用多个微服务，任何一个微服务不可用都可能造整个调用过程失败

spring cloud sleuth 可以跟踪调用链路，分析链路中每个节点的执行情况

## 微服务中添加 spring cloud sleuth 依赖

修改以下微服务的 pom.xml，添加 sleuth 依赖

- item-service
- user-service
- order-service
- zuul

 


```xml

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

## debug 查看链路跟踪日志

> - 通过 zuul 网关，访问 order-service<br />
> http://localhost:3001/order-service/112233
> 
> 四个微服务的控制台日志中，可以看到以下信息：<br />
> ==`[服务id,请求id,span id,是否发送到zipkin]`==

- 请求id：请求到达第一个微服务时生成一个请求id，该id在调用链路中会一直向后面的微服务传递
- span id：链路中每一步微服务调用，都生成一个新的id

[zuul,==6c24c0a7a8e7281a==,**6c24c0a7a8e7281a**,false]

[order-service,==6c24c0a7a8e7281a==,**993f53408ab7b6e3**,false]

[item-service,==6c24c0a7a8e7281a==,**ce0c820204dbaae1**,false]

[user-service,==6c24c0a7a8e7281a==,**fdd1e177f72d667b**,false]


---
# 二十六、sleuth + zipkin 链路分析
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

zipkin 可以收集链路跟踪数据，提供可视化的链路分析

## 链路数据抽样比例

默认 10% 的链路数据会被发送到 zipkin 服务。可以配置修改抽样比例

```yml
spring:
  sleuth:
    sampler:
      probability: 0.1
```

## zipkin 服务

### 下载 zipkin 服务器

> - https://github.com/openzipkin/zipkin

> ![image](https://note.youdao.com/yws/res/5208/EECDE887029E42E5B4919BCF7C5F2AE5)

### 启动 zipkin 时，连接到 rabbitmq 

> ==`java -jar zipkin-server-2.12.9-exec.jar --zipkin.collector.rabbitmq.uri=amqp://admin:admin@192.168.64.140:5672`==
> 
> ![image](https://note.youdao.com/yws/res/5232/DAA3A068FC664BC7866AB274A75A5091)

> http://localhost:9411/zipkin
> 
> ![image](https://note.youdao.com/yws/res/5227/5B01205E83DD404A9E1C40F6A500025D)

## 微服务添加 zipkin 起步依赖

修改以下微服务

- item-service
- user-service
- order-service
- zuul

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

==如果没有配置过 spring cloud bus，需要再添加 `spring-cloud-starter-stream-rabbit` 依赖和 rabbitmq 连接信息==

## 启动并访问服务，访问 zipkin 查看链路分析

> - http://localhost:3001/order-service/112233<br />
> 刷新访问多次，链路跟踪数据中，默认只有 10% 会被收集到zipkin


> - 访问 zipkin
> - http://localhost:9411/zipkin
> 
> ![image](https://note.youdao.com/yws/res/5287/DDF04591FD814C3EB2D969C125E21A99)

> ![image](https://note.youdao.com/yws/res/5552/D7496A8A161F46FCB18D48A5AD4CD345)

> ![image](https://note.youdao.com/yws/res/5292/934CFFFEBB14476BA9C20D98EDEB2662)
