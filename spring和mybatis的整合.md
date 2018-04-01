#spring和mybatis的整合
##@MackyHuang

##基本步骤
- 导包
> spring<br>
> mybatis<br>
> mybatis-spring<br>
> ojdbc(oracle)  mysql-connector(mysql)<br>
> dbcp

- 配置springmvc.xml文件，因为mybatis已经集成在spring中，所以mybatis的配置也在springmvc的配置文件中
<br>配置`SqlSessionFactoryBean`，这里是配置数据库连接池和sql映射文件位置
- 实体类的编写
- sql映射文件的编写
- dao层接口的编写（mapper映射器）
- springmvc配置文件中添加`MapperScannerConfigurer`，这里代表映射接口的包（替代原本的getMapper()方法），创建符合映射器接口的对象，然后将这个bean加入spring容器


##具体步骤
###第一次的spring配置文件<br>

	      <!-- 配置文件 -->
		 <util:properties id="config" location="classpath:db.properties"></util:properties>
			<!-- 连接池 -->
	      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	      		<property name="driverClassName" value="#{config.driver}"></property>
	      		<property name="url" value="#{config.url}"></property>
	      		<property name="username" value="#{config.username}"></property>
	      		<property name="password" value="#{config.password}"></property>
	      </bean>
	    
	    	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	    		<!-- 指定连接的数据库连接池 -->
	    		<property name="dataSource" ref="dataSource"></property>
	    		<!-- 指定sql映射配置文件位置 -->
	    		<property name="mapperLocations" value="classpath:entity/*.xml"></property>
	    	</bean>
这里不在使用mybatis自带的连接池，而是用spring配置提供的连接池，然后创建sqlSessionFactoryBean的bean，然后再属性dataSource中加入数据库连接池的引用，在mapperLocations加入sql映射文件的位置，这里是`classpath:entity/*.xml`，代表在entity下的所有xml,在开发时，将类的相应的Sql映射文件放在entity中<br>
mybatis中，不再需要编写一堆的相关于sqlSession创建的代码，因为通过这个配置，spring会帮你把sqlSession给创建好

###第二次的spring配置文件<br>

		<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
			<property name="basePackage" value="dao"></property>
		</bean>
这个配置可以让spring找到dao层中你写的mapper映射器，通过这个接口来产生相应的符合接口条件的对象，然后存在spring容器中<br>
值得注意的是，在这个配置文件中，没有加入组建扫描，可是，spring容器依然可以找到默认名字的dao接口（类名首字母小写）或者是@Repository注解重命名的bean（还记得这个持久层的组件注解吗），因为配置了这个`MappersScannerConfiguer`，它就扮演这组件扫描的角色，会在这个包下去查找接口

###是否会有疑问，如果dao层存在不需要mybatis自动根据接口创建对象的接口时，怎么办？
以上的这种情况有可能出现，mybatis再好用，也是封装好的数据库框架，在效率上不如原生的jdbc，如果在一个业务中，大部分代码需要mybatis框架，少部分需要高效的部分需要jdbc，那么，根据解耦的原理，在dao层内要出现一个接口，然后通过一个自己编写的类来实现这个接口，这个时候，就不希望这个接口被mybatis所“看到”<br>
**这个时候，就需要自己编写一个注解：**<br>
在`MapperScannerConfiguer`中，有一个name为annotationClass的属性，需要给他的值是自己写的一个注解的限定名

		<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
			<property name="basePackage" value="dao"></property>
			<property name="annotationClass" value="annotation.MyDAOAnnotation"></property>
		</bean>

这是加进`annotationClass`的`MapperScannerConfiguer`，这个注解只需要创建即可，不需要在里面添加任何逻辑代码，然后在dao层下，需要被mybatis解析的接口前加上这个注解，就可以完成接口的解析

##另外一个集成方式：
- 导包
- springmvc配置文件（删除MapperScannerConfiguer配置）
- 实体类
- 映射文件
- dao接口
- dao实现类（依赖注入sqlSessionTemplate）
- sqlSessionTemplate,调用该对象方法
- 配置sqlSessionTemplate

	     		<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
	    			<constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
	    		</bean>
- 组件扫描