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
maven不能为2（注意看错误提示 注意修改eclipse关联的maven安装配置）  
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
