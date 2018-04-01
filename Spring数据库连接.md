#SpringJDBC
##@author:MackyHuang

mybatis hibernate
##使用步骤
- 导包
> spring-mvc <br>
>  spring-jdbc  <br>
>  ojdbc(oracle)  mysql-connector(mysql)<br>
>  dbcp<br>

- 添加springmvc配置文件
- 配置JdbcTemplate
JdbcTemplate提供了一些方法，用来访问数据库
- 调用JdbcTemplate提供的方法来访问数据库，将JdbcTemplate注入到DAO。 

		      <!-- 配置文件 -->
			 <util:properties id="config" location="classpath:db.properties"></util:properties>
				<!-- 连接池 -->
		      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		      		<property name="driverClassName" value="#{config.driver}"></property>
		      		<property name="url" value="#{config.url}"></property>
		      		<property name="username" value="#{config.username}"></property>
		      		<property name="password" value="#{config.password}"></property>
		      </bean>
		      <!-- jdbcTemplate -->
		      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		      	<property name="dataSource" ref="dataSource"></property>
		      </bean>
####连接过程中，配置文件用util:properties读取，然后创建DBCP连接池，用spring表达式对里面的属性进行赋值，然后创建jdbcTemplate的bean然后给他的dataSource一个连接池的引用

###这里是数据库连接池配置文件的内容：
		    # db.properties
		    driver=com.mysql.jdbc.Driver
		    #url=jdbc:mysql://localhost:3306/worktime
		    url=jdbc:mysql:///worktime
		    username=root
		    password=8023
		    
		    # paramter for BasicDataSource
		    initSize=1
		    maxActive=2
		    
###DAO层的代码三个部分，sql,object数组和jdbcTemplate的函数
		public void save(Time time)
		{
			String sql = "INSERT time (date, start, end) VALUES (?,?,?)";
			Object[] objects = new Object[] {time.getDate(), time.getStart(), time.getEnd()};
			jdbcTemplate.update(sql, objects);
		}

####下面是查询所有的代码

		public List<Time> findAll()
		{
			String sql = "SELECT * FROM time";
			List<Time> times = jdbcTemplate.query(sql, new Timemap());
			return times;
		}
这里需要有一个类，`Timemap()`,

	    class Timemap implements RowMapper<Time>
	    {
	    	public Time mapRow(ResultSet resultSet, int index) throws SQLException
	    	{
	    		Time time = new Time(resultSet.getString("date"),resultSet.getString("start"),resultSet.getString("end"));
	    		return time;
	    	}
	    }
这个类需要实现一个接口`RowMapper<Time>`,然后实现一个函数，`mapRow(ResultSet resultSet, int index)`这个函数里面有怎么把resultset转化成
对象的过程，然后把这个类传进jdbcTemplate的方法，在有返回对象列表的时候需要这个参数
####下面是查询一个的代码
	
		public Time findById(int id)
		{
			String sql = "SELECT * FROM time WHERE id = ?";
			Object[] objects = new Object[] {id};
			Time time = jdbcTemplate.queryForObject(sql, objects, new Timemap());
			return time;
		}
代码与上面类似，一样需要那个`Timemap()`

####下面是更新数据的代码
 
		public void update(Time time, int id)
		{
			String sql = "UPDATE time set date = ?, start = ?, end = ?";
			Object[] objects = new Object[] {time.getDate(), time.getStart(), time.getEnd()};
			jdbcTemplate.update(sql, objects);
		}
####下面是删除数据的代码
		public void delete(int id)
		{
			String sql = "DELETE FROM time WHERE id = ?";
			Object[] objects = new Object[] {id};
			jdbcTemplate.update(sql, objects);
		}

<br>
----------

----------

----------

<br>
#mybatis
apache->google->github
###开源的持久层框架
##编程步骤
- 导包
>  mybatis <br>
>  ojdbc(oracle)  mysql-connector(mysql)<br>

- 添加mybatis配置文件
- 写实体类，注意实体类的属性名和表的字段名一样，不区分大小写
- 映射文件，映射文件写完之后需要在配置文件中指定位置，并且目录的分隔符使用  **/**
- 调用mybatis提供的SqlSession提供的方法来访问数据库


##具体如下：
###配置文件`sqlMapConfig.xml`

		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE configuration
		PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-config.dtd">
		
		<!-- 通过这个配置文件完成mybatis与数据库的连接 -->
		<configuration>
		<environments default="environment">
			<environment id="environment">
				<!--配置事务管理，采用JDBC的事务管理 -->
				<transactionManager type="JDBC"></transactionManager>
				<!-- POOLED:mybatis自带的数据源，JNDI:基于tomcat的数据源 -->
				<dataSource type="POOLED">
					<property name="driver" value="com.mysql.jdbc.Driver"/>
					<property name="url" value="jdbc:mysql:///worktime"/>
					<property name="username" value="root"/>
					<property name="password" value="8023"/>
					</dataSource>
			</environment>
		</environments>

		<!-- 将mapper文件加入到配置文件中 -->
		<mappers>
			<mapper resource="entity/timeMapper.xml"/>
		</mappers>
		</configuration>
mytatis自带一个数据库连接池，这里一样的配置

###然后是实体类
就是包含数据库中的字段名，属性名与其一模一样，设置get/set方法

###映射文件
命名规范一般是实体类名字+Mapper

		<?xml version="1.0" encoding="UTF-8"?> 
		<!DOCTYPE mapper PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"      
		 "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
		
		<mapper namespace="test">
			<insert id="insert" parameterType="entity.Time">
				INSERT INTO time (date, start, end) VALUES (#{date}, #{start}, #{end})
			</insert>
		</mapper>

这里的namesapce确定命名空间，到时候调用的时候可以根据前面的namespace来区分不同表的查询<br>
然后这个insert标签里面，id是这个标签的名字，唯一性，到时候通过这个id来调用，然后是parameterType，里面是这个查询的相应的实体类对应的表<br>
然后就是一条sql语句，和之前的纯jdbc以及jdbcTemplate相比，这里的sql语句还是有区别的，这里的不再是用？来代表可替换参数，而是直接用spring表达式来读取类中属性值

###SqlSession的使用来访问数据库
	SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
			SqlSessionFactory sessionFactory = sessionFactoryBuilder.build(Test.class.getClassLoader().getResourceAsStream("sqlMapConfig.xml"));
			SqlSession sqlSession = sessionFactory.openSession();
			sqlSession.insert("test.insert", new Time("2018-03-24", "18:00:00", "18:00:00"));
			sqlSession.commit();
			sqlSession.close();
注：上面这句build函数中的参数，是使用系统类加载器来获取资源<br>

- 首先获取`SqlSessionFactoryBuilder`
- 通过上一个类的`build()`方法然后获取`SqlSessionFactory`
- 最后从上一个类的`openSession()`方法得到`SqlSession`
- `SqlSession`调用相关的方法来访问数据库
- mybatis不会自动提交，增删改的时候需要手动调用commit函数提交
- 最后关闭`SqlSession`

现在添加一个查询函数，映射文件:

	<select id="findAll" resultType="entity.Time">
		SELECT * FROM time
	</select>
在查询的时候，需要的是返回结果的类型，所以这里的属性是resultType<br>
在更新的时候，需要的是传入的参数类型，所以这里需要的属性是parameterType

##mybatis基本原理
- `SqlSessionFactory`在创建的时候需要一个mybatis的配置文件，这个时候读取这个配置文件，这个配置文件主要包含俩个部分，数据库连接池和sql语句的映射文件的位置，这里就包含了sql语句
- `SqlSessionFactory`在读取到这个文件以后，根据数据库连接和sql语句，创建了许多预编译的PrepareStatement

    		`PrepareStatement prepareStatement = connection.prepareStatement(sql);`

就是类似于以上的这种语句结构，不过这个时候只是有sql语句，还缺少必要的参数<br>
这些`prepareStatement`被存在Map中，Map中的key的值就是映射文件中的id，而value就是那些`prepareStatement`<br>

- 接下来`SqlSessionFactory`创建了`SqlSession`
- 用户调用`SqlSession`的函数，参数列表第一个就是映射文件中的id，后面就是访问数据库必要的参数<br>
很明显，这些就是`prepareStatement`缺少的参数
- `SqlSession`通过id在Map中找到了对应的`prepareStatement`，然后传入参数执行这个语句
- 返回执行结果，如果是查询，会将记录转换成相应的实体对象，因为在创建实体类的时候就需要严格要求实体类的属性名需要和表中的字段名完全一致

###增删改查映射文件部分

		<mapper namespace="test">
			<insert id="insert" parameterType="entity.Time">
				INSERT INTO time (date, start, end) VALUES (#{date}, #{start}, #{end})
			</insert>
			<select id="findAll" resultType="entity.Time">
				SELECT * FROM time
			</select>
			<select id="findById" parameterType="int" resultType="entity.Time">
				SElect * FROM time WHERE id = #{id}
			</select>
			<update id="update" parameterType="entity.Time">
				UPDATE time SET date=#{date}, start=#{start}, end=#{end} WHERE id = #{id}
			</update>
			<delete id="delete" parameterType="int">
				DELETE FROM time WHERE id = #{id}
			</delete>
		</mapper>
- 添加的标签是< insert ><br>
添加需要的是传入参数，不需要关心返回类型
- 查询的标签是< select ><br>
无条件查询所有只需要关心返回类型，只查询某一个就需要多一个查询条件<br>
如果是数字的话，参数类型的int或者java.lang.Integer
- 修改的标签是< update ><br>
修改的时候，传入类型，如果需要传入多个参数，需要有其他方法
- 删除的标签是< delete ><br>
- 删除只需要传入类型即可

###返回的不是对象，而直接是一个map
在内部，封装成对象之前，从数据库中得到的数据的格式其实就是map，属性名和属性值是一个键值对，所以这里可以直接返回一个map，所以这里可以直接拿到这个中间结果map <br>
这里用findById的查询方法来演示：

		<select id="findById2" parameterType="int" resultType="map">
			SElect * FROM time WHERE id = #{id}
		</select>

这里的改变就是返回类型的地方，不再是实体类，而是map或者java.util.Map，这样返回的就是一个map
并且在去map里面的属性的时候，在oracle中属性名（key）需要全部大写，而mysql需要首字母大写
> data.get("Date")

 ##若属性名和表的字段名不一 样
（1）取别名<br>
（2）使用resultMap
如果属性名和表的字段名不一样，获取到的结果会只有俩个名字匹配的属性，其他的属性值为null<br>
这时候使用结果映射

	<resultMap id="time2Map" type="entity.Time2">
		<result property="tdate" column="date"></result>
		<result property="tstart" column="start"></result>
		<result property="tend" column="end"></result>
	</resultMap>
这里type就是需要转换的实体类，id唯一，在property中填入实体类的属性名，然后column填入表中的字段名
写完之后，回到查询中，不需要resultType，更换为resultMap，然后名字为那个id

## mybatis的mapper映射器（接口）
在开发过程中，需要在持久层创建一个接口，然后用这个接口创建一个实体的持久层对象，在这里使用映射接口之后，可以省略按个实体类的编写

###映射接口的要求：
- 接口的方法名称要和sql配置文件中的id一样
- 返回类型要和resultType一样
- 参数类型要和paramterType一样
- 这个接口的全限定类名需要和sql配置文件的namespace一样

###编程步骤：
- 编写一个接口
- 调用sqlSession的getMapper方法,返回一个持久层对象（映射器要求的对象）

      	  <mapper namespace="dao.TimeDAO"></mapper>
举例：<br>
sql配置文件中：

		<select id="findById" parameterType="int" resultType="entity.Time">
			SElect * FROM time WHERE id = #{id}
		</select>
DAO映射器接口中：

		    public Time findById(int id);
测试函数中：
	    	
		    TimeDAO timeDAO = sqlSession.getMapper(TimeDAO.class);
		    List<Time> times = timeDAO.findAll();


值得注意的是，如果使用上面提到的resultMap的方法处理表的字段名和类的属性名不一样，那么在映射器中的方法的返回类型需要写的是resultMap中的type中的类型<br>
**还有一个重点，这样的操作，最后的增改删还是需要sqlSession.commit()来提交事务**


