# Architecture and implementation plan

## High-level diagram (Mermaid)
```mermaid
flowchart LR
    subgraph Client
        UI[Flask Web UI]
    end

    subgraph Frontend
        APISvc[Azure App Service (Flask)]
    end

    subgraph Storage
        Blob[Azure Blob Storage]
    end

    subgraph Compute
        Func[Azure Functions (ETL)]
        MLN[Azure ML Notebook (scheduled)]
    end

    subgraph Data
        SQL[Azure SQL Database]
        KV[Azure Key Vault]
    end

    subgraph Ops
        Mon[Azure Monitor + Log Analytics]
        AAD[Microsoft Entra ID (Azure AD)]
    end

    UI --> APISvc
    APISvc --> Blob
    Blob -- Event Grid --> Func
    Func --> SQL
    MLN --> SQL
    APISvc --> SQL
    KV --- APISvc
    KV --- Func
    Mon --> APISvc
    Mon --> Func
    AAD --> APISvc
    AAD --> MLN

