##1.5CRM综合练习

###1.5.1搭建开发环境

####1.5.1.1第一步：创建web项目。引入jar包。

####1.5.1.2第二步：引入配置文件

	struts2的框架
		web.xml配置
			<!-- 配置Struts2的核心过滤器 -->
		 	<filter>
		 		<filter-name>struts2</filter-name>
		 		<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
		 	</filter> 
		 	
		 	<filter-mapping>
		 		<filter-name>struts2</filter-name>
		 		<url-pattern>/*</url-pattern>
		 	</filter-mapping>

		Struts.xml配置
			
		spring框架
		jdbc.properties		
		log4j.properties
		applicationContext.xml
		web.xml
			<!-- spring的核心监听器 -->
		  	<listener>
		  		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		  	</listener>
		  
		  	<!-- 用来加载spring的配置文件的路径的，默认加载的/WEB-INF/applicationContext.xml -->
		  	<context-param>
		  		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
  	</context-param>
####1.5.1.3添加相关的包结构

####1.5.1.4添加相关的jsp页面

####1.5.1.5创建相关的类



####1.5.1.6相关的类配置到Spring

	<!-- 配置Action -->
	<bean id="userAction" class="top.miaodou.crm.web.action.UserAction">
	
	</bean>
	
	<!-- 配置Servicd -->
	<bean id=userService"" class="top.miaodou.crm.service.impl.UserServiceImpl">
	
	</bean>
	
	<!-- 配置Dao -->
	<bean id=userDao"" class="top.miaodou.crm.dao.impl.UserDaoImpl">
	
	</bean>

###1.5.2用户模块

####1.5.2.1提供一个注册页面

####1.5.2.2创建表

	CREATE TABLE `sys_user` (
	  `user_id` bigint(32) NOT NULL AUTO_INCREMENT COMMENT '用户id',
	  `user_code` varchar(32) NOT NULL COMMENT '用户账号',
	  `user_name` varchar(64) NOT NULL COMMENT '用户名称',
	  `user_password` varchar(32) NOT NULL COMMENT '用户密码',
	  `user_state` char(1) NOT NULL COMMENT '1:正常,0:暂停',
	  PRIMARY KEY (`user_id`)
	) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
	
####1.5.2.3创建实体和映射

	实体：
	public class User {
	
	private Long user_id;
	private String user_code;
	private String user_name;
	private String user_password;
	private String user_state;

	映射：
	<hibernate-mapping>
		<class name="top.miaodou.crm.domain.User" table="sys_user">
			<id name="user_id" column="user_id">
				<generator class="native"/>	
			</id>
			<property name="user_code" column="user_code"/>
			<property name="user_name" column="user_name"/>
			<property name="user_password" column="user_password"/>
			<property name="user_state" column="user_state"/>
		</class>
	</hibernate-mapping>

####1.5.2.4编写Action

	编写regist的方法

		public class UserAction extends ActionSupport implements ModelDriven<User>{
		//模型驱动使用的对象
		private User user = new User();
		@Override
		public User getModel() {
			return user;
		}	
		//注入Service：
		private UserService userService;
	
	
		public void setUserService(UserService userService) {
			this.userService = userService;
		}
		/**
		 * 用户注册的方法
		 * */
		public String regist(){
			uservice.regist(user);	
			return NONE;
		}
	
	在Action中注入Service
			<!-- 配置Action -->
		<bean id="userAction" class="top.miaodou.crm.web.action.UserAction">
			<property name="userService" ref="userService"/>
		</bean>

####1.5.2.5编写Service实现类

	public class UserServiceImpl implements UserService {
	
	//	注入DAO：
	private UserDao userDao;

	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}

	@Override
	//业务层注册用户的方法
	public void regist(User user) {
		//对密码进行加密处理
		user.setUser_password(MD5Utils.md5(user.getUser_password()));
		//把用户的状态改为1正常
		user.setUser_state("1");		
		//调用DAO
		userDao.save(user);
	}

####1.5.2.6编写DAO

	整合hibernate：
	<!-- Spring整合hibernate -->
	<!-- 引入Hibernate的配置信息 -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<!-- 注入连接池 -->
		<property name="dataSource" ref="dataSource"/>
		<!-- 配置Hibernate的相关属性 -->
		<property name="hibernateProperties">
			<props>
				<!-- 配置Hibernate的方言 -->
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>			
				<!-- 可选配置================ -->
				<!-- 打印SQL -->
				<prop key="hibernate.show_sql">true</prop>
				<!-- 格式化SQL -->
				<prop key="hibernate.format_sql">true</prop>
				<!-- 自动创建表 -->
				<prop key="hibernate.hbm2ddl.auto">update</prop>
			</props>
		</property>
		<!-- 引入映射文件 -->
		<property name="mappingResources">
			<list>
				<value>top/miaodou/crm/domain/User.hbm.xml</value>
			</list>
		</property>
	</bean>

	在DAO中继承HibernateDaoSupport类:
	public class UserDaoImpl extends HibernateDaoSupport implements UserDao {

	@Override
	//DAO中保存用户的方法
	public void save(User user) {
		this.getHibernateTemplate().save(user);
	}
	
####1.5.2.7配置Spring的事务管理

	配置事务管理器
		<!-- 配置事务管理器 -->
		<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
			<property name="sessionFactory" ref="sessionFactory"/>
		</bean>

	开启注解事务
		
		<!-- 开启注解事务 -->
		<tx:annotation-driven transaction-manager="transactionManager"/>


	在业务层添加事务注解
		@Transactional
		public class UserServiceImpl implements UserService {


