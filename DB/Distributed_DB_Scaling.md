# 분산 데이터베이스 확장 전략(Distributed Database Scaling)
### 📚 목차
- [1 DB 확장의 필요성](#1-db-확장의-필요성)
- [2 Replication (데이터 복제)](#2-replication-데이터-복제)
- [3 Sharding (데이터 분할)](#3-sharding-데이터-분할)
- [4 Replication vs Sharding 비교](#4-replication-vs-sharding-비교)
- [5 Replication + Sharding 함께 쓰기](#5-replication--sharding-함께-쓰기)
- [6 실제 DB 시스템에서의 적용](#6-실제-db-시스템에서의-적용)

## 1 DB 확장의 필요성
DB 확장은 "요구되는 부하(트래픽/데이터/연산)가 단일 DB 1대로 감당이 안 될 때"하는 선택이다.

원인은 보통 세 가지 중 하나다.  
- **성능** - 응답 시간이 느려짐
- **용량** - 데이터가 너무 많아짐
- **가용성** - 장애 발생 시 서비스 중단

### 1.1 단일 DB의 한계
- **CPU 한계**

  쿼리 실행과 트랜잭션 처리, 락 관리, MVCC 정리 등에서 CPU를 소비함

  근데 트래픽이 늘면서 동시에 처리해야 할 작업이 많아지면  
  → 평균은 괜찮더라도 p95/p99에서 매우 안좋아짐

- **I/O 한계**

  DB는 저장장치에 쓰고 읽는 작업의 반복이다.

  트래픽이 증가하면 → 랜덤 I/O가 증가하고 → 디스크 대기시간이 늘어난다.

  - 쓰기 시 로그 기록 및 flush
  - 읽기 시 메모리에 없으면 디스크 접근
  - 인덱스가 많으면 쓰기 시 추가 작업
  - 인덱스가 크고 많으면 메모리에 못 올림

- **메모리 한계**

  DB는 메모리 싸움이다.  
  메모리가 적으면 **버퍼 캐시**가 작아지고,  
  → 그러면 데이터와 인덱스 페이지를 읽을 때 **캐시 미스**가 나고 지옥이 시작된다.

  정렬이나 조인 등에서 메모리가 부족하면 메모리에서 중간 결과를 처리할 수 없어서  
  디스크에 임시로 쓰는 **디스크 스필(Disk Spill)** 이 발생할 수 있다.

- **커넥션/동시성 한계**

  동시 요청이 많아지면  
  → 커넥션 수 증가  
  → **락 경합** 증가  
  → 병목 발생

- **유지보수/운영 한계**

  단일 DB가 너무 커진다.  
  - 백업 시간, 복구 시간이 길어짐
  - 스키마 변경 부담
  - 장애 한 번에 전체 서비스 영향

⇒ 단일 DB는 성능/가용성/운영 면에서 한계가 존재한다.

### 1.2 Scale-up vs Scale-out
- **Scale-up (수직 확장)**

  더 좋은 기기로 바꾸는 것이다.  
  → CPU, 메모리, 디스크 업그레이드

  - **장점**  
    구현/개발 난이도 낮음  
    기존 쿼리/트랜잭션 모델 유지  
    효과는 확실함

  - **단점**  
    한계가 명확함 + 비용이 기하급수적으로 증가  
    여전한 SPOF  
    특정 병목(락 경합 등)은 스펙으로 해결이 안됨

- **Scale-out (수평 확장)**

  DB 노드를 여러 대로 늘리는 방식이다.  
  그 중 대표 전략이 **Replication**과 **Sharding**이다.

  - **장점**  
    더 큰 확장 가능  
    장애 내성이 좋아짐 (replication)  
    읽기/쓰기/용량을 "역할 분리"해서 확장 가능

  - **단점**  
    설계 난이도 급상승  
    분산 환경의 문제 (Consistency, lag, 쿼리 난이도)

⇒ 상황에 맞는 적절한 선택이 필요하다.

## 2 Replication (데이터 복제)
> 같은 데이터를 여러 DB 노드에 복제해서  
> 읽기 성능과 가용성(HA)을 확보하는 DB 확장 전략
> Replication은 **Read를 분산**하는 기술

### 2.1 Replication의 개념
- **동일 데이터 복제**

  하나의 DB에 있는 데이터 변경 사항을 다른 DB에 그대로 적용한다.  
  → 여러 DB가 같은 데이터를 보유

- **Primary-Replica 구조**

  <img width="700" alt="image" src="https://github.com/user-attachments/assets/9038d92e-c51e-4486-ac7b-7bd3e8389949" />

  가장 일반적인 Replication 구조이다.

  - **Primary**
 
    모든 Write(INSERT, UPDATE, DELETE)를 담당한다.  
    변경 로그를 생성한다.

  - **Replica**
 
    Primary의 로그를 받아 replay한다.  
    보통 Read 전용 노드이다.

  핵심은 **Replica는 쓰기 부하를 나눠갖지 않는다**는 것이다.  
  모든 쓰기는 단일 Primary에서 이루어진다.

- **Primary와 Replica의 쓰기 비용 차이**

  > Primary는 쓰기에 대해 모든 책임을 지고,  
  > Replica는 결과만 받아서 Replay한다.

  - **Primary write 시 발생 비용**
 
    - 클라이언트 트랜잭션 처리
    - Row-level lock / MVCC 관리
    - 인덱스 갱신
    - Buffer Pool 더티 페이지 생성
    - WAL 기록 + fsync
    - commit ordering
    - 동시성 제어

  - **Replica write 시 발생 비용**

    - Primary 로그 수신
    - 순차 로그 append
    - 순서대로 replay
    - 락이 거의 없음
   
    Replica에서 일어나는 쓰기는 병렬성이 제한되기 때문에 동시성 제어에서 훨씬 가볍다.

### 2.2 Replication의 목적
- **읽기 성능 확장(Read Scale-out)**

  Read 요청을 여러 Replica로 분산시킨다.
  → 대부분의 서비스는 Read 요청 비율이 높기 때문에 부하 분산에 효과적임

  → 읽기가 80~90%인 서비스에서는 Replication이 가장 먼저 검토된다.

- **가용성 확보(High Availability) & 장애 대응(Failover)**

  DB 한 대는 SPOF

  → Replica가 있으면 Primary 장애 시에 Replica를 승격(Promote)  
  → 서비스 지속

  (failover는 따로 설정과 도구를 사용해야 한다.)

### 2.3 Replication 방식
- **Asynchronous Replication (비동기)**

  Primary는 Replica 반영을 기다리지 않고 바로 커밋한다.
  
  가장 널리 쓰이는 방식이다.

  **장점**: Write 성능 우수, 지연(latency) 최소

  **단점**: Replication Lag 발생, Primary 장애 시 최근 데이터 유실 가능

- **Synchronous Replication (동기)**

  Replica 반영 완료 후에만 커밋을 허용한다.

  고일관성 시스템에서만 제한적으로 사용한다.

  **장점**: 강한 일관성, 데이터 유실 거의 없음

  **단점**: Write latency 증가, Replica가 느리면 전체 Write가 느려짐

- **Semi-synchronous Replication**

  최소 1개 Replica가 로그를 수신하면 커밋을 허용한다.

  성능과 안정성의 현실적인 타협안이다.

### 2.4 Replication의 한계
- **Write 확장 불가**

  모든 Write를 Primary 1대에서 처리한다.  
  → Write TPS는 그대로임..  
  → Write 병목이 문제라면 Replication은 해결책이 되지 못한다.

- **Replication Log**

  Primary와 Replica 간 **데이터 반영 시간 차이**를 말한다.

  Replica 성능 부족, 네트워크 지연, Primary의 Write 폭주 등으로 발생한다.

  → Write 직후 Read 시 데이터가 안 보임  
  → 사용자 입장에서 "저장했는데 안 보이는" 경험 발생

- **Read Consistency 문제**

  Replication 환경에서는 다음이 깨질 수 있다.
  - **Read-after-write consistency**
  - **Monotonic read**

  → 글을 썼는데 조회가 안될 수 있음  
  → 한 번 조회됐던 글이 새로고침하면 사라질 수 있음

  - **대응 방법**
    
    - 중요한 Read는 Primary로
    - 세션 단위 Read routing
    - Lag 감지 기반 라우팅

## 3 Sharding (데이터 분할)
> Sharding은 **데이터를 여러 DB 노드에 나눠서 저장**하는 전략이다.  
> Replication과 다르게 Sharding은 서로 다른 데이터를 분산한다.  
> (Sharding은 기본적으로 **수평 분할**이다.)

### 3.1 Sharding의 개념
- **Horizontal Partitioning(수평 분할)**

  **행(row) 단위**로 데이터를 쪼갠다.  
  → pk 범위 or 해시로 여러 DB에 나눠서 저장

  각 샤드는 전체 데이터의 **부분 집합**

  (Vertical Partitioning(컬럼 분리)은 보통 스키마 분리나 정규화로 부른다.)

- **샤드(Shard)와 샤드 키(Shard Key)**

  **Shard**: 분할된 데이터 조각을 저장하는 단위 (DB 인스턴스/노드/클러스터)

  **Shard Key**: 어떤 데이터가 어떤 샤드로 가야 하는지 결정하는 **기준 필드**

  → user_id가 샤드 키면, user_id % N으로 샤드 번호 결정  

  샤드 키는 Sharding 설계의 80%다.  
  → 키를 잘못 잡으면 확장성/성능/운영 난이도 힘들어짐

### 3.2 Sharding의 목적
- **데이터 크기 확장 (Storage Scale-out)**

  단일 DB에 데이터가 너무 많으면  
  → 테이블이 커짐 → 인덱스가 커짐 → 캐시 적중률 떨어짐 → I/O 증가

  → 샤딩을 통해 각 샤드의 데이터 크기 관리

- **Write 성능 확장 (Write Scale-out)**

  Replication과 다르게 Sharding은 **쓰기 부하가 여러 샤드로 분산**된다.

  ex) 유저별 데이터가 user_id 기반으로 분산 → 유저 별로 서로 다른 샤드에 병렬 처리

- **병렬 처리 (Parallelism)**

  샤드 단위로 독립적인 쿼리가 실행된다.  
  → 이 효과는 대부분의 요청이 단일 shard에서 처리될 때 극대화
  → cross-shard 쿼리가 많아질수록 성능과 복잡도가 급격히 악화

### 3.3 Sharding 전략
- **Range-based Sharding (범위 기반)**

  특정 값의 범위를 특정 샤드에 할당  
  ex) user_id 1~1,000,000은 shard1 / 1,000,001~2,000,000은 shard2

  **장점**: 범위 조회가 빠름, 라우팅이 직관적  
  **단점**: 특정 범위에 트래픽이 몰리면 **Hot Shard** 위험

- **Hash-based Sharding (해시 기반)**

  shard = hash(key) % N (또는 consistent hashing)

  **장점**: 데이터/트래픽 분산이 비교적 균등, Hot Shard 위험 감소  
  **단점**: **범위 조회가 매우 불리**, **샤드 수 변경 시 재배치** 문제

- **Directory-based Sharding (디렉토리/룩업 테이블)**

  키별 샤드 매핑 정보를 별도 메타데이터(룩업 테이블)로 관리

  **장점**: 유연함, 샤드 추가/변경에 대응이 쉬움  
  **단점**: 라우팅을 위한 추가 조회/시스템이 필요, 그것 자체가 SPOF/병목이 될 수 있음

### 3.4 Sharding의 난이도
- **샤드 키 선정 문제**

  좋은 샤드 키를 고르는 것이 매우 중요한다.
  - **카디널리티가 높음** (분산 잘 됨)
  - **트래픽이 고르게 분포** (Hot 방지)
  - 대부분의 **쿼리가 샤드 키를 포함**해서 라우팅이 가능
  - 시간이 지나도 **편향이 심해지지 않아야**

  나쁜 예시: create_at → 최근 데이터에 hot shard / region → 특정 지역에 폭증 가능성

- **Hot Shard**

  > 특정 샤드에만 트래픽/데이터가 몰리는 현상

  - **원인**  
    키 선택 편향  
    "최근 데이터"가 몰리는 패턴  
    특정 VIP/인기 콘텐츠에 집중
  - **대응**  
    키 변경(이게 가장 어려움)  
    해시 기반 설계  
    "hot key"를 따로 분리하는 예외 처리(실무에서 자주 함)  
    캐시나 큐 사용

- **Re-sharding**

  > 샤드를 늘리거나 줄이거나, 분포가 깨져서 재배치해야 하는 상황

  데이터 이동량이 많고, 이동하면서도 서비스는 동작해야함  
  → 어려움

  → consistent hashing이나 directory 방식 등장

- **Cross-shard Query**

  > 여러 샤드를 조회해야 하는 쿼리를 말한다.  
  > (쿼리에 샤드키가 포함 안됨 or JOIN or 집계 등)
  > (90% 이상의 쿼리가 단일 샤드로 처리가 되어야 Sharding이 의미가 있다.)

  ex) 전체 유저 중 상위 100명  
  ex) 전체 주문 합계  
  ex) 유저 테이블과 주문 테이블 JOIN  
  ex) 검색 조건이 샤드 키를 안 타는 경우

  → **모든 샤드에 쿼리**를 날리고, 결과를 합침

  - **대응 방식**  
    - 샤드 키를 JOIN 키와 맞추기
    - 비정규화, CQRS, 별도 집계 시스템
    - 중앙 집계 테이블 (스트리밍/배치로 유지)

## 4 Replication vs Sharding 비교
둘 다 "Scale-out"이다.  
하지만, 해결하려는 문제가 다르고, 비용도 다르다.

### 4.1 해결하는 문제의 차이
- **Read vs Write**

  | 항목       | Replication              | Sharding            |
  | -------- | ------------------------ | ------------------- |
  | Read 확장  | ✅ 매우 잘 됨 (Replica로 분산)   | ⚠️ 가능하지만 설계/쿼리 제약 큼 |
  | Write 확장 | ❌ 거의 안 됨 (Primary 1대 집중) | ✅ 가능 (샤드로 분산)       |

  → Replication은 read scale-out, Sharding은 write + storage scale-out

- **Availability vs Capacity**

  | 항목                | Replication                          | Sharding                               |
  | ----------------- | ------------------------------------ | -------------------------------------- |
  | Availability(가용성) | ✅ 핵심 목적 (Failover)                   | ⚠️ 샤드 수가 늘수록 ‘부분 장애’는 쉬워지지만 ‘전체 관점’ 복잡 |
  | Capacity(용량)      | ❌ 용량이 늘긴 하는데 “같은 데이터 복제”라 효율적 확장이 아님 | ✅ 핵심 목적 (데이터를 나눔)                      |

### 4.2 언제 Replication만으로 충분한가
일반적으로 대부분의 서비스는 읽기 비중이 높고, Replication이 효과 대비 난이도가 낮기 때문에 Replication이 가장 먼저 고려된다.

- **읽기 트래픽이 압도적일 때**
  
- **데이터 크기가 아직 단일 노드에 들어갈 때** (인덱스, 핵심 워킹셋이 버퍼풀에 올라감)
  
- **Write 병목이 "DB가 아니라" 다른 곳일 때**
  
- **강한 일관성이 일부만 필요할 때** (일관성을 어느정도 희생해도 될 때)

읽기 확장 + HA → Replication!!

### 4.3 언제 Sharding이 필요한가
Sharding은 도입 비용이 크지만, 필요해졌을 때는 **피할 수 없는 선택**이다.

- **Write TPS가 단일 Primary 한계를 넘을 때** (Write Latency 증가, 락 경합 증가)

- **데이터 크기가 단일 노드 한계를 넘을 때** (캐시 적중률 하락, 디스크 I/O 증가)

- **"고객/테넌트 단위로 자연스러운 분리"가 가능할 때**

- **비용/복잡도를 감수할 만큼 규모가 커졌을 때**

## 5 Replication + Sharding 함께 쓰기
### 5.1 실전 아키텍처
- **Shard + Replica 구조**

  <img width="700" alt="image" src="https://github.com/user-attachments/assets/8d12d157-303d-46ef-b06e-60a2d552460c" />

  


### 5.2 장애 상황별 동작

### 5.3 트랜잭셕놔 일관성

## 6 실제 DB 시스템에서의 적용
### 6.1 MySQL

### 6.2 MongoDB

### 6.3 Redis


