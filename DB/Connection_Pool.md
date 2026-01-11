# Connection Pool
### 📚 목차
- [1 Connection Pool의 필요성](#1-connection-pool의-필요성)
- [2 Connection Pool의 기본 개념](#2-connection-pool의-기본-개념)
- [3 Connection Pool 내부 구조](#3-connection-pool-내부-구조)

## 1 Connection Pool의 필요성
### 1.1 DB Connection의 실제 비용
요청이 올 때마다 DB에 연결을 새로 만드는 것은 생각보다 비싼 작업이다.

- **네트워크 레벨: TCP 연결 비용**  
  3-way handshake 비용 발생
- **보안 레벨: TLS 비용**  
  (DB와 서버 사이에 사용한다면) TLS handshake 비용 발생
- **DB 레벨: 인증 + 세션 생성 비용**  
  사용자 인증, 세션/스레드 구조 할당, 권한 로딩 등

→ 이런 것을 하느라 정장 DB가 **쿼리를 처리할 시간이 줄어든다**.

### 1.2 Connection 없는 구조의 한계
- **평균은 괜찮을 수 있지만...**  
  트래픽이 몰릴 때 연결을 맺고 설정하는 단계로 인해 지연이 폭증할 수 있다.
- **Connection storm**  
  앱 서버에 요청이 몰리면서 DB 연결이 몰려버리면 전부 연결을 설정해야 하기 때문에  
  DB는 CPU를 여기에 다 써버리고, 쿼리 처리가 지연된다.  
  → 서비스 느려짐  
  → 요청이 쌓임  
  → 재시도 폭발

### 1.3 Pool이 해결하는 2가지 핵심
- **연결 생성 비용을 한 번만 지불한다.**  
  앱 시작 시 미리 연결을 만들어놓고 재사용해서  
  → 이후 연결 비용을 없앤다.  
  → 응답이 빨라지고, DB는 쿼리 처리에 대부분의 시간을 쓸 수 있게됨
- **DB 접근 동시성을 제한한다.**  
  DB는 최대 연결 수가 제한되어 있고, DB가 감당할 수 있는 수준 이상의 요청이 오는 것을 막아야 한다.  
  → Connection Pool을 통해 동시성을 제한한다.

## 2 Connection Pool의 기본 개념
### 2.1 Connection Pool이란?
> Connection Pool은 앞서 본 한계를 극복하기 위한 방식으로,  
> DB에 대한 "물리적 연결"을 제한된 개수로 미리 확보해 두고 요청마다 빌려주고 돌려받는 관리 장치이다.

- **Physical Connection vs Logical Connection**  
  - **Physical Connection**  
    실제 애플리케이션이 DB와 맺은 TCP 연결이다.  
    → `TCP 소켓 + (옵션) TLS + DB 세션`으로 구성된다.
  - **Logical Connection**  
    → 애플리케이션(JDBC)이 들고 있는 `Connection` 핸들  
      (Physical Connection을 감싸고 있는 **프록시 객체**이다.)  
    → `close()`를 호출해도 실제 TCP 연결이 끊기는 것이 아니라 프록시를 통해 **풀로 반납**한다. (연결은 유지됨)

### 2.2 기본 동작 흐름
1. **애플리케이션 시작**  
  DataSource가 초기화된다.
2. **풀에 N개의 커넥션 생성**  
   DB와 연결을 맺고 커넥션을 관리한다.  
   (Idle: 빌려줄 수 있는 커넥션, Active: 대여 중인 커넥션, Pending/Wait: 커넥션이 없어서 기다리는 요청들)
3. **요청이 오면 풀에서 대여**  
  idle인 커넥션이 있으면 바로 빌려준다.

  idle이 없고, 아직 maxPoolSize 미만이면 새로 만들어서 준다.

  둘 다 아니면 대기열에서 기다린다. (너무 오래 기다리면 timeout)
4. **사용 후 반납**  
  `conn.close()`를 호출하면 반납되고, 다시 Idle 상태가 된다.  
  
  이 반납이 빨리빨리 이루어져야(=트랜잭션 범위가 짧아야) 풀의 효율이 좋아진다.  
  (커넥션을 빠르게 반납해서 풀이 고갈되지 않게 해야한다.)

### 2.3 Pool vs DB Thread Pool
> Connection Pool은 DB로 들어가는 일종의 "**입장권**"이고,  
> DB Thread Pool은 DB 내부에서 실제로 일하는 "**인력**"이다.  
> → Connection Pool이 아무리 커도 **DB Thread Pool이 작으면 그 요청을 동시에 처리 못함**  
> (연결 당 스레드 구조)

이 두 개를 모두 조정해야 튜닝이 의미가 있다.

## 3 Connection Pool 내부 구조
### 3.1 핵심 구성 요소
- **Idle Pool**  
  **지금 당장 빌려줄 수** 있는 커넥션 목록이다.
- **Active Pool**  
  어떤 스레드가 대여해서 **사용중인** 커넥션이다.
- **Waiting Queue**  
  Idle Connection이 없어서 대기중인 요청 목록이다.  
  (여러 구현 방식이 있지만, HikariCP 기준으로는 전통적인 FIFO 구조는 아니다.)

### 3.2 대여 알고리즘
여러 알고리즘이 있지만, HikariCP는 `ConcurrentBag`라는 자료구조를 이용해서  
빠른 속도와 최소한의 락 경합을 제공한다.  
(HikariCP는 Fairness보다 Throughput에 초점이 맞춰져있다.)

### 3.3 Connection Pool의 Evict
- **Idle Timeout**  
  Connection Pool은 연결을 계속 유지하지 않는다.  
  → **일정 시간 동안 사용되지 않은 idle 상태의 연결은 자동으로 제거**된다.  
  (HikariCP `idleTimeout` 디폴트 10분, `minimumIdle` 설정 보다 더 적어지진 않음)
- **Max Lifetime**  
  연결이 생성된 시점부터 유지될 수 있는 **최대 수명**이다.  
  → 이 시간이 지나면 최근 사용 여부와 상관 없이 제거 대상이 된다.  
  (HikariCP `maxLifetime` 디폴트 30분)

- **Eviction이 있는 이유**  
  - 좀비 커넥션(실제 DB 서버와의 연결은 끊어진 커넥션) 방지 및 유효성 유지
  - 리소스 효율적 관리 → 현재 요청이 많지 않은데 스레드 풀을 max로 채울 필요가 없음
