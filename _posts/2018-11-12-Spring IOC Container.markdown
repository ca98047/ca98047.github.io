---
title: Spring IOC Container
author: jinow
cover: "/assets/posts/1.jpg"
categories: Spring
layout: post
---
### 1.1 Spring IOC Container와 Spring Beans   
**IOC 컨테이너**는 스프링 프레임워크의 핵심기술이다. 객체들의 생성부터 소멸에 이르기 까지 완벽한 life cycle로 설계하고 관리한다. 컨테이너는 **DI(Dependency Injection)** 기술을 사용하여 application을 구성하는 component들을 관리한다.   
컨테이너는 객체들을 생성하는데 이러한 객체들을 **Spring Beans** 라고 한다.
Spring Beans는 특별한 것이 아니라 컨테이너 안에서 생성되는 모든 객체들을 의미한다.  

### 1.2 Spring IOC Container
```org.springframework.context.ApplicationContext```는 스프링 컨테이너를 대표하는 interface로 ```BeanFactory```를 확장한 것이다. Configuration Metadata를 읽어들여 bean 객체들을 생성·설정·조립하는 역할을 한다.  
아래의 Diagram은 spring 공식문서에 포함된 이미지로 Spring 컨테이너가 동작하는 과정을 보여준다.
![https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#spring-core](/assets/posts/ioc container2.png)

### 1.2.1 Configuration MetaData
앞선 다이어그램에서 보듯이 IOC 컨테이너는 메타데이터를 읽어들여서 application을 설정한다. 이 메타데이터는 컨테이너에게 bean 객체들을 어떤 기능으로서 사용할지 전달해주는 역할을 한다.
메타데이터는 XML, Java Annotation 혹은 Java Code를 통해 표현된다.  

#### XML 사용예시
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```  
```id``` : bean 객체의 식별자  
```class``` : 사용하는 클래스의 full 경로를 포함한 이름  

> Spring 3.0 도입 이후 XML을 기반으로한 메타데이터 설정 방식에서 JavaConfig를 사용한 [Java-based configuration](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-java) 방식으로 옮겨가고 있는 추세이다.   


지금까지 Spring Container와 Bean에 대해 간단히 살펴보았고, 요약하자면 다음과 같다.
1. Spring Container는 Bean객체를 생성하고 여러 객체들을 관리한다. 관리하는 주요기술은 DI라고 볼 수 있다.  
2. Bean객체는 XML, Annotation, Java Code를 통하여 Spring Container에 생성되고 관리된다.  

다음 시간에는 Spring Container에서 사용하는 DI의 개념에 대해 살펴볼 것이다.
