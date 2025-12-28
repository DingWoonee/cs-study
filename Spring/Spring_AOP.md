# 스프링 AOP
### 📚 목차
- [1 AOP 개념과 용어](#1-aop-개념과-용어)
- [2 스프링 AOP 동작 원리](#2-스프링-aop-동작-원리)
- [3 스프링 AOP 적용 방법](#3-스프링-aop-적용-방법)
- [4 스프링 AOP 한계와 주의점](#4-스프링-aop-한계와-주의점)
- [5 스프링 AOP 실무](#5-스프링-aop-실무)

## 1 AOP 개념과 용어
### 1.1 AOP 란
> AOP(Aspect-Oriented Programming)는 **공통 기능을 비즈니스 로직에서 분리해서, 필요한 지점에 일괄 적용**하는 프로그래밍 패러다임이다.


<img width="691" height="128" alt="image" src="https://github.com/user-attachments/assets/71f4203d-113a-476d-bcc2-d32c4df492a0" />

애플리케이션 로직은 **핵심 기능**과 **부가 기능**으로 나뉜다.  
→ AOP는 여기서 로그 추적 로직과 같이 여러 곳에서 공통적으로 사용하는 **부가 기능**을 분리해서 일괄적으로 적용한다.

- **스프링에서 AOP가 특별히 중요한 이유**  

  스프링의 대표 기능은 거의 AOP 기반이다.  
  (`@Transactional`, `@Async`, `@Cacheable` 등)  
  → 스프링을 정확하게 활용하기 위해서는 AOP에 대한 이해가 필수적이다.

### 1.2 AOP 용어
- **Target**

  AOP 적용되는 원래 객체(**핵심 기능**)이 있는 객체**

- **Advice**

  핵심 기능의 전/후 **언제 샐행될지** + 실제 실행되는 **공통 로직**  

  - **스프링 AOP의 Advice**

    `@Before` → 메서드 실행 전  
    `@AfterReturning` → 메서드 정상 반환 후  
    `@AfterThrowing` → 메서드에서 예외 발생 후  
    `@After` → 메서드 종료 후 (finally)  
    **`@Around`** → 실행 전/후를 감쌀 수 있음 → 가장 강력  
    
    (`@Around`가 사실상 나머지를 다 대체할 수 있다.)

    - **`@Around`외에 다른 Advice가 존재하는 이유**

      이유 1 - Advice의 의도가 명확하게 들어난다.  
      
      이유 2 - `@Around`는 적절한 위치에 `proceed`로 실제 객체를 호출해야 하지만, 다른 Advice는 알아서 해준다.

- **JoinPoint**

  Advice가 끼어들 수 있는 **모든 후보 지점**을 말한다.  
  → **스프링 AOP**는 프록시 패턴을 이용하기 때문에 **메서드 호출에만 적용**될 수 있다.

- **Pointcut**

  "어떤 JoinPoint에 적용할지"에 대한 조건식/필터  
  → **어떤 JoinPoint에 적용될지 결정**  
  ex) `execution(* com.app.service..*(..))`

- **Aspect**

  공통 관심사(Cross-cutting Concern)를 하나의 모듈로 묶은 단위  
  → 간단히 말해서 **Advice(실행 로직) + Pointcut(적용 대상)**  
  → 이름이 AOP(Aspect Oriented Programming)인 이유가 공통 로직을 **Aspect라는 별도 모듈**로 분리하고 적용하는 패러다임이기 때문이다.

- **Weaving**

  Advice를 Target 실행 흐름에 **끼워 넣는 행위**  
  → 스프링 AOP는 보통 런타임에 프록시를 만들어서 Weaving한다.

## 2 스프링 AOP 동작 원리

### 2.1 프록시 기반 AOP
> 스프링 AOP는 **프록시 기반 AOP**이다.

→ 런타임에 프록시를 만들어서 가로챈다.  
→ **프록시를 빈으로 등록**하고, **실제 객체(Target)는 그 내부에서 호출**한다.

### 2.2 동적 프록시 기술
런타임에 프록시를 만들기 위한 기술을 **동적 프록시**라고 하고,  
스프링에는 **JDK 동적 프록시**와 **CGLIB**가 있다.

- **JDK 동적 프록시**

  > 인터페이스 기반
  
  프록시 객체를 만들지 않아도 되지만, **Target 객체와의 공통 인터페이스가 필요**하다.  
  → 프록시 적용을 위해 인터페이스를 만드는 것이 매우 비효율적...

- **CGLIB**

  > 클래스 상속 기반
  
  **바이트코드 조작**으로 인터페이스 없이 **Target 클래스를 상속하는 클래스를 생성**하고,  
  메서드를 오버라이딩해서 공통 로직을 삽입한다.  
  → **상속**을 이용하기 때문에 **`final` 키워드에는 적용할 수 없고, 생성자 가로채기가 안된다**.

### 2.3 프록시 팩토리
과거 스프링은 상황에 따라 JDK 동적 프록시와 CGLIB 중 선택해야 했으나,  
현재 스프링은 **프록시 팩토리**를 통해 **동적 프록시를 편리하게 생성**한다.

<img width="997" height="973" alt="image" src="https://github.com/user-attachments/assets/f80606b1-672f-4fa8-a608-c193e29dd998" />

Target에 **인터페이스가 있으면** → JDK 동적 프록시를 적용하고,  
Target에 **인터페이스가 없으면** → CGLIB를 적용한다.

### 2.4 스프링 AOP 전체 과정
- **어드바이저(`Adviser`)**

    <img width="726" height="385" alt="image" src="https://github.com/user-attachments/assets/97dcbb4b-f02f-41f1-9b7b-f691ed152e60" />

    <img width="986" height="464" alt="image" src="https://github.com/user-attachments/assets/332df2e3-00de-4937-8fcb-c02db8136796" />
    
    **하나의 포인트컷과 하나의 어드바이스**를 어드바이저라고 한다.  
    (조언자는 어디에 어떤 조언을 해야할 지 알고 있음)

1. **컨테이너 시작**

   `ApplicationContext` 생성  
   → `BeanFactory` 준비

2. **Aspect 스캔 & Advisor 생성**

   `@Aspect` 클래스 스캔  
   → `Advisor` 객체 생성

   (빈이나 프록시는 아직 생성되지 않고, Advisor 목록만 준비)

3. **일반 빈 생성**

   스프링이 순수 객체를 만든다.

4. **BeanPostProcessor 개입**

   빈 생성 직후에 컨테이너에 등록되기 직전에 AOP에 적용할지 여부를 판단한다.
   
   > "이 빈에 적용 가능한 Advisor가 하나라도 있나?

   빈 단위로 `Advisor`의 **포인트컷을 통해 적용 여부를 판단**한다.  
   → **하나라도 매칭되면 프록시 대상**이 된다.

5. **ProxyFactory 동작**

   `ProxyFactory`에서 프록시를 생성한다. (동적 프록시 방식 결정)

6. **빈 등록 완료**

   프록시가 적용된 빈의 경우 스프링 컨테이너에 **프록시 객체를 빈으로 등록**한다.

7. **프록시 빈 호출**

   컨테이너에 프록시 빈이 등록되었기 때문에 프록시 빈이 호출되고,  
   내부에서 Target 객체가 호출된다.

## 3 스프링 AOP 적용 방법
- **annotation 클래스 생성**
  ```java
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Retry {
      int value() default 3;
  }
  ```

- **Aspect 클래스 생성**
  ```java
  @Aspect
  public class RetryAspect {
  
      @Around("@annotation(retry)")
      public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {
          log.info("[retry] {} retry={}", joinPoint.getSignature(), retry);
  
          int maxRetry = retry.value();
          Exception exceptionHolder = null;
  
          for (int retryCount = 0; retryCount < maxRetry; retryCount++) {
              try {
                  log.info("[retry] try count={}/{}", retryCount, maxRetry);
                  return joinPoint.proceed();
              } catch (Exception e) {
                  exceptionHolder = e;
              }
          }
          throw exceptionHolder;
      }
  }
  ```

- **적용**
  ```java
  @Retry(4)
  public String save(String itemId) {
      seq++;
      if (seq % 5 == 0) {
          throw new IllegalStateException("예외 발생");
      }
      return "ok";
  }
  ```

## 4 스프링 AOP 한계와 주의점
### 4.1 프록시 내부 호출 문제
스프링 AOP는 프록시 기반으로 동작한다.  
→ 같은 클래스 내의 메서드를 호출할 때는 AOP가 적용이 안된다.

- **대안 1 - 자기 자신 주입**

  자기 자신을 주입해야 하기 때문에 생성자 주입은 안되고, **setter 주입 방식을 활용**할 수 있다.

- **대안 2 - 지연 조회**

  `ApplicationContext`나 `ObjectProvider`를 사용해서 내부에서 호출 시점에 등록된 빈을 조회해서 사용한다.  
  → AOP 메서드인 경우, 프록시가 반환됨

- **대안 3 - 메서드 분리**

  내부에서 호출하는 그 함수를 외부 클래스로 분리한다.

### 4.2 생성자와 `@PostConstruct`는 AOP 적용 불가
AOP가 적용되기 위해서는 빈이 생성되고 프록시가 생성된 후여야 한다.

`@PostConstruct`는 원본 객체가 생성되고, 의존성 주입이 된 후에 호출이 된다.  
→ 당연히 아직 프록시가 감싸기 전임 → AOP 적용 안됨
