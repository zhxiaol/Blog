---
layout: post
title: Listener和Filter
tags:
- java

categories: java
description: Listener和Filter
---
## javaweb常见监听
```
监听域对象创建销毁
	|
	|--监听ServletContext创建和销毁
    |		    |
	|		    |--ServletContextListener
	|
	|--监听HttpSession创建和销毁
	|			|
	|			|--HttpSessionListener
	|
	|--监听HttpServletRequest
	|		    |
	|			|--ServletRequestListener
	|

监听域对象的属性变化
	|
	|--ServletContext属性变化
	|			|
	|			|--ServletContextAttributeListener
	|
	|--HttpSession属性变化
	|			|
	|			|--ServletSessionAttributeListener
	|			
	|--HttpServletRequest属性变化
	|			|
	|			|--ServletRequestAttributeListener


监听Session绑定JavaBean
	|
	|--用于监听javaBean是否绑定到了session域
	|		   |
	|		   |--HttpSessionBindingListener
    |                       |
    |                       |--JavaBean 实现HttpSessionBindingListener和Serializable  
	|
	|--用于监听javaBean对象的活化和钝化
	|		   |
	|		   |--HttpSessionActivationListener

```

## 创建监听器步骤
```
创建监听器步骤
	|
	|--创建一个类，实现指定的监听器接口
	|
	|--重写接口中的方法
	|
	|--在web.xml文件中对监听器注册
	|		 |
	|		 |-- <listener>
    |    			<listener-class></listener-class>
   	|			  </listener>
    |
```
##### ServletRequestListener
```
public class MyServletRequestListener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("servletRequest销毁了");
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("servletRequest创建了");
    }
}
```
###### ServletRequestAttributeListener
```
public class MyRequestAttributeListener implements ServletRequestAttributeListener{
    @Override
    public void attributeAdded(ServletRequestAttributeEvent srae) {
        System.out.println("servletRequest添加属性 name:"+srae.getName()+" value:"+srae.getValue());
    }

    @Override
    public void attributeRemoved(ServletRequestAttributeEvent srae) {
        System.out.println("servletRequest移出属性 name:"+srae.getName()+" value:"+srae.getValue());
    }

    @Override
    public void attributeReplaced(ServletRequestAttributeEvent srae) {
        System.out.println("servletRequest替换属性 name:"+srae.getName()+" value:"+srae.getValue());
    }
}
```
###### HttpSessionBindingListener
```
public class User implements HttpSessionBindingListener,Serializable{
    private String name;
    private int age;

    @Override
    public void valueBound(HttpSessionBindingEvent event) {
        System.out.println("session绑定user对象");
    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent event) {
        System.out.println("session解除user绑定");
    }
}
```
###### 定时移出session
```
//myServletContextListener
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContext application=sce.getServletContext();
        final ArrayList<HttpSession> list = new ArrayList<>();
        application.setAttribute("sessions",list);
        Timer t = new Timer();
        t.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("开始扫描了");
                for(int i=0;i<list.size();i++){
                    HttpSession session = list.get(i);
                    long l = System.currentTimeMillis() - session.getLastAccessedTime();
                    if(l>5000){//如果时间大于5秒 把session销毁
                        list.remove(session);
                        System.out.println("session 移出了 id:"+session.getId());
                        i--;
                        try {
                            session.invalidate();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        },2000,5000);
    }
}
//MySessionListener
public class MySessionListener implements HttpSessionListener {
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        HttpSession session = se.getSession();
        ServletContext application = session.getServletContext();
        ArrayList<HttpSession> list = (ArrayList<HttpSession>) application.getAttribute("sessions");
        list.add(session);
        System.out.println("添加了session id:"+session.getId());
    }
}
```
## Filter
> javaweb中的过滤器可以拦截所有访问web资源的请求或响应操作
```
Filter使用步骤
	 |
	 |--创建一个类实现Filter接口
	 |
	 |--重写接口方法，doFilter方法是真正过滤的
	 |			|
	 |			|--filterChain.doFilter(req,resp)
	 |
	 |--在web.xml文件中配置
	 |		 |
	 |		 |--<filter>
     |   	 |		<filter-name>MyFilter</filter-name>
     |   	 |		<filter-class>com.itheima.filter.MyFilter</filter-class>
     |		 |	</filter>
   	 |		 |   <filter-mapping>
     |  	 |		<filter-name>MyFilter</filter-name>
     |   	 |		<url-pattern>/*</url-pattern>
     |		 |	</filter-mapping>
     |
     |--Filter生命周期
     |		|
     |		|--init方法 当服务器启动会创建Filter对象并调用一次init方法
     |		|--doFilter方法 当访问资源时，路径与Filter的拦截路径匹配时调用该方法
     |		|--destory方法 当服务器关闭时会调用该方法销毁
     |
     |--Filter配置
     		|
     		|--<url-pattern>
     		|		|
     		|		|--完全匹配 以‘/’开始不包含通配符*
     		|		|--目录匹配 以‘/’开始 *结尾
     		|		|--扩展名匹配 以‘.xxx’
     		|
     		|--<servlet-name>
     		|       |
     		|       |--对指定servlet名称的servlet拦截
     		|
     		|--<dispatcher>
     				|
     				|--REQUEST 拦截直接访问
     				|--FORWARD 拦截转发
     				|--ERROR 拦截异常 拦截error-page
     				|--INCLUDE 拦截包含请求
```
###### Filter
```
//MyFilter
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("doFilter过滤开始");
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("doFilter过滤结束");
    }
}
//web.xml
<filter>
    <filter-name>MyFilter</filter-name>
    <filter-class>com.itheima.filter.MyFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MyFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
## FilterChain
> FilterChain是servlet容器为开发人员提供的对象，他提供了对某些资源的已过滤请求调用链的视图，过滤器使用FilterChain 调用链中的下一个过滤器，如果调用的过滤器是链中最后一个过滤器，则调用链末尾的资源
> 执行顺序由<filter-mapping>的顺序决定
## FilterConfig
> 在Filter的init方法有一个参数FilterConfig,它是Filter的配置对象
> + 获取Filter名称
> + 获取Filter初始化参数
> + 获取ServletContext对象
###### FilterConfig
```
public class MyFilterConfigTest implements Filter {
    private FilterConfig filterConfig;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        this.filterConfig=filterConfig;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        String encoding = filterConfig.getInitParameter("encoding");
        System.out.println("filterConfig取得初始化值:"+encoding);
        servletRequest.setCharacterEncoding(encoding);
        filterChain.doFilter(servletRequest,servletResponse);//放行
    }
}
```
###### Filter配置
```
<filter>
    <filter-name>MyFilter3</filter-name>
    <filter-class>com.itheima.filter.MyFilter3</filter-class>
</filter>
<filter-mapping>
    <filter-name>MyFilter3</filter-name>
    <url-pattern>/servlet/demo2</url-pattern><!--完全匹配-->
    <url-pattern>*.do</url-pattern><!--扩展名匹配-->
    <url-pattern>/*</url-pattern><!--目录匹配-->
    <servlet-name>FirstServlet</servlet-name><!--servlet名称拦截-->
    <dispatcher>FORWARD</dispatcher><!--转发拦截-->
    <dispatcher>REQUEST</dispatcher><!--请求拦截-->
    <dispatcher>INCLUDE</dispatcher><!--包含拦截-->
    <dispatcher>ERROR</dispatcher><!--异常拦截 拦截error-page-->
</filter-mapping>

<error-page>
    <!--第一种方式-->
    <exception-type>java.lang.Exception</exception-type>
    <!--第二种方式-->
    <error-code>404</error-code>
    <location>/error.jsp</location>
</error-page>
```
## MD5加密
> mysql md5加密 update 表名 set 字段名 = md5(字段名);
###### java md5加密
```
public static String md5(String plainText){
    byte[]secretBytes=null;
    try {
        secretBytes=MessageDigest.getInstance("md5").digest(plainText.getBytes());
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
    String md5code=new BigInteger(1,secretBytes).toString(16);
    for(int i=0;i<32-md5code.length();i++){
        md5code+="0"+md5code;
    }
    return md5code;
}
```
###### 自动登录
```
//LoginServlet
public class LoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        password= MD5Util.md5(password);
        UserService service=new UserService();
        User u=service.findUser(username,password);
        if(u!=null){
            String autologin=req.getParameter("autologin");
            Cookie cookie=new Cookie("user",username+"&"+password);
            cookie.setPath("/");
            if(autologin!=null){//自动登录
                cookie.setMaxAge(60*60*24*7);
            }else{//清除cookie登录数据
                cookie.setMaxAge(0);
            }
            resp.addCookie(cookie);
            req.getSession().setAttribute("user",u);
            req.getRequestDispatcher("/home.jsp").forward(req,resp);
        }else{
            req.setAttribute("msg","用户名或者密码错误");
            req.getRequestDispatcher("/login.jsp").forward(req,resp);
        }
    }
}

//AutoLoginFilter
public class AutoLoginFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req= (HttpServletRequest) servletRequest;
        HttpServletResponse resp= (HttpServletResponse) servletResponse;
        String uri = req.getRequestURI();
        String path=uri.substring(uri.lastIndexOf("/")+1);
        System.out.println(path);
        if(!"login.jsp".equals(path)||"loginServlet".equals(path)) {
            User user = (User) req.getSession().getAttribute("user");
            if (user == null) {
                Cookie[] cookies = req.getCookies();
                for (int i = 0; cookies != null && i < cookies.length; i++) {
                    Cookie cookie = cookies[i];
                    if ("user".equals(cookie.getName())) {
                        String str = cookie.getValue();
                        String[] arr = str.split("&");
                        if (arr != null && arr.length > 1) {
                            UserService service = new UserService();
                            User u = service.findUser(arr[0], arr[1]);
                            if (u != null) {
                                req.getSession().setAttribute("user", u);
                            }
                        }
                    }
                }
            }
        }
        filterChain.doFilter(req,resp);
    }
}

```

###### 全局编码过滤
```
public class GlobarFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req= (HttpServletRequest) servletRequest;
        HttpServletResponse resp= (HttpServletResponse) servletResponse;
        req.setCharacterEncoding("utf-8");
        filterChain.doFilter(new MyRequest(req),resp);
    }

    @Override
    public void destroy() {

    }
    class MyRequest extends HttpServletRequestWrapper{
        HttpServletRequest request;
        public MyRequest(HttpServletRequest request) {
            super(request);
            this.request=request;
        }

        @Override
        public String getParameter(String name) {
            Map<String, String[]> map = getParameterMap();
            String[] values = map.get(name);
            return values[0];
        }

        @Override
        public String[] getParameterValues(String name) {
            Map<String, String[]> map = getParameterMap();
            String[] values = map.get(name);
            return values;
        }

        @Override
        public Map<String, String[]> getParameterMap() {
            Map<String, String[]> map = request.getParameterMap();
            if(flag) {
                for (Map.Entry<String, String[]> m : map.entrySet()) {
                    String[] values = m.getValue();
                    for (int i = 0; values != null && i < values.length; i++) {
                        try {
                            values[i] = new String(values[i].getBytes("iso-8859-1"), "UTF-8");
                        } catch (UnsupportedEncodingException e) {
                            e.printStackTrace();
                        }
                    }
                }
                flag=false;
            }
            return map;
        }
        private  boolean flag=true;
    }
}
```
