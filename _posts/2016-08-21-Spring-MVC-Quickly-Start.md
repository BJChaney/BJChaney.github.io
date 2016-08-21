---
layout: post
title:  "Spring MVC Quickly Start"
date:   2016-08-21 23:19:10 +0800
categories: jekyll update
---

### Spring MVC Quickly Start
 
1. #### 配置pom文件依赖库版本，及spring依赖管理 （Maven自动来管理Spring版本）
	
		<properties>
	        <spring.version>4.1.3.RELEASE</spring.version>
	        <commons.lang.version>2.6</commons.lang.version>
	        <slf4j.version>1.7.6</slf4j.version>
	    </properties>
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework</groupId>
	                <artifactId>spring-framework-bom</artifactId>
	                <version>${spring.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	     </dependencyManagement>
	     
	     
	* **bom : 材料清单**
		
		**Spring为了解决jar包冲突，推出spring-framework-bom、spring-boot-dependencies、platform-bom用来统一管理Spring版本。**
 
2. #### 导入 Spring MVC 及其依赖jar包
		
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
	     </dependency>
	     <dependency>
	           <groupId>commons-lang</groupId>
	           <artifactId>commons-lang</artifactId>
	           <version>${commons.lang.version}</version>
	     </dependency>
	     
3. #### 配置web.xml 加入Spring核心类 DispacherServlet

		<?xml version="1.0" encoding="UTF-8"?>
		<web-app version="2.4" 
		    xmlns="http://java.sun.com/xml/ns/j2ee" 
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
		        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
		    
		    <servlet>
		        <servlet-name>mvc-dispatcher</servlet-name>
		        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		        <!-- DispatcherServlet 对应的上下文配置,默认为WEB-INF/${servlet-name}-servlet.xml -->
		        <init-param>
		            <param-name>contextConfigLocation</param-name>
		            <param-value>/WEB-INF/configs/spring/mvc-dispatcher.xml</param-value>
		        </init-param>
		        <load-on-startup>1</load-on-startup>
		    </servlet>
		
		    <servlet-mapping>
		        <servlet-name>mvc-dispatcher</servlet-name>
		        <!--默认拦截所有请求-->
		        <url-pattern>/</url-pattern>
		    </servlet-mapping>
		</web-app>
	    
	* **init-param**
		
		可以不去配置，省略后默为**/WEB-INF/${servlet-name}-servlet.xml**
		
		配置时配置**contextConfigLocation**参数 value = url
		
	* **web-app**
	
		 2.3及之前版本默认关闭el表达式
		 
		 **2.4版本默认开启el表达式**
	    
4. #### 添加Spring MVC 配置文件

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xmlns:p="http://www.springframework.org/schema/p"
		    xmlns:context="http://www.springframework.org/schema/context"
		    xmlns:mvc="http://www.springframework.org/schema/mvc"
		    xsi:schemaLocation="
		        http://www.springframework.org/schema/beans
		        http://www.springframework.org/schema/beans/spring-beans.xsd
		        http://www.springframework.org/schema/context
		        http://www.springframework.org/schema/context/spring-context.xsd
		        http://www.springframework.org/schema/mvc
		        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
		
		    <!--激活注解配置-->
		    <context:annotation-config/>
		    
		    <!--配置只关注@Contraller标注的Bean-->
		    <context:component-scan base-package="com.bjchaney.springmvc">
		        <context:include-filter type="annotation" 
		        	expression="org.springframework.stereotype.Controller"/>
		    </context:component-scan>
		    
		    <!--HandlerMapping 不需要配置 SpringMVC 默认启用DefaultAnnotationHandlerMapping-->
		
		    <!--扩充了注解驱动-->
		    <mvc:annotation-driven/>
		    
		    <!--引入静态资源-->
		    <mvc:resources mapping="/resources/*" location="/resources/"/>
		        
		    <!--配置ViewResolver
		        可以配置多个ViewResolver,
		        使用order属性排序,
		        InternalResourceViewResolver 放在最后
		        -->
		    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		        <!--配置使用支持jstl的view-->
		        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
		        <property name="prefix" value="/WEB-INF/jsp/"/>
		        <property name="suffix" value=".jsp"/>
		    </bean>
		</beans>
		
	* **Spring-MVC 所有的配置完成**
	* 之后加入Spring来管理Service层和dao层，实现**ApplicationContext层次化**
		
		
5. #### 在 web.xml中加入Spring应用的上下文

		<!--Spring 应用上下文 层次化 applicationContext-->
	    <context-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>/WEB-INF/configs/spring/applicationContext*.xml</param-value>
	    </context-param>
	    
	    <listener>
	        <listener-class>
	            org.springframework.web.context.ContextLoaderListener
	        </listener-class>
	    </listener>

6. #### 添加Spring容器的配置文件
	
	
		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
		    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xmlns:p="http://www.springframework.org/schema/p"
		    xmlns:context="http://www.springframework.org/schema/context"
		    xmlns:mvc="http://www.springframework.org/schema/mvc"
		    xsi:schemaLocation="
		        http://www.springframework.org/schema/beans
		        http://www.springframework.org/schema/beans/spring-beans.xsd
		        http://www.springframework.org/schema/context
		        http://www.springframework.org/schema/context/spring-context.xsd
		        http://www.springframework.org/schema/mvc
		        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
		
		    <!--激活注解配置-->
		    <context:annotation-config/>
		    
		    <!--配置关注除@Contraller标注之外的Bean-->
		    <context:component-scan base-package="com.bjchaney.springmvc">
		        <context:exclude-filter type="annotation" 
		        	expression="org.springframework.stereotype.Controller"/>
		    </context:component-scan>
		
		</beans>


7. #### 添加常用 ViewResolver

		
		 <!--文件上传viewResolver-->
	    <bean id="multipartResolver" 
	    	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	        <property name="maxUploadSize" value="2000000"/>
	        <property name="defaultEncoding" value="UTF-8"/>
	        <property name="resolveLazily" value="true"/>
	    </bean>
    
	    <!--  ContentNegotiatingViewResolver
	        提供相同内容不同表现形式的viewResolver
	        此处用来返回Json数据
	    -->
	    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
	        <property name="order" value="1"/>
	        <property name="mediaTypes">
	            <map>
	                <entry key="json" value="application/json"/>
	                <entry key="xml" value="application/xml"/>
	                <entry key="html" value="text/html"/>
	            </map>
	        </property>
	        <property name="defaultViews">
	            <list>
	                <bean 
	                class="org.springframework.web.servlet.view.json.MappingJackson2JsonView">
	                </bean>
	            </list>
	        </property>
	        <property name="ignoreAcceptHeader" value="true"/>
	    </bean>


8. #### 编写Controller进行 相应操作Example

		@Controller
		@RequestMapping("/hello")
		public class HelloController {
		    
		    @Autowired
		    UserService userService;
		    
		    @RequestMapping(value = "/impl1",method = RequestMethod.GET)
		    public String hello(String name,Model model){
		        model.addAttribute("name", userService.getFotmatName(name));
		        return "hello";
		    }
		    
		    @RequestMapping(value = "/impl2",method = RequestMethod.GET)
		    public String hello(String name,Map<String,String> model){
		        model.put("name", userService.getFotmatName(name));
		        return "hello";
		    }
		    
		    @RequestMapping(value = "/impl3",method = RequestMethod.GET)
		    public String hello(HttpServletRequest request){
		        String name = request.getParameter("name");
		        request.setAttribute("name", userService.getFotmatName(name));
		        return "hello";
		    }
		
		    @RequestMapping(value = "/json",method = RequestMethod.GET)
		    @ResponseBody
		    public RestResponse<String> hello(@ModelAttribute("name") String name){
		        RestResponse restResponse = new RestResponse();
		        return restResponse.success(name);
		    }
		    
		    @RequestMapping(value = "/upload",method = RequestMethod.GET)
		    public String hello(){
		        return "upload";
		    }
		    @RequestMapping(value = "/doUpload",method = RequestMethod.POST)
		    public String doUpload(MultipartFile file) throws Exception{
		        if(!file.isEmpty()){
		            userService.CopyFile(file.getInputStream(), "~/Documents/1.jpg");
		        }
		        return "success";
		    }
		}














