# 初始操作

##### git init   		 初始化仓库

#####  git status  	 查看仓库的状态

* 分支左侧带有 ” * “,表示当前分支

# 提交操作

#####  git add  		  向暂存区中添加文件

##### git commit	保存仓库的历史记录

* git commit -m " "               -m参数后的“ ”称作提交信息，是对这个提交的概述
* git commit                         
  * 编辑器第一行：用一行文字简述提交的更改内容
  * ​            第二行：空行
  * ​     第三行以后：记述更改的原因和详细内容
* 中止提交：
  * 如果在编辑器启动后想中止提交，请将提交信息留空并直接关闭编辑器，随后提交就会被中止

##### git commit --amend     修改提交信息

##### git commit -a -m ""      不需要进行git add命令可直接提交

# 推送操作

##### git remote add       添加远程仓库

```bash
$ git remote add github git@github.com:LanPeoplea/Test1.git
```

##### git push      推送到master分支

* git push -u origin master ：-u 参数可以在推送的同时，将origin仓库的master分支设置为本地仓库当前分支的upstream（上游）。添加了这个参数，将来运行git pull命令从远程仓库获取内容时，本地仓库的这个分支就可以直接从origin的master分支获取内容，省去了另外添加参数的麻烦。

# 获取操作

##### git clone      获取远程仓库

* git clone 仓库地址

  ```bash
  $ git clone git@gitee.com:lanpeople/Course.git
  #clone仓库里的所有文件
  ```

##### git pull     获取最新的远程仓库分支

```bash
$ git pull origin feature-D
```



# 最多使用

##### git log           查看提交日志

* git log --pretty=short	:	只显示提交信息的第一行（作者）
* git log 文件名                 ：  显示指定目录、文件的日志
* git log -p   [文件名]                      ：  显示文件的改动

##### git diff         查看更改前后的差别

* 执行git diff 命令，查看当前工作树与暂存区的差别
* "  +  "号标出的是新添加的行；"  -  "号标出的是被删除的行；

##### git branch       显示分支一览表

* git checkout -b 分支名    ：    创建新的分支

* git branch 分支名

  git checkout bro-A          :        创建新的分支

  ```bash
  $ git checkout -b bro-A
  #创建一个名为bro-A的分支
  $ git branch bro-A
  $ git checkout bro-A                
  #创建一个名为bro-A的分支
  ```

##### git checkout      切换分支

* git checkout 分支名：切换至指定分支
* git checkout - ：切换回上一个分支