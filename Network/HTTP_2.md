# HTTP/2
### 📚 목차
- [1 HTTP/2 등장 배경](#1-http2-등장-배경)
- [2 HTTP/2의 핵심 목표](#2-http2의-핵심-목표)
- [3 HTTP/2의 핵심 개념](#3-http2의-핵심-개념)
- [4 Flow Control & Priority](#4-flow-control--priority)
- [5 HAPCK (헤더 압축)](#5-hapck-헤더-압축)
- [6 HTTP/2 연결 수명 주기](#6-http2-연결-수명-주기)
- [7 HTTP/2와 TCP의 관계](#7-http2와-tcp의-관계)
- [8 HTTP/2 실전 성능 이슈](#8-http2-실전-성능-이슈)

## 1 HTTP/2 등장 배경
> [요약]  
> HTTP/1.1이 "한 연결에서 요청을 직렬로 처리하는 구조"라서,  
> 웹이 커질수록 지연이 구조적으로 커졌기 때문이다.

### 1.1 HTTP/1.1의 구조적 한계
- **HOLB(Head-of-Line Blocking)**

  앞의 요청이 느리면 뒤의 요청들이 다 막히는 성능 저하 현상을 말한다.
  
  → HTTP/1.1은 같은 연결에서 응답이 온 후에 다음 요청을 보낼 수 있음  
  (HTTP/1.1 pipelining인 경우에는 응답을 기다리지 않고 다음 요청을 계속 보내긴 한다.  
  하지만, pipelining을 사용하더라도 이전 응답이 늦어짐으로 인한 HOLB는 피할 수 없다.)

- **브라우저의 다중 연결**

  HTTP/1.1의 경우는 HOLB을 해결할 방법이 없기 때문에  
  브라우저는 연결을 여러개 만들어서 동시에 요청을 보내고 응답을 받는다. (보통 최대 6개)

  → TCP 연결 수 증가로 인한 오버헤드 발생  
  (TCP handshake, TLS handshake, TCP slowstart, 관리해야 하는 연결 증가 등)

- **헤더 중복 전송**

  HTTP/1.1은 헤더가 바뀐 것이 없어도 그대로 반복 전송한다.  
  → 대역폭 낭비  
  → 특히 모바일/고지연 환경에서 더 치명적

- **한계를 피하려 하다보니 개발 복잡도가 증가함**

  **도메인 샤딩**: 서브 도메인을 늘려서 도메인 연결 제한 우회

  **스프라이트**: 이미지 여러 개를 한 장으로 합침  
  → 빌드/캐싱/부분 갱신이 어려워짐

  **번들링/인라이닝**: js와 css을 html에 합침 / 이미지를 base64 인코딩해서 html에 합침

### 1.2 SPDY → HTTP/2 표준화 과정
- **구글의 SPDY**

  구글이 SPDY(스피디)를 만들어서 하나의 TCP 연결 위에서 요청을 여러 개 동시에(멀티플렉싱) 흘려보내고 헤더를 압축하면,  
  웹이 빨리지는 것을 보여줬다.

  → 이를 IETF가 표준화했고, 그것이 HTTP/2이다.

## 2 HTTP/2의 핵심 목표
> [요약]  
> 연결 수를 늘리지 않고, 한 연결에서 더 많이/더 효율적으로/더 빨리 처리하자

### 2.1 지연(latency) 감소 전략
- **요청 직렬화 제거 (Multiplexing 전제)**

  HTTP/1.1의 지연은 대부분 **대기**에서 발생했다.

  → HTTP/2의 목표는 **요청을 동시에 보내면서 응답도 동시에 받을 수 있게**하는 것이다.  
  → 지연의 원인을 실제 서버 처리 시간에 더 가까워지게

- **연결 재사용 극대화**

  여러 연결을 사용하던 것을 하나의 연결로 바꾸면서  
  TCP handshake, TLS handshake 비용을 최소화했다.

### 2.2 네트워크 효율 개선
- **헤더 전송 비용 최소화**

  헤더를 반복 전송하는 HTTP/1.1과 다르게  
  HTTP/2는 헤더를 저장하자 → 고지연 환경에서 효과 극대회

- **불필요한 패킷 왕복 감소**

  **작은 요청이 많으면** 패킷 수가 폭증함.  
  HTTP/2는 여러 요청을 묶어 보낸다.

### 2.3 서버/클라이언트 리소스 절약
- **서버 관점**

  HTTP/2에서는 적은 열결 수로 더 많은 요청 처리  
  → 스케일링 예측 가능성 증가

- **클라이언트 관점**

  리소스 요청 로직 단순화  
  (HTTP/1.1의 한계를 극복하기 위한 올라간 개발 복잡도 정상화)

### 2.4 웹 생태계 복잡도 감소
HTTP/1.1의 한계를 극복하기 귀한 여러 복잡한 것들을 정상화하자!!  
→ 작은 리소스를 작은 단위로 요청해도 OK!!

→ 성능 때문에 아키텍처를 희생하지 않아도 된다.

## 3 HTTP/2의 핵심 개념
### 3.1 Binary Framing Layer
HTTP/1.1는 텍스트 기반이었지만, HTTP/2는 **바이너리(프레임)로 쪼개**서 보낸다.

- **HTTP/1.1**

  ```http
  GET /api/profile HTTP/1.1
  Host: example.com
  User-Agent: Chrome/120
  Cookie: session=abc
  Accept: application/json
  ```
  - 헤더와 바디를 문자열(줄바꿈)로 판단함  
  - 프로토콜 확장이 어려움(고정된 형태)  
  
- **HTTP/2**

  ```text
  00 00 2a    // Length (42 bytes)
  01          // Type = HEADERS
  05          // Flags = END_HEADERS | END_STREAM
  00 00 00 01 // Stream ID = 1
  
  82 86 84 41 8c f1 e3 c2 e5 f2 3a 6b a0 ab 90 f4 ff
  ```
  - 파싱이 빠르고 명확함
  - 프로토콜 확장이 쉬움 (새 프레임 타입 추가)
  - 프레임 단위여서 멀티플렉싱 구현이 쉬움

### 3.2 Stream / Message / Frame 구조
HTTP/2에서는 데이터를 Frame, Message, Stream 세 가지로 계층적으로 나눈다.

- **Frame(프레임)**

  > HTTP/2에서 **전송의 최소 단위  
  > (헤더와 바디 모두 프레임으로 쪼개진다.

  각 프레임은 **Stream ID**가 붙어서 어느 요청/응답에 속하는지 표시되어 있다.

  - **대표 프레임 타입**

    `HEADERS`: 요청/응답 헤더  
    `DATA`: 바디(본문)  
    `SETTINGS`: 연결 설정 교환  
    `WINDOW_UPDATE`: 흐름 제어  
    `RST_STREAM`: 스트림 강제 종료  
    `PING`: keepalive/RTT 측정  
    `GOAWAY`: 연결 종료 예고

- **Message(메시지)**

  > HTTP에서 **요청 1개 / 응답 1개**를 의미

  한 개의 HTTP 요청(또는 응답)이 프레임으로 쪼개져서 구성된다.

- **Stream(스트림)**

  > 요청/응답을 담는 논리적 채널

  하나의 TCP 연결 안에는 스트림이 여러 개 동시에 존재한다.  
  스트림마다 ID가 있다. (Stream ID)

  클라이언트가 여는 스트림의 Stream ID - 홀수  
  서버가 여는 스트림의 Stream ID - 짝수

<img width="1262" height="1102" alt="image" src="https://github.com/user-attachments/assets/c94d8ffc-2c14-476b-b8be-866460b31008" />

⇒ Stream = 요청/응답 1쌍의 논리적 통로  
⇒ Message = 그 "요청 전체" 또는 "응답 전체"  
⇒ Frame = 그 Message를 잘게 쪼갠 최소 전송 단위

### 3.3 Multiplexing(다중화)
- **HTTP/1.1**

  ```
  [요청1] --------> [응답1]
  [요청2] --------> [응답2]
  ```
  하나의 연결에서 이전 요청/응답이 끝나야 다음 요청과 응답이 가능하다.

- **HTTP/2**

  ```
  Stream 1 (요청 A):
    HEADERS(A)  DATA(A1)  DATA(A2) ...
  
  Stream 3 (요청 B):
    HEADERS(B)  DATA(B1)  DATA(B2) ...
  
  실제 전송은 프레임이 섞여서(interleaving) 감:
    HEADERS(A) → HEADERS(B) → DATA(A1) → DATA(B1) → DATA(A2) → ...
  ```
  하나의 TCP 연결 위에서 여러 요청/응답이 프레임으로 나뉘어져서 **섞여서 동시에 간다**.

⇒ Multiplexing을 통해 HTTP 레벨에서의 HOLB는 해결되었다.  
(하지만, TCP 레벨에서의 HOLB는 해결 못함)

## 4 Flow Control & Priority

> [요약]  
> HTTP/2는 다중화되었기 때문에, 누가 얼마나 보낼 수 있는지를 엄격히 관리해야 한다.

### 4.1 Flow Control이 필요한 이유
HTTP/2에서는 하나의 TCP 연결 위에 수십~수백 개의 스트림이 동시에 DATA 프레임을 보낼 수 있다.  
→ 어떤 스트림이 대용량 데이터를 계속 줄 수 있다.  
→ **수신 측 버퍼 폭발 & 다른 스트림 처리가 늦어짐**

⇒ Multiplexing이 곧 Flow Control의 필요 원인

### 4.2 Flow Control의 기본 개념
HTTP/2의 Flow Control은 **수신자 기반**이다.  
→ 수신자가 더 얼만큼 보내도 되는지 알려준다.  
→ 송신자는 이 범위(window) 안에서만 프레임 전송 가능  
(TCP의 흐름 제어와 비슷)

### 4.3 두 단계 Flow Control 구조
HTTP/2에는 두 가지 흐름 제어가 존재한다.

- **Connection-level window**  

  하나의 TCP 연결 전체에 대한 수신 허용량

  → 스트림이 많아질 때 합산 폭주 방지;

- **Stream-level window**

  각 스트림별 수신 허용량

  → 한 스트림의 영향력 조절

⇒ 실제로 전송 가능한 양 = **min(Connection-level window, Stream-level window)**

### 4.4 WINDOW_UPDATE 프레임
> 수신자가 송신자에게 "이만큼 더 보내도 돼"를 알려주는 프레임

→ **WINDOW_UPDATE가 늦으면 송신자는 대기할 수밖에** 없다.  
→ HTTP/2의 튜닝 포인트

### 4.5 Priority
HTTP/2는 스트림마다 "**중요도**"를 표현할 수 있다.

- **구성 요소**

  - **Weight (1~256)**

    상대적 비중  
    숫자가 크다고 먼저 처리가 아니라, Weight 비율에 따라 처리량이 달라진다.  
    → 더 큰 비중을 받는 것  
    (형제 관계(같은 부모)에서만 Weight 비교로 처리의 순서가 나뉜다.)
  - **Dependency (부모 스트림)**
 
    이를 통해 Priority는 **트리 구조**가 된다.  
    → 부모가 막히면 자식도 막힌다.
  - **Exclusive 플래스**
 
    부모의 다른 자식들을 자신의 자식으로 재배치 한다.  
    → 트리 구조 재편성

- **예시**

  HTML → CSS → JS → 이미지 순으로 처리

- **현실**

  Priority는 구현 복잡도가 올라가고, 브라우저마다 정책이 다르다.  
  → Priority는 잘 쓰이지 않고, "힌트" 정도의 역할만 한다.  
  (실제 성능은 Flow Control이 좌우한다.)

## 5 HAPCK (헤더 압축)
> [요약]  
> HTTP/2는 헤더를 '번호(인덱스)'로 참조하고, 상태를 공유해 **반복 전송을 제거**한다.

### 5.1 HPACK의 기본 아이디어
- **기존**
  
  HTTP/1.1은 헤더를 매 요청마다 그대로 재전송했다.  
  또한 헤더를 압축하지도 않았다.
  
  → 매우 비효율적이다.

- **HPACK의 기본 아이디어**

  1. 헤더 이름/값을 번호로 참조
  2. 연결 단위로 상태를 공유
  3. 압축 방식은 결정적(deterministic)

### 5.2 HPACK 구성 요소
- **Static Table (고정 테이블)**

  - HTTP에서 자주 쓰는 테이블을 미리 정의 (헤더 각 요소의 인덱스가 지정됨)
  - 이름만 정의된 항목도 있고, 이름+값이 함께 정의된 항목도 있다.
  - 각 항목은 고정된 인덱스를 가지며 통신 중 변경되지 않는다.
  - 예시  
    `:method: GET`은 2번 인덱스  
    `:status: 200`은 8번 인덱스  
    `:status: 400`은 12번 인덱스  
    `:accept-charset:`은 15번 인덱스

- **Dynamic Table (동적 테이블)**

  - **연결마다 달라질 수 있는 헤더**들이 저장된다. (인증 토큰 등)

  - 이 연결에서 실제로 주고받은 헤더를 저장한다.  
  (클라이언트와 서버가 **동기화된 상태**로 유지)

- **Header Encocding Rules**

  - 이전에 주고 받은 내용이랑 **같은 경우**  
    → 인덱스만 보낸다.

  - **이름만 같은 경우**  
    → 인덱스와 새로운 값을 보낸다.

  - **처음 등장하는 경우**  
    → 이름과 값을 literal로 전송  
    → 필요 시 Dynamic Table에 추가

### 5.5 Header 표현 방식
HPACK은 헤더를 보낼 때 **의도를 명확히 표현**한다.

- **Indexed Header Field**

  Static / Dynamic Table에 있는 항목 → **압축률 높음**

- **Literal Header Field (with indexing)**

  새 헤더 전송 하면서 테이블에 추가  
  → **이후 재사용**

- **Literal Header Field (without indexing)**

  한 번만 쓰고 **테이블에 저장 안함**  
  → 민감한 값에 사용

- **Never Indexed**

  절대 테이블 저장 금지
