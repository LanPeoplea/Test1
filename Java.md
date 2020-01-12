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
        * 功能：用于判断字符串、集合、数组长度对象是否为null并且长度是否为0，
        * ${empty list}
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