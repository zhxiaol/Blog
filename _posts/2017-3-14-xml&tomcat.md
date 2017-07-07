---
layout: post
title: xml&tomcat
tags:
- java

categories: java
description: xml&tomcat
---
## XML Xtensible Markup Language 可扩展标记语言
> 可扩展:所有标签自定义
> 功能:数据存储，配置文件，数据传输
### 转义
> &lt;![CDATA[1>2&&3>4]]&gt;
### DTD约束
> xml的书写规则

* dtd 内部dtd
dtd 外部dtd 本地文件:<!DOCUTYPE students SYSTEM "student.dtd">
dtd 外部dtd 网络文件:<!DOCUTYPE students PUBLIC  "名称空间" "student.dtd">
##### student.dtd
```xml
<!ELEMENT students (student*)><!-- 根标签 student可以出现0至多个-->
<!ELEMENT student (name,age,sex)><!--student标签要出现的内容，顺序-->
<!ELEMENT name (#PCDATA)><!-- name标签中可以写文本-->
<!ELEMENT age (#PCDATA)>
<!ELEMENT sex (#PCDATA)>
<!ATTLIST student number ID #REQUIRED><!-- student里面可以写属性 属性名称ID -->
```
##### student.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE students SYSTEM "student.dtd">
<students>
<student number="s001">
    <name>张三</name>
    <age>23</age>
    <sex>男</sex>
</student>
</students>
```
* shema
编写根标签
引入实列名称空间 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
引入名称空间 xsi:schemaLocation="http://www.itcast.cn/xml student.xsd "
引入默认的名称空间
##### student.sxd
```xsd
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.itcast.cn/xml"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.itcast.cn/xml"
        elementFormDefault="qualified">
<xsd:element name="students" type="studentsType"/>
<xsd:complexType name="studentsType">
    <xsd:sequence>
        <xsd:element name="student" type="studentType" minOccurs="0" maxOccurs="unbounded"></xsd:element>
    </xsd:sequence>
</xsd:complexType>
<xsd:complexType name="studentType">
    <xsd:sequence>
        <xsd:element name="name" type="xsd:string"/>
        <xsd:element name="age" type="ageType"/>
        <xsd:element name="sex" type="sexType"/>
    </xsd:sequence>
    <xsd:attribute name="number" type="numberType" use="required"/>
</xsd:complexType>
<xsd:simpleType name="sexType">
    <xsd:restriction base="xsd:string">
        <xsd:enumeration value="男"/>
        <xsd:enumeration value="女"/>
    </xsd:restriction>
</xsd:simpleType>
<xsd:simpleType name="ageType">
    <xsd:restriction base="xsd:integer">
        <xsd:minInclusive value="0"></xsd:minInclusive>
        <xsd:maxInclusive value="256"></xsd:maxInclusive>
    </xsd:restriction>
</xsd:simpleType>
<xsd:simpleType name="numberType">
    <xsd:restriction base="xsd:string">
        <xsd:pattern value="itcast_\d{4}"/>
    </xsd:restriction>
</xsd:simpleType>

</xsd:schema>
```
##### student.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<students xmlns="http://www.itcast.cn/xml"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.itcast.cn/xml  student.xsd">
<student number="itcast_1001">
    <name>张三</name>
    <age>23</age>
    <sex>男</sex>
</student>
</students>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
     version="3.1">
</web-app>
```
## xml解析
* DOM
将文档加载到内存，形成一颗dom树，将文档的各个组成部分封装为一些对象
优点：可以增删改查
缺点: dom树非常占内存，解析速度慢

* SAX
逐行读取，基于事件驱动
优点：不占内存，速度快
缺点：只能读取，不能回写

* JAXP:sun公司提供的解析。支持dom和sax
* JDOM:
* DOM4J: dom for java 民间方式，但是是事实方式。非常好。支持dom
1.导入jar包 dom4j.jar
2.创建解析器  SAXReader reader=new SAXReader();
3.解析xml 获得document对象
    Document document=reader.read(url);
```java
@Test
public void test1() throws Exception {
SAXReader reader=new SAXReader();
Document document=reader.read("src/book.xml");
Element root=document.getRootElement();
Element booksNode=root.element("book");
System.out.println(booksNode.getName());
List list = root.elements();
Element secondBook= (Element) list.get(1);
String name=secondBook.element("name").getText();
System.out.println(name);
}
```
```java
@Test
public  void test2() throws DocumentException {
SAXReader reader=new SAXReader();
Document document=reader.read("src/book.xml");
Element root=document.getRootElement();
treeWalk(root);
}

private  void treeWalk(Element e) {
System.out.println(e.getName());
for(int i=0;i<e.nodeCount();i++){
    Node node = e.node(i);
    if(node instanceof Element){
        treeWalk((Element) node);
    }
}
}
```
* XPATH: 专门用于查询
定义一种规则
使用的方法1.selectSingleNode(); 2.selectNodes();
使用步骤
1.导入jaxen..jar
2.创建解析器 SaxReader reader=new SaxReader()
3.获得decument Document document = reader.read(url)

|符号|备注|
|----|------|
|nodename| 选取此节点|
|/ |从根节点选取|
|//| 不考虑他的位置|
|.. | 父节点|
|@ |选取属性|
|[@属性名] |属性过滤|
|[标签名]  | 子元素过滤|
```java
@Test
public void test1() throws DocumentException {
SAXReader reader=new SAXReader();
Document document = reader.read("src/book.xml");
Node node = document.selectSingleNode("/books/book[2]/name");//第二本书名
System.out.println(node.getText());
}
```
```java
@Test
public void test2() throws DocumentException {
SAXReader reader=new SAXReader();
Document document = reader.read("src/book.xml");
List list = document.selectNodes("//*");
for(int i=0;i<list.size();i++){
    Node node = (Node) list.get(i);
    System.out.println(node.getName()+"\t"+node.getText());
}
}
```
```java
@Test
public void test3() throws  DocumentException {
SAXReader reader=new SAXReader();
Document document = reader.read("src/Dom4JTest.xml");
List list = document.selectNodes("bookstore//book/title");
for(int i=0;i<list.size();i++){
    Node node = (Node) list.get(i);
    System.out.println(node.getText());
}

}
```
## Tomcat
> 小型Servlet/jsp调试工具 免费 稳定 高效 可以配合Apache搭建集群 是企业javaweb应用最佳servlet容器选择之一

## 配置tomact
http://tomcat.apache.org/ 下载tomcat tar.gz版本 解压拷贝到/usr/local/目录下
启动 sudo /usr/local/tomcat目录名称/bin/startup.sh
停止 sudo /usr/local/tomcat目录名称/bin/shutdown.sh
## 修改tomcat端口
> tomcat目录/conf/server.xml 第70行 &lt;Conector port="8081"/&gt;
## Tomcat主要目录
```html
Tomcat主要目录
|
|--bin 可执行文件
|	|
|	|--start.sh 启动tomcat
|	|--shutdown.sh  关闭tomcat
|
|--conf 配置文件
|   |
|   |--server.xml
|   |--Catalina
|		  |
|         |--localhost 配置虚拟目录
|
|--lib tomcat运行时依赖的jar包
|
|--logs tomcat运行时产生的日志文件
|
|--temp tomcat运行时的临时文件
|
|--webapps 存放应用的目录
|
|--work tomcat运行时的工作目录
```

## 标准JavaWeb应用的目录结构
```html
webapp
|
|---xx.html
|
|---css
|	|--myStyle.css
|
|---js
|    |--myjs.js
|
|---WEB-INF 固定写法 此目录不能被外部直接访问
  |
  |--classes	程序代码.class
  |     |--xxx.class
  |
  |--lib  应用需要的jar
  |   |--xxx.jar
  |
  |--web.xml  应用配置信息
```

## 集成Tomcat和自动部署
### eclipse
>集成
window--Prefereces--Servers--Tomcat--Tomcat Home directory
>部署

servers窗口 >> Tomcat右键 add Deployment
### idea
> 集成

References >> Application Servers +>> Tomcat目录  
> 部署 (tomcat9 要配置jdk8才能运行)

1. Project Structure >> Project >> Project compiler out(WEB-INF/classes)
2. Project Structure >> Modules >> Sources >> 右键src(Sources)
Project Structure >> Modules >> Paths >> output path(WEB-INF/classes)
Project Structure >> Modules >> Dependencies >> +(libraries/tomcat) +jars or directories(WEB-INF/lib)
3. double shift 输入 Edit Configurations >> + >> Tomcat Server
## 部署应用到Tomcat
> 开发目录部署方式

把应用直接复制到tomcat/webapps 目录下
>把应用打成war包

打war包命令: jar -cvf 应用名称.war .
把war包直接复制到tomcat/webapps下，应用会自动解压

##URI和URI
> URL:统一资源定位符(网址)
> URI:统一资源标识符
```html
URL统一资源定位符http://localhost:8080/myapp.index.html
--------------------------------------------------------------------
http://[协议]											
|													
|--localhost:[主机]                         			       
  |											
  |--8080[端口]												
    |										
    |--/myapp/index.html[URI资源标识符]
```

## 虚拟目录
```html
Tomcat目录
|
|--conf
|	 |--Catalina(方式一:推荐使用，无需重启服务器)
|	 |		|
 |		|--localhost
 |				|
 |				|--添加myapp.xml
 |						|
 |						|--<Context docBase="D:\myApp"/>
 |
 |--server.xml(方式二:不推荐使用，需重启服务器)
    |
    |--<Host> 标签内添加配置
      |
      |--添加<Context path="" docBase=""></Context>		
            |
            |--	path:虚拟目录，docBase:实际目录
```
##### myapp.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<Context docBase="D:\myApp"/>
```
## 默认主页
>web.xml
```
<welcome-file-list>
<welcome-file>1.html</welcome-file>
<welcome-file>2.html</welcome-file>
</welcome-file-list>
```
## HTTP
> HyperText Transfer Protocol 超文本传输协议
> 封装了请求和响应的数据
### 请求部分

##### 请求消息行

GET  /day08_02/1.html  HTTP/1.1
请求方式：Get（默认）  POST  DELETE  HEAD等
GET：明文传输 不安全，数据量有限，不超过1kb
GET /day08_02/1.html?uName=tom&pwd=123 HTTP/1.1
POST: 暗文传输，安全。数据量没有限制。
URI：统一资源标识符。去协议和IP地址。
协议/版本 ：

##### 请求消息头

从第2行到空行处，都叫消息头
Accept:浏览器可接受的MIME类型
告诉服务器客户端能接收什么样类型的文件。
Accept-Charset: 浏览器通过这个头告诉服务器，它支持哪种字符集
Accept-Encoding:浏览器能够进行解码的数据编码方式，比如gzip
Accept-Language:浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。 可以在浏览器中进行设置。
Host:初始URL中的主机和端口
Referrer:包含一个URL，用户从该URL代表的页面出发访问当前请求的页面
Content-Type:内容类型
告诉服务器浏览器传输数据的MIME类型，文件传输的类型
application/x-www-form-urlencoded
If-Modified-Since: Wed, 02 Feb 2011 12:04:56 GMT利用这个头与服务器的文件进行比对，如果一致，则从缓存中直接读取文件。
User-Agent:浏览器类型.
Content-Length:表示请求消息正文的长度
Connection:表示是否需要持久连接。如果服务器看到这里的值为“Keep -Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接
Cookie:这是最重要的请求头信息之一 (在讲会话时解析)
Date：Date: Mon, 22 Aug 2011 01:55:39 GMT请求时间GMT
##### 请求正文
当请求方式是POST方式时，才能看见消息正文
Name=tom&pwd=123

### 响应

##### 响应信息行

第一行：
HTTP/1.1   200   OK
协议/版本   响应状态码  对响应码的描述（一切正常）
响应状态码：
常用的就40多个。
200(正常)  一切正常
302/307(临时重定向)
304(未修改)
表示客户机缓存的版本是最新的，客户机可以继续使用它，无需到服务器请求。
404(找不到)  服务器上不存在客户机所请求的资源。
500(服务器内部错误)

##### 响应消息头

Location: http://www.it315.org/index.jsp指示新的资源的位置
通常和302/307一起使用，完成请求重定向
Server:apache tomcat指示服务器的类型
Content-Encoding: gzip服务器发送的数据采用的编码类型
Content-Length: 80 告诉浏览器正文的长度
Content-Language: zh-cn服务发送的文本的语言
Content-Type: text/html; charset=GB2312服务器发送的内容的MIME类型
Last-Modified: Tue, 11 Jul 2000 18:23:51 GMT文件的最后修改时间
Refresh: 1;url=http://www.it315.org指示客户端刷新频率。单位是秒
Content-Disposition: attachment; filename=aaa.zip指示客户端下载文件
Set-Cookie:SS=Q0=5Lb_nQ; path=/search服务器端发送的Cookie
Expires: -1
Cache-Control: no-cache (1.1)  
Pragma: no-cache   (1.0)  表示告诉客户端不要使用缓存
Connection: close/Keep-Alive   
Date: Tue, 11 Jul 2000 18:23:51 GMT

##### 响应消息正文

和网页右键“查看源码”看到的内容一样。
