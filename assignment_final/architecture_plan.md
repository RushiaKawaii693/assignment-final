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
        EG[Event Grid]
    end

    subgraph Compute
        Func[Azure Functions (ETL)]
        MLN[Azure ML (Notebook/Scheduled job)]
    end

    subgraph Data
        SQL[Azure SQL Database]
        KV[Azure Key Vault]
    end

    subgraph Ops & Identity
        Mon[Azure Monitor + Log Analytics]
        Entra[Microsoft Entra ID]
    end

    UI --> APISvc
    APISvc --> Blob
    Blob --> EG
    EG --> Func
    Func --> SQL
    MLN --> SQL
    APISvc --> SQL
    KV --- APISvc
    KV --- Func
    Mon --> APISvc
    Mon --> Func
    Entra --> APISvc
    Entra --> SQL
    Entra --> MLN
