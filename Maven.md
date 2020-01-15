### 标准目录结构

​		src/main/java               核心代码部分

​		src/main/resources	 配置文件部分

​		src/test/java				  测试代码部分

​		src/test/resources		测试配置文件

​		src/main/webapp         页面资源，js、css、图片等

### 常用命令

#### mvn -v              ：查看Maven版本

#### mvn clean		:删除掉了target目录

* 把编译好的项目中的信息直接删掉，清除项目编译信息

#### mvn complie 		：编译，多一个target目录

#### mvn test				 ：编译测试下的代码，也编译了main下的代码

#### mvn package		 ：编译并打成war包

#### mvn install			 ：编译打包并安装到本地仓库

