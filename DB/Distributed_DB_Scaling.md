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

### 1.2 Scale-up vs Scale-out

## 2 Replication (데이터 복제)
### 2.1 Replication의 개념

### 2.2 Replication의 목적

### 2.3 Replication 방식

### 2.4 Replication의 한계

## 3 Sharding (데이터 분할)
### 3.1 Sharding의 개념

### 3.2 Sharding의 목적

### 3.3 Sharding 전략

### 3.4 Sharding의 난이도

## 4 Replication vs Sharding 비교
### 4.1 해결하는 문제의 찿이

### 4.2 언제 Replication만으로 충분한가

### 4.3 언제 Sharding이 필요한가

## 5 Replication + Sharding 함께 쓰기
### 5.1 실전 아키텍처

### 5.2 장애 상황별 동작

### 5.3 트랜잭셕놔 일관성

## 6 실제 DB 시스템에서의 적용
### 6.1 MySQL

### 6.2 MongoDB

### 6.3 Redis


