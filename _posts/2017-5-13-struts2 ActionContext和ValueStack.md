---
layout: post
title: struts2 ActionContext和ValueStack
tags:
- java

categories: java
description:struts2 ActionContext和ValueStack
---
## contextMap
> 在每次动作执行之前核心控制器StrutsPrepareAndExecuteFilter都会创建ActionContext和ValueStack对象，且每次访问都会创建
ValueStatck 是个list集合
ActionContext是一个Map<String,Map>
|key|value|
|----|------|
|applicaton|map|
|session|map|
|request|map|
|attr|map|
## ActionContext的存取
> OGNL取值  #key+名称
```
public class Demo1Action extends ActionSupport {
    @Override
    public String execute() throws Exception {
        ActionContext context = ActionContext.getContext();//从当前线程局部变量获得
        context.put("contextMap","helloContextMap");
        Map<String,Object>sessionAttribute= (Map<String, Object>) context.get("session");
        sessionAttribute.put("name1","张三");
        context.getSession().put("name2","李四");
        ServletActionContext.getRequest().getSession().putValue("name3","王五");
        context.getApplication().put("name1","刘亦菲");
        ServletActionContext.getServletContext().setAttribute("name2","周芷若");
        return SUCCESS;
    }
}

//demo1.jsp
<%@ taglib prefix="s" uri="/struts-tags" %>
<html>
<head>
    <title>取ActionContext数据</title>
</head>
<body>
    <s:debug></s:debug>
    <!--#key+名称-->
    <s:property value="#contextMap"/>
    <s:property value="#session.name1"/>
    <s:property value="#application.name1"/>
</body>
```

## ValueStack存取
> 只能取元素的属性 取对象属性时不写#
> 从栈顶查找，查找到就不找了
> 获取指定位置的属性 [index].属性名
> setValue(expr,value) expr使用#就存到ContextMap中，没有就存到ValueStack中，在valuestack中找到第一个name赋值，如果不到就报错
|方法|备注|
|----|----|
|ActionContext.getContext().getValueStack()|获得ValueStack对象|
|ActionContext.getContext().get("request")|获得request域|
|request.get("struts.valueStack")|获得ValueStack对象|
|vs.push(new Student("戴斯特",21))|压到栈顶|
|vs.setValue("#name","张三")|expr使用#就存到ContextMap中，没有就存到ValueStack中|
|vs.setValue("name","李四")|在valuestack中找到第一个name赋值，如果不到就报错|
|&lt;s:debug></s:debug>|jsp中显示ActionContext和ValueStack域的数据|
|&lt;s:property value="name"/>|jsp中获取valuestack元素属性,如果没有找到就到ActionContext继续查找|
|&lt;s:property value="[1].name"/>|jsp中获取valueStack中第二个元素属性以后查找第一个name|
|&lt;s:poperty value="[index].field"|去掉前面index元素，从栈顶开始往下查找|
|set(String key,Object o)|如果栈顶是map,map.put(key,o),如果不是map创建map map.put(key,o)|
|&lt;s:property/>|默认取栈顶元素|


```
public class Demo2Action extends ActionSupport {
    @Override
    public String execute() throws Exception {
        HttpServletRequest request = ServletActionContext.getRequest();
        ValueStack vs1 = (ValueStack) request.getAttribute("struts.valueStack");
        System.out.println(vs1.hashCode());
        Map<String, Object> req = (Map<String, Object>) ActionContext.getContext().get("request");
        ValueStack vs2 = (ValueStack) req.get("struts.valueStack");
        System.out.println(vs2.hashCode());
        ValueStack vs = ActionContext.getContext().getValueStack();
        System.out.println(vs.hashCode());
        vs.push(new Student("戴斯特",21));
        vs.push(new Student("戴斯特2",23));
        //setValue(expr,value) expr使用#就存到ContextMap中，没有就存到ValueStack中
        vs.setValue("#name","张三");
        vs.setValue("name","李四");//在valuestack中找到第一个name赋值，如果不到就报错
        /**
         * set(String key,Object o)
         * String key:map的key
         * Object o:map的value
         * 如果栈顶是一个Map元素,直接把key作为map的key,把Object作为map的value.存入栈顶
         * 如果栈顶不是一个map,创建一个map对象，把key作为map的Key,把Object作为map的value亚入栈顶
         *
         */
        vs.set("s1",new Student("王五",18));
        vs.push(new Student("test",23));
        vs.push(new Student("test2",23));
        System.out.println(vs.getRoot());
        return SUCCESS;
    }
}


demo2.jsp
<body>
    <s:debug></s:debug>
    <!--只能取元素的属性 取对象属性时不写 从下标位置开始往下查找#-->
    <s:property value="name"/>
    <s:property value="[1].name"/>
    <s:property value="[2].name"/>
    <s:property value="[3].name"/>
    <s:property value="[4].name"/>
    <s:property value="[2].s1.name"/>
    <s:property/>
    <%//模拟原理:其实全是ValueStack的findValue和findString
        ValueStack vs= ActionContext.getContext().getValueStack();
        Object obj=vs.findValue("name");
        out.print("<br/>---------------------<br/>");
        out.print(obj);

    %>
</body>
```
## struts2对el表达式的改变
EL 查找顺序: page--request--session--application
struts2 el表达式查找顺序: page scope --- request scope ---valueStack --- contextMap --- session scope ---application scope
```
public class Demo3Action extends ActionSupport{
    private String name="动作类中的name";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String execute() throws Exception {
        HttpServletRequest request = ServletActionContext.getRequest();
        //request.setAttribute("name","请求域中的name");
        ServletContext applicaton = ServletActionContext.getServletContext();
        applicaton.setAttribute("name","应用域中的name");
        ActionContext context = ActionContext.getContext();
        context.put("name","actionContext的name");
        return SUCCESS;
    }
}


//demo3.jsp
EL表达式:${name }<!--pageContext.findAttribute('name')-->
<br/>
OGNL表达式:<s:property value="name"/>
<s:debug/>
```

## s:iterator
|属性|备注|
|-----|-----|
|value属性|要遍历的集合，是OGNLB表达式|
|有var属性|把var的值当作key,把当前元素作为value,存到ActionContext这个大Map中|
|无var属性|把当前遍历的元素压入栈顶|
|status属性|遍历时的一些计数信息，也是存在到ActionContext的map中 .index索引 .count计数|
```
public class Demo4Action extends ActionSupport {
    private List<Student> students;
    @Override
    public String execute() throws Exception {
        students=new ArrayList<>();
        students.add(new Student("张三",13));
        students.add(new Student("李四",14));
        students.add(new Student("王五",15));
        students.add(new Student("赵六",16));
        return SUCCESS;
    }

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }
}

//demo4.jsp
<body>
    <table width="500px" border="1" align="center">
        <tr>
            <th>序号</th>
            <th>姓名</th>
            <th>年龄</th>
        </tr>
        <%--
        value属性:要遍历的集合，是OGNLB表达式
        var属性:如果写了该属性:把var的值当作一个key,把当前遍历的元素作为value,存到ActionContext这个大Map中
               如果不写
        --%>
        <s:iterator value="students" var="s" status="vs">
            <tr>
                <td><s:property value="#vs.index"/></td>
                <td><s:property value="#s.name"/></td>
                <td><s:property value="#s.age"/></td>
            </tr>
        </s:iterator>
    </table>
    <hr/>
    <table width="500px" border="1" align="center">
        <tr>
            <th>序号</th>
            <th>姓名</th>
            <th>年龄</th>
        </tr>
        <%--
        value属性:要遍历的集合，是OGNLB表达式
        var属性:如果写了该属性:把var的值当作一个key,把当前遍历的元素作为value,存到ActionContext这个大Map中
               如果不写 把当前遍历的元素压入栈顶
        --%>
        <s:iterator value="students" status="vs">
            <tr>
                <td><s:property value="#vs.count"/></td>
                <td><s:property value="name"/></td>
                <td><s:property value="age"/></td>
            </tr>
        </s:iterator>
    </table>
    <s:debug/>
</body>
```
## 添加过滤条件
|表达式|备注|
|-------|------|
|a.{?#this }|过滤所有符合条件的集合，如:users.{?#this.age > 19}|
|b.{^#this }|过滤第一个符合条件的元素，如：users.{^#this.age > 19}|
|c.{\$#this }|过滤最后一个符合条件的元素,如:users.{$#this.age > 19}|
|students.{name}|只取指定的字段|
```
<hr/>
<table width="500px" border="1" align="center">
    <tr>
        <th>序号</th>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <s:iterator value="students.{?#this.age>21}" status="vs">
        <tr>
            <td><s:property value="#vs.count"/></td>
            <td><s:property value="name"/></td>
            <td><s:property value="age"/></td>
        </tr>
    </s:iterator>
</table>
<%--OGNL 指定输出内容--%>
<hr/>
<table width="500px" border="1" align="center">
    <tr>
        <th>序号</th>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <s:iterator value="students.{name}" status="vs">
        <tr>
            <td><s:property value="#vs.count"/></td>
            <td><s:property value="name"/></td>
            <td><s:property value="age"/></td>
        </tr>
    </s:iterator>
</table>
```
## struts2中的一些其他标签
|标签|备注|
|----|----|
|&lt;s:set var="str" value="'test'"/>|存入ActionContext map中 var作为key value为表达式|
|&lt;s:action name="action1" executeResult="true"/>|name指定一个动作名称,executeResult 是否执行动作，结果包含|
|&lt;s:if test="#grade=='D'">差</s:if>|
|&lt;s:elseif test="#grade=='C'">中</s:elseif>|
|&lt;s:else>其他</s:else>|
|&lt;s:url value="action1"/>|直接把取值输出到页面|
|&lt;s:url action="action1"/>|把请求地址输出到页面|
|&lt;s:url action="action1" var="url"/>|存入到ActionContext map中|

##### \#
> 在contextMap中key时使用,例如:<s:property value="#name"/>
> OGNL中创建map对象使用,例如:<s:radio list="#{'male':"男",'female':'女'}"
##### \$
> 在jsp中使用EL表达时使用,列如${name}
> 在xml配置文件中,编写OGNL表达式使用,假如文件下载,文件名编码
     struts.xml---${@java.net.URLEncoder.encode(filename)}
##### %
> 把普通字符串当成OGNL <s:textfield value="%{username}"/>

## checkboxlist
```
public class Demo6Action extends ActionSupport {
    //初始化表单的爱好列表
    private String[] hobbyarr=new String[]{"吃饭","睡觉","敲代码"};
    //用户提交表单时的数据 封装到此属性中
    private String hobby;
    public String save(){
        System.out.println(hobby);
        return NONE;
    }

    public String getHobby() {
        return hobby;
    }

    public void setHobby(String hobby) {
        this.hobby = hobby;
    }

    public String[] getHobbyarr() {
        return hobbyarr;
    }

    public void setHobbyarr(String[] hobbyarr) {
        this.hobbyarr = hobbyarr;
    }
}

//demo6.jsp
<s:form action="save">
    <s:checkboxlist list="hobbyarr" name="hobby"></s:checkboxlist>
    <s:submit value="提交"/>
</s:form>
<s:debug/>

public class Demo7Action extends ActionSupport implements ModelDriven<Customer> {
    private Customer customer=new Customer();
    @Override
    public Customer getModel() {
        return customer;
    }

    public Customer getCustomer() {
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }
    public String save(){
        System.out.println(customer);
        return NONE;
    }
}
//demo7.jsp
<body>
    <s:form action="saveCustomer">
        <s:textfield name="name" label="用户名"></s:textfield>
        <s:password name="password" label="密码"></s:password>
        <s:checkbox name="married" label="已婚" value="true"></s:checkbox>
        <s:checkboxlist list="{'吃饭','睡觉','敲代码'}" name="hobby" label="爱好"></s:checkboxlist>
        <s:select list="#{'bj':'北京','sh':'上海','sz':'苏州'}" name="city" label="故乡" headerValue="---请选择---"></s:select>
        <s:textarea name="description" label="个人介绍" rows="5" cols="25"></s:textarea>
        <s:radio list="#{'male':'男','frmale':'女'}" name="gender" value="'male'"></s:radio>
        <s:submit value="提交" theme="simple"/>
        <s:reset value="重置" theme="simple"/>
    </s:form>
    <hr/>
    <s:form action="saveCustomer" theme="simple">
        <s:textfield name="name" label="用户名"></s:textfield>
        <s:password name="password" label="密码"></s:password>
        <s:checkbox name="married" label="已婚" value="true"></s:checkbox>
        <s:checkboxlist list="{'吃饭','睡觉','敲代码'}" name="hobby" label="爱好"></s:checkboxlist>
        <s:select list="#{'bj':'北京','sh':'上海','sz':'苏州'}" name="city" label="故乡" headerValue="---请选择---"></s:select>
        <s:textarea name="description" label="个人介绍" rows="5" cols="25"></s:textarea>
        <s:radio list="#{'male':'男','frmale':'女'}" name="gender" value="'male'"></s:radio>
        <s:submit value="提交" theme="simple"/>
        <s:reset value="重置" theme="simple"/>
    </s:form>
    <%--<constant name="struts.ui.theme" value="simple"/>--%>
</body>
```
## 表单重复提交
###### 模拟原理
```
@WebServlet("/servlet/LoginServlet.servlet")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        String username = req.getParameter("username");
        String fcode = req.getParameter("fcode");
        String scode = (String) req.getSession().getAttribute("scode");
        if(fcode.equals(scode)){
            //第一次提交
            req.getSession().removeAttribute("scode");
            resp.getWriter().println("第一次提交 成功");
        }else {
            if(scode==null||"".equals(scode)){
                //重复提交
                resp.getWriter().println("重复提交");
            }else {
                //输入的验证码不正确
                resp.getWriter().println("验证码不正确");
            }
        }
        System.out.println(username);
        System.out.println(fcode);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}
```

```
//login.jsp login2.jsp
<body>
  <s:form action="login.action">
    <s:token/>
    <s:textfield name="name" label="用户名"/>
    <s:submit value="提交"/>
  </s:form>
</body>

//struts.xml
<package name="p2" extends="struts-default">
    <action name="login" class="com.itheima.web.action.LoginAction" method="login">
        <interceptor-ref name="defaultStack"></interceptor-ref>
        <!--拦截器 通过令牌避免重复提交-->
        <interceptor-ref name="token"></interceptor-ref>
        <result type="redirect">/success.jsp</result>
        <result name="invalid.token">/message.jsp</result>
    </action>
    <action name="login2" class="com.itheima.web.action.Login2Action" method="login">
        <interceptor-ref name="defaultStack"></interceptor-ref>
        <!--只处理第一次，以后的不再处理-->
        <interceptor-ref name="tokenSession"></interceptor-ref>
        <result type="redirect">/success.jsp</result>
        <result name="invalid.token">/message.jsp</result>
    </action>
</package>
```
