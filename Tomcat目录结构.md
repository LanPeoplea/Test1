* bin目录：可执行文件

* conf目录：   存放配置文件
* lib：          存放jar包
* logs：       存放日志文件
* temp：       存放临时文件
* webapps：  存放web项目
* work：          存放运行时的数据

#### tomcat启动

打开**bin**目录下的**startup.bat**文件     不要关闭

#### tomcat访问

浏览器输入 http://127.0.0.1:8080     或者http://localhost:8080        **访问自己**

​                    http://别人的ip:8080                                                            **访问别人**

##### 可能遇到的问题

* 黑窗口一闪而过  :没有正确配置JAVA_HOME环境变量

* 启动报错：           已经启动过或者端口号8080被占用

  ​	解决办法：

  ​			1.找到占用的端口号，并且找到相应的进进程关闭

  ​					**netstat -an**o  查看端口号使用情况     找到对应的**PID**（进程标识符）后关闭

  ​			2.修改自身的端口号

  ​					在**conf**目录中找到**server.xml**中找到**port=“ ”**并更改端口号   

     			**一般会将tomcat的默认端口号修改为80，80是http协议的默认端口号** (好处：在访问时，就不用输入端口号)
  
  ​					 <Connector port="8080" protocol="HTTP/1.1"
​               connectionTimeout="20000"
  ​               redirectPort="8443" />
  
  #### tomcat的关闭
  
  * 正常关闭：
  
  	* **bin/shutdown.bat**
  	* **ctrl + c**
  
  * 强制关闭：
    * 点击启动窗口的 x
  
  #### tomcat的配置
  
  ​	*直接将项目放到**webapps**目录下即可
  
  ​				*/项目名  ：项目的访问路径 -->虚拟目录  
  
  ​				*简化部署：将项目打成一个war包，再将war包放到webapps目录下。war包会自动解压缩
  
  ​	*配置 **conf/server.xml** 文件
  
  ​		在<Host>标签中配置
  
  ​           <Context docBase="项目路径"  path="/虚拟路径名" />
  
  ​	*在 **conf\Catalina\localhost** 创建任意名称的xml文件，在文件中编写
  
  ​			<Context docBase="项目路径" />
  
  ​			*虚拟目录：xml文件的名称			
  
  #### 静态项目和动态项目
  
  java动态项目的目录结构：
  
  --- 项目的根目录
  
  ​		---WEB-INF目录
  
  ​			---web.xml：web项目的核心配置文件
  
  ​			---classes目录：放置字节码文件的目录
  
  ​			---lib目录：放置依赖的jar包

#### 将Tomcat集成到IDEA中，并且创建JAVAEE项目

