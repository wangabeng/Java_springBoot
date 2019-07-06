#  maven 创建springBoot项目

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
https://o7planning.org/en/11679/spring-boot-file-upload-example

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
解决方案  
解决方案：查询中使用的表名不是真实数据库中的表名，它应该是实体类的名称。如果您尝试映射到数据库表并且实际实体类名称不同，则会抛出此错误。另请注意，查询字符串应具有实体类名称的确切大小写（大写或小写）才能正常工作。
