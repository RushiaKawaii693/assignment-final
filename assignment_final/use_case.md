# Use case: Remote diabetes monitoring mini-pipeline

## Problem statement
Diabetes clinics need timely, low-friction visibility into patients’ home glucose and activity data to prioritize outreach. Today, device data is fragmented across apps and portals, and staff sift through CSV exports manually. The goal is a lightweight cloud pipeline that ingests patient-generated health data (PGHD) daily, aggregates risk flags (e.g., frequent hypoglycemia, sustained hyperglycemia, missed readings), and presents a simple triage dashboard for care teams. 

**Users:** Clinicians (MD/NP/PA), diabetes educators, care coordinators, and clinic managers.  
**Value:** Faster triage and outreach, reduced manual processing, and defensible audit trails for compliance and quality reporting.

## Data sources
- **Device/app JSON:** Daily glucose readings, timestamps, device metadata (from partner APIs or simulated JSON drops).
- **CSV uploads:** Manual exports with readings and notes if API access isn’t available.
- **Reference data:** Patient registry with minimal identifiers (internal IDs), clinical targets, and alert thresholds.
- **Operational logs:** Ingestion events, processing results, error metrics for auditability.

## Basic workflow
1. **Upload/ingest:** Staff uploads a CSV via a Flask UI or the system receives JSON from a device/app API.
2. **Landing storage:** Raw files land in Azure Blob Storage (separate containers for CSV vs. JSON).
3. **Event trigger:** A Blob-created event triggers Azure Functions for validation, parsing, and normalization.
4. **Transform/load:** Validated records are written to Azure SQL Database (patient_readings, daily_summary).
5. **Analytics:** A scheduled Azure ML Notebook computes daily risk flags and aggregates (e.g., time-in-range).
6. **Serve:** Flask (Azure App Service) displays clinician-friendly summaries, filters, and export options.
7. **Monitor & audit:** Azure Monitor + Log Analytics capture events, errors, and performance for governance.

## Scope boundaries
- **No real PHI:** Use internal pseudonymous patient IDs and synthetic data in student/testing environments.
- **Minimal UI:** Simple upload page and a summary dashboard; deep BI visualizations are out of scope.

