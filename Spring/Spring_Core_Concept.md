# 스프링 핵심 원리(Spring Core Concepts)
### 📚 목차
- [1 스프링 프레임워크 개요](#1-스프링-프레임워크-개요)
- [2 IoC & DI (스프링 객체 관리 핵심)](#2-ioc--di-스프링-객체-관리-핵심)
- [3 AOP(Aspect-Oriented Programming)](#3-aopaspect-oriented-programming)
- [4 스프링 MVC 아키텍처](#4-스프링-mvc-아키텍처)
- [5 스프링 요청 처리 전체 흐름](#5-스프링-요청-처리-전체-흐름)
- [6 스프링 빈 생명주기 & 확장 포인트](#6-스프링-빈-생명주기--확장-포인트)
- [7 스프링 부트 자동 구성 원리](#7-스프링-부트-자동-구성-원리)

## 1 스프링 프레임워크 개요
### 1.1 스프링이란 무엇인가?
> 자바 기반 엔터프라이즈 애플리케이션(웹 서비스, 백엔드 서비스)을 만들기 위한 프레임워크 집합

- **POJO 기반**  
  평범한 자바 객체(Plain Old Java Object, POJO) 중심으로 개발할 수 있게 해준다.  
  → 순수 비즈니스 로직으로 테스트 및 유지보수가 쉬워짐
- **IoC/DI 중심**  
  프레임워크(컨테이너)가 객체 생성 및 연결을 대신해준다.  
  → 개발자는 비즈니스 로직에 집중
- **인프라 추상화**  
  트랜잭션, 보안, 데이터 접근, 메시징 등의 기능들을 공통된 방식으로 묶어 제공한다.  
  → 특정 기술에 의존하지 않고, 스프링을 통해 추상 계층으로 다룸

### 1.2 스프링의 등장 배경
- **EJB의 문제점**
  - 너무 무겁고 복잡함
  - 비즈니스 로직보다 기술 코드가 많음
  - 테스트가 거의 불가능함
  - 개발 비용이 너무 높음

- **스프링의 목표**
  - POJO 기반 개발 → 상속/인터페이스 제약 없이 순수 자바 코드로 개발
  - IoC/DI 제공 → 객체를 직접 생성/관리하지 않아도 됨
  - AOP 기반 선언적 트랜잭션 → 공통 기능을 AOP로 별도로 분리
  - 가벼운 컨테이너 → JVM만 있어도 스프링 실행 가능

### 1.3 스프링과 스프링 부트 관계
> Spring은 핵심 기술이고,  
> Spring Boot는 Spring을 쉽고 빠르게 사용할 수 있게 만든 시작 도구이다.

- **스프링 프레임워크**  
  IoC 컨테이너, AOP, MVC 등 기능 그 자체

- **스프링 부트**  
  - starter 의존성 제공
  - 자동 구성(AutoConfiguration)
  - 내장 서버(Tomcat, Jetty, Netty)
  - `application.yml` 기반 설정
  - 배포 단순화(Jar 하나로 동작)

## 2 IoC & DI (스프링 객체 관리 핵심)
### 2.1 IoC 개념
> IoC(Inversion of Control)는 객체의 생성과 제어 권한을 개발자가 아닌 외부 컨테이너가 갖는 것을 말한다.  
> 원래 개발자에게 있던 권한이 프레임워크로 옮겨졌기 때문에 **제어의 역전(IoC)**라고 한다.

→ 객체를 직접 생성하지 않음  
→ 스프링 컨테이너가 대신 객체를 생성하고, 연결한다.

- **IoC가 필요한 이유**  
  1. 객체 생성/초기화/연결 로직이 여기저기 흩어진다. → 유지보수가 어려움
  2. 구현체를 바꾸려면 코드 수정이 필요함 → 결합도가 높아짐
  3. 테스트를 하려면 가짜 객체를 넣기 어려움 → 테스트 비용 증가
  4. 애플리케이션 규모가 커질수록 객체 조립 구성이 복잡해짐
  
### 2.2 DI 개념
> DI는 **필요한 객체(의존성)를 외부에서 넣어주는 기법**이다.  
> IoC가 개념이라면, DI는 그것을 실현하는 구체적인 방식이다.

- **DI가 강력한 이유**
  - 구현체를 외부에서 선택해서 변경에 유연하다.
  - 테스트 시 Mock 객체 주입으로 테스트가 쉽다.

- **주입 방식**
  - **생성자 주입**  
    생성자를 통해 **의존성을 객체 생성 시점에 넣는 방식**이다.  
    
    → `final`을 통해 불변을 보장할 수 있다.  
    → 객체 생성 시점에 필요하기 때문에 누락이 방지된다.  
    → 순환 참조를 빠르게 감지할 수 있다.
  
  - **Setter 주입**  
    객체 생성 후 Setter로 주입하는 방식이다.  
    Setter 함수에 `@Autowired`를 붙여서 사용할 수 있다.  

    의존성이 필수는 아니고 선택적인 경우나 런타임 중에 변경해야 하는 경우 사용할 수 있다.
    
  - **필드 주입**  
    필드에 `@Autowired`를 붙여서 사용하는 방식으로,  
    자바의 리플렉션(reflection)을 사용해서 직접 객체를 꽂아 넣는다.  
    (리플렉션으로 `@Autowired`가 붙은 필드를 찾는다.)

- **생성자 주입이 권장되는 이유**  
  - 의존성이 빠짐없이 주입된 완전한 객체만 생성될 수 있다. → 불변성 보장
  - 생성자 주입 단계에서 순환 참조 감지
  - 테스트가 쉬움 (필드 주입의 경우 private 필드를 직접 넣을 수 없고, Setter 주입은 Setter 메서드가 열려있어야 한다.)
  - DI 컨테이너 없이도 객체 생성 가능 → 테스트가 편해짐
  - `final`을 통해 불변성 보장 가능

### 2.3 IoC 컨테이너 구조
> IoC 컨테이너는 객체를 직접 `new`로 생성하지 않고,  
> 스프링이 대신 생성하고 초기화하고 의존성을 주입해주고 소멸까지 관리하는 객체이다.

- **`BeanFactory`**  
  스프링 IoC 컨테이너의 최상위 인터페이스  
  빈 생성 및 조회, 의존성 주입 같은 **기본 기능**을 제공  
  → 스프링 컨테이너의 최소 기능
  
- **`ApplicationContext`**  
  `BeanFactory`를 확장한 컨테이너  
  (실무에서는 거의 이걸 사용한다.)  
  국제화, 이벤트 발행, AOP, 환경 변수 관리 등의 기능 제공

### 2.4 스프링 DI 구현 방식
- **컴포넌트 스캔(`@ComponentScan` + `@Component` 등)**  
  `@Component`, `@Service`, `@Repository`, `@Controller` 등이 붙은 클래스를 자동으로 검색해서 빈으로 등록한다.

- **자바 설정(`@Configuration` + `@Bean`)**  
  개발자가 직접 빈을 등록하는 방식으로, Configuration 클래스에서 등록한다.

- **생성자 DI + `@Autowired`**  
  스프링은 생성자를 통해 빈을 주입한다.  
  스프링 4.3부터 생성자가 1개면 `@Autowired`가 생략 가능하다.  
  (보통 `@RequiredArgsConstructor`를 많이 사용함)

### 2.5 설계 원칙(DIP, OCP)과 DI
- **DIP(의존성 역전 원칙)**  
  > 고수준 모듈은 저수준 모듈에 의존하면 안된다.  
  > 둘 다 추상화(인터페이스)에 의존해야 한다.

  인터페이스를 의존하고 구현체를 빈으로 등록하면 이 원칙을 지킬 수 있다.  
  → 클래스 내부에서는 인터페이스만 의존하고 `new` 등의 작업을 할 필요가 없다.

- **OCP(개방-폐쇄 원칙)**  
  > 확장에는 열려 있고, 변경에는 닫혀있어야 한다.

  빈으로 등록하는 구현체만 바꾸면 되기 때문에  
  인터페이스를 의존하는 클래스의 코드는 수정할 필요 없고, 구현 코드만 수정하면 된다.

## 3 AOP(Aspect-Oriented Programming)

## 4 스프링 MVC 아키텍처
### 4.1 MVC 패턴과 스프링 MVC
- **MVC 패턴**  
  데이터(Model), 화면(View), 요청 로직(Controller)을 명확히 분리해서  
  **유지보수성과 확장성을 높이**는 아키텍처 패턴이다.
  
- **스프링 MVC**  
  Model → 화면에 보여줄 데이터, 비즈니스 로직의 결과  
  View → 사용자에게 보이는 화면 (Thymeleaf, JSON 응답 등)  
  Controller → 요청을 받아서 비즈니스 로직을 호출하고, 결과(Model)을 만들어서 View에게 전달

- **스프링 MVC의 목표**  
  개발자는 **비즈니스 로직에 집중**하고,  
  웹 관련 처리(요청 파싱, 인코딩, 예외 처리, 뷰 선택, 리다이렉트 등)은 스프링 MVC의 인프라가 대신 처리하게 한다.

### 4.2 스프링 MVC의 핵심 구조
<img width="772" height="353" alt="image" src="https://github.com/user-attachments/assets/e351973f-8b01-4997-91b6-5e67b01f5cd3" />

- **DispatcherServlet(프론트 컨트롤러)**  
   모든 HTTP 요청이 먼저 들어오는 입구.  
   서블릿 클래스(`HttpServlet`)의 구현체.
  
  - **하는 일**  
    1. 요청 URL, HTTP Method 등을 보고 어느 컨트롤러에 보낼지 조회(`HandlerMapping`)
    2. 해당 컨트롤러를 실행할 수 있는 어댑터 찾기(`HandlerAdapter`)
    3. 컨트롤러 호출 → `ModelAndView` 또는 다른 형태의 응답 결과를 받음
    4. `ViewResolver`로 실제 View를 찾고 렌더링해서 응답

- **Handler (Controller)**  
  `@Controller`, `@RestController` 클래스의 메서드들  
  → 핵심 비즈니스 흐름만 신경 쓰면, 요청 파싱/응답 변환은 스프링이 해준다.

- **HandlerMapping**  
  URL, HTTP Method에 해당하는 컨트롤러 메서드를 찾아준다.  
  (스프링 부트에서는 `@RequestMapping`, `@GetMapping` 등을 분석해서 내부적으로 매핑 정보를 전부 만들어 놓는다.)
  
  ex) `/restaurants/1` 요청에 대해 `RestaurantController#getRestaurant()` 메서드를 찾아줌

- **HanderAdapter**  
  `HandlerMapping`이 메서드 핸들러를 찾아줬다면,  
  `HandlerAdapter`는 해당 핸들러를 **실제로 실행하는 방법을 알고 있는 객체**이다.  
  → 해당 메서드를 실제로 호출하고, 결과를 `ModelAndView` 형태로 만들어준다.  

  (스프링은 다양한 형태의 핸들러(컨트롤러)를 지원하기 때문에,  
  호출 방법을 알고 있는 객체(HandlerAdapter)가 필요하다.)

- **ViewResolver**  
  컨트롤러에서 반환한 View 이름으로 **실제 View 객체**를 찾는다.  

  ex) `"restaurant/detail"`을 반환 → `src/main/resources/templates/restaurant/detail.html` 실제 파일을 찾아서 **View 객체**를 만든다.

  Thymeleaf를 사용하면 `ThymeleafViewResolver`이고, JSP면 `InternalResourceViewResolver`가 구현체가 된다.  
  `@RestController`나 `@ResponseBody`를 사용한 경우는 ViewResolver를 거치지 않고, 바로 `HttpMessageConverter`를 통해 JSON 같은 바디로 변환된다.

- **View**  
  최종적으로 **렌더링을 수행**하는 객체이다.  
  **실제 응답 바디**를 만든다.  

  SSR(Thymeleaf)라면 템플릿에 `model`을 넣고 HTML을 렌더링한다.  
  View 객체에 대해 **`render` 함수를 호출**한다.
  
  REST라면 JSON으로 직렬화한다.  
  View 객체가 생성되지 않고, `HttpMessageConverter`로 직렬화된 바디를 생성한다.

## 5 스프링 요청 처리 전체 흐름
클라이언트에서 리버스 프록시를 거쳐, 톰캣 및 스프링까지의 흐름

### 5.1 클라이언트 → Nginx (리버스 프록시)
1. DNS 조회
2. TCP 연결
3. TLS 핸드셰이크 (HTTPS의 경우)
4. HTTP 요청 전송

### 5.2 Nginx (리버스 프록시)
1. 요청 수신 & 서버 블록 매칭
2. 리버스 프록시로 업스트림에 전달 (톰캣과 TCP 연결 생성, 또는 기존 keep-alive 재사용)
3. 버퍼링 / 타임아웃 / 로드밸런싱

### 5.3 Nginx → Tomcat (스프링 부트 내장 톰캣)
1. **톰캣 커넥터가 요청 수신**
   
   스프링 부트 기준으로 톰캣 요청 수신 포트 번호는 `server.port=8080`으로 설정되어 있다.  
   → **8080 포트로 오는 요청**은 OS 커널이 **Tomcat 프로세스로 전달**해준다.  
   
   이때 실제로 요청을 받는 주체는 **Tomcat의 Connector**이고,  
   `Connector → ProtocalHandler → Executor` 구조로 되어있다.  

   - `Connector`: 8080 포트와 연결된 HTTP 커넥터
   - `ProtocolHandler`: HTTP/1.1, NIO 같은 프로토콜 처리 담당
   - `Executor`: ThreadPoolExecutor 기반의 스레드 풀

   → 요청이 들어오면 Executor의 워커 스레드 중 하나를 빌려와서 해당 요청을 처리하게 된다.
   
2. **HTTP 요청 파싱**  
   
   톰캣이 HTTP 헤더, 메서드, URI, 바디 등을 파싱하고,  
   `HttpServletRequest`, `HttpServletResponse` 객체를 생성한다.  

   → 서블릿 호출 전에 파싱이 완료된다.

   - **서블릿(Servlet)**
      
     Servlet은 HTTP 요청을 받아 로직을 수행하고 HTTP 응답을 만드는 **자바 기반 서버 측 컴포넌트(스펙)** 이다.  
     → Servlet은 규약(인터페이스, `HttpServlet`)이고, Tomcat같은 WAS는 이를 구현한 것이다.  
     → Tomcat은 Servlet을 지원하는 Servlet Container.
   
3. **서블릿 컨테이너로 진입**
   
   톰캣은 **서블릿 스펙**에 따라 요청을 해당하는 서블릿으로 전달한다.  

   스프링 부트의 경우는 `DispatcherServlet`을 대표 서블릿으로 `/`에 매핑해둬서 모든 요청이 `DispatcherServlet`의 `service()` 메서드로 들어간다.

4. **FilterChain 호출**  

   톰캣은 실제로는 DispatcherServlet의 `service()`를 직접 호출하지 않는다.  
   FilterChain을 호출하고, 그 안에서 `service()`를 호출한다.  

   → 마지막 필터에서 `service()`를 호출한다.  
   → 필터가 등록되어 있다면, `service()` 전후에 필터 체인 로직들이 호출된다.

5. **DispatcherServlet → 스프링**  

   `HandlerMapping`으로 핸들러 찾기  
   → 핸들러 타입에 맞는 `HandlerAdapter` 찾기  
   → 핸들러 호출하기  
   → `ViewResolver` 또는 `HttpMessageConverter`로 응답 생성  
   → 응답(HTML, JSON 등)을 `HttpServletResponse`의 body에 기록

## 6 스프링 빈 생명주기 & 확장 포인트
### 6.1 Bean LifeCycle
1. **BeanDefinition 로딩 단계**  
   > 스프링이 설정 정보를 읽어서 "빈 설계도"를 만든다.  

   - `@Configuration` + `@Bean`
   - `@ComponentScan`으로 찾은 `@Component`, `@Service`, `@Repository`, `@Controller` 등
   - XML, JavaConfig 등  

   ⇒ "실제 객체"를 만들지는 않고, 생성을 위한 "메타데이터"만 갖고 있는 단계이다.

2. **인스턴스 생성 및 의존성 주입 단계**  
   >설계도(BeanDefinition)를 보고 실제 객체를 생성하는 단계이다.  

   - 생성자 주입 → 객체를 생성할 때 의존성이 필요함
   - 필드 주입 / 세터 주입 → 객체를 생성하고 이후에 의존성을 주입해도 됨  

   ⇒ 주입 방식에 따라 객체 생성 및 의존성 주입이 진행된다.  
   ⇒ 또한 `@Value` 같은 값도 주입된다.

3. **(`Aware` 인터페이스를 구현한 경우) Aware 단계**  
   > "스프링이 빈에게 스프링 내부 인프라 객체들을 알려주는 단계"이다.  

   `BeanNameAware`  
   → 이 빈이 컨테이너에 등록된 이름을 알 수 있음 (로깅, 디버깅 등에 사용)  
   
   `BeanFactoryAware`  
   → `BeanFactory` 자체를 받아서 특정 빈을 지연 조회할 수 있음  
   
   `ApplicationContextAware`  
   → `BeanFactory`보다 많은 기능을 제공하는 `ApplicationContext` 전체에 접근  

   `EnvironmentAware`  
   → 활성 프로필이나 시스템 환경 변수, `application.yml` 설정 값 등에 접근  

   이 외에 여러 Aware 인터페이스가 존재한다.

4. **초기화 콜백(Initialization) 단계**  
   > 빈에 모든 필요한 정보가 다 제공된 이후의 단계로 초기화 함수들을 호출해준다.  

   `@PostConstruct`, `InitializingBean` 인터페이스, `@Bean(initMethod = "init")`의 초기화 콜백 방식이 있다.    

   초기화 콜백은 해당 빈 하나를 초기화하는 시점에서 필요한 작업을 수행할 때 사용한다.  
   ex) 초기 계산 로직, 초기 값 세팅, 생성자에 넣기에는 적합하지 않은 무거운 초기화 작업

   - **주의점**  
    초기화 콜백은 AOP가 적용되기 전에 수행되고, 이로 인해 **`@Transactional`, `@Async`, `@Cacheable` 같은 것이 적용되지 않는다**.  
    (모든 빈이 생성되고 의존성이 주입되었기 때문에 DB 접근은 가능하다.)

5. **사용 단계**  
   > 애플리케이션이 동작하는 동안 빈을 사용한다.  
   
   스코프에 따라 생명주기가 달라진다.  
   - **singleton scope**  
     → 컨테이너 생성 시 만들고, 종료 시까지 유지된다.
   - **prototype scope**  
     → 요청할 때마다 새로 만들고, 컨테이너는 소멸 관리를 하지 않는다. (그래서 `@PreDestroy`는 실행되지 않는다.)
   - **request/session scope**  
     → 웹 요청/세션의 라이프사이클과 함께 생성되고 소멸된다.

6. **소멸(Destroy) 콜백 단계**  
   > 컨테이너가 종료될 때, 빈이 정리 작업을 할 수 있는 구간이다.  

   1. `@PreDestroy` 호출
   2. `DisposableBean` 인터페이스의 `destroy()` 호출
   3. `@Bean(destroyMethod = "close")` 호출

   (prototype scope인 빈은 이것들이 자동 호출되지 않는다.)

### 6.2 BeanPostProcessor / FactoryPostProcessor
### 6.3 커스텀 확장 포인트

## 7 스프링 부트 자동 구성 원리
### 7.1 AutoConfiguration 동작 원리
### 7.2 조건부 빈 등록(Conditional 계열)
### 7.3 SpringFactoriesLoader / AutoConfiguration.imports
### 7.4 내장 서버 구조
