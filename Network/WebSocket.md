# WebSocket
### 📚 목차
- [1 WebSocket 등장 배경](#1-websocket-등장-배경)
- [2 WebSocket 기본 개념](#2-websocket-기본-개념)
- [3 WebSocket 프로토콜 구조 (RFC 6455)](#3-websocket-프로토콜-구조-rfc-6455)
- [4 WebSocket Frame 구조](#4-websocket-frame-구조)
- [5 WebSocket 연결 관리](#5-websocket-연결-관리)
- [6 WebSocket 서버 아키텍처](#6-websocket-서버-아키텍처)
- [7 WebSocket vs 다른 실시간 기술](#7-websocket-vs-다른-실시간-기술)
- [8 WebSocket 보안](#8-websocket-보안)
- [9 WebSocket과 스케일링](#9-websocket과-스케일링)

## 1 WebSocket 등장 배경
### 1.1 HTTP의 근본적 한계
- **요청-응답 모델**

  HTTP는 기본적으로 요청-응답 모델로,  
  **클라이언트가 요청해야만** 서버가 응답이 가능하다.  
  → 서버는 자발적으로 데이터를 밀어줄 수 없다.  
  → 실시간 이벤트에 부적합

- **Stateless + Short-lived Connection**
  실시간 상태 동기화가 어렵다.

### 1.2 기존의 실시간 통신 우회 방법과 한계
- **Polling**

  클라이언트가 주기적으로 요청을 보내는 방식이다.

  → 불필요한 요청 폭증 / 서버 부하 증가

- **Long Polling**

  서버가 이벤트가 생길 때까지 응답을 붙잡는다.  
  → 이벤트가 있을때까지 커넥션을 계속 유지한다.  
  → 이벤트가 없으면 타임아웃 후 응답한다.

  → 커넥션이 계속 붙잡힘 / 타임아웃/재연결 비용

- **HTTP Streaming**

  하나의 HTTP 응답을 끊지 않고 계속 전송한다.
  (SSE)

⇒ 이런 것들은 서버 측에서 전송 시작을 할 수가 없다.

### 1.3 실시간 서비스의 요구사항 변화
> 웹이 "문서"에서 "애플리케이션"으로 변화했다.  
> ex) 채팅, 알림, 협업 도구, 실시간 모니터링 등

⇒ 높은 실시간성을 가진, 서버 푸시가 가능한, 지속 연결이 필요하다.

## 2 WebSocket 기본 개념
### 2.1 WebSocket이란 무엇인가
> WebSocket은 브라우저와 서버 간에 **하나의 TCP 연결**을 유지하면서  
> **양방향(real-time) 통신**을 가능하게 하는 프로토콜이다.

WebSocket의 핵심 특징
- 지속 연결
- 양방향 통신
- Full-Duplex
- 낮은 오버헤드

또한, WebSocket은 HTTP에서 시작해서 `Upgrade`를 통해 WebSocket 프로토콜로 전환한다.

### 2.2 HTTP와 WebSocket의 차이
| 구분      | HTTP                   | WebSocket  |
| ------- | ---------------------- | ---------- |
| 통신 모델   | Request–Response       | Message 기반 |
| 방향성     | Client → Server 단방향 요청 | 양방향        |
| 서버 푸시   | 없음                      | 가능          |
| 연결      | 짧음 (논리적으로)             | 지속         |
| 헤더 오버헤드 | 큼                      | 매우 작음      |
| 실시간성    | 낮음                     | 높음         |

### 2.3 Full-Duplex 통신 개념
> Full-Duplex는 클라이언트와 서버가 동시에 송수신 가능한 통신방식을 말한다.

| 방식          | 설명                     |
| ----------- | ---------------------- |
| Half-Duplex | 한쪽이 말하면 다른 쪽은 대기 (무전기) |
| Full-Duplex | 동시에 말하고 듣기 가능 (전화)     |

- **HTTP는 왜 Full-Duplex가 아닌가?**

  HTTP는 요청-응답 모델로 여러 기능을 제공하는데,  
  이게 깨질 경우 HTTP의 정체성이 흐려지기에 서버의 시작을 제한하였다.
  (HTTP에서 서버는 클라이언트 요청의 연장선 리소스만 가능하다. → 기존 요청에 종속)

- **WebSocket은 왜 가능한가?**

  WebSocket은 실시간 메시지 교환을 위해 개발된 프로토콜로,  
  연결 이후에는 양쪽 모두 제한 없이 메시지 전송이 가능하다.

### 2.4 Connection-Oriented 통신
WebSocket은 Connection-Oriented 방식으로 연결이 설정된 후에 그 연결을 기반으로 데이터를 주고받는다.
**연결 상태는 서버에 유지**된다.

- 온라인 접속자 목록 관리 가능
- 1대1 / 1대N 메시지 라우팅 가능
- 특정 사용자에게만 push 가능

- **HTTP와의 차이**

  | 항목     | HTTP           | WebSocket |
  | ------ | -------------- | --------- |
  | 연결 상태  | 없음 (Stateless) | 있음        |
  | 세션 인식  | 매 요청마다         | 연결 단위     |
  | 리소스 사용 | 요청당            | 연결당       |


## 3 WebSocket 프로토콜 구조 (RFC 6455)
### 3.1 Handshake 과정 (HTTP → WebSocket Upgrade)
WebSocket 연결은 **무조건 HTTP 요청으로 시작**한다.

- **1번 클라이언트 → 서버 (HTTP 요청)**

  ```http
  GET /chat HTTP/1.1
  Host: example.com
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
  Sec-WebSocket-Version: 13
  ```
  - `Upgrade: websocket`: 이 연결을 WebSocket으로 바꾸고 싶다.  
  - `Connection: Upgrade`: 이 요청은 프로토콜 전환 목적이다.  
  - `Sec-WebSocket-Key: ...`: 보안 검증용 난수  
  - `Sec-WebSocket-Version: 13`: 현재 표준은 `13`

- **2번 서버 → 클라이언트 (프로토콜 전환 응답)**

  ```http
  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
  ```
  - `101 Switching Protocols`: 프로토콜 변경을 승인함  
  - `Sec-WebSocket-Accept: ...`: `Sec-WebSocket-Key`를 가공해서 생성함  
  
  이 응답 이후로는 **HTTP는 끝**이다.  
  → 이제 WebSocket 프레임 프로토콜 사용

### 3.2 Sec-WebSocket-Key / Accept 의미
해당 서버가 **WebSocket 프로토콜을 이해하고 정상적으로 응답했음을 증명**한다.

- **Sec-WebSocket-Key (클라이언트 → 서버)**

  무작위 값이다.

  **진짜 WebSocket 업그레드 요청**임을 표시

- **Sec-WebSocket-Accept (서버 → 클라이언트)**

  클라이언트 한테 받은 `Sec-WebSocket-Key`에 GUID 값을 더하고,  
  SHA-1과 Base64를 거쳐서 만든다.  
  (여기서 GUID는 WebSocket 스펙에 고정적으로 정해진 상수이다.)

- **클라이언트 확인**

  클라이언트는 직접 계산해서 `Sec-SebSocket-Accept`와 비교하고,  
  WebSocket이 정상적으로 맺어졌음을 확신할 수 있다.

### 3.3 연결 수립 이후 상태 변화
서버의 `101 Switching Protocol` 응답 후부터
- HTTP Request / Response 사용 안함
- HTTP Header 사용 안함
- **WebSocekt Frame 기반 통신 시작**
- **Message 단위 송수신 시작**

### 3.4 포트 사용 (80, 443)
WebSocket은 **기존 HTTP 인프라를 그대로 사용**한다.

| 스킴       | 설명                     | 포트  |
| -------- | ---------------------- | --- |
| `ws://`  | WebSocket              | 80  |
| `wss://` | WebSocket Secure (TLS) | 443 |

→ 방화벽, 프록시, 로드밸런서 통과가 쉽다.  
→ HTTPS 위에서 자연스럽게 WebSocket 사용이 가능하다.

### 3.5 왜 "Upgrade" 방식인가?
- **이유**

  - 기존 웹 인프라 재사용
  - 브라우저 호환성
  - 점진적 도입 가능

- **만약 전용 포트를 썼다면?**

  - 방화벽 차단
  - 기업 네트워크 불가
  - 브라우저 지원 어려움

⇒ HTTP로 문을 열고, WebSocket으로 갈아탄다.

## 4 WebSocket Frame 구조

## 5 WebSocket 연결 관리

## 6 WebSocket 서버 아키텍처

## 7 WebSocket vs 다른 실시간 기술

## 8 WebSocket 보안

## 9 WebSocket과 스케일링

