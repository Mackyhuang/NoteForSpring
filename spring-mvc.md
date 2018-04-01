# Spring-MVC
##@author:MackyHuang
一个mvc框架，用来简化基于mvc架构的web应用开发<br>

##五大组件
###DispatcherServlet（前端控制器）
接受请求，依据HandlerMapping的配置调用相应的模型来处理
###HandlerMapping
包含了请求路径和模型的对应关系
###Controller（处理器）
负责处理业务逻辑
###ModelAndView
封装了处理结果
注：处理结果除了数据之外，还可能有视图名
###ViewResolver（视图解释器）
DispatcherServley依据ViewResolver的解析，
调用真正的视图对象来生成相应的页面

### Spring-MVC五大组件关系图 ###
![Spring-MVC五大组件关系图](https://i.imgur.com/XwLVBIC.png)


###五大组件工作步骤：
- DispatcherServlet收到请求之后，依据HandlerMapping的配置，调用相应的Contrller来处理
- Controller将处理结果封装成ModelAndView对象，然后返回给
- DispatcherServlet依据ViewResolver的解析调用相应的视图对象（比如某个jsp）来生成相应的页面

##编程步骤


###导包，springframe的包


###添加spring配置文件


###在`WEB-INF\web.xml`中配置DispatcherServlet<servlet>

		      	<servlet-name>springmvc</servlet-name>
		      	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		      	<!-- 
		      		DispatcherServlet的初始化方法中会获取初始化参数的值
		      		这些值可以让DispatcherServlet获取spring配置文件的位置
		      		然后启动spring容器
		      	 -->
		      	<init-param>
		      		<param-name>contextConfigLocation</param-name>
		      		<param-value>classpath:springmvc.xml</param-value>
		      	</init-param>
		      	<load-on-startup>1</load-on-startup>
		      	</servlet>
		      	<servlet-mapping>
		      	<servlet-name>springmvc</servlet-name>
		      	<url-pattern>*.do</url-pattern>
		      	</servlet-mapping>
    
此时的错误详细见Splendid Erroer.大概就是关于MAVEN的jar包的管理<br>

###Controller的创建

创建的时候，创建的类需要继承`Controller`的接口，实现里面的一个`handleRequest`的方法
		
			public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception
			{
				//添加业务处理
				System.out.println("handleRequest()");
				/*
				 * ModelAndView的俩个构造器
				 * (1)ModelAndView(String viewName)
				 * ViewName 为 视图名称
				 * (2)ModelAndView(String viewName, Map map)
				 * map用于返回处理的结果数据
				 * */
				return new ModelAndView("hello");
			}
这里有一个问题：
>   无法解析org.springframework.web.servlet.mvc.Controller
  
原因未知，解决办法是，将mawen中的springframe的jar包删除，只是使用自己下载添加进去的springframe，就可以运行

###在`springmvc.xml`配置`HandlerMapping`

	      <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	      	<property name="mappings">
	      		<props>
	      			<prop key="/hello.do">helloController</prop>
	      		</props>
	      	</property>
	      </bean>
key的是请求的网址，然后转跳到prop标签里面的那个id对应的处理器的bean
	      	随后处理器返回视图名通过ModelAndView对象给ViewResolver
###在`springmvc.xml`配置`Controller`

	      <bean id="helloController" class="controller.HelloController"></bean>
控制器的名字将会用来使用在handlemapping里面
###在`springmvc.xml`配置`ViewResolver`
	  
	      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	      	<property name="prefix" value="/WEB-INF/"></property>
	      	<property name="suffix" value=".jsp"></property>
	      </bean>
 Controller封装成ModelAndView对象传过来的视图名，会通过以下配置映射到/WEB-INF/hello.jsp

(小提示：如何正确保证自己在导入spring包的时候文件名的正确：<br>
在库中找到相应的那个包，然后右击->复制限定名->粘贴->删去尾巴的.class)

###重新梳理一次整个spring-mvc的运行过程：<br>
1. 用户在网页上输入`http://localhost:8023/springmvc/hello.do`发送请求
2. 启动`DispatcherServlet`，此时，在`web.xml`文件中会获得spring配置文件的地址，然后启动spring容器
3. spring容器一旦创建，就会创建好三个对象(默认为单例singleton),分别是`HandlerMapping`, `Controller`, `ViewResolver`
4. `DispatcherServlet`依据`HandlerMapping`的配置，找到其`prop`标签内id名称对应的bean，调用这个bean的`Controller`对象
5. `Controller`处理业务信息，返回一个`ModelAndView`对象给`DispatcherServlet`
6. `DispatcherServlet`根据`ModelAndView`中的视图名，根据`ViewResolver`解析出一个视图对象（比如某个`.jsp`），响应给用户

###回过头来，用简单的方法配置一次springmvc
(此处直接显示出不同之处)

		    @Controller
		    public class Login 
		    {
		    	@RequestMapping("/login.do")
		    	public String login()
		    	{
		    		return "login";
		    	}
		    }

 * 可以不使用Controller的接口直接创建多个方法，每个方法处理一种请求
 * 方法名不做要求，和前面不同的还有返回类型，之前是ModelAndView,现在返回字符串类型
 * 使用`@Controller`来代替配置文件的bean,直接纳入容器进行管理
 * （记得这个需要在配置文件组件扫描）
 * 使用`@RequestMapping`,告知`DispatcherServlet`请求的路径和
 * 处理器类种方法的对应关系（spring配置文件种不用配置`HandlerMapping`）
 * 为了可以找的到`@RequestMapping`，需要mvc注解扫描
 `<mvc:annotation-driven></mvc:annotation-driven>`
* `springmvc.xml`中只需要组件扫描，mvc注解扫描和ViewResolver配置
 * @RequestMapping也可以添加在类的前面，当作模块名
 * 未添加需要访问：    `http://ip:port/springmvc02/login.do`
 * 添加需要访问：`http://ip:port/springmvc02/demo/login.do  `
 
##Controller类读取参数
> jstl小记
> <%=request.getParameter("name") %>等效于${param.name }
> <%=request.getAttribute("name") %>等效于${name }

这是之前读取参数的办法，在jsp中读取表单或者servlet中传过来的值

### 第一种方式：
通过HttpServletRequest对象，使用`request.getParameter("name") `方法获取表单数据，因为`DispatcherServlet`在调用`Controller`的时候会传一个`Request`对象。

		    @RequestMapping("/hello.do")
		    	public String hello(HttpServletRequest request)
		    	{
		    		String user = request.getParameter("name");
		    		String password = request.getParameter("password");
		    		return "hello";
		    	}
###第二种方式：
通过注解来获取，值得一提的是，在创建一个函数的时候，如果传入的参数不是Request对象，而是一个个变量，如果这个变量的名字和表单中的参数的name的值一样，那么，不需要注解也完全可以让函数得到参数的值，如下：

			@RequestMapping("/hello.do")
			public String hello(String name, String password)
			{
				System.out.println("Hello，" + name);
				System.out.println("确认您的密码，" + password);
				return "hello";
			}
可是如果这个时候参数名字不一样，就无法的到值，这个时候就需要注解`@RequestParam("") `添加在相应的变量参数前面，引号中需要填写表单中的name，如下：
		
			@RequestMapping("/hello.do")
			public String hello(@RequestParam("name") String user, String password)
			{
				System.out.println("Hello，" + user);
				System.out.println("确认您的密码，" + password);
				return "hello";
			}
注：推荐即使参数名字和name一样，也加上`@RequestParam`的注解， 因为根据java的反射机制的一个缺点，有的时候在检查参数的时候无法读到参数名字，只能读到变量的类型，这个时候就会得到一个null。因为eclipse会自动添加所以才能不用这个注解直接访问，将来没有这个环境的时候，为了节省空间可能不会获得这个参数的具体名字

###第三种方式：
通过javabean的封装来获取

			@RequestMapping("/hello.do")
			public String hello(User user)
			{
				System.out.println("Hello，" + user.getName());
				System.out.println("确认您的密码，" + user.getPassword());
				return "hello";
			}
编写一个javabean封装表单中所有的name，其中包括所有的name，并且需要set和get方法，然后在方法中直接传入一个对象。

##Controller类给页面传值
### 第一种方式：
使用Request对象绑定对象然后转发给页面
		
			@RequestMapping("/hello.do")
			public String hello(User user, HttpServletRequest request)
			{
				System.out.println("Hello，" + user.getName());
				System.out.println("确认您的密码，" + user.getPassword());
				request.setAttribute("name", user.getName());
				request.setAttribute("password", user.getPassword());
				return "hello";
			}
这里的用法和servlet的用法一样，设置属性值然后转发给页面，值得注意的是，这里的默认就是转发，所以直接返回就好
### 第二种方式：
返回ModelAndView对象来传参数

			@RequestMapping("/hello.do")
			public ModelAndView hello(User user)
			{
				System.out.println("Hello，" + user.getName());
				System.out.println("确认您的密码，" + user.getPassword());
				Map<String, Object> map = new HashMap<String, Object>();
				map.put("user", user);
				ModelAndView modelAndView = new ModelAndView("hello", map);
				return modelAndView;
			}
这里返回`ModelAndView`，其中有一个带数据结果的Map对象，在内部，map的键值对和request中setAttribute的键值对结果是一样的。key的值和对应绑定名
###第三种方式：
使用`ModelMap`对象

			@RequestMapping("/hello.do")
			public String hello(User user, ModelMap modelMap)
			{
				System.out.println("Hello，" + user.getName());
				System.out.println("确认您的密码，" + user.getPassword());
				modelMap.addAttribute("name", user.getName());
				modelMap.addAttribute("password", user.getPassword());
				return "hello";
			}
在参数传入一个`ModelMap`对象，用法和request大同小异：`modelMap.addAttribute("name", user.getName())`
###第四种方式：
使用`HttpSession`对象

	@RequestMapping("/hello.do")
	public String hello(User user, HttpSession session)
	{
		System.out.println("Hello，" + user.getName());
		System.out.println("确认您的密码，" + user.getPassword());
		session.setAttribute("name", user.getName());
		session.setAttribute("password", user.getPassword());
		return "hello";
	}
` HttpServletRequest`对象都可以传递，`HttpSession`对象肯定也是可以的，用法与request相同

##重定向（默认转发）
###第一种方式（只返回一个字符串的）：

			@RequestMapping("/tohello.do")
			public String tohello()
			{
				System.out.println("tohello()");
				return "redirect:hello.do";
			}
return 语句的string中，加上`redirect: `然后加上请求的页面的对应的RequestMapping
###第二种（返回一个ModelAndView）：
		
			public ModelAndView tohello()
			{
				System.out.println("tohello()");
				RedirectView redirectView = new RedirectView("hello.do");
				ModelAndView modelAndView = new ModelAndView(redirectView);
				return modelAndView;
			}

##系统分层
### 重温MVC 
- Model  控制逻辑
- View  表示逻辑
- Controller  封装业务逻辑

- Model分为<br>
业务逻辑 + 数据访问<br> 
也就是  Service（服务类） + DAO（持久化类）<br>

###如何分类
这里的分层通过mvc继续细化：<br>

- 表示层：数据展示[V]和控制逻辑[C]（请求分发） 
- 业务层：业务逻辑的处理[M]
- 持久层：数据库的访问[M]


###
- 上一层通过接口调用下一层提供的服务，比如，业务层调用持久层提供的接口
- 下一层发生改变，不影响上一层，方便代码的维护和工作的分配


##处理字符
网页出现乱码的原因：
提交表单的时候，浏览器会对中文进行编码，会使用在打开表单所在页面的字符所用的字符集来编码，比如现在使用的utf-8，然而服务器默认会用iso-8859-1来解码
###此时可以使用springmvc提供的一个字符过滤器`CharacterEncodingFilter`进行配置，也有一定的要求
过滤器的条件

-   		表单的提交方式是post
-   		页面的编码和过滤器的配置是一样的
-   		因为这个过滤器其实就是调用了request.setCharacterEncoding()这个方法，其条件就是这个
###详细配置之如下（在web.xml文件中配置）

			<filter>
		  		<filter-name>encodingFilter</filter-name>
		  		<filter-class>
		  			org.springframework.web.filter.CharacterEncodingFilter
		  		</filter-class>
		  		<init-param>
		  			<param-name>encoding</param-name>
		  			<param-value>UTF-8</param-value>
		  		</init-param>
		  	</filter>
		  	<filter-mapping>
		  		<filter-name>encodingFilter</filter-name>
		  		<url-pattern>/*</url-pattern>
		  	</filter-mapping>

## 拦截器（interceptor）
拦截器，其实就是DispatcherServlet收到请求之后，如果有拦截器，会先调用拦截器，然后再调用相应的controller
> 过滤器是属于servlet的规范，而拦截器是属于springmvc框架的组件
> 过滤器是请求到达容器的时候先工作的，而拦截器是要调用DispatcherServlet的时候先工作的

###拦截器编写步骤
- 写一个JAVA类，实现HandlerInterceptor接口
- 实现具体的拦截逻辑处理
- 配置拦截器（告诉servlet哪些需要拦截）


			public class MyInterceptor implements HandlerInterceptor
			{
				public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception
				{
					System.out.println("preHandle()");
					HttpSession session = request.getSession();
					Object object = session.getAttribute("name");
					if(object == null)
					{
						response.sendRedirect("login.do");
						return false;
					}
					return true;
				}
				public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
						ModelAndView modelAndView) throws Exception
				{
					System.out.println("postHandle");
				}
				public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
						throws Exception
				{
					System.out.println("afterCompletion");
				}	
			}


- `preHandle（）`<br>
DispatcherServlet收到请求之后，会先调用preHandle方法
如果这个方法的返回值是True就继续向后调用，如果返回False就
不会往后调用，第三个参数描述的是Controller方法的一个对象*/
- `postHandle()`<br>
处理器Controller的方法已经执行完毕之后。正准备
 将结果ModelAndView对象返回给DispatcherServlet之前
执行这个postHandle方法，可以在这个方法中修改处理结果
- `postHandle`<br>
这是最后执行的方法，只有在preHandle方法返回一个true的时候
这个方法才会执行
可以写一个拦截器，专门用来处理Controller抛出来的异常

###配置文件
		<mvc:interceptors>
			<mvc:interceptor>
				<mvc:mapping path="/**"/>
				<mvc:exclude-mapping path="/login.do"/>
				<mvc:exclude-mapping path="/tohello.do"/>
				<bean class="controller.MyInterceptor"></bean>
			</mvc:interceptor>
		</mvc:interceptors>
- 先后顺序执行
- 配置的时候，如果是/*只会拦截/hello.do，不会拦截/demo/hello.do
- 这个时候配置/** 他会拦截所有请求
- mvc:exclude-mapping里面是允许访问的

##异常处理
###第一种方法(适合全局处理简单异常):
- 直接配置简单异常处理器

		<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
			<property name="exceptionMappings">
				<props>
					<prop key="java.lang.NumberFormatException">error</prop>
				</props>
			</property>
		</bean>
创建一个`SimpleMappingExceptionResolver`的bean，里面Key的值就是异常的全称，而prop里面的内容是一个视图名，可以通过这个转跳到一个提示页面
###第二种方法(annotation):
- 在controller中的一个方法前面注解@ExceptionHandler，然后在里面用instanceof来判定异常的类型，然后做出相关的业务处理，然后返回一个视图名（返回一个ModelAndView对象也是可以的）

			@ExceptionHandler
			public String myException(Exception exception, HttpServletRequest request)
			{
				if(exception instanceof NumberFormatException)
				{
					request.setAttribute("meg", "NumberFormatException");
					return "error";
				}
				else if(exception instanceof StringIndexOutOfBoundsException)
				{
					request.setAttribute("meg", "StringIndexOutOfBoundsException");
					return "error";
				}
				else 
				{
					request.setAttribute("meg", "system_error");
					return "error";
				}
			}
####相比而言，第一种方法只能进行简单的异常获取后跳转到页面，而第二种方法可以再获取异常之后，进行一系列的操作然后转跳到页面

