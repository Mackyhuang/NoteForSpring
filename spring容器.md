#Spring 容器
##@author:MackyHuang

- 是spring框架中的核心模块， 用于管理对象


##启动spring容器步骤
1，导包 （spring-webmvc）  （需要一个设置，在首选项里面的MAWEN设置download）

2，添加配置文件  applicationContext.xml 在 main/resourse

3，启动spring容器

##创建对象



- 方法一，无参构造函数创建*
1，给类创建无参构造函数
2，配置<Bean>元素
3，调用容器的getbean来获取对象



- 方法二，静态工厂方法
调用静态方法 通过factory-method创建对象



- 方法三，实例工厂方法
Factory-bean   一个实例化对象
Factory-method     对象的一个方法
Spring调用方法返回一个对象


##作用域
1，默认情况下， 容器会为一个bean创建一个对象,

2，可以配置作用域， scope=”prototype”

3，scope默认值 singleton 单个实例（一个bean一个实例）

4，Prototype （原型）一个bean会创建多个实例，调用几次就创建几个


##生存周期
init-method 指定初始化函数

destroy-method 指定销毁方法

容器创建对象的时候，会调用初始化方法

容器被销毁时候，会销毁对象，销毁对象的时候会调用销毁方法

如果作用域为原型的时候 (prototype)的时候，不会调用销毁方法，单例才会调用销毁方法


##延迟加载
Spring容器启动的时候，会创建作用域为单例的bean，如果有初始化方法就直接调用初始化方法

Lazy-init可以进行延迟加载，只有在调用getbean的方法的时候才会创建实例


##IOC（inversion of controll  控制反转）


- 对象之间的依赖关系由容器来建立
##DI(dependency injectrion   依赖注入) 

- 容器通过调用对象提供的set方法或者构造器来建立依赖关系
通过DI实现IOC


###set方式注入
			<bean id="b1" class="com.ioc.ClassB"></bean>
		 	<bean id="c1" class="com.ioc.ClassC"></bean>
		  	<bean id="a1" class="com.ioc.ClassA">
		  		<property name="bc" ref="c1"></property>
		 	 </bean>
		      	property:表示用set方法依赖注入
		      	name指属性名，  就是需要注入的属性在类中的实例名字
		      	ref是属性值， 指的是那个实例的bean的id
		      	调用set+name大写的方法,

	       
为了不更改一个源码，需要在设计类的时候，需要灵活变换的类需要实现同一个接口，这样，在调用的主类中就可以直接用接口声明，最后只需要在配置文件中修改ref来更改对象的依赖关系
###构造器方式注入
		 		<bean id="f1" class="com.ioc.AdminDAO"></bean>
  				<bean id="login" class="com.ioc.LoginService">
  					<constructor-arg index="0" ref="f1"></constructor-arg>
  				</bean>
		   		这里用构造函数，所以用constructor-arg
		   		index 表示的是在构造函数中你需要传参数的位置是第几个  0 是第一个
		   		ref还是一样的，就是需要传入的那个bean的id
		    
###自动装配
指的是spring容器依照某种规则，自动建立对象之间的依赖关系，

注意：

a，默认情况下，容器不会自动装配

b，可以通过指定autowire属性告诉spring容器进行自动装配（其实就是容器自己调用set或者构造器的方法进行装配）



- autowire=”byName”

会扫描主类，找到主类中的名字，然后在配置文件中扫描与其名字一样的bean，然后进行set装配，如果找不到对应的值就注入null，不可能找到多个符合条件的bean、



- autowire=”byType”

依据主类中的类型，然后再配置文件中扫描bean的类型，然后用set方法进行装配，如果找不到对应的值就注入null，如果有多个bean都是那个类型，就会报错，


- autowire=”constractor”

和type一样，就是用构造器注入装配

###注入基本类型的值
Value
		  
		      	property:表示用set方法依赖注入
		      	name指属性名，  就是需要注入的属性在类中的实例名字
		      	#ref是属性值， 指的是那个实例的bean的id
		      	如果是基本类型，就用value
		 		<property name="age" value="21"></property>
		  		<property name="name" value="Mackyhuang"></property>

###注入集合类型的值list set map properties
		  		<property name="list">
		  			<list>
		  				<value>Hello</value>
		  				<value>World</value>
		  			</list>
		  		</property>
		  		
		  		<property name="set">
		  			<set>
		  				<value>do</value>
		  				<value>it</value>
		  				<value>again</value>
		  				<value>again</value>
		  			</set>
		  		</property>
		  		
		  		<property name="map">
		  			<map>
		  				<entry key="hello" value="world"></entry>
		  			</map>
		  		</property>
		  		
		  		<property name="properties">
		  			<props>
		  				<prop key="Hello">World</prop>
		  			</props>
		  		</property>
	    

###用引用注入集合类型
需要几行配置文件

			xmlns:util="http://www.springframework.org/schema/util" http://www.springframework.org/schema/util http://www.springframework
			.org/schema/util/spring-util-4.0.xsd)
			  	<util:list id="listBean">
			  		<value>Hello</value>
			  		<value>World</value>
			  		<value>Hello</value>
			  		<value>Spring</value>
			  	</util:list>
			<property name="list" ref="listBean"></property>
			
Ref直接添加那个list的id

###用这个引用的办法读properties文件
			
			  	<util:properties id="config" location="classpath:config.properties">
			  	</util:properties>

Location指定文件路径，固定格式classpath:文件路径

			  	<property name="properties" ref="config">
			  	</property> 
			
这里和上面一样直接应用

###用spring表达式

			格式：#{bean.属性值}
			  	<bean id="el" class="com.ioc.ElBean">
			  		<property name="age" value="#{value.age}"></property>
			  		<property name="name" value="#{value.name}"></property>
			  		<property name="list" value="#{value.list[0]}"></property>
			  		<property name="map" value="#{value.map.hello}"></property>
			  		<property name="property" value="#{config.pageSize}"></property>
			  	</bean>	

列表直接[]  map直接.key  或者[“”]  property直接  util:property的bean的id加上.属性
那个属性必须由get方法


##使用注解简化配置
###组件扫描

在类中进行特殊注解
**@Component**

	  		配置：
	  		context:component-scan base-package=“包名”
	  		这里面指定的包名，就是spring容器会扫描这个包下的类，如果有特殊的注解
	  		（比如@Component）,spring会将其纳入管理
	  		就相当于配置了一个bean属性
	  		@Component  注解 默认类型第一个字母小写 如 SomeBean 那么id为 someBean
	  		@Component("") 引号里面写指定的id
  	 
扫描语句：
**<context:component-scan base-package="annotation"></context:component-scan>**

###还有许多注解   

- **@Component**通用注解
- **@Named**   通用注解
- **@Repository**  持久层组件注解
- **@Service**   业务层组件注解
- **@Controller** 控制层组件注解


###其他注解
- **@Scope  作用域**  在类
- **@Lazy  延迟加载**  在类
- **@PostConstuct**   初始化方法  在方法
- **@PreDestroy**  销毁方法  在方法
- 再次强调：只有单例（singleton）才会调用销毁方法，原型（prototype）不会
- **@Component("sss")**
- **@Scope("singleton")**
- **@Lazy(true)**

			@Component("sss")
			@Scope("singleton")
			@Lazy(true)
			public class SomeBean
			{
				@PostConstruct
				public void init()
				{
					System.out.println("init()");
				}
				
				@PreDestroy
				public void destroy()
				{
					System.out.println("destroy()");
				}
				public SomeBean()
				{
					System.out.println("SomeBean()");
				}
			}




##依赖相关的注解
****
**一共有三种，**

### @Autowired @qualifier("").


- 这个支持对于set和构造器方法的注入


- 对于set :
	
在set的方法前面添加@Autowire的注解，然后在参数的类型前面加上@qualifier(""）， 引号中加上的是那个类型需要的bean的id，如果不加@qualifier，就会类似于自动装配中的Bytype,容易找到多个bean

			@Autowired
		
			public void setSmall(@Qualifier("small") Small small)
			{
				this.small = small;
			}
	
还有这个注解放在另外一个地方的注解

			@Autowired
			@Qualifier("small")
			private Small small;

放在声明的变量的前面，这样做会比放在set前面更简单，电脑直接调用反射机制对变量赋值，而上面那种就需要调用set方法来赋值



- 对于构造器
    
	    	@Autowired
	    	public Stu(@Qualifier("small") Small small)
	    	{
	    		this.small = small;
	    		System.out.println("Stu(small()");
	    	}
上面提到的直接加载在变量前面的注解，因为他是直接赋值，所以set和构造器都可以用那种办法，甚至可以没有构造器和set方法


###@Resourse
使用方法也是类似的，因为**只能用于set方法**，所以，添加在set前面，或者直接添加在变量前面

			@Resource(name="small")
			public void setSmall(Small small)
			{
				this.small = small;
			}

这里**@Resource**里面可以指定参数，name，里面是需要传入的bean的id，如果不添加，就类似于ByType


###注解用spring表达式
**@value("#{}")**

- 可以用表达式，可以设置基本类型的值
- 可以使用在set方法前，可以直接在变量前面添加，原理和前面的一样

			@Value("#{config.pageSize}")
				private int pageSize;
			@Value("Mackuhuang")
			private String name;

和spring表达式的用法一样，注意：配置文件读取注意大小写






