# Distributed-Trace-Workbench
### 실시간 사용자 활동 로그 및 분산 성능 관측 플랫폼

[English 🇺🇸](README.md)

Distributed-Trace-Workbench는 .NET OpenTelemetry SDK를 기반으로 구축된 커스텀 관측(Observability) 플랫폼입니다.

이 시스템은 사용자 활동 로그와 분산 성능 트레이싱을 하나의 텔레메트리 파이프라인으로 통합합니다.  
ASP.NET Core 애플리케이션을 자동 계측하여 Controller → API → SQL 계층의 Span을 생성하고,  
커스텀 Exporter를 통해 구조화된 Trace 데이터를 DynamoDB에 저장합니다.

Angular 기반 Workbench 대시보드를 통해 활동 로그 조회, 성능 벤치마킹, 분산 트레이스 시각화를 제공합니다.  
외부 APM 도구 없이 내부 관측 시스템을 직접 설계·구현한 프로젝트입니다.

---

## 📌 프로젝트 개요

본 시스템은 다음을 수집 및 모델링합니다:

1. 사용자 활동 로그 (CRUD 작업)
2. 시스템 성능 트레이스 (Controller → API → SQL)
3. 분산 트레이스 트리 (Parent / Child Span 관계)

모든 텔레메트리는 ASP.NET Core에서 OpenTelemetry로 계측되며,  
커스텀 Exporter를 거쳐 TraceId / SpanId 기반 모델로 DynamoDB에 저장됩니다.

---

## 🏗 아키텍처 개요

Client (Angular Workbench UI)  
        │  
        ▼  
ASP.NET Core API (OpenTelemetry 계측)  
        │  
        ▼  
Custom Workbench Exporter  
        │  
        ▼  
DynamoDB (Trace 저장)  
        │  
        ▼  
ASP.NET Core Read API  
        │  
        ▼  
Angular Dashboard & Trace Tree UI  

---

## 🔍 실행 흐름

### 1️⃣ HTTP 요청 수신
- ASP.NET Core Controller 진입

### 2️⃣ OpenTelemetry 자동 계측
- AspNetCore → Server Span 생성
- HttpClient → Client Span 생성
- SqlClient → DB Span 생성

### 3️⃣ 분산 트레이스 생성
- TraceId / SpanId 생성
- Parent / Child Span 계층 구조 형성

### 4️⃣ Custom Exporter 처리
- Span → 구조화된 JSON 변환
- Batch 방식으로 DynamoDB 저장

### 5️⃣ DynamoDB 저장 구조
- PK: TraceId
- SK: SpanId
- TTL: 7일
- On-Demand 모드

---

## 🗄 DynamoDB 데이터 모델

| 필드 | 설명 |
|------|------|
| PK | TraceId |
| SK | SpanId |
| Type | controller / api / sql |
| Controller | 컨트롤러 이름 |
| Method | HTTP Method |
| HttpUrl | 요청 URL |
| DbStatement | 실행된 SQL |
| DurationMs | 실행 시간 |
| ParentSpanId | 상위 Span |
| ExpireAt | TTL (7일 자동 삭제) |

TraceId / SpanId 구조를 통해 전체 분산 트레이스 트리를 재구성할 수 있습니다.

---

## 📊 Workbench 주요 기능

### 🧾 Activity Log Dashboard
- 사용자 정보 조회
- CRUD 활동 기록 추적
- 시간 기반 필터링

### 📈 Benchmark Dashboard
- Top 10 느린 Controller
- Top 10 빠른 Controller
- Top 10 API
- Top 10 SQL

### 🌳 분산 Trace Tree
Span 계층 구조 시각화:

Controller  
  └── API  
        └── SQL  

ParentSpanId 기반으로 TraceId 단위 트리 재구성

### 🧠 SQL Modal
- SQL Pretty Format
- Copy 버튼 제공
- 실행 시간 표시

---

## ⚙️ 기술적 핵심 요소

### 통합 Observability 모델
사용자 활동 로그와 성능 트레이스를 단일 OpenTelemetry 파이프라인으로 처리.

### Custom OpenTelemetry Exporter 구현
- Span을 구조화된 JSON으로 변환
- DynamoDB Batch Write 처리
- TTL 기반 자동 데이터 정리

### 분산 트레이스 모델링
- PK = TraceId
- SK = SpanId
- ParentSpanId 기반 계층 재구성

### 내부 APM 설계
외부 SaaS 모니터링 도구 없이 자체 관측 시스템 설계 및 구현.

---

## 🛠 기술 스택

Backend:
- ASP.NET Core
- .NET OpenTelemetry SDK
- Custom Span Exporter

Frontend:
- Angular
- Tree View UI
- Dashboard 컴포넌트

Database:
- Amazon DynamoDB (On-Demand + TTL)

Cloud:
- AWS

---

## 📌 프로젝트 요약

Distributed-Trace-Workbench는 .NET OpenTelemetry와 DynamoDB를 활용하여  
분산 트레이스를 수집·모델링·저장·시각화하는 풀스택 관측 시스템입니다.

사용자 활동 로그와 성능 모니터링을 하나의 트레이스 모델로 통합하고,  
TraceId / SpanId 관계를 기반으로 Span 트리를 재구성합니다.  
대시보드를 통해 성능 벤치마킹 및 트레이스 분석 기능을 제공합니다.

본 프로젝트는 분산 트레이싱 구현, 커스텀 텔레메트리 Export 설계,  
DynamoDB 기반 Trace 저장 모델링, 내부 APM 시스템 설계 역량을 보여줍니다.
