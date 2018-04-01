
----------

# Splendid Error
##@author:MackyHuang

----------
<br><br>

----------

### Springmvc打开tomcat服务器的时候，遇到以下报错：
    		java.util.concurrent.ExecutionException: 
			org.apache.catalina.LifecycleException: Failed to start 
			component [StandardEngine[Catalina].StandardHost
			[localhost].StandardContext[/springmvc]]

Maven包中的springframe的jar包的不支持当前的tomcat，解决方法：<br>

- 手动下载springframe的包
- 导入到WEB-INF/lib目录
- 构建路径
- 项目属性->Deployment Assembly添加依赖


----------

###使用tomcat服务器时候，遇到以下报错：
	Could not publish server configuration for Tomcat v8.5 Server at localhost.
	Multiple Contexts have a path of "/Springmvc".


tomcat下面的`server.xml`里面存在多个这个项目的配置，需要进入然后删除`<Context>`标签

- 在项目列表里面，会在创建tomcat服务器的时候有一个`Servers`的文件夹
> Servers\Tomcat v8.5 Server at localhost-config\server.xml

- eclipse中配置的tomcat还可以在工作空间中找到
 
>  .metadata\.plugins\org.eclipse.wst.server.core\tmp\conf\server.xml

- tomcat的独自服务器
> Tomcat\conf\server.xml

----------
###配置多个tomcat服务器


- 下载解压tomcat服务器到磁盘
- 配置环境变量：<br>
第一个tomcat配置名称`CATALINA_HOME`路径是这个tomcat的根路径<br>
第二个tomcat配置名称`CATALINA_HOME2`路径是这个tomcat的根路径
- 修改`startup.bat`和`catalina.bat`：<br>
在`C:\apache-tomcat-8.5.9\bin\startup.bat`和`C:\apache-tomcat-8.5.9\bin\catalina.bat`中将第二个里面所有的`CATALINA_HOME`修改为`CATALINA_HOME2`
- 修改端口：<br>
俩个服务器的端口不能相同，否则不能同时开启<br>
在`C:\apache-tomcat-8.5.9\conf\server.xml`中修改其中一个的端口号`Connector port="8081"`


----------
### 在创建jsp文件的时候，遇到一下报错：
> The superclass "javax.servlet.http.HttpServlet" was not found on the Java Build Path

原因是项目的tomcat没配置，右击项目选中属性，找到TargetedRuntimes，添加tomcat服务器




----------
###使用eclipse的时候出现
> 访问限制：由于对必需的库 C:\Program Files\Java\jre1.8.0_121\lib\rt.jar 具有一定限制，因此无法访问类型 Resource

这样的错误，是因为JRE的环境库的问题，解决方法如下：<br>
####1、右键该项目（如：QQ）---->  构造路径  ---->  配置构造路径<br>
####2、java构造路径 ----> 库 ---->  JRE系统库（双击）----> 点击“执行环境” ----> 选“JavaSE-1.7（jre1.8.0_20）”----> 点“完成”。<br>


----------
###mybatis和spring整合时候的报错
java.lang.AbstractMethodError: org.mybatis.spring.transaction
这个表示，spring和mybatis的版本，和mybatis-spring的版本不兼容

###@Resource注解找不到
注解找不到，它是在JDK下的`javax.annotation.Resource`
找不到是因为项目的jdk版本不行，解决方法：<br>
右击项目->属性->program facets->java改成1.8+