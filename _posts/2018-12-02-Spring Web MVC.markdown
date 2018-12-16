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
7. ` DispatcherServlet`은 해당 화면에 로직결과물을 전달한다.  
8. data를 랜더링 한 후 response 반환

다음과 같이 Dispatcher Servlet은 각각의 역할에 따라 나눠진 객체들(Controller, Service, DAO, View ... etc)이 무리 없이 본인의 역할을 수행하도록 결과물들을 전달해 주는 역할을 한다.  

> **따라서 개발자는 Business logic을 구현하는 것에만 집중할 수 있다.**

그렇다면 DispatcherServlet을 통한 Web MVC를 직접 구현해보도록 하자.

#### 1. DispatcherServlet 설정
모든 웹애플리케이션이 반드시 가지고 있는 `web.xml`에 DispatcherServlet을 정의해준다.  
`web.xml`에 명시된 설정들은 웹애플리케이션 시작시 메모리에 로딩된다.  

* web.xml  

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
dispatcherServlet에 대한 설정정보들은 xml파일로 분리하여 명시한다.  
sts 플러그인을 통해 spring mvc project 생성 시, /WEB-INF/spring/servlet-context.xml 으로 자동 생성된다.

* servlet-context.xml  

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
* `<annotation-driven />` : 어노테이션 사용가능  
* `<resources>` : 정적자원에 대한 경로 설정  
* `<context:component-scan>` : 해당 패키지 내 경로에 있는 클래스들을 스캔하여  @Component 어노테이션이 있는 클래스들을 자동으로 bean객체로 등록시켜준다.  
* `ViewResolver` : view의 특정 경로를 지정  


### 2.Business logic 구현  
DispatcherServlet은 모든 요청을 직접 처리해주기 때문에 개발자가 서블릿의 생명주기, url 매핑에 대한 처리를 해줄 필요가 없다.
따라서 DispatcherServlet에 대한 설정이 완료되었으므로 이제 Controller, Service, Dao ...에 해당하는 business logic을 구현해보도록 한다.  

#### 2-1. Controller 구현  
Controller는 DispatcherServlet을 통해 처음으로 요청이 들어오는 곳이다. `HomeController.java`를 통해 간단히 살펴보도록 하자.

```
@Controller
public class HomeController {

	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

	/**
	 * Simply selects the home view to render by returning its name.
	 */
	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		logger.info("Welcome home! The client locale is {}.", locale);

		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

		String formattedDate = dateFormat.format(date);

		model.addAttribute("serverTime", formattedDate );

		return "home";
	}
}
```
* @Controller : 컨테이너에게 Controller로 사용될 객체임을 명시
* @RequestMapping : 해당 url에 사용될 메서드임을 명시, GET/POST 방식에 대해 명시
* model.addAttribute : View에 전달할 객체들을 명시
* return "home" : 해당 View의 이름을 명시


![](/assets/posts/controller1.png)
다음과 같이 Request Parameter를 VO 객체로 넘기는게 가능하다. 자세한 내용은 **Spring Command Object**에 대해 살펴보도록 한다.  


이제 실습을 통해 mvc 패턴을 통한 기능을 직접 구현해보도록 하자. 회원가입 양식의 맞게 작성된 데이터를 전달받아 서비스 객체를 통해 회원 등록을 한 뒤 결과 값을 출력해 주는 실습을 할것이다.
```
@Controller
@RequestMapping("/member") // (1)
public class MemberController {

	@Autowired // (2)
	MemberService service;

	@ModelAttribute("serverTime") // (3)
	public String getServerTime(Locale locale) {

		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

		return dateFormat.format(date);
	}

	@RequestMapping(value = "/memJoin", method = RequestMethod.POST)
	public String memJoin(Member member) {

		service.memberRegister(member);

		return "memJoinOk";
	}
}
```
(1) 해당 컨트롤러에 `RequestMapping` 어노테이션을 사용함으로써 /member 하위의 url은 모두 MemberController를 경유함을 명시한다.  
(2) 컨트롤러 안에서 사용할 Service객체를 자동주입한다. 해당 서비스 클래스는 bean객체로 선언이 되어있어야 한다. (xml을 통한 주입이든, 어노테이션을 통한 주입이든)  
(3) 해당 메서드에 `ModelAttribute` 어노테이션을 사용함으로써 리턴 값은 serverTime의 이름으로 Model객체의 속성으로 자동 추가될 것이다.

### 3. View 구현
business logic 구현을 완료하였으니 이제 마지막으로 결과값을 화면에 출력하는 일만 남았다. 앞서 구현한 회원 등록 로직을 통해 등록한 회원의 정보를 화면에 보여주는 것을 실습해보도록 하자.

```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
	<title>Home</title>
</head>
<body>
	<h1> memJoinOk </h1>

	ID : ${member.memId} <br />
	PW : ${member.memPw} <br />
	Mail : ${member.memMail} <br />
	PHONE1 : ${member.memPhones[0].memPhone1} - ${member.memPhones[0].memPhone2} - ${member.memPhones[0].memPhone3} <br />
	PHONE2 : ${member.memPhones[1].memPhone1} - ${member.memPhones[1].memPhone2} - ${member.memPhones[1].memPhone3} <br />
	AGE : ${member.memAge} <br />
	ADULT : ${member.memAdult} <br />
	GENDER : ${member.memGender} <br />
	FAVORITE SPORT :
	<c:forEach var="fSport" items="${member.memFSports}">
		${fSport},
	</c:forEach> <br />

	<P>  The time on the server is ${serverTime}. </P>

	<a href="/lec19/resources/html/memJoin.html"> Go MemberJoin </a>
</body>
</html>

```
* Spring command Object를 랜더링 하기 위해 `jstl` 태그를 사용한다.
* 객체명은 default로 member 이고 변경이 필요할시 `ModelAttribute` 어노테이션을 사용한다.  
* `ModelAttribute` 어노테이션이 붙여진 메서드의 리턴값인 serverTime 변수가 특별한 선언 없이 사용되고 있음을 확인할 수 있다.  
