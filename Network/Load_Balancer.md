# Load Balancer
### 📚 목차
- [1 Load Balancer 개요](#1-load-balancer-개요)
- [2 Load Balancer의 기본 동작 원리](#2-load-balancer의-기본-동작-원리)
- [3 Load Balancer 분류](#3-load-balancer-분류)
- [4 Load Balancing 알고리즘](#4-load-balancing-알고리즘)
- [5 Health Check & 장애 대응](#5-health-check--장애-대응)
- [6 세션 처리와 Sticky Session](#6-세션-처리와-sticky-session)
- [7 Load Balancer 아키텍처](#7-load-balancer-아키텍처)
- [8 클라우드 환경의 Load Balancer](#8-클라우드-환경의-load-balancer)
- [9 성능과 운영 관점](#9-성능과-운영-관점)

## 1 Load Balancer 개요
### 1.1 Load Balancer란 무엇인가
> **Load Balancer(LB)**는 클라이언트의 **요청을 여러 서버로 분산**시켜주는 중간 계층 컴포넌트다.

- 클라이언트는 서버가 여러 대인지 모름.
- 클라이언트는 항상 Load Balancer 하나로만 요청을 보냄.
- Load Balancer는 요청을 적절한 서버로 전달함.
  
⇒ **"트래픽 분산 + 가용성 확보 + 확장성 제공"**

### 1.2 왜 Load Balancer가 필요한가
- **단일 서버의 한계**

  - 트래픽이 증가하면 CPU / 메모리 사용량이 올라가고 여기에는 한계가 있음
  - 서버에서 장애가 발생하면 전체 서비스가 다운됨
  - 배포나 재시작 시에 전체 서비스가 중단됨

- **Load Balancer를 통한 해결**

  - 트래픽을 분산함 → 요청을 여러 서버에 나눠줌
  - 장애 격리함 → 장애가 발생한 서버로는 보내지 않음
  - 무중단 배포가 가능함 → 배포나 재시작 중인 서버로는 보내지 않음

### 1.3 Scale Up vs Scale Out
- **Scale Up (수직 확장)**

  서버 1대를 **더 좋은 스펙으로 교체**함

  장애 시 복구가 어려운건 똑같음

- **Scale Out (수평 확장)**

  서버 **여러 대로 확장**함

  이 경우엔 **Load Balancer가 필수**임  
  고가용성(HA) 확보 가능

## 2 Load Balancer의 기본 동작 원리
### 2.1 트래픽 분산의 핵심 개념
> Load Balancer는 **"들어온 요청을 어떤 서버로 보낼지 결정한다."**

- LB가 이를 위해 사용하는 정보
  
  - 현재 살아있는 서버 목록
  - 각 서버의 상태(Health)
  - 분산 알고리즘
  - (L7의 경우) 요청 내용
    
⇒ 이를 통해 각 요청을 어디로 보낼지 판단함

### 2.2 Connection 기반 vs Request 기반 분산
- **Connection 기반**

  TCP 연결 단위로 서버를 선택함  
  → 같은 TCP 연결에서 온 요청은 계속 같은 서버로 보냄  

  매우 빠르고, 단순하지만, HTTP 요청 단위 제어가 불가능함

- **Request 기반**

  HTTP 요청 하나하나를 기준으로 서버를 선택함  
  → 같은 TCP 연결에서도 요청마다 서버가 달라질 수 있음

  유연하고 라우팅 규칙 적용 가능하지만, 상대적으로 무거움

### 2.3 상태 정보 문제 (Stateless vs Stateful)
- **문제 상황**

  같은 사용자의 요청을 다른 서버로 보내면 서버 세션 정보를 사용을 못하는데?

- **상황 별 설계**

  **Stateless 설계** → 서버는 상태를 들고있지 않고, Redis나 헤더에 정보를 저장함

  **Stateful 설계** → 서버가 상태를 보유하게 하고, 같은 사용자의 요청은 항상 같은 서버로 보냄  
  → Sticky Session 필요함

### 2.4 세션 유지(Session Persistence)의 필요성
**Stikey Session 이란?**  
> "특정 클라이언트는 항상 같은 서버로 보내겠다."

- **방법**

  - 쿠키에 서버 ID 저장
  - IP Hash
  - LB 내부 매핑 테이블

- **단점**

  - 서버 간 부하 불균형
  - 서버 장애 시 세션 소실
  - Scale Out 효과 감소

⇒ 실무에서는 Sticky Session을 피한다.

## 3 Load Balancer 분류
Load Balancer는 **어느 계층까지 패킷을 이해하느냐**에 따라 나뉜다.

### 3.1 L4 Load Balancer (전송 계층)
> OSI 4계층 정보까지 이용한다.  
> → IP, Port, TCP 연결 정보만 사용

TCP 연결이 들어오는 순간  
→ 서버 하나를 선택하고, → **TCP 연결이 종료될 때까지 서버가 고정**된다.  
(HTTP 요청 정보를 보지 않기 때문)

- **장점**

  매우 빠르고, 대규모 트래픽에 강하며, 안정적이다.

- **단점**

  - URL, Header, Cookie 기반 분기 불가
  - 요청 단위 라우팅이 불가능
  - 세밀한 제어가 어려움

### 3.2 L7 Load Balancer (응용 계층)
> OSI 7계층 정보까지 이용한다.  
> → **HTTP 요청을 파싱**함.  

**HTTP 요청을 파싱**하고, 하나의 TCP 연결에서도 **요청마다 서버를 선택**한다.

- **장점**

  - URI, Method, Header, Cookie 기반 분기 가능
  - MSA에 적합
  - A/B 테스트, Canary 배포 가능

- **단점**

  - 처리 비용이 큼
  - L4보다 느림
  - 설정이 복잡함

### 3.3 L4? L7?
- **L4를 선택하는 경우**

  - 초당 연결 수가 매우 많음
  - 요청 내용은 중요하지 않음
  - 지연 시간이 최우선
 
- **L7을 선택하는 경우**

  - URL, API 별로 분기 필요
  - 인증, 헤더 처리 필요
  - 무중단 배포, 트래픽 제어 필요

## 4 Load Balancing 알고리즘

## 5 Health Check & 장애 대응

## 6 세션 처리와 Sticky Session

## 7 Load Balancer 아키텍처

## 8 클라우드 환경의 Load Balancer

## 9 성능과 운영 관점

