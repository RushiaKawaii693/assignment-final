# Architecture & Implementation Plan

## Service Mapping

| Layer        | Service (Cloud)                 | Role in Solution                                           | Related Assignment/Module |
|--------------|---------------------------------|------------------------------------------------------------|---------------------------|
| Storage      | Azure Blob Storage              | Store raw uploaded CSV/JSON files from patient devices      | Cloud Storage & IAM       |
| Compute      | Azure App Service (Flask)       | Run containerized Flask API for uploads and dashboard       | Web App / Container Labs  |
| Compute      | Azure Functions (Serverless)    | Event-driven ETL: validate, parse, normalize, load to SQL   | Serverless Functions Lab  |
| Database/SQL | Azure SQL Database              | Store cleaned/aggregated tables for reporting and queries   | SQL Schema & Query Labs   |
| Analytics/AI | Azure ML Notebook (scheduled)   | Run simple analytics: time-in-range, risk flags, summaries  | ML/Analytics Module       |
| Security     | Azure Key Vault + Entra ID      | Manage secrets, credentials, and RBAC for clinicians/staff  | Identity & Governance     |
| Ops          | Azure Monitor + Log Analytics   | Collect logs, metrics, and alerts for governance            | Monitoring Module         |

---

## Data Flow Narrative

- **Step 1:** Clinician or staff uploads a CSV file via Flask web UI, or JSON arrives from device API.  
- **Step 2:** File lands in Azure Blob Storage in a raw container.  
- **Step 3:** Blob creation triggers Event Grid → Azure Functions.  
- **Step 4:** Function validates schema, normalizes units/timestamps, de-duplicates, and loads into Azure SQL Database (`patient_readings`).  
- **Step 5:** A scheduled Azure ML Notebook computes daily summaries (time-in-range %, hypo/hyper events, missing readings) and writes results into `daily_summary`.  
- **Step 6:** Flask queries SQL and displays clinician dashboard with summary stats and graphs.  
- **Step 7:** Azure Monitor + Log Analytics capture ingestion events, errors, and performance metrics for auditability.

---

## Security, Identity, and Governance Basics

- **Credentials:** All sensitive values (SQL connection strings, storage SAS tokens) are stored in **Azure Key Vault**. Flask and Functions use **Managed Identity** to retrieve secrets at runtime. No secrets are hardcoded.  
- **Access Controls / RBAC:**  
  - Clinicians: App Service access via Entra ID, read-only DB role.  
  - Data engineers: Contributor role on Functions/Storage, writer role on SQL.  
  - Auditors: Monitoring Reader role for logs/metrics.  
  - Enforce least privilege and container-level ACLs on Blob Storage.  
- **PHI Handling:** In student/testing environments, use synthetic or pseudonymized IDs. For production, segregate environments, enable private endpoints, encrypt data at rest/in transit, and maintain audit logs. No real PHI is exposed in public or personal accounts.

---

## Cost and Operational Considerations
In this architecture, the components most likely to drive costs are the always‑on compute resources and the managed database. Hosting the Flask application in Azure App Service requires continuous availability, which means a steady baseline cost, and Azure SQL Database can become expensive depending on the tier and storage size selected. Storage in Azure Blob is relatively inexpensive, especially when lifecycle policies are applied to move older files into cooler or archive tiers, while serverless functions are highly cost‑efficient because they only consume resources when triggered. Analytics workloads, such as scheduled Azure ML notebooks, can also add significant expense if large datasets or high‑end compute tiers are used, but these costs are controllable by limiting runtime and scheduling jobs during off‑peak hours.

To keep the project within a student budget or free tier, the design favors serverless and scheduled jobs over always‑on virtual machines. Azure Functions are used for ETL tasks so that compute scales to zero when idle, and the SQL Database can be deployed in a serverless tier with auto‑pause to reduce costs when not in use. The App Service hosting Flask can be kept at the lowest tier that still supports managed identity, and storage lifecycle rules ensure that raw data is moved to cheaper tiers over time. By combining these strategies, the solution demonstrates the full architecture while remaining affordable and practical for a student environment.

---

## Implementation Notes

- **Schema:**  
  - `patient_readings`: id, patient_id, source, reading_type, value, unit, timestamp_utc, quality_flag  
  - `daily_summary`: id, patient_id, date_utc, time_in_range_pct, hypo_events, hyper_events, missing_readings, risk_flag  
- **Validation Rules:** Reject negative values, enforce timestamp presence, normalize units, deduplicate by patient_id + timestamp.  
- **Deployment:** Use Infrastructure-as-Code (Bicep/Terraform) to provision resources. CI/CD pipelines (GitHub Actions/Azure DevOps) manage deployments with environment-specific configs.

