# Java EE

## IDEA中那些快捷键

#### 输出语句		sout

#### 遍历语句		iter

#### 生成set/get方法		右键generate	或者	alt + insert

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

# MVC:开发模式

* MVC：

  1. M：Model，模型。JavaBean
     * 完成具体的业务操作，如：查询数据库、封装对象
  2. V：View，视图。JSP
     * 展示数据
  3. C：Controller，控制器。Servlet
     * 获取用户的输入
     * 调用模型
     * 将数据交给视图进行展示

  * 优缺点：
    1. 优点：
       * 耦合性低，方便维护，可以利于分工协作
       * 重用性高
    2. 缺点：：
       * 使得项目架构变得复杂，对开发人员要求高

## EL表达式

#### 概念：Expression Language 表达式语言

#### 作用：替换和简化jsp页面中java代码的编写

#### 语法：${表达式}

#### 注意：

* jsp默认支持EL表达式
  * 想要忽略EL表达式：
    1. 设置jsp中page指令中：isELIgnored="true" 忽略当前jsp页面中的所有EL表达式
    2. ${表达式}   前面加上\：忽略当前这个EL表达式

#### 使用：

1. 运算：
   * 运算符：
     1. 算术运算符：+ - * /(div) %(mod)
     2. 比较运算符：>  <  >=  <=  ==  !=
     3. 逻辑运算符：&&(and)  ||(or)  !(not)
     4. 空运算符：empty
        * 功能：用于判断字符串、集合、数组对象是否为null或者长度是否为0，
        * ${empty list}：判断字符串、集合、数组对象是否为null或者长度为0
        * ${not empty str}：表示判断字符串、集合、数组对象是否不为null 并且 长度>0
2. 获取值
   1. EL表达式只能从域对象中获取值
   
   2. 语法：
      1. ${域名称.键名}：从指定域中获取指定键的值
         * 域名称：
           1. pageScope  --->   pageContext
           2. requestScope  --->  request
           3. sessionScope  --->  session
           4. applicationScope  --->  applocation(ServletContext)
         * 举例：在request域中存储了name=张三
         * 获取：${requestScope.name}
      2. ${键名}：表示依次从最小的域中查找是否有该键对应的值，直到找到为止
      3. 获取对象、List集合、Map集合的值
         1. 对象：${域名称.键名.属性名}
            
            * 本质上会去调用对象的getter方法
            
         2. List集合：${域名称.键名[索引]}
         
         3. Map集合：
         
            * ${域名.键名.key名称}
         
            * ${域名.键名["key名称"]}
         
              ```java
                  <h3>el获取Map中的值</h3>
                  ${map}<br>
                  ${map.sattr}    ${map["sattr"]}<br>
                  ${map.sname}<br>
                  ${map.user.name}-------${map["user"]["age"]}-----${map["user"].name}
              ```
      
   3. 隐式对象：
   
      * EL表达式中有11个隐式对象
      * pageContext：
        * 获取jsp其他八个内置对象
          * ${pageConext.request.contextPath}：动态获取虚拟目录

# JSTL

#### 概念：JavaServer Pages Tag Library     JSP标注标签库

				* 是由Apache组织提供的开源的免费的jsp标签

#### 作用：用于简化和替换jsp页面上的java代码

#### 使用步骤：

1. 导入jstl相关jar包
2. 引入标签库：taglib指令：<%@ taglib %>
3. 使用标签

#### 常用的JSTL标签：

1. if				:相当于java代码的if语句

   * 属性：

     * test			：必须属性，接收Boolean表达式
       * 如果表达式为true，则显示if标签体内容；如果为false，则不显示
       * 一般情况下，test属性值会结合EL表达式一起使用

   * 注意：c:if 标签没有else情况，想要else情况，则可以再定义一个c:if 标签

     ```jsp
     <c:if test="true">
             IDEA真垃圾
     </c:if><hr>
     	<%
     		//判断request域中一个list集合是否为空，如果不为null则显示遍历集合
     		
     		List list = new ArrayList();
     		list.add("小雯");
     		request.setAttribute("list",list);
     		request.setAttribute("number",7);
     	%>
     	
     	<c:if test="${not empty list}">
     		遍历集合......
     		
     	</c:if>
     	<hr>
     	<c:if test="${number % 2 != 0 }">
     		${number }是一个奇数
     	</c:if>
     ```

2. choose      ：相当于java代码的switch语句

   ```jsp
   <% request.setAttribute("number",5);	%>
   	<c:choose>
   		<c:when test="${number == 1}">星期一</c:when>
   		<c:when test="${number == 2}">星期二</c:when>
   		<c:when test="${number == 3}">星期三</c:when>
   		<c:when test="${number == 4}">星期四</c:when>
   		<c:when test="${number == 5}">星期五</c:when>
   		<c:when test="${number == 6}">星期六</c:when>
   		<c:when test="${number == 7}">星期日</c:when>
   		<c:otherwise>数字输入有误</c:otherwise>
   	</c:choose>
   ```

3. forEach      ：相当于java代码的for语句

   * 完成重复的操作

     * 属性：

       * begin：开始值

       * end：结束值

       * var：临时变量

       * step：步长

       * varStatus：循环状态对象

         1. index：容器中元素的索引，从0开始

         2. count：循环次数，从1开始

   * 遍历容器

     * 属性：

       * items：容器对象

       * var：容器中元素的临时变量

       * varStatus：循环状态对象

         1. index：容器中元素的索引，从0开始

         2. count：循环次数，从1开始

            ```jsp
            <c:forEach begin="1" end="10" var="i" step="1" varStatus="s">
            		${i} &nbsp;${s.index}&nbsp;${s.count }<br>
            	</c:forEach>
            	<hr>
            	<% 
            		List list = new ArrayList();
            		list.add("aa");
            		list.add("bb");
            		request.setAttribute("list",list);
            	%>
            	<c:forEach items="${list}" var="str" varStatus="s">
            		${str}&nbsp;${s.index}&nbsp;${s.count }<br>
            	</c:forEach>
            ```

# IDEA与Tomcat的相关配置

1. IDEA会为每一个tomcat部署的项目单独建立一份配置文件
   * 查看控制台的log ： Using CATALINA_BASE:   "C:\Users\lan 人\.IntelliJIdea2019.2\system\tomcat\Tomcat_8_5_50_BiLibili"
2. 工作空间项目    和    tomcat部署的web项目
   * tomcat真正访问的是“tomcat部署的web项目”，“tomcat部署的web项目”对应着"工作空间项目"的web目录下的所有资源
   * WEB-INf目录下的资源不能被浏览器直接访问。
3. 断点调试：使用”debug“启动

# XML

##### 概念：Extensible Markup Langage  可扩展标记语言

- 可扩展：标签都是自定义的。  <user> <student>

##### 功能

* 存储数据

  1、 配置文件

  2、 在网络中传输

##### xml与html的区别

- xml标签都是自定义的，html标签是预定义
- xml的语法严格，html语法松散
- xml是存储数据的，html是展示数据
- w3c：万维网联盟

##### 语法

* 基本语法：

  	1. xml文档的后缀名 .xml
   	2. xml第一行必须定义为文档声明
   	3. xml文档中有且仅有一个根标签
   	4. 属性值必须用引号（单双都可）引起来
   	5. 标签必须正确关闭
   	6. xml标签名称区分大小写

* 快速入门：

  ```xml
  <?xml version='1.0' ?>
  
  <users>
  
  	<user id='1'>
  		<name>zhangsan</name>
  		<age>23</age>
  		<gender>male</gender>
  	</user>
  	
  	<user id='2'>
  		<name>lisi</name>
  		<age>24</age>
  		<gender>female</gender>
  	</user>
  	
  </users>
  ```

* 组成部分：

  1. 文档声明：格式：<?xml 属性列表 ?>

     1. 属性列表：
        * version : 版本号，必须的属性
        * encoding : 编码方式。告知解析引擎当前文档使用的字符集，默认值：ISO-8859-1
        * standalone : 是否独立
          * 取值：
            * yes：不依赖其他文件
            * no：依赖其他文件

  2. 指令：结合css的

     ```xml
     <?xml-stylesheet type="text/css" href="a.css" ?>
     ```

  3. 标签：标签名称自定义的

     * 规则：
       * 名称可以包含字母、数字以及其他的字符
       * 名称不能以数字或者标点符号开始
       * 名称不能以字母xml（或者XML、Xml等等开始）
       * 名称不能包含空格

  4. 属性：

     id属性值唯一

  5. 文本

     * CDATA区：在该区域中的数据会被原样展示

       <![CDATA[ if(a < b && a > c) {} ]]>

       <![CDATA[ 数据 ]]>

##### 约束：规定xml文档的书写规则

* 作为框架的使用者（程序员）：

  1. 能够在xml中引入约束文档
  2. 能够简单地读懂约束文档

* 分类：

  1. DTD：一种简单的约束技术
  2. Schema：一种复杂的约束技术

* DTD：

  * 引入dtd文档到xml文档中
    * 内部dtd：将约束规则定义在xml文档中
    * 外部dtd：将约束规则定义在外部的dtd文件中
      1. 本地：<!DOCTYPE 根标签名  SYSTEM "dtd文件的位置">
      2. 网络：<!DOCTYPE 根标签名 PUBLIC "dtd文件名字" "dtd文件的位置">

* Schema：

  ```xml
  <?xml version='1.0' ?>
  <!--
  	1.填写xml文档的根元素
  	2.引入xsi前缀.  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	3.引入xsd文件命名空间.  xsi:schemaLocation="http://www.itcast.cn/xml	student.xsd"
  	4.为每一个xsd约束声明一个前缀，作为标识  xmlns="http://www.itcast.cn/xml"
  <students	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  		    xmlns="http://www.itcast.cn/xml"
  		  xsi:schemaLocation="http://www.itcast.cn/xml	student.xsd"
  			>
  	<student number="heima_0001">
  		<name>zhangsan</name>
  		<age>11</age>
  		<sex>male</sex>
  	</student>
  </students>
  ```

##### 解析：操作xml文档，将文档中的数据读取到内存中

* 操作xml文档
  1. 解析（读取）：将文档中的数据读取到内存中
  2. 写入：将内存中的数据保存到xml文档中。持久化地存储
* 解析xml的方式：
  1. DOM：将标记语言文档一次性加载进内存中，在内存中形成一棵DOM树（多用于服务器端）
     * 优点：操作方便，可以对文档进行CRUD的所有操作
     * 缺点：占内存
  2. SAX：逐行读取，基于事件驱动的（多用于移动端）
     * 优点：不占内存
     * 缺点：只能读取，不能增删改
* xml常见的解析器：
  1. JAXP：sun公司提供的解析器，支持DOM和SAX两种思想。
  2. DOM4J：一款非常优秀的解析器。
  3. Jsoup：jsoup是一款Java的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。
  4. PULL：Android操作系统内置的解析器，SAX方式的。

##### Jsoup

* 快速入门：

  * 步骤：
    1. 导入jar包
    2. 获取Document对象
    3. 获取对应的标签Element对象
    4. 获取数据

* 代码

  ```java
  //获取Document对象，根据xml文档获取
          //获取student.xml的path
          String path = JsoupDemo1.class.getClassLoader().getResource("student.xml").getPath();
          //解析xml文档，加载文档进内存，获取dom树--->Document对象
          Document document = Jsoup.parse(new File(path), "UTF-8");
          //获取元素对象 Element
          Elements elements = (Elements) document.getElementsByTag("name");
  //        System.out.println(elements);
          //获取第一个name的element对象
          Element element = elements.get(1);
          System.out.println(element.text());
  ```

* 对象的使用：

  1. Jsoup：工具类，可以解析html或xml文档，返回Document

     * parse：解析html或者xml文档，返回Document

       * parse(File in,String charsetName):解析xml或者html文件的

         ```java
         //获取student.xml的path
                 String path = JsoupDemo2.class.getClassLoader().getResource("student.xml").getPath();
                 //1 解析xml文档，加载文档进内存，获取dom树--->Document对象
                 Document document = Jsoup.parse(new File(path), "UTF-8");
                 System.out.println(document);
         ```

         

       * parse(String html)：解析xml或者html字符串

         ```java
         String str = "<?xml version='1.0' ?>\n" +
                         "\n" +
                         "<students>\n" +
                         "\t<student number=\"heima_0001\">\n" +
                         "\t\t<name>tom</name>\n" +
                         "\t\t<age>18</age>\n" +
                         "\t\t<sex>male</sex>\n" +
                         "\t</student>\n" +
                         "\t<student number=\"heima_0002\">\n" +
                         "\t\t<name>xiaowen</name>\n" +
                         "\t\t<age>20</age>\n" +
                         "\t\t<sex>female</sex>\n" +
                         "\t</student>\n" +
                         "</students>";
                 Document document = Jsoup.parse(str);
                 System.out.println(document);
         ```

         

       * parse(URL url,int timeoutMillis)：通过网络路径获取指定的html或xml的文档对象

         ```java
         URL url = new URL("https://baike.baidu.com/item/%E6%9C%B1%E8%8C%B5/10617?fr=aladdin5");       //代表网络中的一个资源路径
                 Document document = Jsoup.parse(url,10000);
                 System.out.println(document);
         ```

         

  2. Document：文档对象。代表内存中的dom树

     * 获取ELement对象

       * getElementById(String id)：根据id属性值获取唯一的element对象

       * getElementsByTag(String tagName)：根据标签名称获取元素对象集合

       * getElementsByAttribute(String key)：根据属性名称获取元素对象集合

       * getElementsByAttributeValue(String key,String value)：根据对应的属性名和属性值获取元素对象集合

         

  3. Elements：元素Element对象的集合。可以当作ArrayList<Element>来使用

  4. Element：元素对象

     1. 获取子元素对象

        * getElementById(String id)：根据id属性值获取唯一的element对象

        - getElementsByTag(String tagName)：根据标签名称获取元素对象集合
        - getElementsByAttribute(String key)：根据属性名称获取元素对象集合
        - getElementsByAttributeValue(String key,String value)：根据对应的属性名和属性值获取元素对象集合

     2. 获取属性值

        * String attr(String key)：根据属性名称获取属性值     不区分大小写

     3. 获取文本内容

        * String text()：获取所有子标签的纯文本内容
        * String html()：获取标签体的所有内容（包括子标签的标签和文本内容）

  5. Node：节点对象

     * 是Document和Element的父类

* 快捷查询方式：

  1. **selector：选择器**

     * 使用的方法：Elements select(String cssQuery)

       * 语法：参考Selector类中定义的语法

         ```java
         //获取id为heima_0001的student标签的age标签
         System.out.println(document.select("student[number='heima_0001']>age"));
         ```

         使用 > 获取直接子标签

         使用空格获取所有子标签

  2. **XPath**：

     * XPath即为[XML](https://baike.baidu.com/item/XML)路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言
     * 使用Jsoup的XPath需要额外导入jar包
     * 查询w3cshool参考手册，使用XPath的语法完成查询

# jQuery

#### 概念：一个javaScript框架。简化js开发

* JavaScript框架：本质上就是一些js文件，封装了js的原生代码而已

#### 快速入门

* 步骤：

  1. 下载jQuery

     * jquery-xxx.js与jquery-xxx.min.js
       * jquery-xxx.js:开发版本。给程序员看的，有良好的缩进和注释，体积大一些
       * jquery-xxx.min.js:生产版本。程序中使用，没有缩进，体积小一些，程序加载更快

  2. 导入jQuery的js文件：导入min.js文件

  3. 使用

     ```js
     var div1 = $("#div1");
     alert(div1.html());
     ```

* jQuery对象和js对象区别与转换

  * 都可以当作数组来使用

  * jQuery对象在操作时，更加方便

  * jQuery对象和js对象方法不通用的

  * 两者之间可以相互转换

    * jq -------->  js：jq对象[索引]  或者   jq对象.get(索引)
    * js  -------->  jq：$(js对象)

    ```js
    //1. 通过js方式来获取名称叫div的所有html对象
            var divs = document.getElementsByTagName("div");
            alert(divs.length);    //可以将其当作数组来使用
            //对divs中所有的div 让其标签体内容变为"aaa"
            for (var i = 0; i < divs.length; i++) {
                // divs[i].innerHTML = "aaa";
                // $(divs[i]).html("哈哈哈");
            }
            //2. 通过jQuery方式
            var $divs = $("div");
            alert($divs.length);   //也可以当作数组使用
    
            //对divs中所有的div 让其标签体内容变为"bbb"
            // $divs.html("bbb");
            $divs[0].innerHTML = "20200114";
            $divs.get(1).innerHTML = "this is wawu";
    ```

* ##### 选择器：筛选具有相似特征的元素（标签）

  1. 基本语法学习：

     1. 事件绑定

        ```js
        $(".b1").click(function(){
            alert("弹弹弹");
        });
        ```

     2. 入口函数

        * window.onload 和 $(function) 区别:
          * window.onload只能定义一次，如果定义多次，后边的会将前边的覆盖掉
          * $(function)可以定义多次的

        ```js
        $(function () {
                // 获取b1按钮
                $(".b1").click(function(){
                    alert("弹弹弹");
                });
        })
        ```

     3. 样式控制
     
        ```js
        $(function () {
                    $("#div1").css("background-color","red");
                    $("#div2").css("backgroundColor","blue");
                })
        ```
     
  2. 分类
  
     1. 基本选择器
  
        1. 标签选择器（元素选择器）
           * 语法：$("html标签名") 获得所有匹配标签名称的元素
        2. id选择器
           * 语法：$("#id的属性值") 获得与指定id属性值匹配的元素
        3. 类选择器
           * 语法：$(".class的属性值")
        4. 并集选择器：
           * 语法：$("选择器1,选择器2...") 获取多个选择器选中的所有元素
  
     2. 层级选择器
  
        1. 后代选择器
           * 语法：$("A B") 选择A元素内部的所有B元素（所有后代）
        2. 子选择器
           * 语法：$("A > B") 选择A元素内部的所有B子元素（只有一代）
  
     3. 属性选择器
  
        1. 属性名称选择器
           * 语法：$("A[属性名]") 包含指定属性的选择器
        2. 属性选择器
           * 语法：$("A[属性名='值']") 包含指定属性等于指定值的选择器
        3. 复合属性选择器
           * 语法：```$("A[属性名='值']...")```包含多个属性条件的选择器
  
     4. 过滤选择器
  
        1. 首元素选择器
           * 语法：	:first        获得选择的元素中的第一个元素
        2. 尾元素选择器
           * 语法：	:last		获得选择的元素中的最后一个元素
        3. 非元素选择器
           * 语法：	:not(selecter)	不包括指定内容的元素
        4. 偶数选择器
           * 语法：	:even   偶数，从0开始计数
        5. 奇数选择器
           * 语法：	:odd	奇数，从0开始计数
        6. 等于索引选择器
           * 语法：	:eq(index)	指定索引元素
        7. 大于索引选择器
           * 语法：	:gt(index)
        8. 小于索引选择器
           * 语法：	:lt(index)
        9. 标题选择器
           * 语法：	header    获得标题元素，固定写法
  
     5. 表单过滤选择器
  
        1. 可用元素选择器
  
           * 语法：	:enabled	获得可用元素
  
        2. 不可用元素选择器
  
           * 语法：	:disabled	获得不可用元素
  
        3. 选中选择器
  
           * 语法：	:checked	获得单选/复选框选中的元素
           * 语法：    :selected    获得下拉框选中的元素
  
           ```javascript
           $("#ddd").click(function(){
                       for (var i = 0; i < $("input[type='checkbox']:checked").length; i++) {
                           alert($("input[type='checkbox']:checked")[i].value);
                       }
                   });
                   $("#d2").click(function(){
                       alert($("input[type='radio']:checked")[0].value);
                   });
           ```
  

# Mybatis

		### 概述

* 一个优秀的基于java的持久层框架，内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程
* 通过xml或注解方式将各种statement配置起来
* 使用ORM思想实现了结果集的封装

### ORM

* Object Realtional Mapping        对象关系映射


### 环境搭建

* 创建maven工程并导入坐标

  ```xml
  <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.4.5</version>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.6</version>
          </dependency>
  ```

* 创建实体类和dao的接口

* 创建Mybatis的主配置文件

  * 在resources目录下创建 SqlMapConfig.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <!-- mybatis的主配置文件 -->
    <configuration>
        <!-- 配置环境 -->
        <environments default="mysql">
            <!-- 配置mysql的环境 -->
            <environment id="mysql">
                <!-- 配置事务的类型 -->
                <transactionManager type="JDBC"></transactionManager>
                <!-- 配置数据源（连接池） -->
                <dataSource type="POOLEd">
                    <!-- 配置连接数据库的四个基本信息 -->
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/maven"/>
                    <property name="username" value="root"/>
                    <property name="password" value="123"/>
                </dataSource>
            </environment>
        </environments>
    
        <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
        <mappers>
            <mapper resource="dao/UserDao.xml" />
        </mappers>
    </configuration>
    ```

  * 在resources对应目录下创建对应dao.xml文件

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="dao.UserDao" resultType="domain.User">
        <!-- 配置查询所有 -->
        <select id="findAll">
            select * from user
        </select>
    </mapper>
    ```

### 环境搭建注意事项

1. 在Mybatis中把持久层的操作接口名称和映射文件也叫做Mapper

   所以  **UserDao.xml** 和 **UserMapper.xml**是一样的

2. Mybatis的映射配置文件结构必须和dao接口的包结构相同

3. 映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名

4. 映射配置文件的操作配置（select），id属性的取值必须是dao接口的方法名

### Mybatis基于注解的入门案例

* 把UserDao.xml移除，在Dao接口的方法上使用@Select注解，并且指定SQL语句，同时需要在SqlMapCinfig.xml中的mapper配置时，使用class属性指定Dao接口的全限定类名

  ```java
  //1、读取配置文件
          InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
          //2、创建SqlSessionFactory工厂
          SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
          SqlSessionFactory factory = builder.build(in);
          //3、使用工厂生产SqlSession对象
          SqlSession session = factory.openSession();
          //4、使用SqlSession创建Dao接口的代理对象
          UserDao userDao = session.getMapper(UserDao.class);
          //5、使用代理对象执行方法
          List<User> users = userDao.findAll();
          for (User user: users) {
              System.out.println(user);
          }
          //6、释放资源
          session.close();
          in.close();
  ```


### mybatis中的一些问题

* 向数据库中插入中文乱码

  * 解决办法：在数据库连接信息后添加?useUnicode=true&amp;characterEncoding=UTF-8"

    & 在 xml会需要转义 如

```xml
?useUnicode=true&amp;characterEncoding=UTF-8"
```



