# Café Clickstream Analytics & Observability

## 📌 Project Overview
Deployed a full-stack observability solution for a web application running on **Amazon EC2**. The goal was to translate raw server logs into actionable business intelligence and real-time operational alerts.

**Role:** Cloud Engineer (Simulation)
**Technologies:** AWS CloudWatch, EC2, Systems Manager, S3, Linux (Apache).

---

## 🏗 Architecture
![Architecture Diagram](architecture-diagram.jpg)
*Designed a custom log ingestion pipeline moving data from EC2 -> CloudWatch Agent -> CloudWatch Logs -> S3 Archival.*

---

## 🔧 Implementation Highlights

### 1. Custom Log Ingestion
Configured the `amazon-cloudwatch-agent` on a Linux server to capture custom Apache access logs (`/var/log/www/access/access_log`).
- **Challenge:** The default metrics (CPU/RAM) were insufficient for tracking user behavior.
- **Solution:** Configured the agent to parse standard HTTP log formats to extract `RemoteIP`, `RequestURL`, and `Status` codes.

### 2. Operational Monitoring (Alarms)
Created a "404 Error" metric filter to detect broken links or scanning attempts.
- **Threshold:** > 5 errors within 1 minute.
- **Action:** CloudWatch Alarm triggers immediately, allowing for rapid incident response.
![Alarm Config](alarm-state.png)

### 3. Business Intelligence Dashboard
Built a "Single Pane of Glass" dashboard for the operations team, visualizing:
- **Traffic by Region** (Geolocation analysis).
- **Top Cities** by order volume.
- **Real-time Order Count**.
![Dashboard](dashboard-full.png)

---

## 🔍 Critical Discovery: Root Cause Analysis
During the final business review, the dashboard showed high traffic to the Menu page but **zero purchases**.

Using **CloudWatch Logs Insights**, I ran the following query to investigate the user journey:

```sql
fields @timestamp, status, request
| filter request = "/cafe/menu.php"
| filter status = "500"
| sort @timestamp desc

Result: The query confirmed that the application was failing with HTTP 500 Internal Server Errors, preventing users from viewing the menu. This analysis identified the specific PHP file (menu.php) responsible for 100% of the revenue loss.
