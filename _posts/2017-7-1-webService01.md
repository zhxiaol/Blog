---
layout: post
title: webService01
tags:
- java

categories: java
description: webService
---
## Webservice
* Webservice就是一种远程调用技术，他的作用就是从远程系统中获取业务数据
* Webservice是使用Http发送SOAP协议的数据的一种远程调用技术
* Webservice客户端开发需要阅读服务端的使用说明书（WSDL）
* UUID 提高webservice服务端的注册和搜索功能





#### Webservice的入门程序 天气查询
##### 服务端：
开发步骤：
第一步：创建SEI（Service Endpoint Interface）接口，本质上就是Java接口
第二步：创建SEI实现类，在实现类上加入@WebService
第三步：发布服务，Endpoint发布服务，publish方法，两个参数：1.服务地址；2.服务实现类
第四步：测试服务是否发布成功，通过阅读使用说明书，确定客户端调用的接口、方法、参数和返回值存在，证明服务发布成功

WSDL地址：服务地址+”?wsdl”
WSDL阅读方式：从下往上

###### 服务器
```java
public interface WeatherInterface {
    public String queryWeather(String cityName);
}

@WebService //表示该类是一个服务 发布public 方法
public class WeatherInterfaceImpl implements WeatherInterface {
    @Override
    public String queryWeather(String cityName) {
        System.out.println("from client..."+cityName);
        String weather="晴";
        return weather;
    }
}

public class WeatherServer {
    public static void main(String[] args) {
        //发布服务
        //1.服务地址,2.实现类
        Endpoint.publish("http://127.0.0.1:12345/weather",new WeatherInterfaceImpl());
    }
}
```







##### 客户端：
开发步骤
第一步：wsimport命令生成客户端代码
wsimport -s . http://127.0.0.1:12345/weather?wsdl
wsimport -p cn.itcast.weather -s . file:///zhxiaol/WeatherWS.asmx.xml //本地文件
第二步：根据使用说明书，使用客户端代码调用服务端
               创建服务视图，视图是从service标签的name属性获取
               获取服务实现类，实现类从portType的name属性获取
第三步：获取查询方法，从portType的operation标签获取

```java
public class WeatherClient {
    public static void main(String[] args) {
        WeatherInterfaceImplService service=new WeatherInterfaceImplService();
        WeatherInterfaceImpl port = service.getPort(WeatherInterfaceImpl.class);
        String result = port.queryWeather("北京");
        System.out.println(result);
    }
}
```

#### Webservice的优缺点
##### 优点：
* 发送方式采用http的post发送，http的默认端口是80，防火墙默认不拦截80，所以跨防火墙
* 采用XML格式封装数据，XML是跨平台的，所以webservice也可以跨平台。
* Webservice支持面向对象
##### 缺点：
* 采用XML格式封装数据，所以在传输过程中，要传输额外的标签，随着SOAP协议的不断完善，标签越来越大，导致webservice性能下降

##### Webservice应用场景
* 软件集成和复用


* 适用场景
发布一个服务（对内/对外），不考虑客户端类型，不考虑性能，建议使用webservice
服务端已经确定使用webservice，客户端不能选择，必须使用webservice

* 不适用场景
考虑性能时不建议使用webservice
同构程序下不建议使用webservice，比如java 用RMI，不需要翻译成XML的数据


#### WSDL
* 定义
WSDL及web服务描述语言，他是webservice服务端使用说明书，说明服务端接口、方法、参数和返回值，WSDL是随服务发布成功，自动生成，无需编写

* 文档结构

```xml
<service>    服务视图，webservice的服务结点，它包括了服务端点
<binding>     为每个服务端点定义消息格式和协议细节
<portType>   服务端点，描述 web service可被执行的操作方法，以及相关的消息，通过binding指向portType
<message>   定义一个操作（方法）的数据参数(可有多个参数)
<types>        定义 web service 使用的全部数据类型
```

* 阅读方式：从下往上



#### SOAP
##### 定义：
SOAP即简单对象访问协议，他是使用http发送的XML格式的数据，它可以跨平台，跨防火墙，SOAP不是webservice的专有协议。SOAP=http+xml

##### 协议格式
* 必需有 Envelope 元素，此元素将整个 XML 文档标识为一条 SOAP 消息
* 可选的 Header 元素，包含头部信息
* 必需有Body 元素，包含所有的调用和响应信息
* 可选的 Fault 元素，提供有关在处理此消息所发生错误的信息


#### SOAP1.1和SOAP1.2区别
##### 相同点：
* 请求发送方式相同：都是使用POST
* 协议内容相同：都有Envelope和Body标签
##### 不同点：
* 数据格式不同：content-type不同
SOAP1.1：text/xml;charset=utf-8
SOAP1.2：application/soap+xml;charset=utf-8
* 命名空间不同：
SOAP1.1：http://schemas.xmlsoap.org/soap/envelope/
SOAP1.2：http://www.w3.org/2003/05/soap-envelope

##### UDDI
UDDI 是一种目录服务，企业可以使用它对 Web services 进行注册和搜索。UDDI，英文为 "Universal Description, Discovery and Integration"，可译为“通用描述、发现与集成服务”。
UDDI 并不像 WSDL 和 SOAP 一样深入人心，因为很多时候，使用者知道 Web 服务的位置（通常位于公司的企业内部网中）。




#### Webservice的四种客户端调用方式
##### 公网服务地址：
http://www.webxml.com.cn/zh_cn/index.aspx

#### 第一种生成客户端调用方式
##### Wsimport命令介绍
* Wsimport就是jdk提供的的一个工具，他作用就是根据WSDL地址生成客户端代码
* Wsimport位置JAVA_HOME/bin
* Wsimport常用的参数：
-s，生成java文件的
	-d，生成class文件的，默认的参数
	-p，指定包名的，如果不加该参数，默认包名就是wsdl文档中的命名空间的倒序
* Wsimport仅支持SOAP1.1客户端的生成


###### 调用公网手机号归属地查询服务

```java
//第一步：wsimport生成客户端代码
//wsimport -p cn.itcast.mobile -s . http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx?wsdl
//第二步：阅读使用说明书，使用生成客户端代码调用服务端
public class MobileClient {
	public static void main(String[] args) {
		//创建服务视图
		MobileCodeWS mobileCodeWS = new MobileCodeWS();
		//获取服务实现类
		MobileCodeWSSoap mobileCodeWSSoap = mobileCodeWS.getPort(MobileCodeWSSoap.class);
		//调用查询方法
		String reuslt = mobileCodeWSSoap.getMobileCodeInfo("13888888", null);
		System.out.println(reuslt);
	}
}
```

###### 公网天气服务端查询
```java
//第一步：wsimport生成客户端代码
wsimport -p cn.itcast.weather -s . file:///zhxiaol/WeatherWS.asmx.xml
//第二步：阅读使用说明书，使用生成客户端代码调用服务端
public class WeatherClient {
	public static void main(String[] args) {
		WeatherWS weatherWS = new WeatherWS();
		WeatherWSSoap weatherWSSoap = weatherWS.getPort(WeatherWSSoap.class);
		ArrayOfString  arrayOfString = weatherWSSoap.getWeather("北京", "");
		List<String> list = arrayOfString.getString();
		for(String str : list){
			System.out.println(str);
		}
	}
}
```


#### 第二种：service编程调用方式
* 该种方式可以自定义关键元素，方便以后维护，是一种标准的开发方式
```java
public class ServiceClient {

	public static void main(String[] args) throws IOException {
		//创建WSDL的URL，注意不是服务地址
		URL url = new URL("http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx?wsdl");

		//创建服务名称
		//1.namespaceURI - 命名空间地址
		//2.localPart - 服务视图名
		QName qname = new QName("http://WebXml.com.cn/", "MobileCodeWS");

		//创建服务视图
		//参数解释：
		//1.wsdlDocumentLocation - wsdl地址
		//2.serviceName - 服务名称
		Service service = Service.create(url, qname);
		//获取服务实现类
		MobileCodeWSSoap mobileCodeWSSoap = service.getPort(MobileCodeWSSoap.class);
		//调用查询方法
		String result = mobileCodeWSSoap.getMobileCodeInfo("1866666666", "");
		System.out.println(result);
	}
}

```

#### 第三种：HttpURLConnection调用方式
开发步骤：
* 第一步：创建服务地址
* 第二步：打开一个通向服务地址的连接
* 第三步：设置参数
   设置POST，POST必须大写，如果不大写，报如下异常 Invalid Http method:post
   如果不设置输入输出，会报异常 cannot write to a UrlConncetion if doOutput=false
* 第四步：组织SOAP数据，发送请求
* 第五步：接收服务端响应，打印

```java
public class HttpClient {
	public static void main(String[] args) throws IOException {
		//第一步：创建服务地址，不是WSDL地址
		URL url = new URL("http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx");
		//第二步：打开一个通向服务地址的连接
		HttpURLConnection connection = (HttpURLConnection) url.openConnection();
		//第三步：设置参数
		//3.1发送方式设置：POST必须大写
		connection.setRequestMethod("POST");
		//3.2设置数据格式：content-type
		connection.setRequestProperty("content-type", "text/xml;charset=utf-8");
		//3.3设置输入输出，因为默认新创建的connection没有读写权限，
		connection.setDoInput(true);
		connection.setDoOutput(true);

		//第四步：组织SOAP数据，发送请求
		String soapXML = getXML("15226466316");
		OutputStream os = connection.getOutputStream();
		os.write(soapXML.getBytes());
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

			br.close();
		}
		os.close();
	}
	public static String getXML(String phoneNum){
		String soapXML = "<?xml version=\"1.0\" encoding=\"utf-8\"?>"
		+"<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">"
			+"<soap:Body>"
		    +"<getMobileCodeInfo xmlns=\"http://WebXml.com.cn/\">"
		    	+"<mobileCode>"+phoneNum+"</mobileCode>"
		      +"<userID></userID>"
		    +"</getMobileCodeInfo>"
		  +"</soap:Body>"
		+"</soap:Envelope>";
		return soapXML;
	}
}
```



#### Ajax调用方式
```html
<!doctype html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script type="text/javascript">
	function queryMobile(){
		//创建XMLHttpRequest对象
		var xhr = new XMLHttpRequest();
		//打开连接
		xhr.open("post","http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx",true);
		//设置数据类型
		xhr.setRequestHeader("content-type","text/xml;charset=utf-8");
		//设置回调函数
		xhr.onreadystatechange=function(){
			//判断是否发送成功和判断服务端是否响应成功
			if(4 == xhr.readyState && 200 == xhr.status){
				alert(xhr.responseText);
			}
		}
		//组织SOAP协议数据
		var soapXML = "<?xml version=\"1.0\" encoding=\"utf-8\"?>"
		+"<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">"
			+"<soap:Body>"
		    +"<getMobileCodeInfo xmlns=\"http://WebXml.com.cn/\">"
		    	+"<mobileCode>"+document.getElementById("phoneNum").value+"</mobileCode>"
		      +"<userID></userID>"
		    +"</getMobileCodeInfo>"
		  +"</soap:Body>"
		+"</soap:Envelope>";
		alert(soapXML);
		//发送数据
		xhr.send(soapXML);
	}
  </script>
 </head>
 <body>
  手机号查询：<input type="text" id="phoneNum"/> <input type="button" value="查询" onclick="javascript:queryMobile();"/>
 </body>
</html>
```


#### 深入开发：用注解修改WSDL内容
> WebService的注解都位于javax.jws包下:

* @WebService-定义服务，在public class上边
targetNamespace：指定命名空间
name：portType的名称
portName：port的名称
serviceName：服务名称
endpointInterface：SEI接口地址，如果一个服务类实现了多个接口，只需要发布一个接口的方法，可通过此注解指定要发布服务的接口。
* @WebMethod-定义方法，在公开方法上边
	operationName：方法名
	exclude：设置为true表示此方法不是webservice方法，反之则表示webservice方法，默认是false
* @WebResult-定义返回值，在方法返回值前边
	name：返回结果值的名称
* @WebParam-定义参数，在方法参数前边
	name：指定参数的名称
* 作用：
通过注解，可以更加形像的描述Web服务。对自动生成的wsdl文档进行修改，为使用者提供一个更加清晰的wsdl文档。
当修改了WebService注解之后，会影响客户端生成的代码。调用的方法名和参数名也发生了变化
```java
@WebService(targetNamespace = "http://service.cn.itcast",
name = "WeatherWSSoap",
portName = "WeatherWSSoapPort",
serviceName = "WeatherWS") //表示该类是一个服务 发布public 方法
public class WeatherInterfaceAnnotation implements WeatherInterface {
    @WebMethod(operationName = "getWeather",
    exclude = false)
    @Override
    public @WebResult(name = "result") String queryWeather(@WebParam(name = "cityName") String cityName) {
        System.out.println("from client..."+cityName);
        String weather="晴";
        return weather;
    }
}
```
