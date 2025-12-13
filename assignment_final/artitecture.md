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

- **Cost Drivers:**  
  - Azure SQL Database and App Service (always-on compute) are the largest recurring costs.  
  - Blob Storage is inexpensive at low volume.  
  - Azure Functions are cost-efficient since they scale to zero when idle.  
  - Azure ML Notebook incurs compute costs only during scheduled runs.  
- **Optimization:**  
  - Use serverless Functions for ETL instead of VMs to minimize idle costs.  
  - Deploy App Service at the lowest tier that supports managed identity.  
  - Choose SQL serverless tier with auto-pause to reduce costs.  
  - Apply lifecycle policies to move raw blobs to cool/archive storage.  
- **Student Budget:** Keep deployment in a single region, use free/basic tiers, and limit ML runtime. Focus on demonstrating architecture rather than scaling to production workloads.

---

## Implementation Notes

- **Schema:**  
  - `patient_readings`: id, patient_id, source, reading_type, value, unit, timestamp_utc, quality_flag  
  - `daily_summary`: id, patient_id, date_utc, time_in_range_pct, hypo_events, hyper_events, missing_readings, risk_flag  
- **Validation Rules:** Reject negative values, enforce timestamp presence, normalize units, deduplicate by patient_id + timestamp.  
- **Deployment:** Use Infrastructure-as-Code (Bicep/Terraform) to provision resources. CI/CD pipelines (GitHub Actions/Azure DevOps) manage deployments with environment-specific configs.

