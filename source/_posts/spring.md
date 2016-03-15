title: Spring
categories:
  - Java
tags:
  - Java
  - Spring
date: 2016-01-27 13:07:38

---
Spring整理

## 一. 基础配置

1. 导入jar包（dist）
2. src目录添加beans.xml, schema的添加从doc的reference， spring-framework中可以找到；
3. 创建User对象类；在beans中添加所有的累进行对象实例；
		
	```
	// 相当于User user = new User(); 
    <bean id="userDao" class="com.xxx.model.User" scope="四个值可选"></bean>
    ```
    **scope默认singleton， prototype为多例。即fac.getBean时得到的对象是否时单例**
        
4. 测试类中使用user对象,通过工厂方法得到的user对象是被Spring管理的
	
	```	
	// 创建Spring工厂
	private BeanFactory fac = new ClassPathXmlApplicationContext("beans.xml");
	public void test(){
		User user = fac.getBean("userDao", User.class);
	}
	```
	**xml没有代码提示，在ide配置中修改xml的catalog，增加配置文件xsd；**

## 二. IoC注入

分层model,dao,service,action,各添加需要的包、类；    
大项目使用xml配置，中小项目用annotation配置；

### 1. 基于xml配置

#### (1).手动注入

1. 配置所有的类到beans.xml

	```
			// 相当于User user = new User();  在User类中有set/get方法
    	<bean id="user" class="com.xxx.model.User" scope="singleton">
    	</bean>
    	
    	<bean id="userDao" class="com.xxx.dao.UserDao"/>
    	
    	<bean id="userService" class="com.xxx.service.UserService">
    		<property name="userDaoS" ref="userDao"></property>
    		
    		// 此处的userDaoS需要在userService中添加对应的setUserDaoS/getUserDaoS方法；
    		// 需要在service类中添加对应的setUserDaoS(IUserDao userDao)/get方法
    		// ref="userDao"中的userDao即为bean.xml中配置文件的id：userDao
    	</bean>
    	
    	<bean id="userAction" class="com.xxx.action.UserAction" scope="prototype">
    	
    		<property name="userService" ref="userService"></property>
    	
    	</bean>
	```

  **对于Action而言，里面的对象的值会发生改变，需要用多例，比如2个thread添加不同的user，user已经发生了变化了**    
  **如果没有属性、状态变化的，使用单里即可**
3. 在各类中通过工厂方法方式获取注入的类实例；


#### (2). 构造方法注入

1. UserAction添加构造方法，有两个参数；
2. 修改配置文件

	```
	    <bean id="userAction" class="com.xxx.action.UserAction" scope="prototype">
    		<constructor-arg ref="userServie1"/>
    		<constructor-arg ref="userServie2"/>
    	</bean>
	```
#### (3). 自动注入

1. beans.xml配置   
		
	```
 	    <bean id="userAction" class="com.xxx.action.UserAction" 
 	    	scope="prototype"    
 	    	autowire="xx???xxx">    	
    	</bean>
    ```
2. autowire="byName",会调用get/set方法；

3. 在配置文件root节点beans节点添加 default-autowire="",则所有配置都将自动注入。

**虽然减少手动配置代码，但无法通过beans文件了解所有文件结构，不建议使用。**

#### (4). 属性注入
1. 修改beans.xml
    	
    ```
		// 相当于User user = new User();  在User类中有set/get方法    
    	<bean id="user" class="com.xxx.model.User" scope="singleton">
    		<property name="id" value="1"/>
    		<property name="userName" value="ccSun"/>
    		<property name="listXX">
    			<list>
    			</list>
    		</property>
    	</bean>
    ```
    	
### 2. 基于Annotation注入（Spring 3.0之后）

1. beans.xml中beans节点添加context的schema；
2. 修改beans.xml   

	```
		打开spring annotation
		<context:annotation-config/>
		设置从哪些包扫描注解
		<context:component-scan base-package="com.xxx">
	```
3. class文件

	```
		// 相当于<bean id="userDao" class="com.xxx.dao.UserDao"/>
		// @Component("userDao") // 公共的创建bean的annotation,比较少用
		@Repository("userDao") // 一般用于DAO的注入
		public class UserDAO implements IUserDao{
		}
		
		// service 中配置，默认通过名称注入
		// @Component("userService")
		@Service("userService") //使用service特有的注入
		public class Uservice implements IUserService{
			@Resource
			public void setUserDao(IUserDao userDao){}
		}
		
		// action 中修改scope
		@Controller("userAction")
		@scope("prototype")
		public class UserAction{}
	```
## 三. AOP代理自实现

### 1. 静态代理

  假设在userDao的操作中需要加入log输出代码，我们可以写一个ProxyDao，在ProxyDao中写于原userDao相同的方法。方法内添加log输出，调用userDao。把ProxyDao注入到Service中。
	
### 2. 动态代理

1. 创建代理类  

	```
		/**
		* 1. 创建一个类实现InvocationHandler接口
		*/
		public class LogProxy implements InvocationHandler{
		
			// 2. 创建一个代理对象
			private Object target;
		
			//3. 创建一个方法生成对象，参数需要被代理的对象，返回代理对象
			public static Object getInstance(Object o){
				// 3.1 创建LogProxy对象
				LogProxy proxy = new LogProxy();
				// 3.2 设置这个被代理对象
				proxy.target = o;
				// 3.3 通过Proxy的方法创建代理对象
				// result是代理，它代理o
				// 第一个参数 被代理对象的classLoader
				// 第二个参数 被代理对象的实现的所有接口
				// 第三个参数 实现InvocationHandler的类
				Object result = Proxy.newProxyInstance(
					o.getClass.getClassLoader(),
					o.getClass.getInstances(),
					proxy);
				return result;
			
			}
	
			// 当有了代理对象后，都会调用invoke方法
			@Override
			public Object invoke(Object proxy, Method method, Object[] args){
		
				// 自己的代码			
				if(method.getName().equals("add")) // 只在add方法添加代码
					Logger.info("hhhhalll");
				// 自己的代码
			
				Object obj = method.invoke(target, args);
				
				// 自己的代码	
				// 自己的代码
				
				return obj;
			}
	
		}
	```
2. beans.xml注入    
		没有get/set,注入属性要通过factory-method注入。
		
	```
		<bean id="userDynamicDao" class="com.xxxx.LogProxy"
			factory-method="getInstance"">
			
			<constructor-arg ref="userDao"/>  // 此处的userDao会找anotation中的Dao
		</bean>
	```
3. Service中注入userDynamicDao

	```
		@Resource(name="userDynamicDao")
		public void setUserDao(IUserDao userDao){
			this.userDao = userDao;
		}
	```
4. 通过Annotation来添加invoke代码

	(1) new Annotation文件 
		
	```
		@Retention(RetentionPolicy.RUNTIME)
		public @interface LogInfo {
			public String value() default"";
		}
	```
	(2) 在需要添加log输出的接口文件的方法上添加
	
	```
		@LogInfo("Add a new user")
		public void addUser(User user);
	```	
	(3) invoke修改
	
	```
		@Override
		public Object invoke(Object proxy, Method method, Object[] args){
	
			// 自己的代码										if(method.isAnnotationPresent(LogInfo.class)){
				LogInfo li = method.getAnnotation(LogInfo.class);
				Logger.info(li.value());
			}
			// 自己的代码
		
			Object obj = method.invoke(target, args);
			
			// 自己的代码	
			// 自己的代码
			
			return obj;
		}
	```
## 4. 基于Annotation实现AOP代理

1. 修改beans的schema；
	root节点beans添加
		
	```
		xmlns:xsi="xxxxx/XMLSchema-instance"
		xmlns:aop="http://www.springframework.org/schema/context"
		xsi:schemaLocation="xxxxxxxxxxx   aop  aop/spring-aop-3.0.xsd"
	```
2. 打开基于annotation的aop代理
	```
		<aop:aspectj-autoproxy/>
	```
3. 创建aop切面
		
	```
		@Component("logAspect") // 注入切面类给spring管理
		@Aspect // 声明这是一个切面类，Spring通过第三方aspectj实现aop切面
		public class LogAspect{
			
			// 在哪些类里执行
			// 第一个＊表示任意返回值；add*表示add开头的方法；..表示任意参数
			@Before("execution(* com.xxxxx.dao.*.add*(..))"
						  + "|| execution(* com.xxxxx.dao.*.update*(..))")
			public void logStart(JoinPoint jp){
				syso("加入日志");
				// 得到执行对象
				syso:	jp.getTarget()	
				// 得到执行的方法
				syso:	jp.getSignature().getname()	// 
			}
			
			@Before @After 开始、结束执行
			@After("execution(xxx)")
			public void logEnd(JoinPoint jp){
			}
			
			@Around("execution(xxxxxx)")
			public void logAround(ProceedingJoinPoint pjp){
				
				// coding 执行程序前执行
			
				pjp.process(); // 执行程序
				
				// coding 执行完程序后执行
			}
		}
	```
	**@Aspect依赖jar包：aopalliance.jar aspectjrt.jar aspectjweaver.jar,要先导入**

## 5. 基于XML实现AOP代理

1. LogAspect修改

	```
			@Component("logAspect") // 注入切面类给spring管理
		public class LogAspect{
			
			public void logStart(JoinPoint jp){
				syso("加入日志");
				// 得到执行对象
				syso:	jp.getTarget()	
				// 得到执行的方法
				syso:	jp.getSignature().getname()	// 
			}
			
			public void logEnd(JoinPoint jp){
			}
			
			public void logAround(ProceedingJoinPoint pjp){
				
				// coding 执行程序前执行
			
				pjp.process(); // 执行程序
				
				// coding 执行完程序后执行
			}
		}
	```
2. 修改beans.xml

	```
		<aop:config>
			// 定义切面
			<aop:aspect id="myLogAspect" ref="logAspect">
				// 通过execution指定在哪些类加入切入点
				<aop:pointcut id="logPointCut" expression="execution(* xxxx)"/>
				<aop:before method="logStart" pointcut-ref="logPointCut"/>
			</aop:apsect>
		</aop:config>	
	```