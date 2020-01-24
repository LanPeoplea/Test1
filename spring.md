## XML配置

- bean.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- 把对象的创建交给spring来管理 -->
      <bean id="accountService" class="com.lanpeople.service.impl.AccountServiceImpl"></bean>
      <bean id="accountDao" class="com.lanpeople.dao.impl.AccountDaoImpl"></bean>
  </beans>
  ```

### 解耦：降低程序间的依赖关系

- 实际开发中应该做到：编译期不依赖，运行时才依赖
- 解耦思路：
  1. 使用反射来创建对象，而避免使用new关键字
  2. 通过读取配置文件来获取要创建的对象全限定类名

## IOC

- 概念：控制反转(Inversion of Control)把创建对象的权利交给框架，是框架的重要特征，并非面向对象编制的专用术语。它包括依赖注入(Dependency Injextion,简称DI)和依赖查找(Dependency Lookup)
- 作用：削减计算机程序的耦合（解除代码中的依赖关系）

### ApplicationContext

- 三个常用实现类
  1. ClassPathXmlApplicationContext   (更常用)
     - 可以加载类路径下的配置文件，要求配置文件必须在类路径下
  2. FileSystemXmlApplicationContext
     - 可以加载磁盘任意路径下的配置文件(必须用访问权限)
  3. AnnotationConfigApplicationContext
     - 用于读取注解创建容器

### 核心容器的两个接口引发出的问题：

- ApplicationContext:                   (单例对象适用)
  - 它在构建核心容器时，创建对象采取的策略是采用立即加载的方式。即一读取完配置文件马上就创建配置文件中配置的对象
- BeanFactory:                              (多例对象适用)
  - 它在构建核心容器时，创建对象采取的策略是采用延迟加载的方式。即什么时候根据id获取对象，什么时候才真正地创建对象

### spring对bean的管理细节

1. 创建bean的三种方式

   1. 使用默认构造函数创建：

      - 再spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建bean对象。此时如果类中没有m默认构造函数，则对象无法创建

        ```xml
        <bean id="accountService" class="com.lanpeople.service.impl.AccountServiceImpl"></bean>
        ```

   2. 使用普通工厂中的方法创建对象(使用某个类中的方法创建对象，并存入spring容器)

      ```xml
      <bean id="instanceFactory" class="com.lanpeople.factory.InstanceFactory"></bean>
          <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
      ```

   3. 使用工厂中的静态方法创建对象(使用某个类中的静态方法创建对象，并存入spring容器)

      ```xml
      <!-- 第三种方式：使用工厂中的静态方法创建对象(使用某个类中的静态方法创建对象，并存入spring容器) -->
          <bean id="accpuntService" class="com.lanpeople.factory.staticFactory" factory-method="getAccountService"/>
      ```

2. bean对象的作用范围

   - bean标签的scope属性：
     - 作用：用于指定bean的作用范围
     - 取值：
       1. singleton：单例的（默认值）
       2. prototype：多例的
       3. request：作用于web应用的请求范围
       4. session：作用于web应用的会话范围
       5. global-session：作用于集群环境的会话范围（全局会话范围），当不是j集群环境时，它就是session

3. bean对象的生命周期

   - 单例对象:
     - 出生：当容器创建时对象出生
     - 活着：只要容器还在，对象一直活着
     - 死亡：容器销毁，对象消亡
     - 总结：单例对象的生命周期和容器相同
   - 多例对象：
     - 出生：当我们使用对象时spring框架为我们创建
     - 活着：对象只要是在使用过程中就一直活着
     - 死亡：当对象长时间不用且没有别的对象引用时，由Java的垃圾回收器回收

### spring中的依赖注入

#### 依赖注入：Dependency Injection

#### 依赖注入的类型：

1. 基本类型和String
2. 其他bean类型（在配置文件中或者注解配置过的bean）
3. 复杂类型/集合类型

#### 依赖注入的方式：

1. 使用构造函数提供：

   - 优势：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。

   - 弊端：改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供。

   - 使用的标签：constructor-arg

   - 标签出现的位置：bean标签的内部

   - 标签中的属性：

     1. type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型

     2. index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置从0开始

     3. name：用于指定给构造函数中指定名称的参数赋值

     4. value：用于提供基本类型和String类型的数据

     5. ref：用于指定其他的bean类型数据。指的就是在spring的IOC核心容器中出现过的bean对象

        ```xml
         <bean id="accountService" class="com.lanpeople.service.impl.AccountServiceImpl">
                <constructor-arg name="name" value="xaiowen"></constructor-arg>
                <constructor-arg name="age" value="20"></constructor-arg>
                <constructor-arg name="birthday" ref="now"></constructor-arg>
            </bean>
        
            <!-- 配置一个日期对象 -->
            <bean id="now" class="java.util.Date"/>
        ```

2. 使用set方法提供（更常用）

   - 优势：创建对象时没有明确的限制，可以直接使用默认构造函数
   - 弊端：如果由某个成员必须有值，则获取对象是有可能set方法没有执行
   - 涉及的标签：property
   - 出现的位置：bean标签的内部
   - 标签的属性：
     - name：用于指定注入时所调用的set方法名称
     - value：用于提供基本类型和String类型的数据
     - ref：用于指定其他的bean类型数据。指的就是在spring的IOC核心容器中出现过的bean对象

   ```xml
   <!-- set方法注入 -->
       <bean id="accountService2" class="com.lanpeople.service.impl.AccountServiceImpl2">
           <property name="name" value="小雯"/>
           <property name="age" value="20"/>
           <property name="birthday" ref="now"/>
       </bean>
   ```

   - 复杂类型set注入
     - 用于给List结构集合注入的标签：list  array  set
     - 用于给Map结构集合注入的标签：map  props
     - 结构相同，标签可以互换

   ```xml
   <!-- 复杂类型/集合类型    set方法注入 -->
       <bean id="accountService3" class="com.lanpeople.service.impl.AccountServiceImpl3">
           <property name="myStrs">
               <array>
                   <value>AAA</value>
                   <value>bbb</value>
                   <value>CCC</value>
               </array>
           </property>
           <property name="myList">
               <list>
                   <value>a1</value>
                   <value>b2</value>
                   <value>C3</value>
               </list>
           </property>
           <property name="mySet">
               <set>
                   <value>hahaha</value>
                   <value>heiheihei</value>
                   <value>xixixi</value>
               </set>
           </property>
           <property name="myMap">
               <map>
                   <entry key="name1" value="张三"/>
                   <entry key="name2" value="李四"/>
                   <entry key="name3" value="王五"/>
               </map>
           </property>
           <property name="myProps">
               <props>
                   <prop key="testC">C位</prop>
                   <prop key="testD">D杯</prop>
               </props>
           </property>
       </bean>
   ```

3. 使用注解提供

### spring中的注解

- 使用注解注释则不需要set方法

#### 注解的分类：

1. 用于创建对象的：

   - 作用和在xml配置文件中编写一个<bean>标签实现的功能是一样的。

   1. ##### @Component:    用于把当前类对象存入spring容器中

      - 属性：
        1. value：用于指定bean的id，默认值是当前类名且首字母改小写

   2. ##### @Controller:      作用与属性同@Component相同     **一般用在表现层**

   3. ##### @Service:           作用与属性同@Component相同     **一般用在业务层**

   4. ##### @Repository:     作用与属性同@Component相同     **一般用在持久层**

2. 用于注入数据的：

   - 作用和在xml配置文件中的<bean>标签中写一个<property>标签的作用是一样的。

   1. ##### @Autowired:      自动按照类型注入。只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功；如果IOC容器中没有任何bean的类型和要注入的变量类型匹配，则报错；如果IOC容器中有多个类型匹配时，首先匹配类型相符的，然后匹配变量名称相符的，如果都不相符则报错      **只能注入其他bean类型的数据**

      - 出现位置：可以是变量上，也可以是方法上
      - 细节：        在使用注解注入时，set方法就不是必须的了

   2. ##### @Qualifier：          在按照类中注入的基础之上再按照名称注入。它在给类成员注入时不能单独使用，但是在给方法参数注入时可以   **只能注入其他bean类型的数据**

      - 属性：
        - value：    用于指定注入bean的id

      ```java
      	@Autowired
          @Qualifier("accountDao1")
          private AccountDao accountDao;
      
          public void saveAccount(){
              accountDao.saveAccount();
          }
      ```

   3. ##### @Resource：      直接按照bean的id注入。**可以独立使用**    **只能注入其他bean类型的数据**

      - 属性：
        - name：    用于指定bean的id

      ```java
      	@Resource(name = "accountDao2")
          private AccountDao accountDao;
      ```

   4. ##### @Value：      用于注入基本类型和String类型的数据

      - 属性：

        - value：        用于指定数据的值。它可以使用spring中的SpEL（即spring的el表达式）

        - SpEL的写法：${表达式}

          ```java
          @Value("${jdbc.driver}")
              private String driver;
              @Value("${jdbc.url}")
              private String url;
              @Value("${jdbc.username}")
              private String username;
              @Value("${jdbc.password}")
              private String password;
          ```

3. 用于改变作用范围的：

   - 作用和在<bean>标签中使用scope属性实现的功能是一样的。

   1. ##### @Scope：            用于指定bean的作用范围

      - 属性：
        - value：    指定范围的取值。 常用取值：singleton和prototype

4. 和生命周期相关：

   - 作用和在<bean>标签中使用init-method和destory-method的作用是一样的。

   1. ##### PostConstruct：  用于指定初始方法

   2. ##### PreDestroy：        用于指定销毁方法

## 在xml中加入注解需要的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 告知spring在创建容器时要扫描的包，配置所需要的标签不是在bean的约束中，而是一个名称为context名称空间和约束中-->
    <context:component-scan base-package="com.lanpeople"/>
</beans>
```

### Spring中的新注解

#### @Configuration

- 作用：指定当前类是一个配置类
- 细节：当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可以不写

#### @ComponentScan

- 作用：用于通过注解指定spring在创建容器时要扫描的包

- 属性：

  1. value：和basePackages的作用一样，都用于指定创建容器时要扫描的包，使用此注解等同于在xml中配置：

     ```xml
     <context:component-scan base-package="com.lanpeople"/>
     ```

### @Bean

- 作用：用于把当前方法的返回值作为bean对象存入spring的IOC容器中
- 属性：
  1. name：用于指定bean的id。**默认值是当前方法的名称**

### 细节

**** 当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象，查找的方式和Autowired注解的方式相同

### @Import

- 作用：用于导入其他的配置类
- 属性：
  - value：用于指定其他配置类的字节码

### @PropertySource

```java
@PropertySource("classpath:jdbcConfig.properties")
```

- 作用：用于指定properties文件的位置
- 属性：
  - value：指定文件的名称和路径
    - 关键字：classpath：表示类路径下

### @Aspect

- 作用：表示当前类是一个切面类

## Spring整合Junit的配置

#### 第一步：导入spring整合junit的jar（坐标）

```xml
<dependency>
    <groupId>org.springframework</groupId>    					     <artifactId>spring-test</artifactId>       	      <version>5.0.2.RELEASE</version>
</dependency>
```

### 第二步：使用Junit提供的一个注解把原有的main方法替换掉，替换成spring提供的@RunWith

- 在测试类上加上注解

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  ```

### 第三步：告知spring的运行器，spring的Ioc创建是基于xml还是注解的，并且说明位置

- @ContextConfiguration

  - locations：指定xml文件的位置，加上classpath关键字，表示在类路径下
  - classes：表示注解类所在位置

  ```java
  @ContextConfiguration(locations = "classpath:bean.xml")
  @ContextConfiguration(classes = SpringConfiguration.class)
  ```

### 使用Spring 5.x版本的时候，要求Junit的jar必须是4.12及以上

## AOP	

**** **Aspect Oriented Programming      面向切面编程**

#### XML配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
```

#### 配置AOP步骤

1. 把通知Bean也交给spring来管理
2. 使用aop:config标签表明开始AOP的配置
3. 使用aop:aspect标签表明配置切面
   1. id属性：是给切面提供一个唯一标识
   2. ref属性：是指定通知类bean的Id
4. 在aop:aspect标签内部使用对应的标签来配置通知的类型
   - aop:before：表示前置通知

- 在切入点方法执行之前执行

  - aop:after-returning：表示后置通知           **和异常执行永远只能执行一个**

- 在切入点方法正常执行之后执行

  - aop:throwing：表示异常通知

- 在切入点方法执行产生异常之后执行

  - aop:after：表示最终通知

  - 无论方法是否正常执行它都会在其后面执行

  - aop:around：表示环绕通知
  - 写在proceed方法前就是前置通知
    - 写在proceed方法后就是后置通知
  - 写在catch中就是异常通知
    - 写在finally中就是最终通知
  - method属性：用于指定方法
  - pointcut属性：用于指定切入点表达式。该表达式的含义是指对业务层中哪些方法增强

  - 切入点表达式的写法：
    - 关键字：execution(表达式)
    - 表达式：
      - 访问修饰符    返回值   包名.类名.方法名(参数l列表)
      - 访问修饰符可以省略
  - 返回值可以使用通配符表示任意返回值
    - 包名可以使用通配符表示任意包。但是有几级包就需要写几个***.**
  - 包名可以使用**..**表示当前包及其子包
    - 类名和方法名都可以使用*****来实现通配
  - 参数列表：
    - 可以直接写数据类型：
      - 基本类型直接写名称
      - 引用类型写包名.类名的方式
  - 可以使用通配符表示任意类型但是必须有参数

- 可以使用**..**表示有无参数均可，有参数可以是任意类型

  - 实际开发中切入点表达式的通常写法：

    - 切到业务层实现类下的所有方法：

      - ```java
        com.lanpeople.service.impl.*.*(..)
        ```

  - 标准表达式写法：

    ```java
     pointcut="execution(public void com.lanpeople.service.impl.AccountServiceImpl.saveService())"
    ```

  - 全通配写法：

  ```
   * *..*.*(..)
  ```

5. 配置切入点表达式

   - aop:pointcut         写在<aop:aspect>标签内部只能当前切面使用；写在外部则所有切面可用，但必须定义在<aop:aspect>标签之前

   ```xml
   <!-- 配置AOP -->
       <aop:config>
           <aop:aspect id="logAdvice" ref="logger" >
               <!-- 配置前置通知 -->
               <aop:before method="beforePrintLog" pointcut-ref="pt1"/>
   
               <!-- 配置后置通知 -->
               <aop:after-returning method="afterReturnPrintLog" pointcut-ref="pt1"/>
   
               <!-- 配置异常通知 -->
               <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"/>
   
               <!-- 配置最终通知 -->
               <aop:after method="afterPrintLog" pointcut-ref="pt1"/>
   
               <!-- 配置切入点表达式 id属性用于指定表达式的唯一标志 expression属性用于指定表达式内容-->
               <aop:pointcut id="pt1" expression="execution(* com.lanpeople.service.impl.*.*(..))"/>
           </aop:aspect>
       </aop:config>
   ```

#### 注解AOP

##### @Aspect     表示当前类是一个切面类

##### @Before    前置通知

##### @AfterReturning    后置通知

##### @AfterThrowing    异常通知

##### @After    最终通知

##### @Around     环绕通知

##### @Pointcut("execution(* com.lanpeople.service.impl.*.*(..))")    配置切入点表达式

```java
@Pointcut("execution(* com.lanpeople.service.impl.*.*(..))")
    private void pt1(){}
```

#### 配置spring开启注解AOP的支持

- 在xml中写上就支持

```xml
<aop:aspectj-autoproxy/>
```

- 或者在类上添加  @EnableAspectJAutoProxy

## spring中基于xml的声明式事务控制配置步骤 

* [链接xml配置文件](D:/IDEAworkstation/spring_tx/src/main/resources/bean.xml)

#### 第一步：配置事务管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

#### 第二步：配置事务通知

- 需要导入事务的约束

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xmlns:tx="http://www.springframework.org/schema/tx"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/tx
          https://www.springframework.org/schema/tx/spring-tx.xsd
          http://www.springframework.org/schema/aop
          https://www.springframework.org/schema/aop/spring-aop.xsd">
  ```

- 配置事务通知

  - transaction-manager：给事务通知提供一个事务管理器引用

  ```xml
  <tx:advice id="txAdvice" transaction-manager="transactionManager"/>
  ```

#### 第三步：配置AOP中的通用切入点表达式

```xml
<aop:config>
        <aop:pointcut id="pt1" expression="execution(* com.lanpeople.service.impl.*.*(..))"/>
    </aop:config>
```

#### 第四步：建立事务通知和切入点表达式的对应关系

```xml
<aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
```

#### 第五步：配置事务的属性

* 在事务的通知tx:advice标签内部配置
  * isolation：用于指定事务的隔离级别，默认值是DEFAULT，表示使用数据库的默认隔离级别
  * propagation：用于指定事务的传播行为，默认值是REQUIRED，表示一定会有事务，增删改的选择，查询方法可以选择SUPPORTS
  * read-only：用于指定事务是否只读。只有查询方法才能设置为true，默认值为false，表示读写。
  * timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位
  * rollback-for：用于指定一个异常，当产生该异常时事务回滚；产生其他异常时事务不回滚。没有默认值，表示任何异常都回滚。
  * no-rollback-for：用于指定一个异常，当产生该异常时事务不回滚；产生其他异常时事务回滚。没有默认值，表示任何异常都回滚。

## spring中基于注解的声明式事务控制配置步骤

* [链接xml配置文件](D:/IDEAworkstation/spring_tx_annoation/src/main/resources/bean.xml)
* [源项目](D:/IDEAworkstation/spring_tx_annoation)

#### 第一步：配置事务管理器

#### 第二步：开启spring对注解事务的支持

```xml
<!-- 开启spring对注解事务的支持 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
```

#### 第三步：在需要事务支持的地方使用@Transactional注解

## 动态代理：

- 特点：字节码随用随创建、随用随加载

- 作用：不修改源码的基础上对方法增强

- 分类：

  - ### 基于接口的动态代理

    - 涉及的类：Proxy
    - 提供者：JDK官方
    - 如何创建代理对象：
      - 使用Proxy类中的newProxyInstance方法
    - 创建代理对象的要求
      - 被代理类最少要实现一个接口，如果没有则不能使用
    - newProxyInstance方法的参数：
      - ClassLoader:       类加载器
        - 用于加载代理对象字节码，和被代理对象使用相同的类加载器
        - 固定写法           类对象.getClass().getClassLoader()
      - Class[]:                    字节码数组
        - 用于让代理对象和被代理对象有相同方法
        - 固定写法            类对象.getClass().getInterfaces()
      - InvocationHandler:  用于提供增强的代码
        - 写如何代理。一般都是写一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的
        - 此接口的实现类都是谁用谁写
        - invoke方法：      执行被代理对象的任何接口方法都会经过该方法
          - proxy：        代理对象的引用
          - method：     当前执行的方法
          - args：            当前执行方法所需的参数
          - 返回值：        和被代理对象有相同的返回值

    ```java
    		Producer producer = new Producer();
            IProducer proxyProducer =  (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),producer.getClass().getInterfaces(),new InvocationHandler() {
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            return null;
                        }
                    });
    ```

  - ### 基于子类的动态代理

    - 涉及的类：Enhancer
    - 提供者：第三方cglib库
    - 如何创建代理对象：
      - 使用Enhamcer类中的create方法
    - 创建代理对象的要求：
      - 被代理类不能是最终类
    - create方法的参数：
      - Class：    字节码
        - 用于指定被代理对象的字节码
      - Callback：用于提供增强的代码
        - 一般写的都是该接口的子接口实现类：MethodInterceptor           执行被代理对象的任何方法都会经过该方法

  ```java
          final Producer producer = new Producer();
  
          Enhancer.create(producer.getClass(), new MethodInterceptor() {
              public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                  return null;
              }
          });
  ```

  #### 