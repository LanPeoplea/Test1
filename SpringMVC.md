## 入门

#### maven工程pom.xml文件设置

* [SpringMVC入门项目](D:/IDEAworkstation/SpringMVC/springmvc_01_start)

* 版本锁定

  ```xml
    <properties>
      <spring.version>5.0.2.RELEASE</spring.version>
    </properties>
  ```

* 导入坐标

  ```xml
    <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
      </dependency>
  
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
      </dependency>
  
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
      </dependency>
  
      <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
      </dependency>
  
      <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
      </dependency>
    </dependencies>
  ```

* 配置web.xml文件

  * 添加servlet

    ```xml
      <servlet>
      <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
      <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
      </servlet-mapping>
    ```

  * 在servlet标签中添加加载springmvc配置文件

    ```xml
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
      </init-param>
    <load-on-startup>1</load-on-startup>
    ```

* 在resources目录下新建xml配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
          http://www.springframe work.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/mvc
          http://www.springframework.org/schema/mvc/spring-mvc.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
  </beans>
  ```

  * 配置扫描类包

  * 配置视图解析器

    * prefix：表示文件路径
    * suffix：表示文件后缀名

    ```xml
        <!-- 开启注解扫描 -->
        <context:component-scan base-package="com.lanpeople"/>
    
        <!-- 配置视图解析器对象 -->
        <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/pages/"/>
            <property name="suffix" value=".jsp"/>
        </bean>
    ```

* 开启SpringMVC框架注解的支持

  ```xml
  <mvc:annotation-driven/>
  ```


#### 表单提交时post方式中文乱码

* ##### 在web.xml中配置过滤器

  ```xml
    <!-- 配置解决中文乱码的过滤器 -->
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
  ```

#### 自定义类型转换器

* 第一步 ：定义一个类，实现Converter接口，该接口有两个泛型  [点击查看java代码](D:/IDEAworkstation/SpringMVC/springmvc_01_start/src/main/java/com/lanpeople/utils/StringToDateConverter.java)

  * Converter<S,T>   
    * S指转换前类型，T指转换后想得到的类型

* 第二步：在SpringMVC配置文件中配置自定义类型转换器

  ```xml
  <!-- 自定义类型转换器 -->
      <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
          <!-- 注册自己写的转换器 -->
          <property name="converters">
              <set>
                  <bean class="com.lanpeople.utils.StringToDateConverter"/>
              </set>
          </property>
      </bean>
  
   <!-- 开启SpringMVC框架注解的支持,使自定义类型转换器生效 -->
   <mvc:annotation-driven conversion-service="conversionService"/>
  ```

  