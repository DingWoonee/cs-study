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

## 3 Load Balancer 분류

## 4 Load Balancing 알고리즘

## 5 Health Check & 장애 대응

## 6 세션 처리와 Sticky Session

## 7 Load Balancer 아키텍처

## 8 클라우드 환경의 Load Balancer

## 9 성능과 운영 관점

