# Databricks Lakehouse
Data lakehouse contains raw data extracted from ERP and CRM systems as well as three tier data warehouse adhering to medallion architecture. 

[![Layout Diagram](databricks-lakehouse/blob/main/architecture/layout.png)](databricks-lakehouse/blob/main/architecture/layout.png)

```mermaid
flowchart LR
    %% ===== Sources =====
    subgraph SRC["Unity Catalog"]
        CRM["CRM"]
        ERP["ERP"]
    end

    %% ===== Bronze Layer =====
    subgraph BRZ["Bronze Layer (Raw Ingest)"]
        B_CRM_SALES["crm_sales_details"]
        B_CRM_CUST["crm_cust_info"]
        B_CRM_PRD["crm_prd_info"]

        B_ERP_CUST["erp_cust_az12"]
        B_ERP_LOC["erp_loc_a101"]
        B_ERP_PX["erp_px_cat_g1v2"]
    end

    %% ===== Silver Layer =====
    subgraph SLV["Silver Layer (Cleansed)"]
        S_CRM_SALES["crm_sales_details"]
        S_CRM_CUST["crm_cust_info"]
        S_CRM_PRD["crm_prd_info"]

        S_ERP_CUST["erp_cust_az12"]
        S_ERP_LOC["erp_loc_a101"]
        S_ERP_PX["erp_px_cat_g1v2"]
    end

    %% ===== Gold Layer =====
    subgraph GLD["Gold Layer (Analytics)"]
        FACT_SALES["fact_sales"]
        DIM_CUSTOMERS["dim_customers"]
        DIM_PRODUCTS["dim_products"]
    end

    %% ===== Source to Bronze =====
    CRM --> B_CRM_SALES
    CRM --> B_CRM_CUST
    CRM --> B_CRM_PRD

    ERP --> B_ERP_CUST
    ERP --> B_ERP_LOC
    ERP --> B_ERP_PX

    %% ===== Bronze to Silver =====
    B_CRM_SALES --> S_CRM_SALES
    B_CRM_CUST --> S_CRM_CUST
    B_CRM_PRD --> S_CRM_PRD

    B_ERP_CUST --> S_ERP_CUST
    B_ERP_LOC --> S_ERP_LOC
    B_ERP_PX --> S_ERP_PX

    %% ===== Silver to Gold =====
    S_CRM_SALES --> FACT_SALES
    S_CRM_PRD --> DIM_PRODUCTS
    S_ERP_PX --> DIM_PRODUCTS

    S_CRM_CUST --> DIM_CUSTOMERS
    S_ERP_CUST --> DIM_CUSTOMERS
    S_ERP_LOC --> DIM_CUSTOMERS
```
### Modeling Principles

- Bronze Layer

    - Immutable, replayable raw data

    - Schema changes tolerated

    - No joins or aggregations

- Silver Layer

    - One row = one business entity/event

    - Cleaned, deduplicated, standardized

    - Still traceable to source

- Gold Layer

    - Star schema optimized for analytics

    - Facts reference conformed dimensions

    - Business logic is applied once and reused everywhere
