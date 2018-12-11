---
title: Spring Web MVC
author: jinow
cover: "/assets/posts/1.jpg"
categories: Spring
layout: post
date: '2018-12-02 23:18:25'
---

---
Spring에서 Web MVC 를 구현하기 전에 Dispatcher Servlet의 동작방식에 대해 알아보자.

![dispatcher servlet architecture](/assets/posts/dps.png)

1. 요청을 받는다.  
2. `Handler Mapping`을 통해 적합한 컨트롤러를 찾는다. 없을 시 자원을 호출  
3. ` DispatcherServlet`은 적합한 컨트롤러 정보를 전달한다.  
4. `Handler Adapter`를 통해 적합한 메서드를 호출하여 로직을 수행한다.  
5. business 로직을 수행 후 결과물을 model 객체에 저장 후 결과물을 보여줄 화면이름을 반환한다.  
6. `ViewResolver`를 통해 해당 화면의 경로를 찾는다.  
7.` DispatcherServlet`은 해당 화면에 로직결과물을 전달한다.  
8. data를 랜더링 한 후 response 반환

다음과 같이 Dispatcher Servlet은 각각의 역할에 따라 나눠진 객체들(Controller, Service, DAO, View ... etc)이 무리 없이 본인의 역할을 수행하도록 결과물들을 전달해 주는 역할을 한다.  

> 따라서 개발자는 Business logic을 구현하는 것에만 집중할 수 있다.

그렇다면 DispatcherServlet을 통한 Web MVC를 직접 구현해보도록 하자.

#### 1. DispatcherServlet 설정
모든 웹애플리케이션이 반드시 가지고 있는 `web.xml`에 DispatcherServlet을 정의해준다. `web.xml`에 명시된 설정들은 웹애플리케이션 시작시 메모리에 로딩된다.  
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>

	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<filter>
	    <filter-name>CharacterEncodingFilter</filter-name>
	    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	    <init-param>
	      <param-name>encoding</param-name>
	      <param-value>UTF-8</param-value>
	    </init-param>
	    <init-param>
	      <param-name>forceEncoding</param-name>
	      <param-value>true</param-value>
	    </init-param>
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

</web-app>
```
dispatcherServlet에 대한 설정정보들은 xml파일로 분리하여 명시한다. sts 플러그인을 통해 spring mvc project 생성 시, /WEB-INF/spring/servlet-context.xml 으로 자동 생성된다.
```
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->

	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>

	<context:component-scan base-package="com.jinow" />
</beans:beans>
```
`<annotation-driven />` : 어노테이션 사용가능  
`<resources>` : 정적자원에 대한 경로 설정  
`<context:component-scan>` : 해당 패키지 내 경로에 있는 클래스들을 스캔하여 @Component 어노테이션이 있는 클래스들을 자동으로 bean객체로 등록시켜준다.  
`ViewResolver` : 컨트롤러가 찾으려는 view의 full 경로를 찾아준다.  
