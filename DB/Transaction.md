# Transaction(트랜잭션)
### 📚 목차
- [1 트랜잭션(Transaction) 개념](#1-트랜잭션transaction-개념)
- [2 ACID 원칙](#2-acid-원칙)
- [3 Commit / Rollback](#3-commit--rollback)
- [4 트랜잭션 상태 변경](#4-트랜잭션-상태-변경)
- [5 격리 수준(Isolation Level)](#5-격리-수준isolation-level)
- [6 Lock과 Deadlock](#6-lock과-deadlock)
- [7 DBMS의 트랜잭션 로그 구조 (Undo / Redo)](#7-dbms의-트랜잭션-로그-구조-undo--redo)

## 1 트랜잭션(Transaction) 개념
### 1.1 트랜잭션의 정의
  
  DB에서 하나의 논리적 작업 단위를 구성하는 일련의 연산 집합을 **트랜잭션**이라고 한다. → 트랜잭션 내의 모든 연산은 전부 성공하거나, 전부 실패한다.
    
### 1.2 트랜잭션이 필요한 이유
    
시스템 장애나 여러 사용자의 동시 접근에도 데이터의 일관성과 무결성을 보장하기 위해 사용한다.
    
### 1.3 트랜잭션의 생명주기
    
`BEGIN / START` - 트랜잭션 시작

`DML 수행` - INSERT, UPDATE, DELETE 등

`COMMIT / ROLLBACK` - 성공 시 영속화, 실패 시 복구

`END` - 종료
    
### 1.4 트랜잭션 경계 설정
    
“어디에서 시작하고 어디서 끝나는가”를 명확히 지정해야 한다.

Spring에서는 `@Transactional`로 경계를 지정한다.
    
### 1.5 트랜잭션 단위 예시
    
“계좌 이체”  
→ 출금 UPDATE와 입금 UPDATE  
→ 둘 다 성공해야만 전체 트랜잭션이 성공한다.
    
### 1.6 Auto-commit
    
기본적으로 DB는 명령어마다 자동으로 반영되는 `auto-commit` 기능이 켜져있다.  
→ 트랜잭션을 제어하려면 이 `auto-commit`을 끄고 명시적으로 `COMMIT`을 수행해야 한다.
    
### 1.7 다중 트랜잭션 환경에서의 문제점
- **Lost Update**
    
    두 트랜잭션이 같은 데이터를 읽고 각각 수정한 뒤 저장할 때, 한 쪽의 수정 결과가 덮어써져 먼저 수정한 내용이 사라지는 것을 말한다.  
    (의도하지 않게 갱신한 값이 사라질 수 있다.)
    
- **Dirty Read**
    
    하나의 트랜잭션이 아직 커밋되지 않은 변경 내용을 다른 트랜잭션이 읽는 경우 이전 트랜잭션이 롤백되는 경우에 문제가 생길 수 있다.
    
- **Non-repeatable Read**
    
    같은 트랜잭션 내에서 같은 데이터를 두 번 읽었는데 값이 다르게 나오는 경우를 말한다.  
    → 다른 트랜잭션이 중간에 데이터를 수정하거나 커밋한 경우 발생한다.
    
- **Phantom Read**
    
    같은 조건으로 조회했는데 중간에 삽입된 행 때문에 결과 집합이 달라지는 것을 말한다.
    
- **Deadlock**
    
    두 트랜잭션이 서로가 가진 자원을 기다리며 무한 대기 상태에 빠지는 것을 말한다.
    
- **Non-repeatable Read와 Phantom Read의 차이**
    
    Non-repeatable Read → 같은 행을 읽었는데 그 행의 값(내용)이 달라짐
    
    Phantom Read → 같은 조건으로 읽었는데 결과의 집합의 행들이 달라짐 (새로운 행이 유령처럼 나타남)

## 2 ACID 원칙
트랜잭션이 안전하게 수행되기 위해 가져야 할 4가지 속성을 말한다.

### 2.1 Atomicity (원자성)
    
트랜잭션의 모든 연산은 전부 수행되거나 전혀 수행되지 않아야 한다.  
ex) 계좌 이체 시 출금과 입금 잔액 변경이 모두 성공적으로 반영되어야 한다.
    
### 2.2 Consistency (일관성)
    
트랜잭션 전후로 데이터의 일관성이 유지되어야 한다.  
→ **무결성 제약조건**이 항상 유효해야 한다.  
→ 이는 **단순 필드 조건 뿐만 아니라 비즈니스 로직이 보장해야 할 모든 조건을 포함**한다.
    
### 2.3 Isolation (격리성)
    
여러 트랜잭션이 동시에 실행될 때 서로의 중간 상태를 볼 수 없어야 한다.  
→ 다른 트랜잭션의 미완료 데이터 접근 금지

⇒ 이것은 Isolation의 최소 수준이고, 이것을 넘어서 `Serializable Isolation` 수준까지 지원하는 것이 이상적이다.)  
→ **성능적인 이유로 이 특성은 가장 유연성** 있는 제약 조건이다.
    
### 2.4 Durability (지속성)
    
`COMMIT`된 데이터는 장애가 발생해도 영구히 보존되어야 한다.  
→ 이미 커밋된 트랜잭션은 어떤 상황에도 유지되어야 한다.  
→ 전원이 꺼져도 WAL(로그) 기반 복구로 유지

## 3 Commit / Rollback
### 3.1 Commit의 의미
    
트랜잭션 내에서 수행된 모든 변경 사항을 영구적으로 DB에 반영하는 명령이다.  
→ **트랜잭션의 성공적인 종료 지점**을 의미.

→ **`DML`이 이에 해당한다. `DDL`은 DB 구조를 바꾸기 때문에 트랜잭션에 상관없이 바로 자동 커밋**된다.

- **효과**
    - 변경 사항이 디스크에 기록되어 다른 트랜잭션에서도 조회가 가능함.  
    - Undo 로그(복구 로그)는 더 이상 필요하지 않음  
    → Undo를 통한 복구 불가능(Undo는 트랜잭션 커밋 이전의 시점에만 사용 가능)  
    - 하나의 논리적 작업 단위가 완료됨
- **명령**
    
    ```sql
    BEGIN; -- 또는 START TRANSACTION
    -- 데이터 변경 명령어들
    COMMIT;
    ```
        
### 3.2 Rollback의 의미
    
트랜잭션 내에서 이루어진 모든 변경 사항(`DML`)을 원래 상태로 되돌리는 명령이다. (트랜잭션 실패나 예외 발생 시 수행됨)

- **효과**
    - Undo 로그를 이용해 변경 전 상태로 복구
    - 해당 트랜잭션에서 수행된 모든 DML 취소
    - 다른 트랜잭션에는 아무 영향도 주지 않음(격리성 보장)
- **명령**
    
    ```sql
    BEGIN; -- 또는 START TRANSACTION
    -- 명령어들
    ROLLBACK;
    ```
        
### 3.3 Partial Rollback
    
트랜잭션 전체를 되돌리는 대신, **특정 지점까지만 복구**하는 것.  
→ 중간에 오류가 발생해도 트랜잭션 전체를 취소하지 않고, 일부만 되돌린다.

- **방법**
    
    `SAVEPOINT`를 설정한 후에, 해당 지점까지 `ROLLBACK TO SAVEPOINT`로 복구한다.
    
- **명령**
    
    ```sql
    BEGIN;
    UPDATE A SET x = 1;
    SAVEPOINT s1;
    UPDATE B SET y = 2;
    ROLLBACK TO s1; -- B 변경만 취소되고, A 변경은 유지됨
    COMMIT;
    ```
        
### 3.4 Implicit Commit (암묵적 커밋)
    
사용자가 명시적으로 `COMMIT`을 호출하지 않아도 **DB가 자동으로 COMMIT을 수행**하는 상황이다.

- **발생 상황**
    - 모든 `DDL` 실행 시 (`CREATE`, `ALTER`, `DROP` 등)
    - 대부분의 `DCL` 실행 시 (`GRANT`, `REVOKE` 등)
    - 정상 종료 시 (`EXIT`, `DISCONNECT`)
    - Auto-commit 모드가 활성화된 경우
- **주의점**
    
    DDL 실행 시 자동으로 COMMIT 되기 때문에, 이전 트랜잭션의 원자성이 깨질 수 있다.  
    → **실무에서는 DDL을 트랜잭션 블록 내에서 섞어 쓰지 않는 것이 원칙**이다.
        
### 3.5 Savepoint를 이용한 중간 복구
    
트랜잭션 내에서 복구용 중간 지점을 미리 지정하는 명령이다.  
→ 이후 `ROLLBACK TO` 명령어를 통해 트랜잭션 안에서 특정 시점으로 이동(롤백)할 수 있다.

- **효과**
    - 하나의 트랜잭션에 대해 세밀한 제어 가능
    - 복잡한 로직에서 부분 복구를 통해 성능과 유연성을 높인다.
- **예시**
    
    ```sql
    BEGIN;
    UPDATE users SET age = 30 WHERE id = 1;
    SAVEPOINT s1;
    UPDATE users SET age = 'NaN';
    ROLLBACK TO s1;
    COMMIT;
    ```
        
### 3.6 트랜잭션에서 에러 발생 시
    
SQL 실행 자체가 실패한다.  
→ 그리고 DBMS에 따라 이후 SQL문이 수행될 수도 있고, 수행되지 않을 수도 있다.  
⇒ 다른 SQL문이 수행되지 않는 경우는 `ROLLBACK`만 받아들여진다.  
⇒ 다른 SQL문이 수행되는 경우는 오류 문장만 실패하는 것이다.
    
### 3.7 트랜잭션 경계 설정 방식
    
트랜잭션이 “언제 시작하고 언제 끝나는가”를 명확히 지정하는 것.

**명시적 방식**  
→ 개발자가 직접 `TCL` 입력 (JDBC, SQL 스크립트)

**프로그래밍 방식**  
→ 코드 상에서 API로 트랜잭션 제어 (`conn.setAutoCommit(false)`)

**선언적 방식**  
→ 프레임워크가 자동으로 관리 (`@Transactional`)

## 4 트랜잭션 상태 변경
### 4.1 상태
- **`Active` - 활성**
    
    트랜잭션이 시작되어서 SQL 문을 실행 중인 상태.  
    (`COMMIT`이나 `ROLLBACK` 수행 전)
    
- **`Partially Commited` - 부분 커밋**
    
    마지막 SQL 문이 성공적으로 실행되어, 이제 **`COMMIT`만 남은 상태.**  
    → 여기서 장애 발생 시, 트랜잭션 복구 과정에 따라 `COMMIT`/`ROLLBACK`이 결정된다.
    
- **`Committed` - 커밋 완료**
    
    `COMMIT` 명령이 성공적으로 실행되어 변경 내용이 영구 저장됨.  
    → 트랜잭션 종료
    
- **`Failed` - 실패**
    
    SQL 문 중 실행 중 오류 발생(무결성 위반, 형변환 오류 등)으로 트랜잭션이 더 이상 정상 진행 불가능한 상태
    
    - **DB별 특징**
        
        **MySQL이나 Oracle**은 SQL 실행 오류가 발생해도 이어서 실행이 가능하다.  
        → 실패해도 `Active`
        
        PostgreSQL은 SQL 실행 오류가 발생하면 바로 해당 트랜잭션은 `ABORTED` 상태가 된다.
        
- **`Aborted` - 중단/롤백 완료**
    
    `ROLLBACK`을 통해 모든 변경이 취소되고, 트랜잭션이 종료된 상태.  
    → 복구 완료 후 다시 새 트랜잭션 시작 가능
        
### 4.2 상태 전이 트리거
- `Active` → `Failed`
    
    SQL 오류, 무결성 위반, 예외 발생 (Mysql이나 Oracle은 해당 안됨)
    
- `Active` → `Partially Committed`
    
    마지막 SQL 성공 후 `COMMITED` 준비
    
- `Partially Committed` → `Committed`
    
    `COMMIT` 성공 (로그 기록 및 반영 완료)
    
- `Failed` → `Aborted`
    
    `ROLLBACK` 실행 (자동 또는 명시적)
    
- `Active` → `Aborted`
    
    사용자가 직접 `ROLLBACK` 호출
        
### 4.3 `COMMITTED`가 디스크에 변경 사항이 반영되었다는 의미인가?
    
`COMMITTED`는 논리적으로 트랜잭션이 성공적으로 끝났음을 보장하는 상태이다.  
→ 반드시 디스크에 데이터 파일로 써졌다를 의미하지는 않고, 로그 등으로 복구 가능한 형태로 안전하게 저장되었다는 의미이다.

## 5 격리 수준(Isolation Level)
### 5.1 격리 수준 개요 (성능 vs 일관성의 트레이드오프)
    
**격리(Isolation)**는 여러 트랜잭션이 동시에 실행될 때 서로 간섭하지 않도록 하는 속성(ACID 중 I)이다.

완벽한 격리를 보장하려면 **직렬화(Serializability)** 수준이 필요하지만,  
→ **성능이 급격히 저하**된다.  
⇒ DBMS는 상황에 따라 트랜잭션에서의 데이터 조회 일관성을 조금 양보하고 성능을 높이는 다양한 수준의 격리 단계를 제공한다.
    
### 5.2 ANSI SQL 표준 4단계
- **`READ UNCOMMITED`**
    
    아직 `COMMIT`되지 않은 변경 내용도 읽을 수 있다.  
    → Dirty Read, Non-repeatable Read, Phantom Read, Lost Update 발생 허용
    
    - **발생 가능한 이상 현상**
        
        Dirty Read, Non-repeatable Read, Phantom Read, Lost Update
        
    - **특징**
        
        성능은 최고(락이 없음) → But, 실제로는 거의 사용 못한다.
        
- **`READ COMMITTED`**
    
    `COMMIT`된 데이터만 읽을 수 있다. (가장 일반적이다.)  
    → Non-repeatable Read, Phantom Read, Lost Update 발생 허용
    
    - **발생 가능한 이상 현상**
        
        Non-repeatable Read, Phantom Read, Lost Update
        
    - **특징**
        
        Dirty Read 방지
        
        대부분의 DBMS의 기본 수준(읽기 일관성과 성능 균형이 좋음)
        
        `SELECT` 시점마다 **다시 최신 커밋된 버전**을 읽는다.  
        → 스냅샷이 계속 바뀐다는 말.  
        → 당연히 `Non-repeatable Read가 발생할 수 있음
        
- **`REPEATABLE READ`**
    
    트랜잭션 동안 같은 행을 반복 조회해도 동일한 결과를 보장한다.  
    → Phantom Read 허용 (DBMS에 따라 해결됨)
    
    - **`REPEATABLE READ`가 어떻게 Lost Update를 해결?**
        
        `REPEATABLE READ`는 데이터를 **쓸 때 그 행에 대한 배타적 잠금을 획득**한다.  
        (ANSI 표준에 명시된 것은 아니고, 보통 DBMS 구현상 이것까지 보장된다.)
        
    - **발생 가능한 이상 현상**
        
        Phantom Read
        
    - **특징**
        
        **트랜잭션 시작 시점의 스냅샷을 고정**한다.  
        → 이후 `SELECT`는 항상 그 시점 기준으로 읽는다.  
        (새로운 행이 추가되는 것은 감지하지 못해서 Phantom이 발생할 수 있다.)
      
        ⇒ 추가적으로 관리해야 할 것들이 생기기 때문에(MySQL은 이를 보장하려면 버전 관리가 추가됨) 성능이 저하된다.
        
- **`SERIALIZABLE`**
    
    모든 트랜잭션이 순차적으로 실행된 것처럼 보이게 한다.  
    → 허용되는 이상 현상 없음.
    
    - **발생 가능한 이상 현상**
        
        없음
        
    - **특징**
        
        완벽한 일관성을 보장한다. ACID를 완전 충족한다.
        
        동시성이 급격히 저하된다.  
        (`SELECT`문에서도 공유락이 걸리는 등 락 경합이 많이 발생하고, 데드락 가능성이 증가한다.)  
        → 대부분은 `REPEATABLE READ`로 충분히 안전하게 설계한다.
            
### 5.3 DBMS별 기본 격리 수준
- MySQL → `REPEATABLE READ`
- PostgreSQL → `READ COMMITTED`
- Oracle → `READ COMMITTED`
- SQL Server → `READ COMMITTED`
- 
### 5.4 MVCC와 Isolation Level의 관계
- **MVCC (Multi-Version Concurrency Control)**
    
    동시에 여러 트랜잭션이 접근해도 각 트랜잭션이 자신만의 데이터 버전(snapshot)을 읽게 하는 방식이다.  
    → 읽기와 쓰기를 완전히 분리하여 Lock 충돌을 줄이는 동시성 제어 기술이다.
    
    > “**스냅샷**”은 트랜잭션 단위로, “**버전**”은 행 단위로 관리된다.
    
    
    ”**스냅샷**”  
    → 트랜잭션이 “어떤 버전들의 행을 볼 수 있는가”를 정의하는 시점  
    ”**버전**”  
    → 각 행의 과거/현재 상태를 나타내는 실제 데이터  
    → 버전은 데이터 수정 시마다 생성되고, 각 행의 버전에 링크드 리스트처럼 다음 버전으로 이어지게 된다.
    
    - **트랜잭션이 데이터를 읽을 때**
        
        가장 최신 버전부터 시작해서, “이 버전을 생성한 트랜잭션이 내 스냅샷 시점에 이미 커밋된 상태인가?”를 확인한다.  
        또한 “이 버전이 내 스냅샷 이후에 삭제된 버전이 아닌가?”도 확인한다.  
        → 그리고 조건에 만족하는 해당 행의 첫 번째(최신) 버전을 반환한다.
        
    - **트랜잭션이 데이터를 쓸 때**
        
        항상 새 버전이 추가된다.
        
        데이터를 덮어쓰지 않는다.
        
    - **Phantom Read가 발생하는 이유**
        
        DBMS가 “**트랜잭션 도중에 생긴 새로운 행을 스냅샷의 가시 범위에 포함시킬지 말지**”에 따라 달라진다.  
        → DBMS의 구현과 격리 수준 설정에 따라 다르다.
        
- **Undo 로그를 통한 버전 관리**
    
    MVCC는 데이터를 수정할 때 이전 버전을 Undo 로그(또는 Rollback Segment)에 보관한다.  
    → 읽는 트랜잭션은 Undo 로그를 참조하여 자신의 스냅샷 시점에 맞는 버전을 읽는다.
    
    `UPDATE`/`DELETE` → Undo 로그에 이전 버전 기록
    
    `SELECT` → Undo 로그에서 스냅샷 시점의 데이터 복원

## 6 Lock과 Deadlock
### 6.1 Lock 개념
Lock은 여러 트랜잭션이 동시에 같은 데이터를 접근할 때 데이터의 일관성과 무결성을 보장하기 위한 동기화 메커니즘이다.

- **Shared Lock (S-Lock, 읽기 잠금)**  
    데이터를 읽을 때 설정한다. 다른 트랜잭션도 동시에 읽을 수는 있지만, 쓰기는 불가능하다.
    
- **Exclusive Lock (X-Lock, 쓰기 잠금)**  
    데이터를 변경할 때 설정한다. 다른 트랜잭션은 읽기와 쓰기가 모두 불가능하다.
    
- **Intent Lock, Record Lock, Gap Lock (InnoDB)**
  
### 6.2 Lock Granularity (잠금 단위)
- **Table-level**
- **Page-level**
- **Row-level**
  
### 6.3 락 경합 Concurrency Issue)
- **Deadlock**
- **Deadlock 발생 조건**
    
    아래 4가지 조건이 모두 성립하면 Deadlock 발생.
    
    - 상호 배제(Mutual Exclusion)
    - 점유 대기(Hold and Wait)
    - 비선점(No Preemption)
    - 순환 대기(Circular Wait)
      
- **Deadlock 탐지 / 회피 / 해결 전략**
    - 탐지(detection)  
        DB 엔진이 Lock Wait Graph를 분석하여 **순환 구조를 탐지**한다.
        
    - 회피(avoidance)  
        락 요청 순서를 미리 정하거나, **타임아웃**을 설정한다.
        
    - 해결(resolution)  
        Deadlock 발생 시에, 비용이 적은 트랜잭션을 희생(victim)시켜 롤백한다.  
        (탐지 단계에서 탐지한 Deadlock을 강제로 `ROLLBACK` 시킨다.)
        
- **Starvation(기아 상태)**
    
    일부 트랜잭션이 계속해서 Lock을 얻지 못하고 무한히 기다리는 상태를 말한다.  
    (높은 우선순위 트랜잭션이 계속 들어오는 경우)
        
### 6.4 Lock Escalation (락 승격)
    
너무 많은 Row-level Lock이 걸릴 경우, 시스템 오버헤드를 줄이기 위해 상위 단위(Table-level Lock)로 자동 전환하는 기능이다.  
(DBMS에 의해 상위 잠금 단위(Lock Granularity)로 **자동 승격**된다.)

MySQL InnoDB의 경우 락 승격을 하지 않는다.
MySQL에서 락은 항상 필요한 단위(row/page/table)로 별도로 관리된다.

## 7 DBMS의 트랜잭션 로그 구조 (Undo / Redo)
### 7.1 Undo Log
> **데이터 변경 이전(before image)** 을 저장하는 로그로,  
> **Rollback(복구)** 과 **MVCC(스냅샷 읽기)** 를 위해 사용된다.

⇒ Undo Log는 단순히 "되돌리기" 기능만이 아니라, **트랜잭션 격리성(Isolation)** 을 구현하는 핵심이다.

- **Undo Log가 필요한 이유**  
  1. **복구(Rollback) 용도**  
     트랜잭션이 실패하거나 롤백될 때, Undo Log를 역순으로 적용해서 **변경하기 이전 상태로 되돌린다.**
     
  3. **MVCC 용도**  
     InnoDB의 가장 중요한 기능 중 하나로,  
     Undo LOg는 MVCC에서 **과거 버전을 제공**한다.  
     
     ⇒ Undo Log = "트랜잭션이 참조할 수 있는 과저 버전 스냅샷"  
     이를 통해 Non-Repeatable Read나 Dirty Read를 방지한다.

### 7.2 Redo Log
> **데이터 변경 이후(after image)** 를 기록하는 로그로,  
> **장애 복구(Crash Recovery)** 와 **Durability(지속성)** 를 보장하기 위해 사용된다.

⇒ Undo Log가 "과거로 되돌리기"라면, Redo Log는 "미래를 다시 만들어내기"이다.

- **Redo Log가 필요한 이유 (트랜잭션에서 가장 중요한 이유)**  
  1. **Durability(ACID의 D) 보장**  
     COMMIT했으면 서버가 죽어도 데이터는 반드시 살아 있어야 한다.  
     → 하지만, 실제 데이터 페이지는 **즉시 디스크에 쓰는 방식이 아니다.** (성능 문제)
     
     InnoDB는 페이지를 나중에 디스크에 쓴다.  
     → 대신 "변경됐다는 로그(Redo)"를 먼저 안전하게 기록한다.  

     ⇒ **COMMIT = Redo Log Flush 완료**

  2. **Crash Recovery(장애 복구)**  
     서버가 죽은 후 재시작할 때 Redo Log를 읽어서  
     마지막 체크포인트 이후 변경된 내용을 복구한다.  

  ⇒ Undo Log는 "롤백"용, Redo Log는 "재적용(복구)"용.

### 7.3 Undo vs Redo 비교
| 구분 | Undo Log | Redo Log |
|------|----------|-----------|
| 목적 | 과거 상태로 복구(Rollback), MVCC 스냅샷 제공 | 장애 복구(Crash Recovery), Durability 보장 |
| 저장 내용 | 변경 전(before image) 데이터 | 변경 후(after image) 데이터 |
| 사용 시점 | ROLLBACK, SELECT(MVCC) | COMMIT, 재시작 시 복구 |
| 기록 시점 | DML 실행 시 즉시 기록 | DML 실행 시 Redo Log Buffer에 기록 |
| COMMIT과의 관계 | COMMIT하면 Undo는 더 이상 MVCC 외에는 필요 없음 | COMMIT은 **Redo Log가 디스크에 flush된 순간** 완성 |
| 데이터 파일 반영 | Undo는 실제 데이터 파일 변경 X | Redo는 Dirty Page가 디스크에 기록될 때 사용 |
| 삭제 시점 | 더 이상 참조할 트랜잭션이 없으면 purge | 체크포인트 후 불필요한 영역 재사용 |

> **Undo = 과거로 돌아가는 로그**  
> **Redo = 미래를 재현하는 로그**

- **Dirty Page란?**  
  메모리(Buffer Pool)에 올라온 페이지가 변경되었지만, **아직 디스크에 반영되지 않은 상태의 페이지**를 말한다.

### 7.4 Checkpoint
> Checkpoint는 InnoDB가 주기적으로 Dirty Page를 디스크에 flush하고,  
> Redo Log의 사용 가능 영역을 확보하는 과정이다.

- **목적**
  - Redo Log가 무한히 커지는 것을 방지
  - 재시작 시 복구 시간을 단축
  - 메모리(Buffer Pool)의 Dirty Page를 디스크에 반영

- **동작 개념**
  - InnoDB는 Buffer Pool의 Dirty Page를 디스크로 flush
  - 해당 페이지의 Redo Log 범위를 "Checkpoint 이전"으로 표시  
    (Redo Log 구간을 checkpoint 이전으로 이동시켜서, 그 Redo Log 공간은 덮어써서 재사용할 수 있게 함)
  - Redo Log의 해당 영역을 재사용 가능하게 만듦

⇒ **Checkpoint = Dirty Page flush + Redo Log 공간 확보 + 복구 시간 단축**

### 7.5 InnoDB 전체 흐름
