---
layout: post
title: Oracle02
tags:
- java

categories: java
description: Oracle02
---
HttpServletResponse
* 响应消息行
> TTP/1.1 200
setStatus(int sc)设置响应状态码
* 响应消息头
>sendRedirect(String location)请求重定向
setHead(String name,String value)设置响应头
* 响应正文
>getWrite();字符流
getOutputSteam();字节流
setCharacterEncoding(String)
setContentType(String)
```java
public class ServletDemo1 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //服务器默认的编码为ISO-8859-1,不支持中文，tomcat规定的。告示服务器用什么编码
        resp.setCharacterEncoding("utf-8");
        //告示客户端用什么编码解析
        resp.setHeader("context-type","text/html;charset=utf-8");
        //告示服务器和客户端用什么编码解析
        resp.setContentType("text/html;charset=utf-8");
        //得到一个字符输出流
        PrintWriter out = resp.getWriter();
        //向客户端响应文本内容
        out.write("你好!");
    }
}
```
## 文件下载

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String path = this.getServletContext().getRealPath("/WEB-INF/classes/hotgril.jpg");
    String name = path.substring(path.lastIndexOf("/") + 1);
    System.out.println(name);
    name = URLEncoder.encode(name, "UTF-8");
    resp.setHeader("content-disposition","attachment;filename="+name);
    resp.setHeader("content-type","image/jpeg");
    FileInputStream fin=new FileInputStream(path);
    ServletOutputStream out = resp.getOutputStream();
    byte[]arr=new byte[8192];
    int len=-1;
    while((len=fin.read(arr))!=-1){
        out.write(arr,0,len);
    }
    fin.close();

}
```
## 验证码
###### ServletDemo4
```java
public class ServletDemo4 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException, IOException {
        int width=110;
        int height=25;
        //内存中创建一个图像对象
        BufferedImage img=new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);
        //创建一个画笔
        Graphics g=img.getGraphics();
        g.setColor(Color.PINK);//设置颜色
        g.fillRect(1,1,width-2,height-2);//填充矩形
        g.setColor(Color.RED);//设置颜色
        g.drawRect(0,0,width-1,height-1);//画矩形边框
        //设置文本样式
        g.setFont(new Font("宋体",Font.BOLD|Font.ITALIC,15));
        Random r = new Random();
        String captcha="";
        //给图片添加文本
        for(int i=1;i<5;i++){
            int num = r.nextInt(10);
            captcha+=num;
            g.drawString(num+"",20*i,20);
        }
        //添加9条干扰线
        for(int i=0;i<9;i++){
            g.drawLine(r.nextInt(width),r.nextInt(height),r.nextInt(width),r.nextInt(height));
        }
        //将图片以对象以流的方法输出给客户端
        ImageIO.write(img,"jpg",resp.getOutputStream());
        System.out.println(captcha);
    }
```
###### login.html
```html
<body>
    <form action="#" method="post">
    用户名:<input type="text" name="username"><br/>
    密码:<input type="password" name="password"><br/>
        验证码:<input type="text" name="code"><img src="/demo4" onclick="changeCode()"><a href="javascript:changeCode()">看不清换一张</a><br/>
    <input type="submit" value="登录">
    </form>
</body>
```
###### 工具类验证码ValidateCode
```java
public class ServletDemo4 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException, IOException {
        ValidateCode vc=new ValidateCode(110,25,4,9);//宽，高，验证码数量，干扰线数量
        vc.write(resp.getOutputStream());
        String code = vc.getCode();
        System.out.println(code);

    }
}
```
###### 告示客户端不缓存
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException, IOException {
    //告知不同的客户端不缓存
    resp.setHeader("pragma","no-cache");
    resp.setHeader("cache-control","no-cache");
    resp.setIntHeader("expires",0);//多少毫秒后缓存失效
}
```
## 刷新功能
```java
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//      resp.setIntHeader("refresh",1);//设置1秒刷新一次
//      Random r=new Random();
//      resp.getWriter().write(r.nextInt()+"");
        resp.setContentType("text/html;charset=utf-8");
        resp.getWriter().write("注册成功！3秒钟跳到主页");
        resp.setHeader("refresh","3;url=/demo6");
    }
```

## 请求重定向
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //方式一
    resp.setStatus(302);//告示客服端重新定向新的资源
    resp.setHeader("location","/demo8");//告示客户端要去访问那个URL
    //方式二
   resp.sendRedirect("/demo8");
}
```
## response细节
> getOutputStream和getWriter分别用于得到输出二进制数据和输出文本数据的流对象
> getOutputStream和getWriter相互排除，调用一个不能调用另一个
> Servlet程序向ServletOutputStream或PrintWriter写入数据，将被Servlet引擎(Tomcat)从response里面获取当做消息正文，与响应状态行和响应头组合输出给客服端
> Servlet的service方法结束后，Servlet引擎(Tomcat)检查getWriter或getOutputStream返回的流对象是否已经调用close方法，如果没有，Servlet引擎将调用close()方法

## HttpServletRequest
|方法|备注|
|-----|-----|
|请求消息行||
|getMethod()|获得请求方式|
|getRequestURL()|返回完整URL|
|getRequestURI|返回资源名部分|
|getContextPath()|当前应用虚拟目录/day09|
|getQueryString()|返回请求行中参数部分|
|请求消息头|
|getHeader()str|根据头名得到头信息值
|getHeaderNames()|得到所有头信息name|
|getHeaders(str)|根据头名得到相同名称的头信息值|
|请求正文|
|getParameter(name)|根据表单name获取value值|
|getParameterValues(name)|为复选框提供的方法|
|getParameterNames()|得到表单提交所有name的方法|
|getParameterMap|得到表单提交的所有值的方法,做框架用，非常实用|
|getInputStream|以字节流的方式得到所有表单数据|
|request域对象||
|void setAttribute(name,value)||
|obj getAttribute(name)||
|void removeAttribute(name)||

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    System.out.println("method:"+req.getMethod());
    System.out.println("getRequestURL:"+req.getRequestURL());
    System.out.println("getRequestURI:"+req.getRequestURI());
    System.out.println("getContextPath:"+req.getContextPath());
    System.out.println("getQueryString:"+req.getQueryString());
    String header = req.getHeader("User-Agent");
    System.out.println(header);
    if(header.toLowerCase().contains("chrome")){
        System.out.println("你使用的是谷歌浏览器");
    }else if(header.toLowerCase().contains("firefox")){
        System.out.println("你使用的是火狐浏览器");
    }else if(header.toLowerCase().contains("msie")){
        System.out.println("你使用的是IE浏览器");
    }else {
        System.out.println("你使用的其他浏览器");
    }
    Enumeration<String> names = req.getHeaderNames();
    while (names.hasMoreElements()){
        String name = names.nextElement();
        System.out.println(name+" : "+req.getHeader(name));
    }
    Enumeration<String> headers = req.getHeaders("accept-language");
    while (headers.hasMoreElements()){
        String s = headers.nextElement();
        System.out.println("相同name的消息头："+s);
    }
}
```
```javascript
//RequestDemo3
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.setCharacterEncoding("utf-8");
    String username = req.getParameter("username") ;
    String pwd = req.getParameter("pwd");
    String sex = req.getParameter("sex");
    String[] hobbies = req.getParameterValues("hobby");
    String city = req.getParameter("city");
    System.out.println("username:"+username+"\npassword:"+pwd+"\nsex:"+sex+"\nhobbies:"+ Arrays.toString(hobbies)+"\ncity:"+city);
}
//register.html
<body>
    <form action="/request3" method="post">
        用户名:<input type="text" name="username"/><br/>
        密码:<input type="password" name="pwd"><br/>
        性别:<input type="radio" name="sex" value="男" checked="checked">男
            <input type="radio" name="sex" value="女">女<br/>
        爱好:<input type="checkbox" name="hobby" value="篮球">篮球
            <input type="checkbox" name="hobby"value="唱歌">唱歌
            <input type="checkbox" name="hobby" value="编码">编码<br/>
        所在城市:<select name="city">
            <option selected="selected">----请选择----</option>
            <option value="bj">北京</option>
            <option value="sh">上海</option>
            <option value="gz">广州</option>
    </select><br/>
        <input type="submit" value="注册">
    </form>
</body>
```
```java
req.setCharacterEncoding("utf-8");
Enumeration<String> names = req.getParameterNames();
while(names.hasMoreElements()){
    String name = names.nextElement();
    String[] values = req.getParameterValues(name);
    System.out.print(name+":");
    for(String value:values){
        System.out.print(value+"\t");
    }
    System.out.println();
}
```
```java
req.setCharacterEncoding("utf-8");
Map<String, String[]> map = req.getParameterMap();
User u = new User();
for(Map.Entry<String, String[]> m:map.entrySet()){
    String name=m.getKey();
    String[]value=m.getValue();
    try {
        PropertyDescriptor pd=new PropertyDescriptor(name,User.class);
        Method setter=pd.getWriteMethod();
        if(value.length==1){
            setter.invoke(u,value[0]);
        }else {
            setter.invoke(u,(Object) value);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
System.out.println(u);
```
```java
try {
    req.setCharacterEncoding("utf-8");
    User u = new User();
    Field[] fields = u.getClass().getFields();
    Map<String, String[]> map = req.getParameterMap();
    for(Field field:fields){
        String[] arr = map.get(field.getName());
           if(field.getType()==String[].class){
                field.set(u,arr);
            }else if(field.getType()== List.class){
                field.set(u,Arrays.asList(arr));
            }else {
                field.set(u,arr[0]);
            }
    }
    System.out.println(u);
} catch (Exception e) {
    e.printStackTrace();
}
```
###### commons-beanutils.jar
```java
req.setCharacterEncoding("utf-8");
User u=new User();
try {
    BeanUtils.populate(u,req.getParameterMap());
    System.out.println(u);
} catch (Exception e) {
    e.printStackTrace();
}
```
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.setCharacterEncoding("utf-8");
    ServletInputStream sis = req.getInputStream();
    int len=0;
    byte[]arr=new byte[8194];
    while((len=sis.read(arr))!=-1){
        String s = new String(arr, 0, len);
        String s2 = URLDecoder.decode(s,"utf-8");
        System.out.println(s2);
    }
}
```
```java
//reuqest5
req.setCharacterEncoding("utf-8");
resp.setContentType("text/html;charset=utf-8");
System.out.println("A:我想办事");
System.out.println("B:我办不了，但是我可以找人给你办");
req.setAttribute("s","aaa");
//req.getRequestDispatcher("/request6").forward(req,resp);
//resp.sendRedirect(req.getContextPath()+"/request6?s=aaa");
req.getRequestDispatcher("/request6").include(req,resp);
//再返回到原来的servlet执行response的输出（如果有）
resp.getWriter().println("我还能输出呢");//request6
resp.setContentType("text/html;charset=utf-8");
System.out.println("这事我能办！");
String s =(String) req.getAttribute("s");
String s1 = req.getParameter("s");
System.out.println(s1);
System.out.println(s);
```
