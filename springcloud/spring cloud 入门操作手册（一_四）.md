> tedu.cn
> 
> 2019.4<br />
> spring boot 2.1.4<br />
> spring cloud Greenwich.SR1<br />

> STS 插件安装<br />
> http://note.youdao.com/noteshare?id=42c30d89f50ee57a54e013f2b8fc4fe5&sub=601E92B306E6412C81EAF36637A7029A

> lombok 插件安装<br />
> http://note.youdao.com/noteshare?id=45bc7db2ad61efd163281d2e5845e8dd&sub=DE9C5A3EE14F4645997E9FB91CD8D1BF

> maven settings.xml<br />
> http://note.youdao.com/noteshare?id=1cd70f3144281cce5e40c46386d2d747&sub=8C0D7F77FA1C4429B36554A9CE246D08

<span style="font-size: 50px">spring cloud 入门手册</span>

# 目录
[TOC]

---
# 一、service - 服务 

<div style="text-align: right"> 

[返回目录](#目录) 

</div>

![service-01](https://note.youdao.com/yws/res/5030/F6CE30C4522045BDAA15DD3856A674DE)
- 商品服务 item service，端口 8001
- 用户服务 user service，端口 8101
- 订单服务 order service，端口 8201


---
# 二、commons 通用项目
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

## 新建 maven 项目
> ![image](https://note.youdao.com/yws/res/5027/2734D117A28D4EEA881D42B0B63A13DD)


## pom.xml


```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.tedu</groupId>
	<artifactId>sp01-commons</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp01-commons</name>
	<dependencies>
		<dependency>
			<groupId>com.fasterxml.jackson.module</groupId>
			<artifactId>jackson-module-parameter-names</artifactId>
			<version>2.9.8</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.datatype</groupId>
			<artifactId>jackson-datatype-jdk8</artifactId>
			<version>2.9.8</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.datatype</groupId>
			<artifactId>jackson-datatype-jsr310</artifactId>
			<version>2.9.8</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.datatype</groupId>
			<artifactId>jackson-datatype-guava</artifactId>
			<version>2.9.8</version>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.6</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.26</version>
		</dependency>
		<dependency>
		    <groupId>org.apache.commons</groupId>
		    <artifactId>commons-lang3</artifactId>
		    <version>3.9</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.0</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

## java 源文件
> ![image](https://note.youdao.com/yws/res/5031/F28A6D0C77734359AFAC8F70172941F3)

### pojo

#### Item
```java
package com.tedu.sp01.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Item {
	private Integer id;
	private String name;
	private Integer number;
}
```

#### User
```java
package com.tedu.sp01.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
	private Integer id;
	private String username;
	private String password;
}
```

#### Order
```java
package com.tedu.sp01.pojo;

import java.util.ArrayList;
import java.util.List;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Order {
	private String id;
	private User user;
	private List<Item> items;
}
```
### service

#### ItemService
```java
package com.tedu.sp01.service;

import java.util.List;

import com.tedu.sp01.pojo.Item;

public interface ItemService {
	List<Item> getItems(String orderId);
	void decreaseNumbers(List<Item> list);
}
```
#### UserService
```java
package com.tedu.sp01.service;

import com.tedu.sp01.pojo.User;

public interface UserService {
	User getUser(Integer id);
	void addScore(Integer id, Integer score);
}

```

#### OrderService
```java
package com.tedu.sp01.service;


import com.tedu.sp01.pojo.Order;

public interface OrderService {
	Order getOrder(String orderId);
	void addOrder(Order order);
}

```

### util

#### CookieUtil
```java
package com.tedu.web.util;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CookieUtil {

	/**
	 * @param response
	 * @param name
	 * @param value
	 * @param maxAge
	 */
	public static void setCookie(HttpServletResponse response,
			String name, String value, String domain, String path, int maxAge) {
		Cookie cookie = new Cookie(name, value);
		if(domain != null) {
			cookie.setDomain(domain);
		}
		cookie.setPath(path);
		cookie.setMaxAge(maxAge);
		response.addCookie(cookie);
	}
	public static void setCookie(HttpServletResponse response, String name, String value, int maxAge) {
		setCookie(response, name, value, null, "/", maxAge);
	}
	public static void setCookie(HttpServletResponse response, String name, String value) {
		setCookie(response, name, value, null, "/", 3600);
	}
	public static void setCookie(HttpServletResponse response, String name) {
		setCookie(response, name, "", null, "/", 3600);
	}

	/**
	 * @param request
	 * @param name
	 * @return
	 */
	public static String getCookie(HttpServletRequest request, String name) {
		String value = null;
		Cookie[] cookies = request.getCookies();
		if (null != cookies) {
			for (Cookie cookie : cookies) {
				if (cookie.getName().equals(name)) {
					value = cookie.getValue();
				}
			}
		}
		return value;
	}

	/**
	 * @param response
	 * @param name
	 * @return
	 */
	public static void removeCookie(HttpServletResponse response, String name, String domain, String path) {
		setCookie(response, name, "", domain, path, 0);
	}

}
```

#### JsonUtil

```java
package com.tedu.web.util;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.Writer;
import java.math.BigDecimal;
import java.math.BigInteger;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.lang3.StringUtils;

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.fasterxml.jackson.datatype.guava.GuavaModule;
import com.fasterxml.jackson.datatype.jdk8.Jdk8Module;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class JsonUtil {
    private static ObjectMapper mapper;
    private static JsonInclude.Include DEFAULT_PROPERTY_INCLUSION = JsonInclude.Include.NON_DEFAULT;
    private static boolean IS_ENABLE_INDENT_OUTPUT = false;
    private static String CSV_DEFAULT_COLUMN_SEPARATOR = ",";
    static {
        try {
            initMapper();
            configPropertyInclusion();
            configIndentOutput();
            configCommon();
        } catch (Exception e) {
            log.error("jackson config error", e);
        }
    }

    private static void initMapper() {
        mapper = new ObjectMapper();
    }

    private static void configCommon() {
        config(mapper);
    }

    private static void configPropertyInclusion() {
        mapper.setSerializationInclusion(DEFAULT_PROPERTY_INCLUSION);
    }

    private static void configIndentOutput() {
        mapper.configure(SerializationFeature.INDENT_OUTPUT, IS_ENABLE_INDENT_OUTPUT);
    }

    private static void config(ObjectMapper objectMapper) {
        objectMapper.enable(JsonGenerator.Feature.WRITE_BIGDECIMAL_AS_PLAIN);
        objectMapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
        objectMapper.enable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY);
        objectMapper.enable(DeserializationFeature.FAIL_ON_READING_DUP_TREE_KEY);
        objectMapper.enable(DeserializationFeature.FAIL_ON_NUMBERS_FOR_ENUMS);
        objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        objectMapper.disable(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES);
        objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        objectMapper.enable(JsonParser.Feature.ALLOW_COMMENTS);
        objectMapper.disable(JsonGenerator.Feature.ESCAPE_NON_ASCII);
        objectMapper.enable(JsonGenerator.Feature.IGNORE_UNKNOWN);
        objectMapper.enable(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES);
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        objectMapper.enable(JsonParser.Feature.ALLOW_SINGLE_QUOTES);
        objectMapper.registerModule(new ParameterNamesModule());
        objectMapper.registerModule(new Jdk8Module());
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.registerModule(new GuavaModule());
    }
    public static void setSerializationInclusion(JsonInclude.Include inclusion) {
        DEFAULT_PROPERTY_INCLUSION = inclusion;
        configPropertyInclusion();
    }

    public static void setIndentOutput(boolean isEnable) {
        IS_ENABLE_INDENT_OUTPUT = isEnable;
        configIndentOutput();
    }

    public static <V> V from(URL url, Class<V> c) {
        try {
            return mapper.readValue(url, c);
        } catch (IOException e) {
            log.error("jackson from error, url: {}, type: {}", url.getPath(), c, e);
            return null;
        }
    }

    public static <V> V from(InputStream inputStream, Class<V> c) {
        try {
            return mapper.readValue(inputStream, c);
        } catch (IOException e) {
            log.error("jackson from error, type: {}", c, e);
            return null;
        }
    }

    public static <V> V from(File file, Class<V> c) {
        try {
            return mapper.readValue(file, c);
        } catch (IOException e) {
            log.error("jackson from error, file path: {}, type: {}", file.getPath(), c, e);
            return null;
        }
    }

    public static <V> V from(Object jsonObj, Class<V> c) {
        try {
            return mapper.readValue(jsonObj.toString(), c);
        } catch (IOException e) {
            log.error("jackson from error, json: {}, type: {}", jsonObj.toString(), c, e);
            return null;
        }
    }

    public static <V> V from(String json, Class<V> c) {
        try {
            return mapper.readValue(json, c);
        } catch (IOException e) {
            log.error("jackson from error, json: {}, type: {}", json, c, e);
            return null;
        }
    }

    public static <V> V from(URL url, TypeReference<V> type) {
        try {
            return mapper.readValue(url, type);
        } catch (IOException e) {
            log.error("jackson from error, url: {}, type: {}", url.getPath(), type, e);
            return null;
        }
    }

    public static <V> V from(InputStream inputStream, TypeReference<V> type) {
        try {
            return mapper.readValue(inputStream, type);
        } catch (IOException e) {
            log.error("jackson from error, type: {}", type, e);
            return null;
        }
    }

    public static <V> V from(File file, TypeReference<V> type) {
        try {
            return mapper.readValue(file, type);
        } catch (IOException e) {
            log.error("jackson from error, file path: {}, type: {}", file.getPath(), type, e);
            return null;
        }
    }

    public static <V> V from(Object jsonObj, TypeReference<V> type) {
        try {
            return mapper.readValue(jsonObj.toString(), type);
        } catch (IOException e) {
            log.error("jackson from error, json: {}, type: {}", jsonObj.toString(), type, e);
            return null;
        }
    }

    public static <V> V from(String json, TypeReference<V> type) {
        try {
            return mapper.readValue(json, type);
        } catch (IOException e) {
            log.error("jackson from error, json: {}, type: {}", json, type, e);
            return null;
        }
    }

    public static <V> String to(List<V> list) {
        try {
            return mapper.writeValueAsString(list);
        } catch (JsonProcessingException e) {
            log.error("jackson to error, obj: {}", list, e);
            return null;
        }
    }

    public static <V> String to(V v) {
        try {
            return mapper.writeValueAsString(v);
        } catch (JsonProcessingException e) {
            log.error("jackson to error, obj: {}", v, e);
            return null;
        }
    }

    public static <V> void toFile(String path, List<V> list) {
        try (Writer writer = new FileWriter(new File(path), true)) {
            mapper.writer().writeValues(writer).writeAll(list);
            writer.flush();
        } catch (Exception e) {
            log.error("jackson to file error, path: {}, list: {}", path, list, e);
        }
    }

    public static <V> void toFile(String path, V v) {
        try (Writer writer = new FileWriter(new File(path), true)) {
            mapper.writer().writeValues(writer).write(v);
            writer.flush();
        } catch (Exception e) {
            log.error("jackson to file error, path: {}, obj: {}", path, v, e);
        }
    }

    public static String getString(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).toString();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get string error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static Integer getInt(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).intValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get int error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static Long getLong(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).longValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get long error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static Double getDouble(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).doubleValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get double error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static BigInteger getBigInteger(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return new BigInteger(String.valueOf(0.00));
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).bigIntegerValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get biginteger error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static BigDecimal getBigDecimal(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).decimalValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get bigdecimal error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static boolean getBoolean(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return false;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).booleanValue();
            } else {
                return false;
            }
        } catch (IOException e) {
            log.error("jackson get boolean error, json: {}, key: {}", json, key, e);
            return false;
        }
    }

    public static byte[] getByte(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        try {
            JsonNode node = mapper.readTree(json);
            if (null != node) {
                return node.get(key).binaryValue();
            } else {
                return null;
            }
        } catch (IOException e) {
            log.error("jackson get byte error, json: {}, key: {}", json, key, e);
            return null;
        }
    }

    public static <T> ArrayList<T> getList(String json, String key) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        String string = getString(json, key);
        return from(string, new TypeReference<ArrayList<T>>() {});
    }

    public static <T> String add(String json, String key, T value) {
        try {
            JsonNode node = mapper.readTree(json);
            add(node, key, value);
            return node.toString();
        } catch (IOException e) {
            log.error("jackson add error, json: {}, key: {}, value: {}", json, key, value, e);
            return json;
        }
    }

    private static <T> void add(JsonNode jsonNode, String key, T value) {
        if (value instanceof String) {
            ((ObjectNode) jsonNode).put(key, (String) value);
        } else if (value instanceof Short) {
            ((ObjectNode) jsonNode).put(key, (Short) value);
        } else if (value instanceof Integer) {
            ((ObjectNode) jsonNode).put(key, (Integer) value);
        } else if (value instanceof Long) {
            ((ObjectNode) jsonNode).put(key, (Long) value);
        } else if (value instanceof Float) {
            ((ObjectNode) jsonNode).put(key, (Float) value);
        } else if (value instanceof Double) {
            ((ObjectNode) jsonNode).put(key, (Double) value);
        } else if (value instanceof BigDecimal) {
            ((ObjectNode) jsonNode).put(key, (BigDecimal) value);
        } else if (value instanceof BigInteger) {
            ((ObjectNode) jsonNode).put(key, (BigInteger) value);
        } else if (value instanceof Boolean) {
            ((ObjectNode) jsonNode).put(key, (Boolean) value);
        } else if (value instanceof byte[]) {
            ((ObjectNode) jsonNode).put(key, (byte[]) value);
        } else {
            ((ObjectNode) jsonNode).put(key, to(value));
        }
    }

    public static String remove(String json, String key) {
        try {
            JsonNode node = mapper.readTree(json);
            ((ObjectNode) node).remove(key);
            return node.toString();
        } catch (IOException e) {
            log.error("jackson remove error, json: {}, key: {}", json, key, e);
            return json;
        }
    }

    public static <T> String update(String json, String key, T value) {
        try {
            JsonNode node = mapper.readTree(json);
            ((ObjectNode) node).remove(key);
            add(node, key, value);
            return node.toString();
        } catch (IOException e) {
            log.error("jackson update error, json: {}, key: {}, value: {}", json, key, value, e);
            return json;
        }
    }

    public static String format(String json) {
        try {
            JsonNode node = mapper.readTree(json);
            return mapper.writerWithDefaultPrettyPrinter().writeValueAsString(node);
        } catch (IOException e) {
            log.error("jackson format json error, json: {}", json, e);
            return json;
        }
    }

    public static boolean isJson(String json) {
        try {
            mapper.readTree(json);
            return true;
        } catch (Exception e) {
            log.error("jackson check json error, json: {}", json, e);
            return false;
        }
    }

    private static InputStream getResourceStream(String name) {
        return JsonUtil.class.getClassLoader().getResourceAsStream(name);
    }

    private static InputStreamReader getResourceReader(InputStream inputStream) {
        if (null == inputStream) {
            return null;
        }
        return new InputStreamReader(inputStream, StandardCharsets.UTF_8);
    }
}
```

#### JsonResult

```java
package com.tedu.web.util;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class JsonResult<T> {
	/** 成功 */
	public static final int SUCCESS = 200;

	/** 没有登录 */
	public static final int NOT_LOGIN = 400;

	/** 发生异常 */
	public static final int EXCEPTION = 401;

	/** 系统错误 */
	public static final int SYS_ERROR = 402;

	/** 参数错误 */
	public static final int PARAMS_ERROR = 403;

	/** 不支持或已经废弃 */
	public static final int NOT_SUPPORTED = 410;

	/** AuthCode错误 */
	public static final int INVALID_AUTHCODE = 444;

	/** 太频繁的调用 */
	public static final int TOO_FREQUENT = 445;

	/** 未知的错误 */
	public static final int UNKNOWN_ERROR = 499;
	
	private int code;
	private String msg;
	private T data;
	
	

	public static JsonResult build() {
		return new JsonResult();
	}
	public static JsonResult build(int code) {
		return new JsonResult().code(code);
	}
	public static JsonResult build(int code, String msg) {
		return new JsonResult<String>().code(code).msg(msg);
	}
	public static <T> JsonResult<T> build(int code, T data) {
		return new JsonResult<T>().code(code).data(data);
	}
	public static <T> JsonResult<T> build(int code, String msg, T data) {
		return new JsonResult<T>().code(code).msg(msg).data(data);
	}
	
	public JsonResult<T> code(int code) {
		this.code = code;
		return this;
	}
	public JsonResult<T> msg(String msg) {
		this.msg = msg;
		return this;
	}
	public JsonResult<T> data(T data) {
		this.data = data;
		return this;
	}
	
	
	public static JsonResult ok() {
		return build(SUCCESS);
	}
	public static JsonResult ok(String msg) {
		return build(SUCCESS, msg);
	}
	public static <T> JsonResult<T> ok(T data) {
		return build(SUCCESS, data);
	}
	public static JsonResult err() {
		return build(EXCEPTION);
	}
	public static JsonResult err(String msg) {
		return build(EXCEPTION, msg);
	}
	
	@Override
	public String toString() {
		return JsonUtil.to(this);
	}
}
```

---
# 三、item service 商品服务
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

1. 新建项目
2. 配置依赖 pom.xml
3. 配置 application.yml
4. ~~配置主程序~~
5. 编写代码


## 新建 spring boot 起步项目
> ![image](https://note.youdao.com/yws/res/4984/C61483A84EF44710A733E6CA0FA1180F)

## 选择依赖项
- 只选择 web

> ![image](https://note.youdao.com/yws/res/5008/8F524AA83345448E8AD4374AF841C43D)

## pom.xml

- ==要填加 sp01-commons 项目依赖==

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
	<artifactId>sp02-itemservice</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp02-itemservice</name>
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

## application.yml

> ![image](https://note.youdao.com/yws/res/5359/B4C8864A72864DABB877F4E5D2C366DE)

```yml
spring:
  application:
    name: item-service
    
server:
  port: 8001
```

## 主程序
- 默认代码，不需要修改
```java
package com.tedu.sp02;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Sp02ItemserviceApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp02ItemserviceApplication.class, args);
	}

}
```

## java 源文件
> ![image](https://note.youdao.com/yws/res/5023/EB26694475A0438BA623AD04E6DC20CC)

### ItemServiceImpl

```java
package com.tedu.sp02.item.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Service;

import com.tedu.sp01.pojo.Item;
import com.tedu.sp01.service.ItemService;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class ItemServiceImpl implements ItemService {

	@Override
	public List<Item> getItems(Integer orderId) {
		ArrayList<Item> list = new ArrayList<Item>();
		list.add(new Item(1, "商品 1",1));
		list.add(new Item(2, "商品 2",2));
		list.add(new Item(3, "商品 3",3));
		list.add(new Item(4, "商品 4",4));
		list.add(new Item(5, "商品 5",5));
		return list;
	}

	@Override
	public void decreaseNumbers(List<Item> list) {
		if (log.isInfoEnabled()) {
			for(Item item : list) {
				log.info("减少库存 - "+item);
			}
		}
	}
}
```

### ItemController
- 如果存在 jackson-dataformat-xml 依赖，会根据请求协议头，可能返回 xml 或 json；可以强制返回 json: <br />==produces=MediaType.APPLICATION_JSON_UTF8_VALUE==

```java
package com.tedu.sp02.item.controller;

import java.util.List;

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
	public JsonResult<List<Item>> getItems(@PathVariable String orderId) {
		log.info("server.port="+port+", orderId="+orderId);
		
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









---
# 四、user service 用户服务
<div style="text-align: right"> 

[返回目录](#目录) 

</div>

1. 新建项目
2. 配置依赖 pom.xml
3. 配置 application.yml
4. ~~配置主程序~~
5. 编写代码


## 新建 spring boot 起步项目
> ![image](https://note.youdao.com/yws/res/4982/756269E32264434BA6A68233B4812041)

## 选择依赖项
- 只选择 web

> ![image](https://note.youdao.com/yws/res/5008/8F524AA83345448E8AD4374AF841C43D)

## pom.xml
- 要填加 sp01-commons 项目依赖

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
	<artifactId>sp03-userservice</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>sp03-userservice</name>
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
## application.yml

- 其中 `sp.user-service.users` 属性为自定义属性，提供用于测试的用户数据

```yml
sp:
  user-service:
    users: "[{\"id\":7, \"username\":\"abc\",\"password\":\"123\"},{\"id\":8, \"username\":\"def\",\"password\":\"456\"},{\"id\":9, \"username\":\"ghi\",\"password\":\"789\"}]"

spring:
  application:
    name: user-service
    
server:
  port: 8101
```

## 主程序

- 默认代码，不需要修改

```java
package com.tedu.sp03;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Sp03UserserviceApplication {

	public static void main(String[] args) {
		SpringApplication.run(Sp03UserserviceApplication.class, args);
	}

}
```

## java源文件

> ![image](https://note.youdao.com/yws/res/5018/DD2AD0535F14481E8C919CD0C4E47D59)

### UserServiceImpl

```java
package com.tedu.sp03.user.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import com.fasterxml.jackson.core.type.TypeReference;
import com.tedu.sp01.pojo.User;
import com.tedu.sp01.service.UserService;
import com.tedu.web.util.JsonUtil;

import lombok.extern.slf4j.Slf4j;

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

### UserController

- 如果存在 jackson-dataformat-xml 依赖，会根据请求协议头，可能返回 xml 或 json；可以强制返回 json: <br />==produces=MediaType.APPLICATION_JSON_UTF8_VALUE==

```java
package com.tedu.sp03.user.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import com.tedu.sp01.pojo.User;
import com.tedu.sp01.service.UserService;
import com.tedu.web.util.JsonResult;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class UserController {
	
	@Autowired
	private UserService userService;
	
	@GetMapping("/{userId}")
	public JsonResult<User> getUser(@PathVariable Integer userId) {
		log.info("get user, userId="+userId);
		User u = userService.getUser(userId);
		return JsonResult.ok(u);
	}
	
	@GetMapping("/{userId}/score") 
	public JsonResult addScore(
			@PathVariable Integer userId, Integer score) {
		userService.addScore(userId, score);
		return JsonResult.ok();
	}
}

```
