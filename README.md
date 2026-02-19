# 🚀 DocVault – Day 3 (Events, Messaging & Observability)

**AZ-204 Capstone Project – Day 3**
**Focus:** Event-Driven Architecture & Azure Monitoring

## 🎯 Day 3 Objective
Enhance DocVault with event-driven architecture and production-level monitoring by implementing:
* Azure Event Grid integration
* Domain event publishing after document upload
* Application Insights custom telemetry
* Upload performance tracking
* Azure monitoring & observability validation

---

## 👥 Team Contributions

### Akshay – Backend Events & Telemetry
**Completed:**
* Integrated Azure Event Grid publisher.
* Published `DocVault.DocumentUploaded` event.
* Added custom telemetry using Application Insights.
* Implemented upload duration tracking.
* Implemented custom `DocumentUploaded` event tracking.
* Verified telemetry ingestion in Azure Portal.
* Tested full end-to-end event publishing flow.

### Vaibhav – Azure Messaging & Monitoring Setup
**Completed:**
* Created Azure Event Grid Topic.
* Generated Event Grid access keys.
* Configured Application Insights resource.
* Verified telemetry ingestion.
* Monitored server response time & availability.
* Validated Azure monitoring dashboards.

### Vikrant – Frontend Monitoring Validation
**Completed:**
* Verified upload flow from Angular UI.
* Confirmed successful event publishing.
* Validated Application Insights metrics update.
* Tested monitoring dashboard integration.

---

## 🏗️ Event-Driven Architecture Implementation
After a document is successfully uploaded:
1. File stored in **Azure Blob Storage**.
2. Metadata stored in **Azure Cosmos DB**.
3. Event published to **Azure Event Grid**.

* **Event Type:** `DocVault.DocumentUploaded`
* **Event Subject:** `documents/{documentId}`
* **Implementation Location:** `api/DocVault.Api/Controllers/DocumentsController.cs`

---

## 📊 Application Insights Integration

### 1. Custom Metric – Upload Duration
* **Metric Name:** `DocumentUploadDurationMs`
* **Description:** Tracks upload performance in milliseconds.

### 2. Custom Event – DocumentUploaded
* **Event Name:** `DocumentUploaded`
* **Tracked Properties:** * `fileName`
  * `contentType`
  * `sizeBytes`
  * `userId`

---

## 🔍 Monitoring & Log Verification

**Custom Event Query (KQL)**
```kusto
customEvents
| where name == "DocumentUploaded"
| order by timestamp desc
