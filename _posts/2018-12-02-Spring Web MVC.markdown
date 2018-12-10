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
