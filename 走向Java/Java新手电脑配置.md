# 1. Java/web环境

### Java



### Tomcat

1. 进入官网[Http://tomcat.apache.org/](http://tomcat.apache.org/),选择download，下载所需要的Tomcat版本。

<img src="Java新手电脑配置.assets/1832356-20191016113630168-1443483862.png" alt="img" style="zoom:50%;" />

​	下载后解压即可。

2. 到目录bin下的startup.bat，点击启动Tomcat；shutdown.bat：关闭Tomcat。
3. 环境变量配置：
   1. Tomcat的安装目录：CATALINA_HOME:\program_work\Javabase\apache-tomcat-9.0.31
   2. Tomcat的工作目录：CATALINA_BASE:\program_work\Javabase\apache-tomcat-9.0.31
   3. CLASSPATH：%CATALINA_HOME%\lib\servlet-api.jar;
   4. PATH：%CATALINA_HOME%\bin;%CATALINA_HOME%\lib
4. IDEA编辑器tomcat设置

# 2. IDEA安装与配置

#### 1.设置

<img src="Java新手电脑配置.assets/image-20200306222432941.png" alt="image-20200306222432941" style="zoom: 50%;" /> 

#### 2. 字体设置

<img src="Java新手电脑配置.assets/image-20200306222701618.png" alt="image-20200306222701618"  />

#### 3.关闭自动更新

![image-20200306222753891](Java新手电脑配置.assets/image-20200306222753891.png)

#### 自动导包

![image-20200306222823935](Java新手电脑配置.assets/image-20200306222823935.png)

Add unambiguous imports on the fly：自动导入不明确的结构;
Optimize imports on the fly： 自动帮我们优化导入的包

#### 禁止自动打开上次的项目

![image-20200306222854896](Java新手电脑配置.assets/image-20200306222854896.png)

#### 代码折叠设置

![image-20200306222940849](Java新手电脑配置.assets/image-20200306222940849.png)

#### 代码风格设置

![image-20200306223015591](Java新手电脑配置.assets/image-20200306223015591.png)

#### 更改文件签名

![image-20200306223045797](Java新手电脑配置.assets/image-20200306223045797.png)

#### 设置主题

![设置主题](Java新手电脑配置.assets/20190519213524520-1583505431491.png)

#### 编辑区主题

![在这里插入图片描述](Java新手电脑配置.assets/20190519213557985.png)

#### 通过插件（plugins）更换主题

![在这里插入图片描述](Java新手电脑配置.assets/20190519213618115.png)

#### 设置鼠标悬浮提示

![在这里插入图片描述](Java新手电脑配置.assets/20190519213655650.png)

#### 设置显示行号和方法间的分隔符

![在这里插入图片描述](Java新手电脑配置.assets/20190519213739145.png)

![在这里插入图片描述](Java新手电脑配置.assets/20190519213755419.png)

#### 忽略大小写提示

![在这里插入图片描述](Java新手电脑配置.assets/20190519213809951.png)

取消勾选，match case

#### 设置取消单行tabs的操作
在打开很多文件的时候， IntelliJ IDEA默认是把所有打开的![在这里插入图片描述](Java新手电脑配置.assets/20190519213843134.png)文件名Tab页单行显示的。

#### 设置默认的字体、字体大小、字体行间距


Editor –-> Color Scheme![在这里插入图片描述](Java新手电脑配置.assets/20190519213900330.png)

#### 修改当前主题的控制台输出的字体及字体大小( 可忽略)

![在这里插入图片描述](Java新手电脑配置.assets/20190519213931225.png)

#### 修改代码中注释的字体颜色
Doc Comment – Text： 修改文档注释的字体颜色;Block comment： 修改多行注释的字体颜色;Line comment： 修改单行注释的字体颜色。

![在这里插入图片描述](Java新手电脑配置.assets/2019051921395177.png)

#### 设置项目文件编码
Transparent native-to-ascii conversion 主要用于转换 ascii，一般都要勾选，不然 Properties 文件中的注释显示的都不会是中文。

![在这里插入图片描述](Java新手电脑配置.assets/20190519214303690.png)

#### 设置快捷键(Keymap)
设置快捷为 Eclipse 的快捷键


![在这里插入图片描述](Java新手电脑配置.assets/20190519214501540.png)




#### 关于模板(Templates)

Postfix Completion 默认如下：

![在这里插入图片描述](Java新手电脑配置.assets/20190519214611160.png)


Live Templates 默认如下：

![在这里插入图片描述](Java新手电脑配置.assets/20190519214807637.png)

二者的区别： Live Templates 可以自定义，而 Postfix Completion 不可以。同时，有些操作二者都提供了模板，Postfix Templates 较 Live Templates 能快 0.01 秒。


自定义模板

![在这里插入图片描述](Java新手电脑配置.assets/20190519214857443.png)

#### 插件安装

![在这里插入图片描述](Java新手电脑配置.assets/20190519215400501.png)


RainBow Brackets（彩虹括号）

Maven Helper

ignore：生成各种ignore文件，一键创建git ignore文件的模板。

lombok：通过该插件可以生成实体的GetXXX和SetXXX方法。lombok的注解(@Setter,@Getter,@ToString,@@RequiredArgsConstructor,@EqualsAndHashCode或@Data)，需要在项目中添加依赖。

FindBugs-IDEA：检测代码中可能的bug及不规范的位置。

GsonFormat：根据json文本生成java类。

VisualVM Launcher：运行java程序的时候启动visualvm，方便查看jvm的情况。

GenerateAllSetter：一键调用一个对象的所有set方法并且赋予默认值。

Grep console：自定义日志颜色，idea控制台可以彩色显示各种级别的log，安装完成后，在console中右键就能打开。

Free Mybatis plugin：mybatis 插件，让你的mybatis.xml像java代码一样编辑。

MyBatis Log Plugin：直接将Mybatis执行的sql脚本显示出来，可以直接运行。

Restfultookit：可以根据web访问的url找到对应的controller类，还可以生成测试数据，不用postman来组装数据。

#### 创建 Java Web Project 或 Module
创建的静态 Java Web

![在这里插入图片描述](Java新手电脑配置.assets/20190519215459459.png)


创建动态的 Java Web

![在这里插入图片描述](Java新手电脑配置.assets/20190519215528823.png)

##### 添加Tomcat

![在这里插入图片描述](Java新手电脑配置.assets/20190519215551847.png)

![在这里插入图片描述](Java新手电脑配置.assets/20190519215613305.png)

![image-20200307111153278](Java新手电脑配置.assets/image-20200307111153278.png)

##### 添加jar包

![在这里插入图片描述](Java新手电脑配置.assets/2019051921562815.png)

##### 添加datasource

![在这里插入图片描述](Java新手电脑配置.assets/20190519215646356.png)

#### 版本控制

![在这里插入图片描述](Java新手电脑配置.assets/20190519215703513.png)

#### 断点调试
Shared memory 是 Windows特有的一个属性，一般在 Windows 系统下建议使用此设置， 内存占用相对较少。

![在这里插入图片描述](Java新手电脑配置.assets/20190519215723426.png)

#### 配置Maven

![在这里插入图片描述](Java新手电脑配置.assets/20190519215737905.png)


Import Maven projects automatically：表示 IntelliJ IDEA 会实时监控项目的pom.xml 文件，进行项目变动设置。

![在这里插入图片描述](Java新手电脑配置.assets/2019051921575451.png)

##### 创建对应的Module

![在这里插入图片描述](Java新手电脑配置.assets/20190519215810455.png)

### Eclipse常用快捷键

执行 (run) alt+r
提示补全 (Class Name Completion) alt+/
单行注释 ctrl + /
多行注释 ctrl + shift + /
向下复制一行 (Duplicate Lines) ctrl+alt+down
删除一行或选中行 (delete line) ctrl+d
向下移动行 (move statement down) alt+down
向上移动行 (move statement up) alt+up
向下开始新的一行 (start new line) shift+enter
向上开始新的一行 (Start New Line before current) ctrl+shift+enter
如何查看源码 (class)
ctrl + 选中指定的结构
或
ctrl + shift + t
万能解错 / 生成返回值变量 alt + enter
退回到前一个编辑的页面 (back) alt + left
进入到下一个编辑的页面 ( 针对于上条 ) (forward) alt + right
查看继承关系 (type hierarchy) F4
格式化代码 (reformat code) ctrl+shift+F
提示方法参数类型 (Parameter Info) ctrl+alt+/
复制代码 ctrl + c
撤销 ctrl + z
反撤销 ctrl + y
剪切 ctrl + x
粘贴 ctrl + v
保存 ctrl + s
全选 ctrl + a
选中数行，整体往后移动 tab
选中数行，整体往前移动 shift + tab
查看类的结构：类似于 eclipse 的 outline ctrl+o
重构： 修改变量名与方法名 (rename) alt+shift+r
大写转小写 / 小写转大写 (toggle case) ctrl+shift+y
生成构造器/get/set/toString alt +shift + s
查看文档说明(quick documentation) F2
收起所有的方法(collapse all) alt + shift + c
打开所有方法(expand all) alt+shift+x
打开代码所在硬盘文件夹(show in explorer) ctrl+shift+x
生成 try-catch 等(surround with) alt+shift+z
局部变量抽取为成员变量(introduce field) alt+shift+f
查找/替换(当前) ctrl+f
查找(全局) ctrl+h
查找文件 double Shift
查看类的继承结构图(Show UML Diagram) ctrl + shift + u
查看方法的多层重写结构(method hierarchy) ctrl+alt+h
添加到收藏(add to favorites) ctrl+alt+f
抽取方法(Extract Method) alt+shift+m
打开最近修改的文件(Recently Files) ctrl+E
关闭当前打开的代码栏(close) ctrl + w
关闭打开的所有代码栏(close all) ctrl + shift + w
快速搜索类中的错误(next highlighted error) ctrl + shift + q
选择要粘贴的内容(Show in Explorer) ctrl+shift+v
查找方法在哪里被调用(Call Hierarchy) ctrl+shift+h



# 3. Postman（接口测试工具）



# 4. 项目管理工具maven/gradle

https://www.runoob.com/maven/maven-setup.html

# 5. 版本控制系统git

https://www.runoob.com/git/git-install-setup.html

# 6. 数据库及可视化工具

[Mysql安装配置](https://www.runoob.com/mysql/mysql-install.html)

 Oracle， Redis， MongoDB（关系型数据库与非关系型数据库）

Navicat（可视化工具）

# 7. 办公软件

xshell（链接服务器软件）

xftp（文件传输工具）

# 8. 编辑器

sublime/notepad

Typora

VSCode



# 9. 作图软件

StarUML（作UML图）

XMind（思维导图）

# 10. 其他软件

1. chrome
2. 微信/企业微信/QQ
3. 迅雷/百度网盘
4. 搜狗输入法
5. office/MathType
6. shadowsock

