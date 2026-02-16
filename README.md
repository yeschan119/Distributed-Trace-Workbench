# APM-Dashboard
### Internal Application Performance Monitoring Platform

[한국어 🇰🇷](README.Ko.md)

APM-Dashboard is a custom-built internal Application Performance Monitoring (APM) system implemented using the .NET OpenTelemetry SDK.

The platform instruments ASP.NET Core applications to capture distributed traces across Controller → API → SQL layers, exports structured telemetry data via a custom exporter, and persists it in DynamoDB. An Angular-based dashboard provides performance benchmarking, trace inspection, and activity analysis.

This system functions as a lightweight internal alternative to external APM tools.

---

## 📌 Project Overview

The APM Dashboard captures and models:

1. Application Performance Traces (Controller → API → SQL)
2. Distributed Trace Trees (Parent / Child Span relationships)
3. User Activity Logs (CRUD operations)
4. Performance Benchmark Metrics (Top10 Slowest / Fastest Endpoints)

All telemetry data is instrumented via OpenTelemetry in ASP.NET Core, processed through a custom exporter, and stored using a TraceId / SpanId model in DynamoDB.

---

## 🏗 Architecture Overview

Client (Angular APM Dashboard)  
        │  
        ▼  
ASP.NET Core API (OpenTelemetry Instrumented)  
        │  
        ▼  
Custom Telemetry Exporter  
        │  
        ▼  
DynamoDB (Trace Storage)  
        │  
        ▼  
ASP.NET Core Read API  
        │  
        ▼  
Angular Dashboard (APM Console)  

---

## 🔍 Execution Flow

### 1️⃣ HTTP Request Instrumentation
- ASP.NET Core Controller receives request.
- OpenTelemetry automatically creates a Server Span.

### 2️⃣ Distributed Span Generation
- AspNetCore → Server Span
- HttpClient → Client Span
- SqlClient → DB Span
- TraceId / SpanId assigned
- Parent / Child hierarchy constructed

### 3️⃣ Telemetry Export
- Custom Exporter converts Span into structured JSON.
- Batched export to DynamoDB.

### 4️⃣ Trace Persistence
- PK: TraceId
- SK: SpanId
- TTL: 7 Days
- On-Demand mode for scalable write throughput.

---

## 🗄 DynamoDB Data Model

| Field | Description |
|--------|-------------|
| PK | TraceId |
| SK | SpanId |
| Type | controller / api / sql |
| Controller | Controller Name |
| Method | HTTP Method |
| HttpUrl | API URL |
| DbStatement | SQL Query |
| DurationMs | Execution Time |
| ParentSpanId | Span Hierarchy |
| ExpireAt | TTL (7 Days) |

The TraceId/SpanId schema enables full reconstruction of distributed trace trees.

---

## 📊 APM Dashboard Features

### 📈 Performance Benchmarking
- Top 10 Slowest Controllers
- Top 10 Fastest Controllers
- Top 10 APIs
- Top 10 SQL Queries
- Execution time ranking

### 🌳 Distributed Trace Viewer
Hierarchical visualization:

Controller  
  └── API  
        └── SQL  

TraceId-based span reconstruction using ParentSpanId relationships.

### 🧾 User Activity Monitoring
- CRUD action tracking
- User-based filtering
- Timestamp-based inspection

### 🧠 SQL Inspection Panel
- Pretty-formatted SQL
- Execution time display
- Copy-to-clipboard support

---

## ⚙️ Technical Highlights

### OpenTelemetry-Based APM Implementation
Implemented distributed tracing using .NET OpenTelemetry SDK.

### Custom Telemetry Exporter
- Span → Structured JSON transformation
- Batch writes to DynamoDB
- TTL-based automatic cleanup

### Distributed Trace Modeling
- TraceId as partition key
- SpanId as sort key
- ParentSpanId for hierarchical reconstruction

### Internal APM System Design
Designed as a lightweight internal performance monitoring solution without relying on external SaaS monitoring platforms.

---

## 🛠 Tech Stack

Backend:
- ASP.NET Core
- .NET OpenTelemetry SDK
- Custom Span Exporter

Frontend:
- Angular
- Dashboard Components
- Trace Tree Viewer

Database:
- Amazon DynamoDB (On-Demand + TTL)

Cloud:
- AWS

---

## 📌 Project Summary

APM-Dashboard is an internal Application Performance Monitoring system that captures, models, stores, and visualizes distributed traces using .NET OpenTelemetry and DynamoDB.

The platform integrates performance tracing and user activity monitoring into a unified telemetry model, reconstructs span hierarchies via TraceId/SpanId relationships, and exposes analytical dashboards for performance benchmarking and trace inspection.

This project demonstrates practical implementation of distributed tracing, custom telemetry exporting, DynamoDB-based trace persistence, and internal APM architecture design without third-party monitoring services.
