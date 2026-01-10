# Connection Pool
### 📚 목차
- [1 Connection Pool의 필요성](#1-connection-pool의-필요성)
- [2 Connection Pool의 기본 개념](#2-connection-pool의-기본-개념)
- [3 Connection Pool 내부 구조](#3-connection-pool-내부-구조)
- [4 Pool Size는 어떻게 결정되는가](#4-pool-size는-어떻게-결정되는가)
- [5 Connection Pool과 성능 병목](#5-connection-pool과-성능-병목)

## 1 Connection Pool의 필요성
### 1.1 DB Connection의 실제 비용

### 1.2 Connection 없는 구조의 한계

## 2 Connection Pool의 기본 개념
### 2.1 Connection Pool이란?

### 2.2 기본 동작 흐름

### 2.3 Pool vs DB Thread Pool

## 3 Connection Pool 내부 구조
### 3.1 핵심 구성 요소

### 3.2 대여 알고리즘

### 3.3 Pool의 상태 머신

## 4 Pool Size는 어떻게 결정되는가
### 4.1 DB가 처리할 수 있는 동시 쿼리 수

### 4.2 서버 쓰레드와의 관계

### 4.3 공식 계산서

## 5 Connection Pool과 성능 병목
### 5.1 Pool이 병목이 되는 순간

### 5.2 증상

### 5.3 Thread Pool vs Connection Pool Deadlock
