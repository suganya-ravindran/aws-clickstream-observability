# Café Clickstream Analytics & Observability Case Study

## 📌 Project Overview
**Role:** Cloud Engineer  
**Environment:** AWS (EC2, CloudWatch, S3)

In this project, I engineered a full-stack observability solution for a web application ("The Café"). The objective was to move beyond basic server monitoring (CPU/RAM) and implement **business-level intelligence** to track user behavior, geographic traffic, and revenue impact.

The system successfully transformed raw Apache web server logs into real-time dashboards and uncovered a critical application failure that was silently causing 100% revenue loss.

---

## 🏗 System Architecture

<img width="1408" height="768" alt="Hybrid Architectural Diagram_Suganya Ravindran" src="https://github.com/user-attachments/assets/c493e842-34bf-4c86-be5f-01eb0a385bba" />

### Architecture Highlights:
* **Data Source:** Amazon EC2 instance hosting a PHP web application.
* **Ingestion:** **Amazon CloudWatch Agent** configured to capture and parse custom application logs (`access_log`).
* **Analysis:** CloudWatch Logs Insights used for ad-hoc SQL-style queries.
* **Archival:** Amazon S3 bucket for long-term audit compliance.

---

## 🔧 Implementation Steps

### 1. Custom Log Ingestion Strategy
The default monitoring only provided infrastructure metrics. To gain application visibility, I:
* Installed the **CloudWatch Agent** on the Linux server.
* Configured the agent to stream `/var/log/www/access/access_log` to a custom Log Group (`apache/access`).
* Parsed HTTP fields (`RemoteIP`, `RequestURL`, `Status Code`) to enable granular filtering.

### 2. Automated Alerting (Operational Excellence)
I created a **Metric Filter** to detect degraded service states before users reported them.
* **Metric:** `404-Errors` (Client-side errors).
* **Threshold:** Trigger ALARM if errors > 5 within 1 minute.
* **Outcome:** The system automatically transitions to an ALARM state during high error rates, enabling rapid incident response.

![Alarm Configuration](alarm-state.png)

### 3. Business Intelligence Dashboard
I built a "Single Pane of Glass" dashboard to visualize traffic patterns for the Operations team.
* **Widgets Created:**
    * **Geographic Map:** Visualizing user traffic by Region/Country.
    * **Top Cities:** Grid view of cities with the highest engagement.
    * **Order Volume:** Bar chart tracking revenue-generating clicks.

![Operational Dashboard](dashboard-full.png)

---

## 🔍 Critical Discovery: Root Cause Analysis
During the final business review, the dashboard showed high traffic to the website but **zero completed orders**. This anomaly required a deep-dive forensic analysis.

Using **CloudWatch Logs Insights**, I wrote the following query to investigate the user journey on the Menu page:

```sql
fields @timestamp, status, request, remoteIP
| filter request = "/cafe/menu.php"
| filter status = "500"
| sort @timestamp desc
| limit 20
```
The Findings:

The query confirmed that the application was failing with HTTP 500 Internal Server Errors specifically on the /cafe/menu.php page.

Evidence: The logs proved that while users were trying to access the menu, the server was rejecting requests, preventing any items from being added to the cart.

Business Impact: This error was the direct cause of the 100% drop in sales revenue.

🚀 Conclusion & Recommendations
This project demonstrated that "Green" infrastructure status lights can be misleading. While the server (EC2) was healthy, the application was failing.

Recommendations provided to the team:

Immediate Fix: Developers must patch menu.php to resolve the 500 errors.

Process Improvement: Configure Amazon SNS notifications to email the DevOps team immediately when HTTP 500 errors are detected, reducing Mean Time to Resolution (MTTR).
