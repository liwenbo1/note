- 前言
	接口想通过servlet进行处理，在servlet中注入了spring管理的bean，导致的空指针异常

- 分析
	web项目启动，web.xml启动加载顺序为context-param=>listener=>filter=>servlet=>spring
	配置项的顺序并不会改变加载顺序；但同类型的配置项回改变；servlet可以通过load-on-startup来指定顺序

- 解决
	1. 通过重写HttpServlet的init()方法
	```
	@Override
    public void init() throws ServletException {
		super.init();
		WebApplicationContext wtx =  WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		wtx.getAutowireCapableBeanFactory().autowireBean(this);
	}
	```

	2. 利用spring的显示配置装配机制，在方法被调用时再从spring容器中读取bean
		1. 通过xml文件将配置加载到IOC容器中
		```
		<?xml version="1.0" encoding="UTF-8"?>
			<beans xmlns="http://www.springframework.org/schema/beans"
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd">
				<!--若没写id，则默认为com.test.Man#0,#0为一个计数形式-->
				<bean id="man" class="com.test.Man"></bean>
			</beans>
		```
		```
		public class Test {
			public static void main(String[] args) {
				// 加载项目中的spring配置文件到容器
			ApplicationContext context = new ClassPathXmlApplicationContext("resouces/applicationContext.xml");
				// 从容器中获取对象实例
				Man man = context.getBean(Man.class);
				man.driveCar();
			}
		}
		```
		2. 通过java注解方式将配置加载到IOC容器
		```
		//同xml一样描述bean以及bean之间的依赖关系
		@Configuration
		public class ManConfig {
			@Bean
			public Man man() {
				return new Man(car());
			}
		}
		```
		```
		public class Test {
			public static void main(String[] args) {
				//从java注解的配置中加载配置到容器
				ApplicationContext context = new AnnotationConfigApplicationContext(ManConfig.class);
				//从容器中获取对象实例
				Man man = context.getBean(Man.class);
				man.driveCar();
			}
		}
		```

#### ApplicationContext
- Application是spring的BeanFactory的实现类
