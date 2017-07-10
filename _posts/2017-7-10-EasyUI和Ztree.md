---
layout: post
title: EasyUI和Ztree
tags:
- java

categories: java
description: EasyUI和Ztree
---
#### BOS Bussiness Operating System 业务操作系统
##### 常见软件类型
* OA系统
办公自动化(Office Automation),办公的便捷方便,提高效率
* CRM系统
Customer Relationship Management客户关系管理系统,管理与客户的关系
* ERP系统
Enterprise Resource Planning企业资源计划,针对物资,人力,财务,信息一体的管理系统
* CMS
Content Mangement System 内容管理系统

#### 搭建开发环境
##### 数据库环境
```sql
--创建数据库
create database bos19 character set utf8;
--创建用户
create user xiaoli identified by '123';
--授权 all增删改查权限 *表,视图,索引,存储过程...
grant all on bos19.* to xiaoli;

```
##### web环境搭建
###### 1.创建动态web项目
###### 2.导入jar包(SSH,Spring依赖,日志,数据库驱动)
###### 3.配置web.xml(struts2的过滤器,spring的监听器,解决Hibernate延迟加载问题的过滤器,解决中文乱码的过滤器)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!--解决中文乱码-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--解决hirbernate延迟加载的问题-->
    <filter>
        <filter-name>openSession</filter-name>
        <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>openSession</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--spring配置文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
###### 4.创建目录结构
```
com.itheima.bos
	  |
	  |--dao
	  |--domain
	  |--service
	  |--utils
	  |--web
	  	  |--action
	  	  |--filter
	  	  |--interceptor
	  	  |--listener
```
###### 5.在config目录下提供log4j,struts的配置文件
```
<struts>
    <constant name="struts.devMode" value="true"/>
    <constant name="struts.objectFactory" value="spring"/>
    <package name="basicstruts2" extends="struts-default">
        <!--需要进行权限控制的页面访问-->
        <action name="page_*_*">
            <result type="dispatcher">/WEB-INF/pages/{1}/{2}.jsp</result>
        </action>
    </package>
</struts>
```
##### 6.在config目录下提供spring的配置文件applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
    <!--加载jdbc属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driverClass}"></property>
        <property name="jdbcUrl" value="${jdbcUrl}"></property>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <!--spring用于整合hibernate的工厂bean-->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql>">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>
        <!--注入映射文件-->
        <property name="mappingDirectoryLocations">
            <list>
                <value>classpath:com/itheima/bos/domain</value>
            </list>
        </property>
    </bean>
    <!--事务管理器-->
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
    <!--组件扫描-->
    <context:component-scan base-package="com.itheima.bos"/>
    <!--引入注解解析器-->
    <context:annotation-config/>
    <!--事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

###### 7.提供项目所需的资源文件 css js image jsp

##### 使用svn管理项目代码
```
svn/bos19/conf
	 |
	 |--svnserve.conf
   	 |		|
     |		|--anon-access = none
     |		|--auth-access = write
     |		|--password-db = passwd
   	 |		|--authz-db = authz
   	 |
   	 |
   	 |--passwd
   	 |	  |
   	 |	  |--zhangsan = 123
   	 |	     lisi = 123
   	 |
   	 |
   	 |--authz
   	 	 |
   	 	 |--[/]
			zhangsan = rw
			lisi = r
			* =
//启动 svnserve -d -r /zhxiaol/svn/
```

##### EasyUI
###### layout页面布局
这个布局容器,有五个区域:北,南,东,西和中心,中心面板是必须的,边缘地区可选,每一个边缘可以缩放的拖动其边境
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>layout---页面布局</title>
<link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath }/js/easyui/themes/default/easyui.css">
<link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath }/js/easyui/themes/icon.css">
<script type="text/javascript" src="${pageContext.request.contextPath }/js/jquery-1.8.3.js"></script>

</head>
<body class="easyui-layout">
   <%--使用div指定区域--%>
   <div title="xx管理系统" data-options="region:'north'" style="height: 100px">北部区域</div>
   <div title="系统菜单" data-options="region:'west'" style="width: 200px">西部区域</div>
   <div data-options="region:'center'">中心区域</div>
   <div data-options="region:'east'" style="width:100px;">东部区域</div>
   <div data-options="region:'south'" style="height:50px;">南部区域</div>
</body>
</html>
```

###### 折叠面板
```
<!-- 折叠面板效果 -->
<div class="easyui-accordion" data-options="fit:true">
   <!-- 每个子div是其中的一个面板 -->
   <div title="面板一">棉衣一</div>
   <div title="面板二">test2</div>
   <div title="面板三">test3</div>
</div>
```
###### 选项卡面板
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>tabs---选项卡面板</title>
<link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath }/js/easyui/themes/default/easyui.css">
<link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath }/js/easyui/themes/icon.css">
<script type="text/javascript" src="${pageContext.request.contextPath }/js/jquery-1.8.3.js"></script>
<script type="text/javascript" src="${pageContext.request.contextPath }/js/easyui/jquery.easyui.min.js"></script>
</head>
<body class="easyui-layout">
   <!-- 使用div指定区域 -->
   <div title="XX管理系统" data-options="region:'north'" style="height: 100px">北部区域</div>
   <div title="系统菜单" data-options="region:'west'" style="width: 200px">
      <!-- 折叠面板效果 -->
      <div class="easyui-accordion" data-options="fit:true">
         <!-- 每个子div是其中的一个面板 -->
         <div title="面板一">
            <a class="easyui-linkbutton" onclick="doAdd();">百度</a>
            <script type="text/javascript">
                  function doAdd(){
                     //动态添加一个选项卡面板
                     $("#tt").tabs("add",{
                        title:'这个可是动态的',
                        content:'<iframe frameborder="0" width="100%" height="100%" src="page_base_staff.action"></iframe>',
                        closable:true,
                        iconCls:'icon-search'
                     });
                  }
            </script>
         </div>
         <div title="面板二">test2</div>
         <div title="面板三">test3</div>
      </div>
   </div>

   <div data-options="region:'center'">
      <!-- 选项卡面板效果 -->
      <div id="tt" class="easyui-tabs" data-options="fit:true">
         <!-- 每个子div是其中的一个面板 -->
         <div data-options="closable:true,iconCls:'icon-help'" title="面板一">棉衣一</div>
         <div title="面板二">test2</div>
         <div title="面板三">test3</div>
      </div>
   </div>
   <div data-options="region:'east'" style="width: 100px">东部区域</div>
   <div data-options="region:'south'" style="height: 50px">南部区域</div>
</body>
</html>
```

##### Ztree树形插件

###### 标准JSON方式
```
<link rel="stylesheet" href="${pageContext.request.contextPath }/js/ztree/zTreeStyle.css" type="text/css">
<div title="面板二">
   <ul id="ztree1" class="ztree"></ul>
   <script type="text/javascript">
      $(function () {
         var setting ={};
         //构造json数据,json数据加到树上,每个json对象对应一个节点数据
         var zNodes=[{
            name:'系统管理'
         },{
            name:'用户管理',children:[
               {name:'用户添加'},
               {name:'用户修改'}
            ]
         },{
             name:'权限管理'
         }];
         $.fn.zTree.init($("#ztree1"),setting,zNodes)
                 })
   </script>
</div>
```

###### 简单JSON方式
```
<link rel="stylesheet" href="${pageContext.request.contextPath }/js/ztree/zTreeStyle.css" type="text/css">
<div title="面板三">
   <%--简单json数据--%>
   <ul id="ztree2" class="ztree"></ul>
   <script type="text/javascript">
                 $(function () {
                     var setting2 ={
                         data:{
                             simpleData:{
                                 enable:true//启用简单数据
               }
            }
         };
                     //构造json数据,json数据加到树上,每个json对象对应一个节点数据
                     var zNodes2=[{
                         id:'1',
            pId:'0',
                         name:'系统管理'
                     },{
                         id:'2',
                         pId:'0',
                         name:'用户管理'

                     },{
                         id:'21',
                         pId:'2',
                         name:'用户添加'

                     },{
                         id:'22',
                         pId:'2',
                         name:'用户修改'

                     },{
                         id:'3',
                         pId:'0',
                         name:'权限管理'
                     }];
                     $.fn.zTree.init($("#ztree2"),setting2,zNodes2)
                 })
   </script>
</div>
```

###### 发送ajax加载ztree
```
<div title="面板四">
   <%--发送ajax请求获取json数据结构的ztree--%>
    <ul id="ztree3" class="ztree"></ul>
   <script type="text/javascript">
      $(function () {
         $.post("${pageContext.request.contextPath}/json/menu.json",{},function (data) {
            var setting={data:{
                simpleData:{enable:true}
            }}
            $.fn.zTree.init($("#ztree3"),setting,data)
                     },'json')
                 })
   </script>
</div>
```

###### 为ztree节点绑定事件
```
<div title="面板四">
   <%--发送ajax请求获取json数据结构的ztree--%>
    <ul id="ztree3" class="ztree"></ul>
   <script type="text/javascript">
      $(function () {
         $.post("${pageContext.request.contextPath}/json/menu.json",{},function (data3) {
            var setting3={
                data:{
                    simpleData : {
                        enable:true
                  }
               },
               callback:{
                    onClick:function(event,treeId,treeNode){
                        var page =treeNode.page;
                     if(page != undefined){
                         //判断当前选项卡是否已经打开
                        if($("#tt").tabs("exists",treeNode.name)){
                            $("#tt").tabs("select",treeNode.name)
                        }else {
                        $("#tt").tabs("add",{
                            title:treeNode.name,
                           content:'<iframe frameborder="0" width="100%" height="100%" src="'+page+'"></iframe>',
                           closable:true,
                           iconCls:'icon-edit'
                          })
                                         }
                     }
                  }
               }
            }
            $.fn.zTree.init($("#ztree3"),setting3,data3)
                     },'json')
                 })
   </script>
</div>
```

###### UML工具PowerDesigner
