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
