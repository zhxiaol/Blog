---
layout: post
title: struts2国际化,拦截器,OGNL
tags:
- java

categories: java
description: struts2国际化,拦截器,OGNL
---
File failed to load: /extensions/MathMenu.js
## 国际化
> 主文件名 message 语言代码 zh/en 国家代码CN/US
>  message_zh_CN.properties
>  message_en_US.properties
>  message.properties 默认的
##### 1.在指定包名(如:com.itheima.resource)下配置message_en_US.properties的全局资源文件
```
key=english message resource
jsp.login.title=i18n
jsp.login.username=Username
jsp.login.password=Password
jsp.login.submit=Submit
```
##### 2.在指定报名(如com.itheima.resouce)下配置文件名为message_zh_CN.properties的全局资源文件
```
key=全局中文的消息资源包
jsp.login.title=国际化
jsp.login.username=用户名
jsp.login.password=密码
jsp.login.submit=登陆
```
##### 3.java中使用
```
public class I18NDemo {
    @Test
    public void test1(){

        ResourceBundle bundle=ResourceBundle.getBundle("com.itheima.resource.message", Locale.US);
        String key = bundle.getString("key");
        System.out.println(key);
    }
}
```
##### 4.jsp中使用
```
<%
    ResourceBundle bundle=ResourceBundle.getBundle("com.itheima.resource.message",Locale.getDefault());
%>
<head>
    <title><%=bundle.getString("jsp.login.title")%></title>
</head>
<body>
    <form action="#" method="post">
        <%=bundle.getString("jsp.login.username")%>:<input type="text" name="username"><br/>
        <%=bundle.getString("jsp.login.password")%>:<input type="password" name="password"><br/>
        <input type="submit" value="<%=bundle.getString("jsp.login.submit")%>">
    </form>
</body>
```
## struts2中的国际化
##### 1.在指定包名下配置message_en_US.properties和message_zh_CN.properties的全局资源文件
##### 2.在struts.xml中声明配置
> 在struts2-core-xxx.jar org.apache.struts2 default.properties中搜索i18n 找到struts.custom.i18n.resources
```
<constant name="struts.custom.i18n.resources" value="com.itheima.resource.message"></constant>
```
##### 3.可以在任意包名下配置package_en_US.properties和package_zh_CN的资源文件
##### 4.可以在动作类的包名下配置:动作类名_en_US.properties和package_zh_CN的资源文件
##### 5.动作类要使用资源文件必须继承ActionSupport
```
public class Demo1Action extends ActionSupport {
    @Override
    public String execute() throws Exception {
        String value=getText("key");
        System.out.println(value);
        return SUCCESS;
    }
}
```
##### 6.jsp要使用struts标签
```
<%@taglib prefix="s" uri="/struts-tags" %>
<html>
<head>
    <title>struts2中的国际化</title>
</head>
<body>
    struts2中的国际,在jsp页面访问消息资源包，必须使用struts的标签<br/>
    <!--当直接访问jsp时，因为没有经过动作类，只会去查找全局消息资源包
        如果经过了动作类，则先去找动作类
    -->
    <s:text name="key"/><br/>
    <!--如果没有找到直接输出name的值-->
    <s:text name="abd"/><br/>
    <!--当自由指定消息资源包不存在，是按照资源搜索顺序搜索 官网文档location中有说明-->
    <s:i18n name="com.itheima.resource.message">
        <s:text name="key"/>
    </s:i18n>
</body>
```
## 自定义拦截器
```
拦截器使用步骤
	|
	|--创建普通类，继承AbstractInterceptor,实现抽象方法
	|				|
	|				|--public String intercept(ActionInvocation)
	|
	|--struts.xml
		 |
		 |--package
		 		|
		 		|--声明拦截器
		 		|	 |
		 		|	 |--<interceptors>
            	|	 |		<interceptor name="my" class="com.MyInterceptor"></interceptor>
           		|	 			 <interceptor-stack name="myDefaultStack">
               	|	  		 		  <interceptor-ref name="defaultStack"></interceptor-ref><！--系统默认-->
                |					  <interceptor-ref name="checkLoginInterceptor"></interceptor-ref>
                |																			
            	|				 </interceptor-stack>
        		|		 </interceptors>
            	|	 
        		|
        	    |    
        		|
        		|--配置拦截器引用
        			 |
        			 |--全局默认
        			 |		|
        			 |		|--package
        			 |			 |
        			 |			 |--<default-interceptor-ref name="myDefaultStack"></default-interceptor-ref>
        			 |
        			 |
        			 |--action配置
        			 		|
        			 		|--如果拦截器继承了MethodFilterInterceptor,其中一个拦截器不需要拦截时
        			 		|		|
        			 		|		|--<interceptor-ref name="myDefaultStack">
               				|				<param name="checkLoginInterceptor.excludeMethods">login</param>
           					|		   </interceptor-ref>		
           					|
           					|--配置使用的拦截器
           							|
           							|--<interceptor-ref name="myDefault"></interceptor-ref>
```

```
//Demo2Action
public class Demo2Interceptor extends AbstractInterceptor {
    @Override
    public String intercept(ActionInvocation actionInvocation) throws Exception {
        System.out.println("demo2Interceptor 拦截了-执行动作方法之前");
        //放行:如果有下一个拦截器就执行下一个拦截器，如果没有就到达方法
        String value = actionInvocation.invoke();
        System.out.println("demo2Interceptor 拦截了-执行动作方法之后");
        return value;
    }
}

//struts.xml
<package name="p2" extends="struts-default">
    <!--声明拦截器-->
    <interceptors>
        <interceptor name="demo2Interceptor" class="com.itheima.interceptor.Demo2Interceptor"></interceptor>
        <interceptor name="demo3Interceptor" class="com.itheima.interceptor.Demo3Interceptor"></interceptor>
    </interceptors>
    <action name="action2" class="com.itheima.web.action.Demo2Action" method="save">
        <!--使用拦截器,多个拦截器的时候引用配置决定顺序-->
        <interceptor-ref name="demo2Interceptor"></interceptor-ref>
        <interceptor-ref name="demo3Interceptor"></interceptor-ref>
        <result>/demo2.jsp</result>
    </action>
</package>
```

```
//CehckLoginInterceptor
public class CheckLoginInterceptor extends AbstractInterceptor {
    @Override
    public String intercept(ActionInvocation actionInvocation) throws Exception {
        //获取httpsession对象
        HttpSession session = ServletActionContext.getRequest().getSession();
        //获取登录标记
        Object obj = session.getAttribute("user");
        if(obj==null){//用户没有登录
            return "input";
        }
        String rtValue=actionInvocation.invoke();
        return rtValue;
    }
}

//CheckLoginInterceptor2
public class CheckLoginInterceptor2 extends MethodFilterInterceptor {

    @Override
    protected String doIntercept(ActionInvocation actionInvocation) throws Exception {
        //获取httpsession对象
        HttpSession session = ServletActionContext.getRequest().getSession();
        //获取登录标记
        Object obj = session.getAttribute("user");
        if(obj==null){//用户没有登录
            return "input";
        }
        String rtValue=actionInvocation.invoke();
        return rtValue;
    }
}

//struts.xml
<package name="p3" extends="struts-default">
    <interceptors>
        <interceptor name="checkLoginInterceptor" class="com.itheima.interceptor.CheckLoginInterceptor2"></interceptor>
        <interceptor-stack name="myDefaultStack">
            <interceptor-ref name="defaultStack"></interceptor-ref>
            <interceptor-ref name="checkLoginInterceptor">
                <param name="excludeMethods">login</param>
            </interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <default-interceptor-ref name="myDefaultStack"></default-interceptor-ref>
    <global-results>
        <result name="input">/login.jsp</result>
    </global-results>
    <action name="showMain" class="com.itheima.web.action.Demo3Action">
        <interceptor-ref name="myDefault"></interceptor-ref>
        <result>main.jsp</result>
    </action>
    <action name="showOther" class="com.itheima.web.action.Demo3Action">
        <interceptor-ref name="myDefault"></interceptor-ref>
        <result>other.jsp</result>
    </action>
    <action name="login" method="login" class="com.itheima.web.action.Demo3Action">
        <interceptor-ref name="myDefaultStack">
            <param name="checkLoginInterceptor.excludeMethods">login</param>
        </interceptor-ref>
        <result type="redirectAction">showMain</result>
    </action>
</package>
```

## 文件上传
> FileUploadInterceptor 拦截器注入获取表单数据
##### 单文件上传
```
//upload1.jsp
<s:form action="upload.action" enctype="multipart/form-data">
    <s:textfield name="username" label="用户名"/>
    <s:file name="photo" label="照片"/>
    <s:submit value="上传"></s:submit>
</s:form>

//struts.xml
<struts>
<!--修改上传文件大小，第一种方式 通过修改struts2的常量，在default.properties中配的struts.multipart.maxSize=2097152-->
<constant name="struts.multipart.maxSize" value="5242880"></constant>
<package name="p4" extends="struts-default">
    <action name="upload" class="com.itheima.web.action.Upload1Action" method="upload">
        <!--限制文件大小第二种方式 失效了-->
        <interceptor-ref name="defaultStack">
            <param name="fileUpload.maximumSize">31457280</param>
            <!--限制上传扩展名-->
            <param name="fileUpload.allowedExtensions">png,jpeg,bmp,jpg</param>
            <!--限制上传类型-->
            <param name="fileUpload.allowedTypes">image/jpg,image/pjpeg,image/png</param>
        </interceptor-ref>
        <result name="input">/upload1.jsp</result>
    </action>
    </package>
</struts>

//upload1Action
public class Upload1Action extends ActionSupport{
    private String username;
    private File photo;
    //struts2在文件上传属性
    private String photoFileName;//上传的文件名.上传字段名+FileName
    private String photoContentType;//上传文件的MIME类型。上传字段名+contentType
    public String upload()throws Exception{
        ServletContext application = ServletActionContext.getServletContext();
        String filePath = application.getRealPath("/WEB-INF/files");
        File file=new File(filePath);
        if(!file.exists()){
            file.mkdirs();
        }
        //剪接，把临时文件剪接过去
        photo.renameTo(new File(file,photoFileName));
        //FileUtils.copyFile(photo,new File(file,photoFileName));
        return NONE;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public File getPhoto() {
        return photo;
    }

    public void setPhoto(File photo) {
        this.photo = photo;
    }

    public String getPhotoFileName() {
        return photoFileName;
    }

    public void setPhotoFileName(String photoFileName) {
        this.photoFileName = photoFileName;
    }

    public String getPhotoContentType() {
        return photoContentType;
    }

    public void setPhotoContentType(String photoContentType) {
        this.photoContentType = photoContentType;
    }
}
```

##### 多文件上传
```
//upload2.jsp
<s:form action="upload2.action" enctype="multipart/form-data">
    <s:textfield name="username" label="用户名"/>
    <s:file name="photo" label="照片"/>
    <s:file name="photo" label="照片"/>
    <s:submit value="上传"></s:submit>
</s:form>


//struts.xml
<action name="upload2" class="com.itheima.web.action.Upload2Action" method="upload">
    <interceptor-ref name="defaultStack">
        <param name="fileUpload.maximumSize">31457280</param>
        <param name="fileUpload.allowedExtensions">png,jpeg,bmp,jpg</param>
        <param name="fileUpload.allowedTypes">image/jpg,image/pjpeg,image/png,image/jpeg</param>
    </interceptor-ref>
    <result name="input">/upload2.jsp</result>
</action>

//upload2Action
public class Upload2Action extends ActionSupport{
    private String username;
    private File[] photo;
    //struts2在文件上传属性
    private String[] photoFileName;//上传的文件名.上传字段名+FileName
    private String[] photoContentType;//上传文件的MIME类型。上传字段名+contentType
    public String upload()throws Exception{
        ServletContext application = ServletActionContext.getServletContext();
        String filePath = application.getRealPath("/WEB-INF/files");
        File file=new File(filePath);
        if(!file.exists()){
            file.mkdirs();
        }
        for(int i=0;i<photo.length;i++) {
            //剪接，把临时文件剪接过去
            photo[i].renameTo(new File(file, photoFileName[i]));
            //FileUtils.copyFile(photo,new File(file,photoFileName));
        }
        return NONE;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public File[] getPhoto() {
        return photo;
    }

    public void setPhoto(File[] photo) {
        this.photo = photo;
    }

    public String[] getPhotoFileName() {
        return photoFileName;
    }

    public void setPhotoFileName(String[] photoFileName) {
        this.photoFileName = photoFileName;
    }

    public String[] getPhotoContentType() {
        return photoContentType;
    }

    public void setPhotoContentType(String[] photoContentType) {
        this.photoContentType = photoContentType;
    }
}
```

## 文件下载
> StreamResult注入结果视图 类型要是stream
```
//struts.xml
<action name="download" class="com.itheima.web.action.DownloadAction" method="download">
    <result name="success" type="stream">
        <!--给steam的结果类型注入参数 -->
        <param name="contentType">application/octet-stream</param>
        <param name="contentDisposition">attachment;filename=photo.jpg</param>
        <!--注入字节输入流-->
        <param name="inputName">is</param>
    </result>
</action>

//downloadAction
public class DownloadAction extends ActionSupport {
    //注意:在给inputsteam指定名称时，不能使用in
    private InputStream is;
    public String download() throws FileNotFoundException {
        String filePath= ServletActionContext.getServletContext().getRealPath("/WEB-INF/files/头像.jpg");
        is=new FileInputStream(filePath);
        return SUCCESS;
    }

    public InputStream getIs() {
        return is;
    }

    public void setIs(InputStream is) {
        this.is = is;
    }
}
```

## 自定义EL方法
> 格式${}
> EL只能调用静态方法
##### 步骤
```
自定义EL表达式方法
	 |
	 |--创建类和静态方法
	 |
	 |--配置
	 |	 |
	 |	 |--在WEB-INF的目录中创建一个扩展名为.tld的xml文件。文件不能放在classes和lib 目录
	 |	 		|
	 |	 		|--仿造tomcat目录下conf/webapps/examples/WEB-INF/jsp2/xxx.tld写
	 |
	 |--引入
	 	 |
	 	 |--在jsp中引入方法库
	 	 		|
	 	 		|--<%@ taglib prefix="myfn" uri="http://www.itheima.com/function/myfunction" %>
```
##### 示例
```
//MyFunction
public class MyFunction {
    public static String toUpperCase(String str){
        return str.toUpperCase();
    }
}

//myfunction.tld
<?xml version="1.0" encoding="UTF-8" ?>
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">
    <description>A tag library exercising SimpleTag handlers.</description>
    <tlib-version>1.0</tlib-version><!--指定标签库或方法版本号-->
    <short-name>myfn</short-name><!--短名称，对应taglib指令中的prefix-->
    <uri>http://www.itheima.com/function/myfunction</uri><!--uri:把当前的方法库绑定到一个uri地址上，在该网址上不一定存在方法库-->
    <function>
        <description>Converts the string to all caps</description>
        <name>toUpperCase</name><!--方法名称 jsp页面上使用的名称-->
        <function-class>com.itheima.web.function.MyFunction</function-class><!--指定执行的类-->
        <function-signature>java.lang.String toUpperCase( java.lang.String )</function-signature><!--指定执行的方法，返回值和参数除了基本类型必须写全路径-->
    </function>
</taglib>

//eldemo.jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="myfn" uri="http://www.itheima.com/function/myfunction" %>
<html>
<body>
    abcdefg---ABCDEFG
    ${myfn:toUpperCase("abcdefg")}
</body>
</html>
```
###### jsp el ognl表达式
>jsp表达式:  <%%>
>el表达式:  ${}
> ognl表达式:  &lt;s:property value="">

## OGNL
> OGNL是Oject Graphic Navigation Language(对象图导航)的缩写，它是一个开源项目，struts2框架使用OGNL作为默认的表达式语言
|属性|备注|
|---|---|
| 格式|lt;s:property value=""|
|开启静态访问|struts.ognl.allowStaticMethodAccess=true|
|value属性|不在是字符串而是表达式，如果想要字符串要加单引号|
|访问静态属性的格式|@全类名@静态属性名称|
|访问静态方法的格式|@全类名@静态方法名|
|xml中使用ognl方法|${@全类名@方法名}
|list集合|{}相当于创建了list集合 集合里面是ognl表达式|
|map集合| '#{}表示创建一个map'|
```
<body>
    abcdefg---ABCDEFG
    ${myfn:toUpperCase("abcdefg")}
    <hr/>
    <!--value 内容是表达式，不是字符串，如果想让他变成字符串套上单引号-->
    <s:property value="OGNL-Expression"/>这是一个OGNL表达式<br/>
    <s:property value="'OGNL-Expression'"/>这是一个普通字符串<br/>
    <s:property value="'OGNL-Expression'.length()"/>使用普通字符串调取了字符串方法
    <hr/>
    <%--ognl访问静态属性格式@全类名@静态属性名称--%>
    <s:property value="@java.lang.Integer@MAX_VALUE"/>
    <%--ognl访问静态方法格式@全类名@静态方法名称 struts2默认是禁止静态方法 配置开启--%>
    <s:property value="@java.lang.Math@random()"/>
    <%--ognl操作list和map
    {}相当于创建了list集合 集合里面是ognl表达式
    #{}表示创建一个map
    --%>
    <s:radio list="{'男','女'}" name="gender"/>
    <s:radio list="#{'1':'男','0':'女'}" name="gender"/>
</body>
```
