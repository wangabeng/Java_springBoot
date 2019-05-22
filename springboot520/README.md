# 1 maven 创建springBoot项目

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
