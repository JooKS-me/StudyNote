## 利用SpringBoot快速生成骨架

前往https://start.spring.io/选择需要的配置，生成，然后下载。

## 打开下载的项目

用idea打开

但是可能会卡住，需要先删除.mvn文件

## 编写HelloSpring

编辑HelloSpringApplication.java

```java
@SpringBootApplication
@RestController
public class HelloSpingApplication {

	public static void main(String[] args) {
		SpringApplication.run(HelloSpingApplication.class, args);
	}

	@RequestMapping("/hello")
	public String hello() {
		return "Hello Spring";
	}
}
```

## 运行

点击运行即可，spring会在8080端口开启tomcat服务。

## 打包

在命令行窗口输入mvn clean package -Dmaven.test.skip

- -Dmaven.test.skip表示跳过测试

- clean表示清除产生的项目
- package表示打包

## 打包后

在项目文件夹中会有target文件夹

里面有两个jar包，一个是.jar（大，内含所有依赖），一个是.jar.original（小）

输入`javac -jar xxxxx.jar`即可运行，同样会在8080端口启动Tomcat

