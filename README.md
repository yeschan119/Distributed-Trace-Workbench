# Distributed-Trace-Workbench
### Real-Time User Activity & Distributed Performance Monitoring

[한국어 🇰🇷](README.Ko.md)

Distributed-Trace-Workbench is a custom observability platform built using the .NET OpenTelemetry SDK.

It unifies real-time user activity logging and distributed performance tracing into a single telemetry pipeline. The system instruments ASP.NET Core applications, generates hierarchical spans (Controller → API → SQL), exports structured trace data via a custom exporter, and persists it in DynamoDB.

The platform provides an Angular-based Workbench dashboard for activity tracking, performance benchmarking, and trace visualization — functioning as an internal APM alternative.

---

## 📌 Project Overview

This system captures and models:

1. User Activity Logs (CRUD operations)
2. System Performance Traces (Controller → API → SQL)
3. Distributed Trace Trees (Parent / Child Span relationships)

All telemetry is instrumented via OpenTelemetry in ASP.NET Core, processed through a custom exporter, and stored in DynamoDB using a TraceId/SpanId data model.

---

## 🏗 Architecture Overview

Client (Angular Workbench UI)  
        │  
        ▼  
ASP.NET Core API (OpenTelemetry Instrumented)  
        │  
        ▼  
Custom Workbench Exporter  
        │  
        ▼  
DynamoDB (Trace Storage)  
        │  
        ▼  
ASP.NET Core Read API  
        │  
        ▼  
Angular Dashboard & Trace Tree UI  

---

## 🔍 Execution Flow

### 1️⃣ Incoming HTTP Request
- ASP.NET Core Controller receives request.

### 2️⃣ OpenTelemetry Instrumentation
- AspNetCore → Server Span
- HttpClient → Client Span
- SqlClient → DB Span

### 3️⃣ Distributed Trace Creation
- TraceId / SpanId generated
- Parent / Child span hierarchy constructed

### 4️⃣ Custom Exporter
- Span converted into structured JSON
- Batch export to DynamoDB

### 5️⃣ DynamoDB Storage
- PK: TraceId
- SK: SpanId
- TTL: 7 Days
- Mode: On-Demand

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

The TraceId/SpanId structure enables full reconstruction of distributed span trees.

---

## 📊 Workbench Features

### 🧾 Activity Log Dashboard
- User information
- CRUD activity tracking
- Timestamp-based filtering

### 📈 Workbench Dashboard
- Top 10 Slowest Controllers
- Top 10 Fastest Controllers
- Top 10 APIs
- Top 10 SQL Queries

### 🌳 Distributed Trace Tree
Visual representation of span hierarchy:

Controller  
  └── API  
        └── SQL  

TraceId-based tree reconstruction using ParentSpanId relationships.

### 🧠 SQL Modal
- Pretty-formatted SQL
- Copy-to-clipboard support
- Execution time display

---

## ⚙️ Technical Highlights

### Unified Observability Model
User activity logging and performance tracing handled through a single OpenTelemetry pipeline.

### Custom OpenTelemetry Exporter
- Converts spans into structured JSON
- Batch writes to DynamoDB
- Supports TTL-based automatic expiration

### Distributed Trace Modeling
- PK = TraceId
- SK = SpanId
- ParentSpanId for hierarchical reconstruction

### Internal APM Design
Designed as a lightweight internal observability system without relying on external SaaS monitoring tools.

---

## 🛠 Tech Stack

Backend:
- ASP.NET Core
- .NET OpenTelemetry SDK
- Custom Span Exporter

Frontend:
- Angular
- Tree View UI
- Dashboard Components

Database:
- Amazon DynamoDB (On-Demand + TTL)

Cloud:
- AWS

---

## 📌 Project Summary

Distributed-Trace-Workbench is a full-stack observability system that captures, models, stores, and visualizes distributed traces using .NET OpenTelemetry and DynamoDB.

It integrates user activity logging and performance monitoring into a unified trace model, reconstructs span hierarchies via TraceId/SpanId relationships, and exposes analytical dashboards for benchmarking and trace inspection.

The project demonstrates practical implementation of distributed tracing, custom telemetry exporting, DynamoDB-based trace persistence, and internal APM system design without third-party monitoring platforms.
