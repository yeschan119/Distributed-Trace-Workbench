# Distributed-Trace-Workbench
Real-Time User Activity &amp; Distributed Performance Monitoring

[한국어 🇰🇷](README.Ko.md)

Built a full-stack observability platform using .NET OpenTelemetry SDK to:

- Capture real-time user activity logs
- Collect distributed performance traces
- Store structured trace data in DynamoDB
- Visualize system performance in a custom Workbench dashboard (Datadog-style)

This project combines user activity tracking and performance tracing into a unified observability system.

---

## 📌 Project Overview

This system captures:

1. User Activity Logs (CRUD actions)
2. System Performance Traces (Controller → API → SQL)
3. Distributed Trace Tree (Parent/Child Span relationships)

All telemetry data is instrumented using OpenTelemetry in ASP.NET Core,
exported via a custom exporter, and stored in DynamoDB.

---

## 🏗 Architecture Overview

# Angular Dashboard & Trace Tree UI
---

## 🔍 Execution Flow

### 1️⃣ Incoming HTTP Request
- ASP.NET Core Controller receives request

### 2️⃣ OpenTelemetry Instrumentation
- AspNetCore → Server Span
- HttpClient → Client Span
- SqlClient → DB Span

### 3️⃣ Distributed Trace Creation
- TraceId / SpanId generated
- Parent / Child Span tree constructed

### 4️⃣ Custom Exporter
- WorkbenchLambdaExporter converts Span → JSON
- Batched export to DynamoDB

### 5️⃣ DynamoDB Storage
- PK: TraceId
- SK: SpanId
- TTL: 7 Days
- Mode: On-Demand

---

## 🗄 DynamoDB Schema

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

---

## 📊 Workbench Features

### 🧾 Activity Log Dashboard
- User information
- CRUD action logs
- Timestamp tracking

### 📈 Benchmark Dashboard
- Top 10 Slowest Controllers
- Top 10 Fastest Controllers
- Top 10 APIs
- Top 10 SQL Queries

### 🌳 Distributed Trace Tree
Visualize:

Controller
  └── API
        └── SQL

Parent-child span hierarchy using TraceId.

### 🧠 SQL Modal
- Pretty formatted SQL
- Copy-to-clipboard support

---

## ⚙️ Technical Highlights

### Unified Observability Model
User activity and performance tracing handled using a single OpenTelemetry pipeline.

### Custom OpenTelemetry Exporter
Implemented custom exporter to:
- Convert spans into structured JSON
- Batch write to DynamoDB
- Support TTL-based auto expiration

### Distributed Trace Modeling
Stored span tree using:
- PK = TraceId
- SK = SpanId
- ParentSpanId for hierarchy reconstruction

### Real-Time Performance Benchmarking
Aggregated trace data to generate Top10 fastest/slowest endpoints.

---

## 🚀 Why This Matters

Instead of using Datadog or external APM tools, this project:

- Implements a custom observability system
- Stores trace data in DynamoDB
- Provides a fully customized monitoring dashboard
- Enables cost-efficient internal monitoring
- Demonstrates deep understanding of OpenTelemetry and distributed tracing

---

## 🛠 Tech Stack

Backend:
- ASP.NET Core
- .NET OpenTelemetry SDK
- Custom Exporter

Frontend:
- Angular
- Tree View UI
- Dashboard Components

Database:
- Amazon DynamoDB (On-Demand + TTL)

Cloud:
- AWS

---

## 🎯 Resume Summary

Built a custom observability platform using .NET OpenTelemetry SDK.

- Captured real-time user activity and distributed performance traces
- Implemented custom span exporter to DynamoDB
- Designed trace tree reconstruction model
- Developed Angular Workbench dashboard with Top10 benchmarking
- Created internal Datadog-style monitoring system
