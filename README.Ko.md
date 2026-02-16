# APM-Dashboard
### 내부 애플리케이션 성능 모니터링(APM) 플랫폼

[English 🇺🇸](README.md)

APM-Dashboard는 .NET OpenTelemetry SDK를 기반으로 구축한 내부 애플리케이션 성능 모니터링(Application Performance Monitoring) 시스템입니다.

ASP.NET Core 애플리케이션을 계측하여 Controller → API → SQL 계층의 분산 트레이스를 생성하고,  
커스텀 Exporter를 통해 구조화된 텔레메트리 데이터를 DynamoDB에 저장합니다.

Angular 기반 APM 대시보드를 통해 성능 벤치마킹, 트레이스 분석, 사용자 활동 모니터링을 제공합니다.  
외부 SaaS APM 도구 없이 내부 성능 관측 시스템을 직접 설계·구현한 프로젝트입니다.

---

## 프로젝트 개요

본 시스템은 다음을 수집하고 모델링합니다:

1. 애플리케이션 성능 트레이스 (Controller → API → SQL)
2. 분산 트레이스 트리 (Parent / Child Span 관계)
3. 사용자 활동 로그 (CRUD 작업)
4. 성능 벤치마크 지표 (Top10 느린/빠른 엔드포인트)

모든 텔레메트리는 ASP.NET Core에서 OpenTelemetry로 자동 계측되며,  
TraceId / SpanId 기반 데이터 모델로 DynamoDB에 저장됩니다.

---

## 아키텍처 개요

Client (Angular APM Dashboard)  
        │  
        ▼  
ASP.NET Core API (OpenTelemetry 계측)  
        │  
        ▼  
Custom Telemetry Exporter  
        │  
        ▼  
DynamoDB (Trace 저장)  
        │  
        ▼  
ASP.NET Core Read API  
        │  
        ▼  
Angular APM Console  

---

## 실행 흐름

### 1️⃣ HTTP 요청 계측
- ASP.NET Core Controller 진입
- OpenTelemetry가 Server Span 자동 생성

### 2️⃣ 분산 Span 생성
- AspNetCore → Server Span
- HttpClient → Client Span
- SqlClient → DB Span
- TraceId / SpanId 할당
- Parent / Child 계층 구조 구성

### 3️⃣ 텔레메트리 Export
- Custom Exporter가 Span을 JSON으로 변환
- Batch 방식으로 DynamoDB에 저장

### 4️⃣ Trace 영속화
- PK: TraceId
- SK: SpanId
- TTL: 7일
- On-Demand 모드

---

## DynamoDB 데이터 모델

| 필드 | 설명 |
|------|------|
| PK | TraceId |
| SK | SpanId |
| Type | controller / api / sql |
| Controller | 컨트롤러 이름 |
| Method | HTTP Method |
| HttpUrl | 요청 URL |
| DbStatement | 실행된 SQL |
| DurationMs | 실행 시간(ms) |
| ParentSpanId | 상위 Span |
| ExpireAt | TTL (7일 자동 삭제) |

TraceId / SpanId 구조를 통해 전체 분산 트레이스 트리를 재구성할 수 있습니다.

---

## APM 대시보드 주요 기능

### 📈 성능 벤치마킹
- Top 10 느린 Controller
- Top 10 빠른 Controller
- Top 10 API
- Top 10 SQL
- 실행 시간 기반 순위 분석

### 분산 트레이스 뷰어
Span 계층 구조 시각화:

Controller  
  └── API  
        └── SQL  

ParentSpanId 기반 TraceId 단위 트리 재구성

### 사용자 활동 모니터링
- CRUD 작업 추적
- 사용자 기준 필터링
- 시간 기반 조회

### SQL 분석 패널
- SQL Pretty Format
- 실행 시간 표시
- Copy 버튼 제공

---

## ⚙️ 기술적 핵심 요소

### OpenTelemetry 기반 APM 구현
.NET OpenTelemetry SDK를 활용하여 분산 트레이싱 구현.

### Custom Telemetry Exporter 설계
- Span → 구조화된 JSON 변환
- DynamoDB Batch Write 처리
- TTL 기반 자동 데이터 정리

### 분산 트레이스 데이터 모델링
- PK = TraceId
- SK = SpanId
- ParentSpanId 기반 계층 구조 복원

### 내부 APM 아키텍처 설계
외부 모니터링 SaaS에 의존하지 않는 경량 내부 성능 관측 시스템 구현.

### AWS 리소스 메트릭 수집

CPU 사용량 및 메모리 사용량과 같은 인프라 레벨 성능 지표는 AWS SDK for .NET을 활용하여 수집합니다.

- AWS SDK for .NET (CloudWatch)
- 실시간 메트릭 조회
- 애플리케이션 트레이스와 함께 APM 대시보드에 통합

이를 통해 애플리케이션 지연 시간과 인프라 상태를 상관 분석할 수 있습니다.

### Batch 기반 텔레메트리 저장 최적화

DynamoDB 쓰기 성능과 비용 최적화를 위해 Hybrid Batch 전략을 적용하였습니다.

- Batch Size: 20개 Span 단위 저장
- Time-based Flush: Batch가 차지 않은 경우 30초마다 저장
- 비동기 백그라운드 Export 처리

이 설계를 통해:

- DynamoDB Write 비용 감소
- Write Amplification 최소화
- 안정적인 Ingestion 처리량 유지
- 텔레메트리 유실 위험 최소화

를 달성하였습니다.
---

## 🛠 기술 스택

Backend:
- ASP.NET Core
- .NET OpenTelemetry SDK
- Custom Span Exporter

Frontend:
- Angular
- Dashboard 컴포넌트
- Trace Tree Viewer

Database:
- Amazon DynamoDB (On-Demand + TTL)

Cloud:
- AWS

---

## 프로젝트 요약

APM-Dashboard는 .NET OpenTelemetry와 DynamoDB를 기반으로  
분산 트레이스를 수집·모델링·저장·시각화하는 내부 애플리케이션 성능 모니터링 시스템입니다.

성능 트레이싱과 사용자 활동 모니터링을 하나의 텔레메트리 모델로 통합하고,  
TraceId / SpanId 관계를 통해 Span 계층 구조를 복원합니다.  
대시보드를 통해 성능 벤치마킹 및 트레이스 분석 기능을 제공합니다.

본 프로젝트는 분산 트레이싱 구현, 커스텀 텔레메트리 Export 설계,  
DynamoDB 기반 Trace 저장 모델링, 내부 APM 아키텍처 설계 역량을 보여줍니다.
