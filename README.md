#  maven 创建springBoot项目  
https://start.spring.io/

# Spring Boot 选择外部Tomcat运行，打war包的修改流程
参照 https://blog.csdn.net/limenghua9112/article/details/81170286
1  修改pom的packaging为war
```
<packaging>war</packaging>
```

2 增加下面依赖覆盖内嵌的Tomcat依赖
```
<!-- 以下为内嵌的Tomcat依赖 注释掉 -->
<!-- <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency> -->
    
<!-- 以下为外部的Tomcat依赖 -->    
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

3 增加SpringBootServletInitializer的子类
在原启动类的同包下创建WebInitializer.java 替换Application.class原来的启动类

```
 /**
 * 使用外部tomcat来启动项目
 */
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.web.SpringBootServletInitializer;

public class WebInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```
4 在eclipse下 项目上右键 - export- WAR file 然后导出到tomcat的webapps文件夹下, 即生成项目的WAR文件（例如springboot520.war）
在浏览器输入http://localhost:3000/springboot520/ 即可打开

5 遇到的问题 放到外部tomcat 无法访问controller 至今无解

# 关闭占用特定端口
打开eclipse的 默认打开了tomcat服务器 并占用了一个端口号8080，如果想运行springboot项目，提示端口被占用。
解决办法：关闭端口程序。
1 比如关闭占用8080的
```
netstat -aon|findstr 8080

```
2 找到进程号 终止这个进程
```
taskkill /f /t /im 9260
```

# 序列化精解 
JavaOSSocketCC++   
1、序列化是干什么的？  
       简单说就是为了保存在内存中的各种对象的状态（也就是实例变量，不是方法），并且可以把保存的对象状态再读出来。虽然你可以用你自己的各种各样的方法来保存object states，但是Java给你提供一种应该比你自己好的保存对象状态的机制，那就是序列化。
  
2、什么情况下需要序列化   
    a）当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；  
    b）当你想用套接字在网络上传送对象的时候；  
    c）当你想通过RMI传输对象的时候；  

3、当对一个对象实现序列化时，究竟发生了什么？  
    在没有序列化前，每个保存在堆（Heap）中的对象都有相应的状态（state），即实例变量（instance ariable）比如：  
   
java 代码  
Foo  myFoo = new Foo();    
myFoo .setWidth(37);    
myFoo.setHeight(70);   
      
       当 通过下面的代码序列化之后，MyFoo对象中的width和Height实例变量的值（37，70）都被保存到foo.ser文件中，这样以后又可以把它 从文件中读出来，重新在堆中创建原来的对象。当然保存时候不仅仅是保存对象的实例变量的值，JVM还要保存一些小量信息，比如类的类型等以便恢复原来的对 象。
java 代码  
```
FileOutputStream fs = new FileOutputStream("foo.ser");    
ObjectOutputStream os = new ObjectOutputStream(fs);    
os.writeObject(myFoo); 
``` 

4、实现序列化（保存到一个文件）的步骤
```
       a）Make a FileOutputStream            
java 代码
FileOutputStream fs = new FileOutputStream("foo.ser");    
       b）Make a ObjectOutputStream            
java 代码
ObjectOutputStream os =  new ObjectOutputStream(fs);   
       c）write the object
java 代码
os.writeObject(myObject1);  
os.writeObject(myObject2);  
os.writeObject(myObject3);  
    d) close the ObjectOutputStream
java 代码
os.close();  
```


5、举例说明
java 代码
```
import java.io.*;
  
public class  Box implements Serializable  
{  
    private int width;  
    private int height;  
  
    public void setWidth(int width){  
        this.width  = width;  
    }  
    public void setHeight(int height){  
        this.height = height;  
    }  
  
    public static void main(String[] args){  
        Box myBox = new Box();  
        myBox.setWidth(50);  
        myBox.setHeight(30);  
  
        try{  
            FileOutputStream fs = new FileOutputStream("foo.ser");  
            ObjectOutputStream os =  new ObjectOutputStream(fs);  
            os.writeObject(myBox);  
            os.close();  
        }catch(Exception ex){  
            ex.printStackTrace();  
        }  
    }  
      
}  
```

6、相关注意事项  
    a）序列化时，只对对象的状态进行保存，而不管对象的方法；  
    b）当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；  
    c）当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；  
    d）并非所有的对象都可以序列化，,至于为什么不可以，有很多原因了,比如：  
        1.安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行rmi传输  等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的。  
       2. 资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分  配，而且，也是没有必要这样实现。  
# 搭建spring-boot项目报错Error parsing lifecycle processing instructions
问题及其产生背景

刚开始学习搭建spring-boot项目，这里遇到一个问题，花了一点时间，现在把它记录下来。

新建完maven项目之后，在向pom.xml文件添加parent节点（内容如下）时文件报错了。
```
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>1.4.2.RELEASE</version>
</parent>
```

报错的信息如下：

Error parsing lifecycle processing instructions

解决办法  

根据问题进行搜索，结果搜索到了两种思路：  

1.说是neon这个版本的eclipse有问题，需要更新maven插件。  
 
2.说是依赖冲突了，可以吧maven仓库清空，重新更新一下。  

试了第一种方法，但是并没有解决这个问题。所以试了第二种方法，将用户主目录下.m2/repository/下的依赖全部清空。然后，在eclipse中右键项目 --> Maven --> Update Project...。等待更新完成后，错误自动消失了。  

# 使用springboot模板快速搭建springboot项目
1 进入https://start.spring.io/ 选择相应的版本 下载；
特别注意版本之间的关系
springboot版本使用2.0.2  
对应jdk版本适用1.8  
maven如果过高，如果出现报错，可以把版本改为2.0.2.RELEASE（注意看错误提示 注意修改eclipse关联的maven安装配置）  
###版本一定要对应上 一定 一定

2 在eclipse中 选择import，选择下载的项目根目录；  
3 导入后，打开paqiang工具，下载依赖。如果报错，可以考虑把parent的版本号降低（比如改为2.0.2.RELEASE 或更低版本1.5.2.RELEASE ），然后重新下载依赖
```
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```
4 重新下载以来后 要更新maven project
5 如果还是保持，可以进入C:\Users\Administrator\.m2\repository，把下载的依赖全部删掉，然后重新安装。安装的时候，记得开启paqiang

# Spring boot项目中自定义的controller不生效的解决办法  
问题描述：
例如 Springboot0610Application.java包是在包com.abeng.east.springboot0610里，如果在此包下新建一个controller，没问题。但是如果新建一个子包：  
com.abeng.east.springboot0610.controller  
在此包下新建一个controller，在未定义错误拦截的情况下，则会出现找不到此controller的页面。  
解决办法：  
在applications类中增加一行注解，意思是扫描该包下及子包下的所有controller类  
@ComponentScan("com.abeng.east.springboot0610")  
com.abeng.east.springboot0610为父包  
```
package com.abeng.east.springboot0610;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan("com.abeng.east.springboot0610")
public class Springboot0610Application {

	public static void main(String[] args) {
		SpringApplication.run(Springboot0610Application.class, args);
	}

}
```

# springboot 文件上传例子
### 前后端不分离
https://o7planning.org/en/11679/spring-boot-file-upload-example 
@ModelAttribute使用详解  
https://blog.csdn.net/li_xiao_ming/article/details/8349115  
例子：  
在themeleaf+springboot中实现表单的数据传递。  
```
新建实体类Greeting
新建Controller类GreetingController
新建html：greeting.html
新建html：result.html
知道GetMapping/PostMapping
当提交form时 GetMapping将对象传入form
form将数据封装进对象th:object=”${greeting}”
th:action=”@{/greeting}”将对象传入指定页面result.html
result.html取出对象属性并显示
注意：引入 不然 th报错。
```

```
//使用默认构造方法
public class Greeting {
    private long id;
    private String content;
    public long getId() {
        return id;
    }
    public void setId(long id) {
        this.id = id;
    }
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
    }

}
```

```
//三个Mapping
@Controller
public class GreetingController {

    //传入greeting对象实现封装
    @GetMapping("/greeting")
    public String greetingForm(Model model) {
        model.addAttribute("greeting", new Greeting());
        return "greeting";
    }
    //将封装的对象转入result.html
    @PostMapping("/greeting")
    public String greetingSubmit(@ModelAttribute Greeting greeting) {
        return "result";
    }

}
```
```
<!-- greeting.html -->
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<h1>Form</h1>
<!-- action:greeting 先通过GetMapping得到对象用于封装数据，在通过PostMapping传递数据 -->
<form action="#" th:action="@{/greeting}" th:object="${greeting}" method="post">
    <p>Id: <input type="text" th:field="*{id}" /></p>
    <p>Message: <input type="text" th:field="*{content}" /></p>
    <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
</form>
</body>
</html>
```

```
<!-- result.html -->
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<h1>Result</h1>
<p th:text="'id: ' + ${greeting.id}" />
<p th:text="'content: ' + ${greeting.content}" />
<a href="/greeting">Submit another message</a>
</body>
</html>
```
### 前后端分离的文件上传
https://blog.csdn.net/oppo5630/article/details/79318715

### 前后端分类SpringBoot实现表单数据与文件同时上传
https://blog.csdn.net/qq_30401609/article/details/82384766


# OKHttp中设置请求头 类似postman中发送get请求 带authorization认证
参考 https://www.jianshu.com/p/cdab05b87a9d  
```
OKHttp中设置请求头特别简单，在创建request对象时调用一个方法即可。 
使用示例如下：

Request request = new Request.Builder()
                .url("http://www.baidu.com")
                .header("User-Agent", "OkHttp Headers.java")
                .addHeader("token", "myToken")
                .build();
```
```
MultipartBody 可以构建复杂的请求体，与HTML文件上传形式兼容。
多块请求体中每块请求都是一个请求体，可以定义自己的请求头，这些请求头可以用来描述这块请求。
例如他的Content-Disposition。如果Content-Length和Content-Type可用的话，他们会被自动添加到请求头中。

private static final String IMGUR_CLIENT_ID = "...";
private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");
private void postMultipartBody() {
    OkHttpClient client = new OkHttpClient();
    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
    MultipartBody body = new MultipartBody.Builder("AaB03x")
            .setType(MultipartBody.FORM)
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"title\""),
                    RequestBody.create(null, "Square Logo"))
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"image\""),
                    RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
            .build();

    Request request = new Request.Builder()
            .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
            .url("https://api.imgur.com/3/image")
            .post(body)
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println(response.body().string());

        }
    });
}
```

# okHttp发送post请求和get请求 失效（解决办法见下一条）
```
package com.abeng.east.springboot0610;

import java.io.IOException;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;

@RestController
public class Hello {
	@RequestMapping("/test1")
	public String sayHello() {
		// 发送post请求
		
		/*

		OkHttpClient client = new OkHttpClient();
		String jsonStr = "{'abeng': 'haha'}";
		
		MediaType JSON = MediaType.get("application/json; charset=utf-8");
		RequestBody body = RequestBody.create(jsonStr, JSON);

		Request request = new Request.Builder()
				.url("https://www.baidu.com")
				.post(body)
				.build();

		try (Response response = client.newCall(request).execute()) {
			return response.body().string();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		*/
		
		// 发送get请求

		OkHttpClient client = new OkHttpClient();
				
		MediaType JSON = MediaType.get("application/json; charset=utf-8");
		
		Request request = new Request.Builder()
				.url("https://api.github.com/user")
				.header("Authorization", "XXXXXXXXX") // 问题点
				.get()
				.build();

		try (Response response = client.newCall(request).execute()) {
			return response.body().string();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}


		return null;
	}
}

```
# Create bearer authorization header in OkHttp java 创建bearer authorization请求头 携带access token获取用户信息的最佳实践
.addHeader("Authorization", "Bearer " + "cd4949a7ef27fb5c856a28dde39460a53632ead6")  
https://stackoverflow.com/questions/49463607/create-bearer-authorization-header-in-okhttp-java  
```
package com.abeng.east.springboot0610;

import java.io.IOException;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import okhttp3.Credentials;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;

@RestController
public class Hello {
	@RequestMapping("/test1")
	public String sayHello() {
		// 发送get请求
				

		OkHttpClient client = new OkHttpClient();
		
		
		
		// MediaType JSON = MediaType.get("application/json; charset=utf-8");
		// RequestBody body = RequestBody.create(jsonStr, JSON);
		
		String credential = Credentials.basic("xxxx", "xxxxxxx");

		Request request = new Request.Builder()
				.url("https://api.github.com/user")
				.addHeader("Authorization", "Bearer " + "XXXXXXXXXXXXXXXX")
				.get()
				.build();

		try (Response response = client.newCall(request).execute()) {
			return response.body().string();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		
		

		return null;
	}
}

```

# Thymeleaf 模板的使用
https://www.jianshu.com/p/ed9d47f92e37

# 使用热部署 就是修改了类或配置文件以后，不用手工启动服务，springboot提供了spring-boot-devtolls， 能够在文件变化的时候自动加载spring boot应用。
```
        <!-- optional=true,依赖不会传递，该项目依赖devtools；之后依赖myboot项目的项目如果想要使用devtools，需要重新引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

# idea中热部署失效的问题
1 添加上面的依赖  
2 setting - build -compiler -  Build Project automatically 勾选  
3 然后 Shift+Ctrl+Alt+/，选择Registry，compiler.automake.allow.when.app.running勾选
4 重新启动即可

# thymleaf在页面中无法获取session数据（本来已经设置好了）报错
原因是：如果我们添加Thymeleaf 依赖，，而没有进行任何配置，或者添加默认目录，启动应用时就会报错。
```
#thymeleaf 配置
＃模板的模式，支持 HTML, XML TEXT JAVASCRIPT
spring.thymeleaf .mode=HTMLS
＃编码 可不用配置
spri 口g thymeleaf encoding=UTF
＃内容类别，可不用配置
spring thymeleaf conte type text/html
＃开发配置为 false ，避免修改模板还要重启服务器
spring.thymeleaf.cache=false
＃配置模板路径，默认是 templates ，可以不用配置
#spring . thymeleaf.prefix=classpath : /templates/ 
```
# thymeleaf模板引擎在没数据的情况下报错 解决办法
类似vue的v-if 要先判断数据是否存在
```
// 正确的方法
<p th:if="${session.user!=null}"  th:text="${session.user.getName()}"></p>
```
如果只像以下这样写，刚开始的时候session.user是不存在的 所以不能执行这个方法 session.user.getName() 所以会报错
```
<p th:text="${session.user.getName()}"></p>
```
# 一个springboot mysql最佳案例
http://zetcode.com/springboot/mysql/

# 一个springboot mybatis mysql的最佳案例  
https://www.cnblogs.com/wangshen31/p/8744157.html

# spring boot导入init工程后 报错 RequestMapping cannot be resolved to a type  
没有引入controller 路由开发的依赖 在pom文件中加入依赖 ，成功解决
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

# Spring Boot 启动错误：To display the conditions report re-run your application with 'debug' enable
原因 在pom文件中引入了mysql依赖 但是没有配置数据库信息  
解决办法：引入数据库配置信息

```
spring.datasource.url=us-cdbr-iron-east-02.cleardb.net
spring.datasource.username=b05ea9c45afa32
spring.datasource.password=5c2a2abd
spring.datasource.driver-class-name: com.mysql.jdbc.Driver
```
# mybatis绑定错误-- Invalid bound statement (not found)  
https://www.jianshu.com/p/800fe918cc7a

# 出现问题 报错
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: No operations allowed after connection closed.
见https://blog.csdn.net/Jack__iT/article/details/80467286
暂时的解决办法
可在mysql的url中加入autoReconnect=true，这样就可以解决。

# 出现问题 报错
java.lang.NullPointerException
先判断对象存不存在 然后再调用该对象的方法 如果一上来就调用，在对象都不存在的情况下 就会报错

# 增加拦截器
https://www.jianshu.com/p/fe7428e2e5b0
1：创建拦截器类DuckTestInterceptor，实现HandlerInterceptor接口  
```
package com.ui.toto.toto.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.jboss.logging.Logger;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

public class DuckTestInterceptor extends HandlerInterceptorAdapter {
	private static final Logger logger = Logger.getLogger(DuckTestInterceptor.class);
	
	@Override
	public boolean preHandle(HttpServletRequest request,
							 HttpServletResponse response,
							 Object handler) {
		logger.info("================ Before Method");
		return true;
	}
	
	@Override
	public void postHandle( HttpServletRequest request,
							HttpServletResponse response,
							Object handler,
							ModelAndView modelAndView) {
		logger.info("================ Method Executed");
	}
	
	@Override
	public void afterCompletion(HttpServletRequest request,
								HttpServletResponse response, 
								Object handler, 
								Exception ex) {
		logger.info("================ Method Completed");
	}

}

```

2：创建java类InterceptorRegister，继承WebMvcConfigurerAdapter，重写addInterceptors方法  
实例化拦截器类并将其添加到拦截器链中。  
```
package com.ui.toto.toto.interceptor;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

@Configuration
public class InterceptorRegister extends WebMvcConfigurationSupport {
	@Bean
	public HandlerInterceptor getMyInterceptor() {
		return new DuckTestInterceptor();
	}

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(getMyInterceptor()).addPathPatterns("/**");
	}

}

```
3：访问http://localhost:8080/freemarker可见日志，后台已经拦截到

# spring boot使用拦截器(以session校验为例)
https://blog.csdn.net/weixin_37891479/article/details/79530669  

# spring boot配置过滤器
Spring Boot 过滤器的实现  https://cloud.tencent.com/developer/article/1362809

# spring boot设置跨域请求
https://www.cnblogs.com/yuansc/p/9076604.html
https://www.jianshu.com/p/470de10b9eec

# spring boot统一错误处理
http://how2j.cn/k/springboot/springboot-error/1643.html#nowhere

# springboot集成ueditor富文本编辑器（不需修改ueditor源码）
https://www.jianshu.com/p/006e65711de0

# 事务管理
比如插入2条数据 A和B
如果插入A成功，插入B不成功，则插入A 也不成功。这就是事务。
```
@Transactional
public void insertTwo () {
	// insert A
	
	// insert B
}
```
注意 如果该方法只有一个任务 比如只插入数据A  A也可能不成功。廖师兄建议，只要不是查询，都要应用事务。

# springboot整合redis进行数据操作(推荐)
https://www.jb51.net/article/125758.htm
redis是一种常见的nosql，日常开发中，我们使用它的频率比较高，因为它的多种数据接口，很多场景中我们都可以用到，并且redis对分布式这块做的非常好。

springboot整合redis比较简单，并且使用redistemplate可以让我们更加方便的对数据进行操作。
### 1、添加依赖
```
<dependency> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-data-redis</artifactId> 
</dependency>
```
### 2、在application.properties中加入相关配置
```
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password= 
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.timeout=5000
```
### 3、编写配置类
```
import org.springframework.cache.CacheManager; 
import org.springframework.cache.annotation.EnableCaching; 
import org.springframework.context.annotation.Bean; 
import org.springframework.context.annotation.Configuration; 
import org.springframework.data.redis.cache.RedisCacheManager; 
import org.springframework.data.redis.connection.RedisConnectionFactory; 
import org.springframework.data.redis.core.RedisTemplate; 
import org.springframework.data.redis.core.StringRedisTemplate; 
@Configuration
@EnableCaching
public class RedisConfig { 
  @Bean
  public CacheManager cacheManager(RedisTemplate<?,?> redisTemplate) { 
   CacheManager cacheManager = new RedisCacheManager(redisTemplate); 
   return cacheManager; 
  } 
  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) { 
   RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>(); 
   redisTemplate.setConnectionFactory(factory); 
   return redisTemplate; 
  } 
  @Bean
  public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory factory) { 
   StringRedisTemplate stringRedisTemplate = new StringRedisTemplate(); 
   stringRedisTemplate.setConnectionFactory(factory); 
   return stringRedisTemplate; 
  } 
}
```
这里定义了两个bean，一个是redisTemplate，另一个是stringRedisTemplate，它们的序列化方式不同，前者默认jdk序列方式，后者默认string的序列化方式，后者一般专门用于存储string格式，前者我们可以用来保存对象等，这里我们都配置上，根据不同业务进行不同使用。
### 4、编写实体类
```
public class User implements Serializable{ 
 /** 
  * 
  */
 private static final long serialVersionUID = 3221700752972709820L; 
 private int id; 
 private String name; 
 private int age; 
 public int getId() { 
  return id; 
 } 
 public void setId(int id) { 
  this.id = id; 
 } 
 public String getName() { 
  return name; 
 } 
 public void setName(String name) { 
  this.name = name; 
 } 
 public int getAge() { 
  return age; 
 } 
 public void setAge(int age) { 
  this.age = age; 
 } 
 public User(int id, String name, int age) { 
  super(); 
  this.id = id; 
  this.name = name; 
  this.age = age; 
 } 
}
```
### 5、编写测试service
```
@Service
public class UserService { 
 @Autowired
 private StringRedisTemplate stringRedisTemplate; 
 @Autowired
 private RedisTemplate<String, Object> redisTemplate; 
 public void set(String key, User user) { 
  redisTemplate.opsForValue().set(key, user); 
 } 
 public User get(String key) { 
  return (User) redisTemplate.boundValueOps(key).get(); 
 } 
 public void setCode(String key, String code) { 
  stringRedisTemplate.opsForValue().set(key, code, 60, TimeUnit.SECONDS); 
 } 
 public String getCode(String key) { 
  return stringRedisTemplate.boundValueOps(key).get(); 
 } 
}
```
这里我们模拟两种操作，一种是根据key存储user对象，另一种是存储key value均为string的操作，并且赋予数据过期时间，这种操作我们可以用于验证码存储，在setcode方法中，我们存储了一个有效时长为60s的数据，当60s过后，数据会自动销毁。
### 6、编写测试controller访问
```
@RestController
@RequestMapping("rest_redis") 
public class RedisController { 
 @Resource
 private UserService userService; 
 @GetMapping("set") 
 public void set() { 
  userService.set("key1", new User(1, "meepoguan", 26)); 
 } 
 @GetMapping("get") 
 public String get() { 
  return userService.get("key1").getName(); 
 } 
 @GetMapping("stringset") 
 public void stringset() { 
  userService.setCode("stringkey", "meepoguan_coke"); 
 } 
 @GetMapping("stringget") 
 public String stringget() { 
  return userService.getCode("stringkey"); 
 } 
}
```
对service中的方法进行测试。  

# jpa最佳实践 SpringDataJpa增删改查
https://www.cnblogs.com/hawell/p/SpringDataJpa.html

# 在Spring Boot中使用JPA
参照 <<一步步学spring boot微服务项目实战>>
### 1 引入依赖  
```
<dependency>
	<groupid>org . springfrarnework . boot</groupid>
	<artifactid>spring boot starter-data-jpa</art fact Id>
</dependenc y> 

```
### 2 继承 JpaRepository 在／src/main/java/com.example.demo
repository 下开发一个 AyUserRepository  这个类是个接口
```
public interface AyUserRepos tory extends JpaRepository<AyUser,
String> {
}
```
<AyUser,String>第一个是实体类 第二个是主键类型

### 3  实体类
与此同时，我们需要在 AyUser 实体类下添加@EEntity 和@EId 注解
在 AyUser   
```
@Entity
@Table(name = "ay_user")
public class AyUser {
  @Id
  private String id;
  
  private String name ;
  private String password ; 
}

```
 @Entity ：每个持久化 POJO 类都是一个实体 Bean ，通过在类的定义中使用  
Entity 注解来进行声明  
 @Table ：声明此对象映射到数据库的数据表。该注解不是必需的，如果没有，  
系统就会使用默认值（实体的短类名）。  
@Id ：指定表的主键  

### 4 服务层
#### 4.1 服务层接口AyUserService
```
public interface AyUserService {
	AyUser findByid(String id);
	List<AyUser> findAll();
	AyUser save(AyUser ayUser} ;
	vroid delete (String id} ; 
}
```
#### 4.2 接口实现类AyUserServiceImpl.java
```
@Service 
public class AyUserServiceimpl implements AyUserService{
	@Resource
	private AyUserRepository ayUserRepository ;
	
	@Override
	public AyUser findByid(String id){
		return ayUserRepository.findOne (id);
	}
	
	@Override
	public List<AyUser> findAll () {
		return ayUserRepository findById (id) . get () ;
	}
	
	@Override
	public AyUser save (AyUser ayUser ) {
		return ayUserRepository . save(ayUser) ;
	}
	
	@Override
	public void delete (String id) {
		ayUserRepository . deleteByid(id)
	}
}
```
@Service: Spring Boot 会自动扫描到＠Component 注解的类，并把这些类纳入
Spring 容器中管理， 也可是@Component 注解，只是＠Service注解更能
表明该类是服务层类  
@Component ：泛指纽件 当组件不好归 类的时候，我们可以使用这个注解进
行标注  
@Repository ：持久层组件 用于标注数据访问纽件，即 DAO 组件

@Resource ：这个注解属于 J2EE ，默认按照名称进行装配，名称可 以通过
name 属性进行指定。如果没有指 name属性， 当注解写在字段上时， 就默认
认取字段名进行查找。如果注解写在 setter 方法上，就默认取属性名进行装配。
当找不到与名称匹配的 bean 时，才按照类型进行装配 但需要注意的是，name 属性一旦指定，就只会按照名称进行装配 具体代码如下：  
```
@Resource(name =“ayUserRepository”)
private AyUserRepository ayUserRepository; 
```

@Autowired ：这个注解 Spring ，默认按类型装配。默认情况下，要求
依赖、对象必须存在，如果要允许 null 值，那么可以设直它的 required 属性为
false ，如＠Autowired(required=false）：如果想使用名称装配，那么可以结合
@Qualifier 注解使用 具体代码如下：
```
@Autowired
@Qualifier (“ayUserRepository”)
private AyUserRepos tory ayUserRepository; 
```

# spring boot 整合JPA 这篇很清晰
https://www.cnblogs.com/sam-uncle/p/8819478.html  

### 1 修改pom，引入依赖
```
  <!-- 引入jpa 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```
### 2 修改application.properties，配置相关信息
```
#修改tomcat默认端口号
server.port=8090
#修改context path
server.context-path=/test

#配置数据源信息
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
#配置jpa
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jackson.serialization.indent_output=true
```

### 3 创建实体类
```
package com.study.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="t_user")
public class User {

    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;
    private String userName;
    private String password;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

}
```

### 4 创建repository接口并继承CrudRepository
```
package com.study.repository;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.query.Param;

import com.study.entity.User;

/**
 * 注意：
 * 1.这里这里是interface，不是class
 * 
 * 2.CrudRepository里面的泛型，第一个是实体类，第二个是主键的类型
 * 
 * 3.由于crudRepository 里面已经有一些接口了，如deleteAll，findOne等， 我们直接调用即可
 * 
 * 4.当然，我们也可以根据自己的情况来实现自己的接口,如下面的getUser()方法，jpql语句和hql语句差不多
 * 
 * */
public interface UserRepository extends CrudRepository<User, Integer> {

    /**
     * 我们这里只需要写接口，不需要写实现，spring boot会帮忙自动实现
     * 
     * */
    
    @Query("from User where id =:id ")
    public User getUser(@Param("id") Integer id);
}
```
### 5 创建service
5.1接口
```
package com.study.service;

import com.study.entity.User;


public interface UserService {
    public User getUser(Integer id);
}
```
5.2 实现  

```
package com.study.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.study.entity.User;
import com.study.repository.UserRepository;
import com.study.service.UserService;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    UserRepository repository;
    
    @Override
    public User getUser(Integer id) {
        //有两种方式：
        //1.调用crudRepository的接口
//        return repository.findOne(id);
        //2.调用我们自己写的接口
        return repository.getUser(id);
    }

    
}
```
### 6 创建controller
```
package com.study.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.study.entity.User;
import com.study.service.UserService;

@RestController
public class UserController {
    @Autowired
    UserService service;
    
    @RequestMapping("/getUser/{id}")
    public User getUser(@PathVariable("id") Integer id){
        
        return service.getUser(id);
    }
}
```
### 7 测试，页面以json格式显示数据库值  
浏览器输入 locahost:8080/test/getuser/1  
输入一个用户json值

# spring boot jpa最佳实践
https://o7planning.org/en/11897/spring-boot-and-spring-data-jpa-tutorial

# spring boot jpa中mysql的语法详解
https://www.kancloud.cn/cxr17618/springboot/428900

# spring boot配置文件application.yml中的配置规范
username: XXXXX中必须有空格 其他位置不能有空格 否则会报错
```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: XXXXX
    password: XXXXX
    url: jdbc:mysql://us-cdbr-iron-east-02.cleardb.net:3306/heroku_7f8XXXXX60249ef5?autoReconnect=true&useUnicode=true&characterEncoding=utf-8
  jpa:
    show-sql:true
```
# spring boot jpa中报错
Caused by: org.hibernate.hql.internal.ast.QuerySyntaxException:  is not mapped  
参考 https://javabeat.net/hibernate-querysyntaxexception/  

解决方案：查询中使用的表名不是真实数据库中的表名，它应该是实体类的名称。如果您尝试映射到数据库表并且实际实体类名称不同，则会抛出此错误。另请注意，查询字符串应具有实体类名称的确切大小写（大写或小写）才能正常工作。

# spring boot jpa报错Caused by: org.hibernate.AnnotationException: No identifier specified for entity
解决方案：https://stackoverflow.com/questions/26471534/no-data-type-for-node-org-hibernate-hql-internal-ast-tree-identnode-hql
SQL queries use column names while HQL queries use Class properties. You're selecting artifact_id from Classification but the Classification class has no property named 'artifact_id'. To fix it, use the class property in your HQL.
```
SELECT artifactId FROM Classification
```
artifactId为类的属性名 Classification为类名非表名

# spring boot jpa报错 java.lang.ClassCastException: java.lang.String cannot be cast to
原因：
jpa查询语句错误
"select userId, userName, userPassword,userEmail from JpaUser where USER_ID =:userId"  
改为 "from JpaUser where USER_ID =:userId"及可 why?

```
public interface  JpaUserRepository extends CrudRepository<JpaUser, Integer> {
    @Query("select userId, userName, userPassword,userEmail from JpaUser where USER_ID =:userId")
    public JpaUser getUser(@Param("userId") Integer userId);
}

```
# 以上的3个jpa报错 都是和数据库查询语句有关，所以必须加强jpa query即JPQL的学习
https://www.baeldung.com/spring-data-jpa-query
1 查询所有列form表where条件  
```
public interface  JpaUserRepository extends CrudRepository<JpaUser, Integer> {
    @Query("select u from JpaUser u where u.userId =:userId")
    public JpaUser getUser(@Param("userId") Integer userId);
}
```
select u from JpaUser[这里要用实体类名] u where u.userId[这里要用实体类的属性名] =:userId[传入的参数]
2 用JPQL中使用原生的sql语句
```
public interface  JpaUserRepository extends CrudRepository<JpaUser, Integer> {
    @Query(value = "SELECT * FROM t_user u WHERE u.USER_ID =:userId", nativeQuery = true)
    public JpaUser getUser(@Param("userId") Integer userId);
}
```
value = "SELECT * FROM t_user【这里要用实际数据库的表名】 u WHERE u.USER_ID【这里要用实际表的字段名】 =:userId", nativeQuery = true

# maven改为国内阿里镜像源方法
1 在eclipse中 widow-属性 找到安装目录 打开安装目录下的conf文件夹 然后编辑settings.xml 在镜像设置里添加阿里镜像如下
```
<mirror> 
  <id>alimaven</id> 
  <name>aliyun maven</name> 
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
  <mirrorOf>central</mirrorOf> 
</mirror> 
```
2 把该setting文件拷贝到user下的.m2文件夹下 然后更新maven即可

# springboot操作数据库时找不到findOne（id:1）方法
https://blog.csdn.net/qq_32003379/article/details/83419280  
```
ProductCategory productCategory = repository.findById(new Integer(1)).get(); // 2.0后的写法findById(new Integer(1)).get()或findById(new Integer(1)).orElse(null)
```
# mysql查询有个连接查询，如何在spring boot jpa中映射一个连接查询的属性到实体类
参考 https://vladmihalcea.com/how-to-map-calculated-properties-with-hibernate-generated-annotation/  
How to map calculated properties with Hibernate @Generated annotation  
方法1（作者建议非生产环境下使用）  
```
@Entity(name = "Hero")
public class Hero {
 
    @Id
    private Long id;
 
    private String firstName;
 
    private String lastName;
 
    private String middleName1;
 
    private String middleName2;
 
    private String middleName3;
 
    private String middleName4;
 
    private String middleName5;
 
    @Generated( value = GenerationTime.ALWAYS )
    @Column(columnDefinition =
        "AS CONCAT(" +
        "   COALESCE(firstName, ''), " +
        "   COALESCE(' ' + middleName1, ''), " +
        "   COALESCE(' ' + middleName2, ''), " +
        "   COALESCE(' ' + middleName3, ''), " +
        "   COALESCE(' ' + middleName4, ''), " +
        "   COALESCE(' ' + middleName5, ''), " +
        "   COALESCE(' ' + lastName, '') " +
        ")")
    private String fullName;
 
    //Getters and setters omitted for brevity
 
    public String getFullName() {
        return fullName;
    }
}
```
The @Generated annotation is used to instruct Hibernate when the associated column value is calculated, and it can take two values:  

INSERT – meaning that the column value is calculated at insert time  
ALWAYS – meaning that the column value is calculated both at insert and update time  

方法2 在创建表的时候增加一个concat字段  
```
CREATE TABLE Hero
(
  id BIGINT NOT NULL ,
  firstName VARCHAR(255) ,
  fullName AS CONCAT(COALESCE(firstName, ''),
                     COALESCE(' ' + middleName1, ''),
                     COALESCE(' ' + middleName2, ''),
                     COALESCE(' ' + middleName3, ''),
                     COALESCE(' ' + middleName4, ''),
                     COALESCE(' ' + middleName5, ''),
                     COALESCE(' ' + lastName, '')) ,
  lastName VARCHAR(255) ,
  middleName1 VARCHAR(255) ,
  middleName2 VARCHAR(255) ,
  middleName3 VARCHAR(255) ,
  middleName4 VARCHAR(255) ,
  middleName5 VARCHAR(255) ,
  PRIMARY KEY ( id )
)
```

# 插入数据时报错
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'sell.hibernate_sequence' doesn't exist  
https://www.cnblogs.com/yangyi9343/p/5807512.html
```
Hibernate 能够出色地自动生成主键。Hibernate/EBJ 3 注释也可以为主键的自动生成提供丰富的支持，允许实现各种策略。
其生成规则由@GeneratedValue设定的.这里的@id和@GeneratedValue都是JPA的标准用法, JPA提供四种标准用法,由@GeneratedValue的源代码可以明显看出.
JPA提供的四种标准用法为TABLE,SEQUENCE,IDENTITY,AUTO.
TABLE：使用一个特定的数据库表格来保存主键。
SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。
IDENTITY：主键由数据库自动生成（主要是自动增长型）
AUTO：主键由程序控制。
在指定主键时，如果不指定主键生成策略，默认为AUTO。
@Id
相当于
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
 
identity:
使用SQL Server 和 MySQL 的自增字段，这个方法不能放到 Oracle 中，Oracle 不支持自增字段，要设定sequence（MySQL 和 SQL Server 中很常用）。
Oracle就要采用sequence了.
 
同时,也可采用uuid,native等其它策略.(相关用法,上网查询)
```
```
也就是说:

auto:        当数据库中  不存在 这张表的时候可以用它建表的时候, 制定自增的方式,  存在的时候插入数据还用它就会出错了

identity:     使用SQL Server 和 MySQL 的自增字段
```
# navicat连接linux的mysql数据慢的解决方法
1.状态：使用Navicat 打开远程数据库很慢。  

2.解决方法：在mysql配置文件 my.cnf  内加入 skip-name-resolve   保存 重启mysql  

 根据文档说明，如果你的mysql主机查询DNS很慢或是有很多客户端主机时会导致连接很慢，由于我们的开发机器是不能够连接外网的，所以DNS解析是不可能完成的，从而也就明白了为什么连接那么慢了。同时，请注意在增加该配置参数后，mysql的授权表中的host字段就不能够使用域名而只能够使用 ip地址了，因为这是禁止了域名解析的结果

# 微信java支付demo
https://www.cnblogs.com/wang-yaz/p/8632624.html

# springBoot jpa 分页
1 jap中有自带的分页方法  
在dao层中使用  
```
Page<LinkUrl> findAll(Pageable pageable);
```  
2 在controller层  
```
public List<LinkUrl>  getlinkList(int page,int size) {
 
	Sort sort = new Sort(Sort.Direction.DESC, "id");
	Pageable pageable = PageRequest.of(page,size,sort);
	int totalElements = (int) datas.getTotalElements(); //总条数
	int totalPages =  datas.getTotalPages(); // 总页数
	List<LinkUrl> content = datas.getContent(); // 数据列表
	return content;
 }
```
# Arrays.asList(new Product()); // 生成一个数组，数组的第一个元素是json格式的Product实例
坑： 曾经因为导包Arrays 导错，卡住了一天多时间。正确的包是：
```
import java.util.Arrays;
```
Arrays.asList(new Product(), new Product()) 则生成 [{},{}]

# Springboot中关于Beanutils.copyProperties( )的用法
https://blog.csdn.net/qq_41603102/article/details/89470246

# spring boot maven项目导入eclipse后 出现大量的报错 
* XXX cannot be resolved XXX *  
比如：
org.springframework.test cannot be resolved also with maven dependency present
根本原因: 运行环境改变 导致一些包依赖找不到了   
解决办法：
查看导入报错的 语句
```
import org.hibernate.annotations.DynamicUpdate;
```
比如他报错，就重新在maven仓库中找到依赖，然后添加到pom文件中重新安装。
```
<!-- https://mvnrepository.com/artifact/org.eclipse.persistence/javax.persistence -->
<dependency>
	<groupId>org.eclipse.persistence</groupId>
	<artifactId>javax.persistence</artifactId>
	<version>2.2.0</version>
</dependency>
```
逐条补救。
比如：无法继承 JpaRepository,不识别 JpaRepository 则需要导入 spring-data-jpa  
```
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-jpa</artifactId>

</dependency>
```
测试类中用@SpringBootTest注解遇到问题： SpringBootTest cannot be resolved to a type  
问题描述： 在src/main/java中引用@SpringBootTest注解报错，而在src/main/test中引用@SpringBootTest正常，需要在设置中修改  
```
build path - source -下的文件夹中 如果是灰色，表示可以使用SpringBootTest注解，如果不是灰色，说明不可以用。在不可用的文件夹最后一个选项contains test sources下双击切换为yes，然后保存即可。
```
### 这个问题卡了好几天，重新导入其他的maven工程，报错原因主要是因为在src/main/java中使用了junit测试，如果在src/main/test中测试，就不会报错。解决方法是在eclipse中 build-path中souce选项下contains test sources下双击切换为yes，然后不用添加任何额外的依赖了。

# springboot统一异常处理
干货  
https://www.jianshu.com/p/accec85b4039  
已经测试过（参见Java_SpringBoot_Mybatis_MySql）  
http://www.imooc.com/article/260354

# VO DTO区别
```
VO（value object）
用于返回给前端交互。传给前端的对象被称作VO（web层）。也就是前端调用你restful接口你返回的对象。

DTO（Data Transfer Object）
(Data Transfer Object 数据传输对象)，有时候我们仅仅需要获得某一个表的几个字段
，所以此时可以用DTO存储这几个字段。

一般来说的流程就是：
1 保存数据
前端  ->  接受form表单（或form对象 OrderForm） -> 转成dto(OrderDTO)  -> 拷贝dto对象属性至entity（OrderMaster）  -> 保存数据库
代码举例
	// 创建订单
	@PostMapping("/create")
	public ResultVO<Map<String, String>> create(@Valid OrderForm orderForm, BindingResult bindingResult) {
		if (bindingResult.hasErrors()) {
			// 打印日志
			throw new SellException(ResultEnum.PARAM_ERROR.getCode(),
					bindingResult.getFieldError().getDefaultMessage());
		}
		// form转成orderDTO
		OrderDTO orderDTO = OrderForm2OrderDTOConverter.convert(orderForm);
		if (CollectionUtils.isEmpty(orderDTO.getOrderDetailList())) {
			// 购物车不能为空
			throw new SellException(ResultEnum.CART_EMPTY);
		}
		// 创建订单 （orderDTO 拷贝属性至 orderMaster 然后存储orderMaster）  （BeanUtils.copyProperties(orderDTO, orderMaster);orderMasterRepository.save(orderMaster);）
		OrderDTO createResult = orderService.create(orderDTO); // 

		Map<String, String> map = new HashMap<>();
		map.put("orderId", createResult.getOrderId());

		// 返回json数据到前端

		return ResultVOUtil.success(map);
	}

2 从数据库中取出数据 及经过处理 返到前端（比如查询操作）orderMaster -》 orderDTO -》 orderVO	

	// 订单列表
	@GetMapping("/list")
	public ResultVO<List<OrderDTO>> list(@RequestParam("openid") String openid,
			@RequestParam(value = "page", defaultValue = "0") Integer page,
			@RequestParam(value = "size", defaultValue = "10") Integer size) {
		if (StringUtils.isEmpty(openid)) {
			// 打印日志
			throw new SellException(ResultEnum.PARAM_ERROR);
		}
		PageRequest request = PageRequest.of(page, size);
		// 查询出orderMaster 然后转成orderDTO  OrderMasterToOrderDTOConverter
		Page<OrderDTO> orderDTOPage = orderService.findList(openid, request);

		return ResultVOUtil.success(orderDTOPage.getContent());
	}


	@Override
	public Page<OrderDTO> findList(String buyerOpenid, Pageable pageable) {
		// TODO Auto-generated method stub
		Page<OrderMaster> orderMasterPage = orderMasterRepository.findByBuyerOpenid(buyerOpenid, pageable);
		List<OrderDTO> orderDTOList = OrderMasterToOrderDTOConverter.convert(orderMasterPage.getContent());
		Page<OrderDTO> orderDTOPage = new PageImpl<OrderDTO>(orderDTOList, pageable,
				orderMasterPage.getTotalElements());
		return orderDTOPage;
	}


从数据库中取出的对象，比如是data ，那么就先需要转换成dataDTO处理数据。然后返回到restful前转换成需要的VO

总结如下：

web页    >   dto（比如form表单转化成dtO）  >    数据库持久层entity 比如保存起来等

web页    <   VO（dto转成VO）  <    dto(service层把实体类转成dto)    <     数据库持久层 entity

```

# 自动注入实体entity报错  
比如
```
 @Autowired 
 private OrderMaster orderMaster;
```

```
Description:

Field orderMaster in com.immoc.sell.service.impl.OrderServiceImpl required a bean of type 'com.immoc.sell.dataobject.OrderMaster' that could not be found.


Action:

Consider defining a bean of type 'com.immoc.sell.dataobject.OrderMaster' in your configuration.

```
出错原因：entity实体类只能通过手工导入，用new来创建，不可以通过 @Autowired private OrderMaster orderMaster创建，否则会出现以上的报错。

# service的接口类和实现类，都要加@Service注解
 
# 插入数据库报错 1062 Dylicate entry "xxx" for key "PRIMARY KEY"
报错原因，PRRIMARY KEY 必须唯一， 就是说一旦列设置为主键，默认就是unique唯一的  
见sell  
```
orderDetail.setDetailId(orderId); // 如果这样写，oder_detail表的主键primary key为detail_id, 如果一次主订单购买2个产品，就会产生2条订单详情记录，这两条order_detail记录的order_id是for循环外生成的，如果orderDetail.setDetailId(orderId)设置的detail_id也用这个for循环外层生成的orderId，就会因为主键冲突，导致无法同时插入这两条数据
```

# SpringBoot2.0集成WebSocket，实现后台向前端推送信息
https://blog.csdn.net/moshowgame/article/details/80275084

# 表单验证 
https://my.oschina.net/u/1584624/blog/912800

# Gjson详解
https://www.cnblogs.com/ryq2014/p/9258790.html

# SpringBoot2.1.3修改tomcat参数支持请求特殊符号问题
异常一：  
    Invalid character found in method name. HTTP method names must be token  
原因： 
   产生这个问题的原因是页面表单提交了大量的数据，而这些数据量可能超过了Tomcat 定义的Header头内容，那么很好解决了，只要设置一下Tomcat的maxHttpHeaderSize  

 解决问题如下：
```
server:
  address: 0.0.0.0
  port: 8080
  max-http-header-size: 10240000
  tomcat:
    max-http-header-size: 10240000
    max-http-post-size: 10240000     

```
 异常二：  

        Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986  
原因：  
     SpringBoot 2.0.0 以上都采用内置tomcat8.0以上版本，而tomcat8.0以上版本遵从RFC规范添加了对Url的特殊字符的限制，url中只允许包含英文字母(a-zA-Z)、数字(0-9)、-_.~四个特殊字符以及保留字符( ! * ’ ( ) ; : @ & = + $ , / ? # [ ] ) (26*2+10+4+18=84)这84个字符,请求中出现了{}大括号或者[],所以tomcat报错。设置RelaxedQueryChars允许此字符(建议)，设置requestTargetAllows选项(Tomcat 8.5中不推荐)。您可以降级为旧版本之一(不推荐-安全性)。根据 https://tomcat.apache.org/tomcat-8.5-doc/config/systemprops.html, requestTargetAllowis设置允许不受欢迎字符。对我来说，这里提出的其他解决办法也不起作用。根据Tomcat文档，我找到了一种方法来设置松弛的QueryChars属性

 解决问题如下：
 最近遇到一个问题，比如GET请求中，key,value中带有特殊符号，请求会报错，见如下URL：

http://xxx.

# thymeleaf局部刷新 通过ajax请求
注意事项： 代码片段要放在渲染页面的内部  
重新渲染的时候 通过ajax请求 controller接收路由 然后重新从数据库获取数据 然后返回html片段到前端 前端替换html数据  
1 html代码
```
<!-- 测试区 -->
<table class="table table-blogs" th:fragment="blogsSearch(blogs)">
  <thead class="thead-dark">
    <tr>
      <th scope="col">序号</th>
      <th scope="col">名称</th>
      <th scope="col">操作</th>
    </tr>
  </thead>
  <tbody>
    <tr th:each="blog:${blogs}">
      <th scope="row" th:text="${blog.title}"></th>
      <td th:text="${blog.content}"></td>
      <td>
        <div class="form-button-action">
          <i class="fa fa-edit mr-2"></i>
          <i class="fa fa-times"></i>
        </div>
      </td>
    </tr>

  </tbody>
</table>
<input type="button" value="测试" class='searchtest'>
```
2 ajax请求  
```
<script>
$(".searchtest").on("click", function () {
	$.ajax({
		url : "/admin/index/search",
		data : {
			/* title : $("#recipient-name").val(),
			page: _this.data("curpage") */
		},
		type : "get",
		// dataType: "json",
		success : function(data) {
			console.log('data:', data);
			$(".table-blogs").html(data);
		}

	});
});
</script>
```
3 controller
```
@Controller
@RequestMapping("/admin")
public class AdminIndexController {
	@Autowired
	private BlogService blogService;
	
	// 登录后管理后台页面 博客列表页面
	@GetMapping("/index")
	public String blogIndex(Model model) {
		Pageable pageable = PageRequest.of(0, 5, Sort.Direction.DESC, "id");
		Page<Blog> sum =  blogService.findAllBlogs(pageable);
		model.addAttribute("blogs", sum);
		return "admin/adminindex";
	}
	
	// 博客搜索列表
	@GetMapping("/index/search")
	public String blogSearch(Model model) {
		Pageable pageable = PageRequest.of(0, 1, Sort.Direction.DESC, "id");
		Page<Blog> sum =  blogService.findAllBlogs(pageable);
		model.addAttribute("blogs", sum);
		return "admin/adminindex::blogsSearch";
		// return "redirect:/admin/index";
	}
}
```

# jackson.databind之 com.fasterxml.jackson.databind.ser...错误  
https://blog.csdn.net/A13432421434/article/details/75330675  
在嵌套的属性上添加一个注解  
@JsonIgnore

# springboot时间戳 在mvc中不能正常显示（以自动生成数据表为例）
1 实体类 在属性上配置  
```
	@Temporal(TemporalType.TIMESTAMP)
	private Date createTime;
```
在mysql中自动生成create_time 格式为datetime  

2 配置文件yml中配置  
只正对responsebody 即以json格式输出有帮助  
```
spring:  
  jackson:
    default-property-inclusion: NON_NULL
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss 
```
3 在前端MVC中输出为    
2020-02-24 20:31:13.0  

json格式输出为 正常  
2020-02-24 20:31:13  

4 在前端模板中这样些  
HH:mm 24小时制  hh12小时制  
```
<td th:text="${#dates.format(blog.createTime, 'yyyy-MM-dd  HH:mm')}"></td>
<!-- <td th:text="${blog.createTime}"></td> -->
```

# mavan打包springboot项目报错
spring boot 生成项目发生程序包org.junit不存在等错误
```
[ERROR] /D:/blog/src/main/java/com/run/blog/repository/BlogRepositoryTest.java:[
13,17] 程序包org.junit不存在
[ERROR] /D:/blog/src/main/java/com/run/blog/repository/BlogRepositoryTest.java:[
14,24] 程序包org.junit.runner不存在
[ERROR] /D:/blog/src/main/java/com/run/blog/repository/BlogRepositoryTest.java:[
16,45] 程序包org.springframework.boot.test.context不存在
[ERROR] /D:/blog/src/main/java/com/run/blog/repository/BlogRepositoryTest.java:[
22,47] 程序包org.springframework.test.context.junit4不存在
```
解决方案：
步骤1：
删除
```
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</pluginRepository>
		<pluginRepository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</pluginRepository>
	</pluginRepositories>
```
2 把test单元测试文件去掉

# Springboot部署流程
## 1 打包 
需要把测试文件删掉 否则无法打包  
打包命令  
```
// -Dmaven.test.skip=true 跳过单元测试
mvn clean package -Dmaven.test.skip=true
```
## 2 打包文件放到linux服务器上
放置位置
```
// 放置位置
/opt/javaapps/

open sis
账号
密码
put something.txt another.txt
put -r D:/www-XXX-com-80 /data/www/www-XXX-com-80
put D:/blog/target/blog-0.0.1-SNAPSHOT.jar /opt/javaapps/blog.jar

conf配置文件位于
cd /etc/nginx/conf.d/

复制命令
 cp dd.txt  ee.txt
nginx命令
现在我们可以使用nginx的控制:

sudo service nginx stop 
sudo service nginx start 
sudo service nginx restart
sudo service nginx reload

1 比如关闭占用8080的
window
netstat -aon|findstr 8080
linux
netstat -tunlp |grep 8080


2 找到进程号 终止这个进程
window
taskkill /f /t /im 9260
linux
kill 4993


```

ngxin配置
```
upstream runjieapi {
    server 127.0.0.1:3009;  # 3 找到服务器本地3009端口
}

server {
    listen       80;
    server_name   runjieapi.benkid.cn; # 1 外部访问runjieapi.benkid.cn;

    location / {
        proxy_pass http://runjieapi; # 2 找到代理服务器runjieapi
        proxy_set_header Host $host;
        proxy_set_header X-Nginx-proxy true;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
    }
}
```

## 3 运行该文件
运行命令  
```
java -jar sell.jar
// 修改端口号运行
java -jar sell.jar -Dserver.port=8030
```
常驻内存运行
```
linux中启动 java -jar 后台运行程序
直接用java -jar xxx.jar，当退出或关闭shell时，程序就会停止掉。以下方法可让jar运行后一直在后台运行。



java -jar xxx.jar &  

说明： 在末尾加入 & 符号
```

# 继承jpa Repository 自定义方法查询  
https://blog.csdn.net/bird_tp/article/details/83651651

# 关于自关联在单元测试里不显示被维护属性的问题
在自关联实体类中，通过单元测试得到的属性值为null，但是在controller中是正常显示的，例如comments中自关联属性为
parentComment 单个  
replyComments List集合  （在单元测试里为查询某个id的comment，其replyCommnet为空 但是在controller中为正常的 请注意注解）
```
@Entity
@Table(name = "t_comment")
@Getter
@Setter
public class Comment {
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	private Long id;
	private String nicename;
	private String email;
	private String content;
	private String avatar;
	@JsonFormat(timezone = "GMT+8",pattern = "yyyy-MM-dd")
	private Date createTime;
	
	@JsonIgnore
	@ManyToOne
	private Blog blog;
	
	
	
	//	自关联关系  回复的评论 和 父一级评论
	@JsonBackReference
	@OneToMany(cascade = CascadeType.ALL, mappedBy="parentComment")
	private List<Comment> replyComments = new ArrayList<>();
	
	@ManyToOne
	private Comment parentComment;

	@Override
	public String toString() {
		return "Comment [id=" + id + ", nicename=" + nicename + ", email=" + email + ", content=" + content
				+ ", avatar=" + avatar + ", createTime=" + createTime + ", blog=" + blog + ", replyComments="
				+ replyComments + ", parentComment=" + parentComment + "]";
	}

	
	
	

}

```
@JsonBackReference和@JsonManagedReference：这两个标注通常配对使用，通常用在父子关系中
https://blog.csdn.net/qq_35357001/article/details/55505659

@JsonManagedReference标注维护端 显示在json中  
@JsonBackReference标注在被维护端 不显示在json中  

# eclipse导入idea出现的几个问题：
## 1 导入的方式为（即maven项目导入idea的正确方式为）
new project - maven - 选择项目文件夹  -finish 即可

## 2 IDEA中@Data注解无getter setter 方法
经过网上一系列查询，得出原因，IDEA编辑器使用依赖 lombok 时，需要安装一个插件，否则，即使导入了正确的依赖，仍然错误。
setting - plugin - 输入Lombok  -  然后安装install - 然后重新启动idea即可

## 3报红提示：Error: Module not specified
解决方法
Run/debug configuration  -   application - blogapplication - Use classpath of mod  - no module(选择blog模块)  -- 应用 即可

# Spring Data JPA使用getOne方法报错：Method threw 'org.hibernate.LazyInitializationException' exception. Cannot evaluate
https://www.bbsmax.com/A/MAzArvBRJ9/  
这篇介绍得很明晰
getOne是懒加载，需要增加这个配置： spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true，但这种方式不太友好，建议不要使用。  

每次初始化一个实体的关联就会创建一个临时的session来加载，每个临时的session都会获取一个临时的数据库连接，开启一个新的事物。这就导致对底层连接池压力很大，而且事物日志也会被每次flush.
设想一下：假如我们查询了一个分页list每次查出1000条，这个实体有三个lazy关联对象,那么，恭喜你，你至少需要创建3000个临时session+connection+transaction.

参考2  
https://segmentfault.com/q/1010000012794754  
使用 shopConfigRepository.findOne("1"); 就不会报错.

不知道怎么回事? 望知情者解惑,万分感谢.  
getOne 是 lazy load 的  

你加上这个 spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
不建议这样用。

总结：  
不要用getOne  用findById()


# 笔记源自视频讲解所做 https://www.bilibili.com/video/av87325381?p=67

https://www.bilibili.com/video/av95156296?p=84
# jpql 是通过操作实体类来改变数据库
## 执行update或delete需要添加3个注解
1 @Modifyng  
2 需要手动添加事务支持 即@Transactional
3 默认执行完成后会回滚事务，但是实际上数据库中值没有更新，此时需要设置不自动回滚
@RollBack(value=false)

# Jpa不同的查询 
1 用repository查询 （面向对象查询）  
2 JPQL查询（面向对象查询）  
例如
```
@Query(value = "update Customer set custName = ?2 where custId = ?!")
@Modifying
public void updateCustomer(Long custId, String custName);
```

3 原生SQL语句 (作为补充) 
```
定义
@Query(value = "select * from cst_customer where cust_name like ?1", nativeQuery = true)
public List<Object []> findSql(String name); // 查询的结果是List集合,每个元素是个 Object数组

使用
@Test
public void testFindSql () {
	List<Object[]> list = customerDao.findSql("传智");
	for (Object [] obj : list) {
		System.out.println(Arrays.toString(obj));
	}
}
// 查询结果如下
[1,null,it教育,null,传智,null,null]
[1,null,it教育,null,传智1,null,null]
[1,null,it教育,null,传智2,null,null]

```

4 根据方法规则名称查询 先在repository中定义
```
findBy开头 + 对象属性首字母大写  + 查询方式Like | isnull 等  

public List<Customer> findByCustNameLike(String custName); 

多条件查询
findBy + 属性名 +  查询方式 + 多条件连接符and | or  + 属性名 +  查询方式
例如: 
使用客户名称模糊匹配 和 客户所属行业精准匹配
public Customer findByCustNameLikeAndCustIndustry(String custName, String custIndustry); // 参数顺序不能反 没有占位符了


```

5 动态查询 即查询条件不固定的 见博客系统

# springdata jpa多表操作
一对一（用得少）
一对多（用得多）
一的一方是主表（例如type表，主键id） ，多的一方是从表（例如blog表, type属性，对应数据表是type_id外键）

多对多（用得多）
用中间表，由两个字段组成，分别指向两个表的主键

# 1 一对多关系
## type 对 blog 

```
type实体类(主表)：(1、2是配置从type找到blog)
// 1 声明关系注解 targetEntity指对方对象的字节码 即Blog类
@OneToMany(targetEntity = Blog.class) 
// 2 配置外键注解 外键是从表上面 即blog表的type_id，然后参照是主表type表的主键即id
// 在type类即（一的一方）添加了外键的配置，也是维护方（即在type上维护blogs）
@JoinColumn(name="type_id", referenceColumnName="id")
private Set<Blog> blogs = new HashSet<>();


blog类：(1、2是配置从blog找到type)  
// 1 声明关系  
@ManyToOne(targetEntity = Type.class) 
// 2 配置外键对应主键的名称 即参照主表的外键名称 name是外键名type_id,referenceColumnName是主键名id 
@JoinColumn(name="type_id", referenceColumnName="id")
private Type type = new Type();

以上配置了从blog找到type，也配置了从type找到blog，是配置了双向的关系。

如何放弃维护权：
不过一般会在一的一方放弃维护权利，放弃很简单，把注解注释掉即可
即在blog上的type上取消注解@ManyToOne括号内的注解以及@JoinColum注解，但是要注明我是被Type的属性blogs维护的。
@ManyToOne(MappedBy="type")
private Type type = new Type();
```

## 一对一级联删除
比如 我想删除一个分类，我就要删除所属分类的blogs，或我要删除一个blog，我要删除这个博客所属的分类。  
级联操作的步骤：  
1 区分操作主体  
2 在需要操作的主体实体类上，添加级联属性注解
3 cascade（配置级联） 

级联添加也是一个道理，比如我要保存一个blog，同时也在type上保存新的type实体。级联删除也是一个道理。  

需求：在type上添加一个blog，同时自动添加一个blog实体，此时是在type上操作的，type就是操作主体。此时在映射关系上加个配置cascade属性（CascadeType.All所有级联操作）
type.class:  
@ManyToOne(mappedBy="type", cascade = CascadeType.All)
private Set<Blog> blogs = new HashSet<>();

# entity的toString方法 只toString基本属性，否则可能会死循环。

# springboot中定义错误页面，jar打包和war打包定义的方式不同，这里说的是jar打包自定义错误页面。
参照 https://www.jianshu.com/p/7c94d1ac2092  
第一种方式  
最简单的方式是直接在resources/templates目录下创建error.html页面，此时如果访问不存在的画面就会直接进入此画面。  
  
第二种方式是实现ErrorPageRegistrar接口，定义具体异常的URL路径：  

```
import org.springframework.boot.web.servlet.ErrorPage;
import org.springframework.boot.web.servlet.ErrorPageRegistrar;
import org.springframework.boot.web.servlet.ErrorPageRegistry;
import org.springframework.http.HttpStatus;

public class MyErrorPageRegistrar implements ErrorPageRegistrar {
    @Override
    public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
        ErrorPage page404 = new ErrorPage(HttpStatus.NOT_FOUND, "/404");
        ErrorPage page500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500");

        errorPageRegistry.addErrorPages(page404, page500);
    }
}
```  

然后在一个配置类中声明该Bean:
```
package com.run.blog.errorPage;

import org.springframework.boot.web.server.ErrorPageRegistrar;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyErrorPageBean {
    @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
        return new MyErrorPageRegistrar();
    }

}

```  
在Controller中添加404和500：  
```
package com.run.blog.errorPage;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class ErrorPageController {
    /**
     * 404 error
     * @return
     */
    @RequestMapping("/404")
    public String error404() {
        return "front/404";
    }

    /**
     * 500 error
     * @return
     */
    @RequestMapping("/500")
    public String error500() {
        return "front/500";
    }
}

```

# idea初始化springboot项目，报错：
```
Error:(3, 32) java: 程序包org.springframework.boot不存在
Error:(4, 46) java: 程序包org.springframework.boot.autoconfigure不存在
Error:(6, 2) java: 找不到符号
  符号: 类 SpringBootApplication
  Error:(10, 9) java: 找不到符号
  符号:   变量 SpringApplication
  位置: 类 com.exdd.demo3.Demo3Application
```
解决方法：
```
SpringBoot项目，maven管理，新建的一个model，代码编译、打包都可以，启动就报找不到类，找不到方法，连jdk的包都找不到，各种删jar包，刷新，重启都试过，不行。

解决办法：  
把导入项目时idea自动生成的文件删除，关闭idea，重新导入项目，就可以了。
```
