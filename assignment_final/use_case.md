# Use case: Remote diabetes monitoring mini-pipeline

## Problem statement
Diabetes clinics require rapid, low-friction access to patients' home glucose and activity data in order to focus outreach. Device data is currently dispersed among apps and sites, and staff manually sort through CSV exports. The goal is to create a lightweight cloud pipeline that ingests patient-generated health data (PGHD) on a daily basis, combines risk flags (such as frequent hypoglycemia, sustained hyperglycemia, and missed readings), and provides care teams with a simple triage display.

**Users:** Clinicians (MD, NP, or PA), diabetes educators, care coordinators, and clinic managers. 
**Value:** Faster triage and outreach, less manual processing, and defensible audit trails for compliance and quality reporting.

## Data sources
- **Device/app JSON:** Daily glucose readings, timestamps, and device metadata (via partner APIs or simulated JSON drops).
- **CSV uploads:** If API access is not available, manually export readings and notes.
- **Reference data:** Patient register with few identities (internal IDs), clinical objectives, and alert thresholds.
- **Operational logs:** Ingestion events, processing outcomes, and error measurements for auditability.

## Basic workflow
1. **Upload/ingest:** Staff uploads a CSV file through the Flask UI, or the system receives JSON from a device/app API.
2. **Landing storage:** Raw files are stored in Azure Blob Storage (with distinct containers for CSV and JSON).
3. **Event trigger:** A Blob-created event initiates Azure Functions for validation, parsing, and normalization.
4. **Transform/load:** Validated records are saved to the Azure SQL database (patient_readings, daily_summary).
5. **Analytics:** A scheduled Azure ML Notebook generates daily risk flags and averages (such as time-in-range).
6. **Serve:** Flask (Azure App Service) provides clinician-friendly summary, filtering, and exporting options.
7. **Monitor & audit:** Azure Monitor + Log Analytics records events, problems, and performance for governance.

## Scope boundaries
- **No real PHI:** Use internal pseudonymous patient IDs and fake data in student/testing settings.
- **Minimal UI:** A simple upload page and a summary dashboard are provided; however, deep BI visualizations are not included.
