# Java EE

#### ServletContext

##### 1、概念：代表整个web应用，可以和程序的容器（服务器）来通信

##### 2、获取：

###### 		1、通过request对象获取

​				request.getServletContext()

###### 		2、通过HttpServlet获取

​				this.getServletContext()

```java
ServletContext context1 = request.getServletContext();      //通过request对象获取
ServletContext context2 = this.getServletContext();         //通过HttpServlet对象获取
System.out.println(context1 == context2);  //True 获取到的为同一个context
```

##### 3、功能：

###### 		1、获取MIME类型：

				* MIME类型：在互联网通信过程中定义的一种文件数据类型
				* 格式：  大类型/小类型           text/html       image/jpeg
				* 获取：  String getMimeType(String file)

```java
ServletContext context = this.getServletContext(); //通过HttpServlet对象获取
String filename = "a.html";                        //定义文件名称
String mimeType = context.getMimeType(filename);   //获取MIME类型
System.out.println(mimeType);
```

###### 		2、域对象：共享数据

​				1.setAttribute(String name,Object value)

​				2.getAttribute(String name)

​				3.removeAttribute(String name)

​				* ServletContext对象范围：所有用户所有请求的数据

###### 		3、获取文件的真实（服务器）路径：

​				方法：String getRealPath(String path)

						*  "/"代表根目录下的web目录
						*  "src"目录下的路径为”/WEB-INF/classes/“

```
String path = context2.getRealPath("/register.jsp");  //获取web目录下文件的真实路径
String src = context2.getRealPath("/WEB-INF/classes/ContextDemo8");//获取src目录下文件的真实路径
```



#### Cookie

​		Cookie存在于客户端（浏览器端）

##### Cookie的创建

```java
Cookie c =new Cookie("name","value");    //创建Cookie
c.setPath("/");              //设置共享目录为根目录,cookie即可被服务器上所有项目共享
c.setDomain("域名");        //设置共享域名,可以被不同服务器上的项目获取
response.addCookie(c);                   //发送Cookie
```

##### Cookie的接收

```java
Cookie[] cs = request.getCookies();          //获取Cookie
//遍历Cookies，获取数据
if(cs != null){
            for (Cookie c : cs) {
                String name = c.getName();  //获取Cookie的键
                String value = c.getValue();//获取Cookie的值
                System.out.println("name:"+name+"\tValue:"+value);
            }
```

##### Cookie的URL编码

```java
//用URL编码可以发送中文Cookie（Tomcat8以上可以直接包含中文）
//发送时编码
String name = URLEncoder.encode("姓名", "UTF-8");
String value = URLEncoder.encode("张三", "UTF-8");
Cookie c = new Cookie(name, value);
//接收时解码
String value = URLDecoder.decode(c.getValue(), "UTF-8");
```

##### Cookie的存活时间

**cookie.setMaxAge()**   

括号中正数为存活时间xx秒

  		 负数为浏览器关闭后删除

​			0为立即删除

```java
Cookie cookie = new Cookie("testCookie","xiaowen");
cookie.setMaxAge(30);         //设置cookie持久化到硬盘，30秒后自动删除
cookie.setMaxAge(-1);         //设置cookie浏览器关闭后删除
cookie.setMaxAge(3000);       //设置cookie持久化到硬盘，300秒后自动删除
cookie.setMaxAge(0);          //设置cookie直接被删除
response.addCookie(cookie);
```

#### Session

##### 1、概念：

​		服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中。 HttpSession

##### 2、获取HttpSession对象：

​		HttpSession session = request.getSession();

##### 3、使用HttpSession对象：

​		Object getAttribute(String name)

​		void setAttribute(String name,Object Value)

​		void removeAttribute(String name)

##### 4、原理：

​		Session存在于服务器端

​		Session依赖于Cookie

##### 5、细节：

###### 		1、客户端关闭后，服务器不关闭，两次获取的session是否为同一个？

   * 默认情况下不是同一个

   * 如果需要相同，可以创建Cookie，键为JSESSIONID，设置最大存活时间，让Cookie持久化保存

     ```java
     Cookie c = new Cookie("JSESSIONID",session.getId());
     c.setMaxAge(60*60);
     response.addCookie(c);
     ```

###### 		2、客户端不关闭，服务器端关闭，两次获取的session是否为同一个？

  * 不是同一个，但是要确保数据不丢失。tomcat自动完成以下工作

     * session的钝化：
       * 在服务器**正常**关闭之前，将session对象系列化到硬盘上。
     * session的活化：
        * 在服务器启动后，将session文件转化为内存中的session对象即可。

    ###### 3、session什么时候被销毁？

    1、服务器关闭

    2、session对象调用invalidate()

    3、session默认失效时间：30分钟

    ​		选择性配置修改

    ```xml
    <session-config>
    	<session-timeout>30</session-tumeout>
    </session-config>
    ```

    ##### 6、session的特点

    ​		1、session用于存储一次会话的多次请求的数据，存在服务器端

    ​		2、session可以存储任意类型、任意大小的数据

      * session与cookie的区别：
        		* session存储数据在服务器端，cookie在客户端
            		* session没有数据大小限制，cookie有
              		* session数据安全，cookie相对于不安全

##### Session的创建

```java
HttpSession session =request.getSession();      //获取session
session.setAttribute("msg","hello session");     //存储数据
```

##### Session的接收

​	Session只能在一次对话中共享数据，浏览器关闭即失效

```java
HttpSession session = request.getSession();
Object msg = session.getAttribute("msg");       //获取session
System.out.println("msg:"+msg);                                
```

# Servlet

​		server applet

##### 概念:	运行在服务器端的小程序

* Servlet就是一个接口，定义了Java类被浏览器访问（tomcat识别）的规则

* c定义一个类，实现Servlet接口
  * public class ServletDemo1 implements Servlet
* 实现接口中的抽象方法
* 配置Servlet
  * 在web.xml中配置

```xml
<!-- 配置Servlet -->
<servlet>
	<servlet-name>demo1</servlet-name>
    <servlet-class>cn.servlet</servlet-class>
</servlet>

<servlet-mapping>
	<servlet-name>demo1</servlet-name>
    <url-pattern>/demo1</url-pattern>
</servlet-mapping>
```

##### 执行原理

* 当服务器接收到客户端浏览器端请求后，会解析请求URL路径，获取访问的Servlet的资源路径
  	* 查找web.xml文件，是否有对应的<url-pattern>标签体内容
  	* 如果有，则再查找对应的<servlet-class>全类名
      	* tomcat会将字节码文件加载进内存，并且创建其对象
 	* 调用其方法



#####  Servlet中的生命周期方法：

1. 被创建：执行init方法，只执行一次
   * Servlet什么时候被创建？
     * 默认情况下，第一次被访问时，Servlet被创建
     * 可以配置执行Servlet的创建时机
       1. 第一次被访问时，创建
          * <load-on-startup>的值为负数
       2. 在服务器启动时，创建
          * <load-on-startup>的值为0或正整数
   * Servlet的init方法，只执行一次，说明Servlet在内存中只存在一个对象，Servlet是单例的
     * 多个用户同时访问时，可能存在线程安全问题
       * 解决：尽量不要在Servlet中定义成员变量。即使定义了成员变量，也不要修改值
2. 提供服务：执行service方法，执行多次
   * 每次访问Servlet时，service方法都会被调用一次
3. 被销毁：执行destroy方法，只执行一次
   * Servlet被销毁时执行。服务器关闭时，Servlet被销毁
   * 只有服务器正常关闭时，才会执行destroy方法
   * destroy方法在Servlet被销毁之前执行，一般用于释放资源

#####  Servlet 3.0

* 支持注解配置，可以不需要web.xml了
* 创建一个JavaEE项目，选择Servlet版本3.0以上，可以不创建web.xml
* 在类上使用@WebServlet注解，进行配置
  * @WebServlet("URL名称")

##### Servlet相关配置

* 一个Servlet可以定义多个访问路径 : @WebServlet({"/url1","/url2","/url33"})
* 路径定义规则：
  1. 可以定义为/xxx/*     /xxx/后任意匹配
  2. 可以定义为/*            优先级最低，匹配不到才会被使用
  3. 可以定义为 *.do       扩展名匹配    前面不要加/

# IDEA与Tomcat的相关配置

1. IDEA会为每一个tomcat部署的项目单独建立一份配置文件
   * 查看控制台的log ： Using CATALINA_BASE:   "C:\Users\lan 人\.IntelliJIdea2019.2\system\tomcat\Tomcat_8_5_50_BiLibili"
2. 工作空间项目    和    tomcat部署的web项目
   * tomcat真正访问的是“tomcat部署的web项目”，“tomcat部署的web项目”对应着"工作空间项目"的web目录下的所有资源
   * WEB-INf目录下的资源不能被浏览器直接访问。
3. 断点调试：使用”debug“启动