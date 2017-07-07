---
layout: post
title: webService02
tags:
- java

categories: java
description: webService
---

#### CXF介绍
* CXF是一个开源的webservice框架，提供很多完善功能，可以实现快速开发
* CXF支持的协议：SOAP1.1/1.2，REST
* CXF支持数据格式：XML，JSON（仅在REST方式下支持）

##### CXF的安装和配置
* 下载地址
http://cxf.apache.org/download.html
* 包结构介绍
```html
目录结构
   |
   |--bin 二进制文件
   |   |
   |   |--wsdl2java 主要
   |
   |--docs 文档目录
   |
   |--etc 内部文件版本控制
   |
   |--lib 第三方jar包
   |
   |--licenses 第三方协议证书
   |
   |--modules 功能模块
   |
   |--samples 示例
   |
   |--LECENSE 自己的协议
   |
   |--NOTICE 注意事项
   |
   |--README 说明
   |
   |--relase-notes.text 升级说明
```

#### 安装和配置

* 第一步：安装JDK，建议1.7

* 第二步：解压apache-cxf-2.7.11.zip到指定目录，创建CXF_HOME

* 第三步：把CXF_HOME加入到Path路径下

* 第四步：测试，在cmd下加入wsdl2java –h


#### CXF发布SOAP协议的服务
##### 需求
* 服务端：发布服务，接收客户端的城市名，返回天气数据给客户端
* 客户端：发送城市名给服务端，接收服务端的响应信息，打印


##### 服务端

```javascript
// 第一步：导入Jar包 cxf里面所有的jar包
// 第二步：创建SEI接口，要加入@WebService
@WebService
public interface WeatherInterface {

	public String queryWeather(String cityName);
}

//第三步：创建SEI实现类
public class WeatherInterfaceImpl implements WeatherInterface {

	@Override
	public String queryWeather(String cityName) {
		System.out.println("from client..."+cityName);
		if("北京".equals(cityName)){
			return "冷且霾";
		} else {
			return "暖且晴";
		}
	}

}

// 第四步：发布服务, JaxWsServerFactoryBean发布，设置3个参数，1.服务接口；2.服务实现类；3.服务地址；
// endpoint仅支持发布实现类，JaxWsServerFactoryBean支持发布接口。
public class WeatherServer {

	public static void main(String[] args) {
		//JaxWsServerFactoryBean发布服务
		JaxWsServerFactoryBean jaxWsServerFactoryBean = new JaxWsServerFactoryBean();
		//设置服务接口
		jaxWsServerFactoryBean.setServiceClass(WeatherInterface.class);
		//设置服务实现类
		jaxWsServerFactoryBean.setServiceBean(new WeatherInterfaceImpl());
		//设置服务地址
		jaxWsServerFactoryBean.setAddress("http://127.0.0.1:12345/weather");
		//发布
		jaxWsServerFactoryBean.create();
	}
}

// 第五步：测试服务是否发布成功，阅读使用说明书，确定关键点
// 如果在CXF发布的服务下，直接访问服务地址 报异常No binding operation info while invoking
// unknown method with params unknown.
// 此时直接访问使用说明书地址即可
```


##### 发布SOAP1.2的服务端
* 在接口上加入注解:@BindingType(SOAPBinding.SOAP12HTTP_BINDING)

##### 拦截器
* 原理：
拦截器可以拦截请求和响应
拦截器可以有多个
拦截器可以根据需要自定义

* 使用
拦截器必须加到服务端，在服务端发布之前
获取拦截器列表，将自己的拦截器加入列表中

```javascript
public class WeatherServer {
    public static void main(String[] args) {
        JaxWsServerFactoryBean jaxWsServerFactoryBean=new JaxWsServerFactoryBean();
        jaxWsServerFactoryBean.setServiceClass(WeatherInterface.class);
        jaxWsServerFactoryBean.setServiceBean(new WeatherInterfaceImpl());
        jaxWsServerFactoryBean.setAddress("http://127.0.0.1:12345/weather");
        //添加拦截器
        jaxWsServerFactoryBean.getInInterceptors().add(new LoggingInInterceptor());
        jaxWsServerFactoryBean.getOutInterceptors().add(new LoggingOutInterceptor());
        jaxWsServerFactoryBean.create();
    }
}
```




##### 客户端
* 第一步：生成客户端代码
Wsdl2java命令是CXF提供的生成客户端的工具，他和wsimport类似，可以根据WSDL生成客户端代码
Wsdl2java常用参数：
	-d，指定输出目录
	-p，指定包名，如果不指定该参数，默认包名是WSDL的命名空间的倒序
        wsdl2java -p cn.itcast.cxf.weather -d . http://127.0.0.1:12345/weather?wsdl
Wsdl2java支持SOAP1.1和SOAP1.2

* 第二步：使用说明书，使用生成代码调用服务端
	JaxWsProxyFactoryBean调用服务端，设置2个参数，1.设置服务接口；2.设置服务地址

```javascript
public class WeatherClient {

	public static void main(String[] args) {
		//JaxWsProxyFactoryBean调用服务端
		JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
		//设置服务接口
		jaxWsProxyFactoryBean.setServiceClass(WeatherInterface.class);
		//设置服务地址
		jaxWsProxyFactoryBean.setAddress("http://127.0.0.1:12345/weather");
		//获取服务接口实例
		WeatherInterface weatherInterface = jaxWsProxyFactoryBean.create(WeatherInterface.class);
		//调用查询方法
		String weather = weatherInterface.queryWeather("保定");
		System.out.println(weather);
	}
}
```


#### CXF+Spring整合发布SOAP协议的服务
##### 服务端
```xml
// 第一步：创建web项目（引入jar包）
// 第二步：创建SEI接口
// 第三步：创建SEI实现类
// 第四步：配置spring配置文件，applicationContext.xml，用<jaxws:server标签发布服务，设置1.服务地址；2.设置服务接口；3设置服务实现类
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				            http://www.springframework.org/schema/beans/spring-beans.xsd
				            http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
				            http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
				            http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
	<!-- <jaxws:server发布SOAP协议的服务 ，对JaxWsServerFactoryBean类封装-->
	<jaxws:server address="/weather" serviceClass="cn.itcast.ws.cxf.server.WeatherInterface">
		<jaxws:serviceBean>
			<ref bean="weatherInterface"/>
		</jaxws:serviceBean>
	</jaxws:server>
	<!-- 配置服务实现类 -->
	<bean name="weatherInterface" class="cn.itcast.ws.cxf.server.WeatherInterfaceImpl"/>
</beans>


第五步：配置web.xml，配置spring配置文件地址和加载的listener，配置CXF的servlet
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>ws_2_cxf_spring_server</display-name>

  <!-- 设置spring的环境 -->
  <context-param>
  	<!--contextConfigLocation是不能修改的  -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- 配置CXF的Servlet -->
  <servlet>
  	<servlet-name>CXF</servlet-name>
  	<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>CXF</servlet-name>
  	<url-pattern>/ws/*</url-pattern>
  </servlet-mapping>

</web-app>


第六步：部署到tomcat下，启动tomcat

第七步：测试服务，阅读使用说明书
	WSDL地址规则：http://ip:端口号/项目名称/servlet拦截路径/服务名称?wsdl
        http://localhost:8080/ws/weather?wsdl

```


##### 拦截器配置
配置applicationContext.xml中
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
       xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
                        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
                        http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
    <!--<jaxws:server>发布SOAP协议服务,对JaxWsServerFactoryBean类封装-->
    <jaxws:server address="/weather" serviceClass="cn.itcast.ws.cxf.server.WeatherInterface">
        <jaxws:serviceBean>
            <ref bean="weatherInterface"/>
        </jaxws:serviceBean>
        <!--配置拦截器-->
        <jaxws:inInterceptors>
            <ref bean="loggingInInterceptor"/>
        </jaxws:inInterceptors>
        <jaxws:outInterceptors>
            <ref bean="loggingOutInterceptor"/>
        </jaxws:outInterceptors>
    </jaxws:server>
    <bean name="loggingInInterceptor" class="org.apache.cxf.interceptor.LoggingInInterceptor"/>
    <bean name="loggingOutInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor"/>
    <bean name="weatherInterface" class="cn.itcast.ws.cxf.server.WeatherInterfaceImpl"/>
</beans>
```

##### Endpoint标签发布服务
```java
<jaxws:endpoint address="/hello" implementor="cn.itcast.ws.cxf.server.HelloWorld"/>

@WebService
public class HelloWorld {
	public String sayHello(String name){
		return "hello,"+name;
	}

}
```




##### 客户端
```java
//第一步：引入jar包
//第二步：生成客户端代码
//第三步：配置spring配置文件，applicationContent.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				            http://www.springframework.org/schema/beans/spring-beans.xsd
				            http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
				            http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
				            http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
	<!-- <jaxws:client实现客户端 ，对JaxWsProxyFactoryBean类封装-->
	<jaxws:client id="weatherClient" address="http://127.0.0.1:8080/ws_2_cxf_spring_server/ws/weather" serviceClass="cn.itcast.cxf.weather.WeatherInterface"/>
</beans>

//第四步：从spring上下文件获取服务实现类
//第五步：调用查询方法，打印
public class WeatherClient {
	public static void main(String[] args) {
		//初始化spring的上下文
		ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
		WeatherInterface  weatherInterface = (WeatherInterface) context.getBean("weatherClient");
		String weather = weatherInterface.queryWeather("保定");
		System.out.println(weather);
	}
}
```

#### CXF发布REST的服务
* 定义：REST就是一种编程风格，它可以精确定位网上资源（服务接口、方法、参数）
* REST支持数据格式：XML、JSON
* REST支持发送方式：GET，POST

##### 需求
* 第一个：查询单个学生
* 第二个：查询多个学生

##### 服务端
```java
//第一步：导入jar包
//第二步：创建学生pojo类，要加入@ XmlRootElement
@XmlRootElement(name="student")//@XmlRootElement可以实现对象和XML数据之间的转换
public class Student {
	private long id;
	private String name;
	private Date birthday;
        //get set..
}

// 第三步：创建SEI接口
@WebService
@Path("/student")//@Path("/student")就是将请求路径中的“/student”映射到接口上
public interface StudentInterface {
	//查询单个学生
	@GET//指定请求方式，如果服务端发布的时候指定的是GET（POST），那么客户端访问时必须使用GET（POST）
	@Produces(MediaType.APPLICATION_XML)//指定服务数据类型
	@Path("/query/{id}")//@Path("/query/{id}")就是将“/query”映射到方法上，“{id}”映射到参数上，多个参数，以“/”隔开，放到“{}”中
	public Student query(@PathParam("id")long id);
	//查询多个学生
	@GET//指定请求方式，如果服务端发布的时候指定的是GET（POST），那么客户端访问时必须使用GET（POST）
	@Produces(MediaType.APPLICATION_XML)//指定服务数据类型
	@Path("/queryList/{name}")//@Path("/queryList/{name}")就是将“/queryList”映射到方法上，“{name}”映射到参数上，多个参数，以“/”隔开，放到“{}”中
	public List<Student> queryList(@PathParam("name")String name);
}


// 第四步：创建SEI实现类
public class StudentInterfaceImpl implements StudentInterface {

	@Override
	public Student query(long id) {
		Student st = new Student();
		st.setId(110);
		st.setName("张三");
		st.setBirthday(new Date());
		return st;
	}

	@Override
	public List<Student> queryList(String name) {

		Student st = new Student();
		st.setId(110);
		st.setName("张三");
		st.setBirthday(new Date());

		Student st2 = new Student();
		st2.setId(120);
		st2.setName("李四");
		st2.setBirthday(new Date());

		List<Student> list = new ArrayList<Student>();
		list.add(st);
		list.add(st2);
		return list;
	}

}

// 第五步：发布服务, JAXRSServerFactoryBean发布服务，3个参数，1：服务实现类；2.设置资源类；3.设置服务地址
public class StudentServer {

	public static void main(String[] args) {
		//JAXRSServerFactoryBean发布REST的服务
		JAXRSServerFactoryBean jaxRSServerFactoryBean = new JAXRSServerFactoryBean();

		//设置服务实现类
		jaxRSServerFactoryBean.setServiceBean(new StudentInterfaceImpl());
		//设置资源类，如果有多个资源类，可以以“,”隔开。
		jaxRSServerFactoryBean.setResourceClasses(StudentInterfaceImpl.class);
		//设置服务地址
		jaxRSServerFactoryBean.setAddress("http://127.0.0.1:12345/user");
		//发布服务
		jaxRSServerFactoryBean.create();
	}
}

/** 第六步：测试服务
http://127.0.0.1:12345/user/student/query/110 查询单个学生，返回XML数据
http://127.0.0.1:12345/user/student/queryList/110?_type=json	 查询多个学生，返回JSON
http://127.0.0.1:12345/user/student/queryList/110?_type=xml	 查询多个学生，返回XML
如果服务端发布时指定请求方式是GET（POST），客户端必须使用GET（POST）访问服务端，否则会报异常
如果在同一方法上同时指定XML和JSON媒体类型，在GET请求下，默认返回XML，在POST请求下，默认返回JSON
**/
```


##### 客户端
* httpclient
http://hc.apache.org/httpclient-3.x/

```java
public class HttpClient {
	public static void main(String[] args) throws IOException {
		//第一步：创建服务地址，不是WSDL地址
		URL url = new URL("http://127.0.0.1:12345/user/student/query/110");
		//第二步：打开一个通向服务地址的连接
		HttpURLConnection connection = (HttpURLConnection) url.openConnection();
		//第三步：设置参数
		//3.1发送方式设置：POST必须大写
		connection.setRequestMethod("POST");
		//3.2设置数据格式：content-type
		//3.3设置输入输出，因为默认新创建的connection没有读写权限，
		connection.setDoInput(true);
		//第五步：接收服务端响应，打印
		int responseCode = connection.getResponseCode();
		if(200 == responseCode){//表示服务端响应成功
			InputStream is = connection.getInputStream();
			InputStreamReader isr = new InputStreamReader(is);
			BufferedReader br = new BufferedReader(isr);

			StringBuilder sb = new StringBuilder();
			String temp = null;
			while(null != (temp = br.readLine())){
				sb.append(temp);
			}
			System.out.println(sb.toString());
			is.close();
			isr.close();
			br.close();
		}
	}
}
```


#### CXF+Spring整合发布REST的服务
##### 服务端
```java
// 第一步：创建web项目（引入jar包）
// 第二步：创建POJO类
// 第三步：创建SEI接口
// 第四步：创建SEI实现类
// 第五步：配置Spring配置文件,applicationContext.xml，<jaxrs:server，设置1.服务地址；2.服务实现类
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				            http://www.springframework.org/schema/beans/spring-beans.xsd
				            http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
				            http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
				            http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
	<!-- <jaxrs:server发布REST的服务 ，对JAXRSServerFactoryBean类封装-->
	<jaxrs:server address="/user">
		<jaxrs:serviceBeans>
			<ref bean="studentInterface"/>
		</jaxrs:serviceBeans>
	</jaxrs:server>
	<!-- 配置服务实现类 -->
	<bean name="studentInterface" class="cn.itcast.ws.rest.server.StudentInterfaceImpl"/>
</beans>

第六步：配置web.xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>ws_2_cxf_spring_server</display-name>

  <!-- 设置spring的环境 -->
  <context-param>
  	<!--contextConfigLocation是不能修改的  -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- 配置CXF的Servlet -->
  <servlet>
  	<servlet-name>CXF</servlet-name>
  	<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>CXF</servlet-name>
  	<url-pattern>/ws/*</url-pattern>
  </servlet-mapping>

</web-app>

// 第七步：部署到tomcat下，启动tomcat

// 第八步：测试服务
// REST服务的使用说明书地址：
// http://127.0.0.1:8080/ws_4_cxf_rest_spring_server/ws/user?_wadl
```


##### 客户端
```html
<!doctype html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script type="text/javascript">
	function queryStudent(){
		//创建XMLHttpRequest对象
		var xhr = new XMLHttpRequest();
		//打开连接
		xhr.open("get","http://127.0.0.1:8080/ws_4_cxf_rest_spring_server/ws/user/student/queryList/110?_type=json",true);
		//设置回调函数
		xhr.onreadystatechange=function(){
			//判断是否发送成功和判断服务端是否响应成功
			if(4 == xhr.readyState && 200 == xhr.status){
				alert(eval("("+xhr.responseText+")").student[0].name);
			}
		}
		//发送数据
		xhr.send(null);
	}
  </script>
 </head>
 <body>
  <input type="button" value="查询" onclick="javascript:queryStudent();"/>
 </body>
</html>
```

##### 综合案例 集成公网手机号归属地查询服务
```java
// 第一步：创建web项目（引入jar包）
// 第二步：生成公网客户端代码
// 第三步：创建SEI接口

@WebService
public interface MobileInterface {

	public String queryMobile(String phoneNum);
}


// 第四步：创建SEI实现类
public class MobileInterfaceImpl implements MobileInterface {

	private MobileCodeWSSoap mobileClient;

	@Override
	public String queryMobile(String phoneNum) {
		return mobileClient.getMobileCodeInfo(phoneNum, "");
	}

	public MobileCodeWSSoap getMobileClient() {
		return mobileClient;
	}

	public void setMobileClient(MobileCodeWSSoap mobileClient) {
		this.mobileClient = mobileClient;
	}

}


// 第五步：创建queryMobile.jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>手机号归属查询网站</title>
</head>
<body>
	<form action="queryMobile.action" method="post">
		手机号归属地查询：<input type="text" name="phoneNum"/><input type="submit" value="查询"/><br/>
		查询结果：${result}
	</form>
</body>
</html>

// 第六步：创建MobileServlet.java
public class MobileServlet extends HttpServlet {

	private MobileInterface mobileServer;

	public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String phoneNum = request.getParameter("phoneNum");
		if(null != phoneNum && !"".equals(phoneNum)){
			ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
			mobileServer = (MobileInterface) context.getBean("mobileServer");
			String result = mobileServer.queryMobile(phoneNum);
			request.setAttribute("result", result);
		}
		request.getRequestDispatcher("/WEB-INF/jsp/queryMobile.jsp").forward(request, response);
	}

	public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		this.doGet(request, response);
	}

}



// 第七步：配置spring配置文件，applicationContext.xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				            http://www.springframework.org/schema/beans/spring-beans.xsd
				            http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
				            http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
				            http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
	<!-- <jaxws:server发布服务-->
	<jaxws:server address="/mobile" serviceClass="cn.itcast.mobile.server.MobileInterface">
		<jaxws:serviceBean>
			<ref bean="mobileServer"/>
		</jaxws:serviceBean>
	</jaxws:server>
	<!-- 配置服务实现类 -->
	<bean name="mobileServer" class="cn.itcast.mobile.server.MobileInterfaceImpl">
		<property name="mobileClient" ref="mobileClient"/>
	</bean>

	<!-- 配置公网客户端 -->
	<jaxws:client id="mobileClient" address="http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx"
		serviceClass="cn.itcast.mobile.MobileCodeWSSoap"/>

</beans>


// 第八步：配置web.xml

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>ws_2_cxf_spring_server</display-name>  
  <!-- 设置spring的环境 -->
  <context-param>
  	<!--contextConfigLocation是不能修改的  -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- 配置CXF的Servlet -->
  <servlet>
  	<servlet-name>CXF</servlet-name>
  	<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>CXF</servlet-name>
  	<url-pattern>/ws/*</url-pattern>
  </servlet-mapping>
  <!-- 配置mobileServlet -->
  <servlet>
  	<servlet-name>mobileServlet</servlet-name>
  	<servlet-class>cn.itcast.mobile.server.servlet.MobileServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>mobileServlet</servlet-name>
  	<url-pattern>*.action</url-pattern>
  </servlet-mapping>

  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
</web-app>


// 第九步：部署到tomcat下，启动tomcat
```
