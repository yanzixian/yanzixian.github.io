---
title: swagger2
date: 2020-01-13 13:46:42
categories: spring
tags: swagger2
---

#### 概述

swagger2用于快速生成api接口文档，也支持在线接口测试，不依赖第三方工具

#### SpringBoot集成Swagger2

##### pom.xml添加maven依赖

```xml
<dependencies>
    ...
    <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
 <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
    <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger2</artifactId>
       <version>2.9.2</version>
    </dependency>
</dependencies>
```

##### 创建swagger2配置类，Swagger2Config.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
​
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
​
@Configuration  
@EnableSwagger2
public class Swagger2Config {
​
   //api接口包扫描路径,即controller类所在文件夹
   public static final String SWAGGER_SCAN_BASE_PACKAGE = "com.***.controller";
​
   public static final String VERSION = "1.0.0";
​
   @Bean
   public Docket createRestApi() {
       return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
.apis(RequestHandlerSelectors.basePackage(SWAGGER_SCAN_BASE_PACKAGE)) 
                   .paths(PathSelectors.any()) // 可以根据url路径设置哪些请求加入文档，忽略哪些请求
                   .build();
   }
​
   private ApiInfo apiInfo() {
       Contact contact=new Contact("联系人","网址","另一网址") 	
       return new ApiInfoBuilder()
                   .title("文档标题") //设置文档的标题
                   .description("文档描述") // 设置文档的描述
           		   .contact
                   .version(VERSION) // 设置文档的版本信息-> 1.0.0 Version information
                   .termsOfServiceUrl("http://www.baidu.com") // 设置文档的License信息->1.3 License information
                   .build();
   }
}
```

##### 使用注解编写接口文档

###### 常用注解使用

- @**Api**

  用于修饰整个Controller类，描述当前Controller的作用

  属性：`value="描述此类，似乎没用",tags="描述此类，将会显示在文档页面中"`

- @**ApiOperation**

  注解在Controller类的方法上，即一个接口上，用于描述此接口

  属性：`value="说明接口的用途、作用",notes="接口的补充说明"`

- @**ApiImpIicitParam**

  注解在Controller类的方法上，描述一个参数的各个方面

  属性：`name="参数名",value="参数的描述，解释",required="参数是否必须要传，默认为false",paramType="参数位于哪个地方，有header--->(请求参数的获取,@RequestHeader)、query-->(请求参数的获取,@RequestParam)、path--->(请求参数的获取,@PathVariable)、body(不常用)、form(不常用)"，dataType="参数类型，默认为String",defaultValue="参数的默认值"`
  
- @**ApiImplicitParams**
  
  注解在Controller类的方法上，描述多个参数，有多个@ApiImplicitParam构成
  
  如：
  
  ```java
  @ApiImplicitParams(
  {
  	@ApiImplicitParam(...),
  	@ApiImplicitParam(...),
  	...
  	...
  }
  )
  ```

- @**ApiResponse**

  注解在Controller类的方法上，描述一个请求响应

  属性：`code="一个整数，http的状态码，如400",message="返回信息，例如'请求参数错误'",response="抛出异常的类"`

- @**ApiResponses**

  注解在Controller类的方法上，描述一组请求响应

  如：

  ```java
  @ApiResponses(
  {
  	@ApiResponse(...),
  	@ApiResponse(...),
  	...
  }
  )
  ```
  
- @**ApiModelProperty**
  
  用在字段属性上，描述实体类字段的属性，即用在JavaBean类的属性上面，说明属性的含义
  
  属性：`value="字段说明",name="字段名",dataType="属性类型",requried=是否必填，默认false,example="举例说明",hidden=是否隐藏,默认false`

- @**ApiModel**

  - 当请求数据描述，即 `@RequestBody` 时， 用于封装请求（包括数据的各种校验）数据。

    ```java
    @ApiModel(description = "用户登录")
    public class UserLoginVO implements Serializable {
    
    	private static final long serialVersionUID = 1L;
    
    	@ApiModelProperty(value = "用户名",required=true)	
    	private String username;
    
    	@ApiModelProperty(value = "密码",required=true)	
    	private String password;
    	
    	// getter/setter省略
    }
    
    ```

    ```
    @Api(tags="用户模块")
    @Controller
    public class UserController {
    
    	@ApiOperation(value = "用户登录", notes = "")	
    	@PostMapping(value = "/login")
    	//请求参数为一个UserLoginVO对象
    	public R login(@RequestBody UserLoginVO userLoginVO) {
    		User user=userSerivce.login(userLoginVO);
    		return R.okData(user);
    	}
    }
    ```

    

  - 当响应值是对象时，即 `@ResponseBody` 时，用于返回值对象的描述。

  ```java
  @ApiModel(description= "返回响应数据")
  public class RestMessage implements Serializable{
  
  	@ApiModelProperty(value = "是否成功",required=true)
  	private boolean success=true;	
  	
  	@ApiModelProperty(value = "错误码")
  	private Integer errCode;
  	
  	@ApiModelProperty(value = "提示信息")
  	private String message;
  	
      @ApiModelProperty(value = "数据")
  	private Object data;
  		
  	/* getter/setter 略*/
  }
  ```

  [传送门](https://blog.csdn.net/xiaojin21cen/article/details/78654652)

- **@ApiIngore**

  注解在Controller类的方法上，表示忽略此方法

##### 访问swagger文档

启动springboot项目，访问 http://localhost:8080/swagger-ui.html

