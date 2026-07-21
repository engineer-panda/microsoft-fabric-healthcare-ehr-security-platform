# Microsoft Fabric Healthcare Data Platform

End-to-end healthcare data engineering and EHR security analytics platform built with Microsoft Fabric, PySpark, Delta Lake, Fabric Data Pipelines, Medallion Architecture, data-quality validation, quarantine handling, Gold reporting tables, and Power BI.

---

## Project Overview

This project demonstrates the ingestion, transformation, validation, orchestration, and reporting of synthetic healthcare data within Microsoft Fabric.

The platform processes four datasets:

- Patients
- Prescriptions
- Clinical encounters
- EHR access logs

The solution follows this data flow:

```text
Healthcare Source Files
        ↓
Raw Lakehouse Files
        ↓
Bronze Delta Tables
        ↓
Silver Cleaning and Validation
        ↙                 ↘
Validated Silver       Quarantine
Records                Invalid Records
        ↓
Gold Reporting Tables
        ↓
Power BI Dashboards
```

---

## Architecture Diagram

![Microsoft Fabric Healthcare Architecture](screenshots/microsoft_fabric_healthcare_data_platform.png)

The architecture includes:

- Microsoft Fabric Lakehouses
- OneLake storage
- Raw healthcare source files
- Bronze Delta tables
- Silver validated tables
- Quarantine tables
- Gold analytical tables
- Fabric Data Pipelines
- Control Lakehouse
- Power BI dashboards

---

## Technology Stack

| Component | Technology |
|---|---|
| Data platform | Microsoft Fabric |
| Storage | Fabric Lakehouse and OneLake |
| Processing | PySpark and Spark SQL |
| Table format | Delta Lake |
| Orchestration | Fabric Data Pipelines |
| Data quality | PySpark validation and quarantine logic |
| Reporting | Power BI |
| Version control | GitHub |

---

## Source Datasets

### Patients

Contains patient demographic, contact, insurance, policy, and activity information.

Example fields:

- `patient_id`
- `first_name`
- `last_name`
- `date_of_birth`
- `gender`
- `email`
- `phone_number`
- `insurance_provider`
- `policy_number`
- `is_active`
- `start_date`
- `end_date`

### Prescriptions

Contains prescription and medication information associated with patients.

### Clinical Encounters

Contains healthcare operational and insurance-claim information.

Example fields:

- `encounter_id`
- `patient_id`
- `provider_id`
- `facility_id`
- `encounter_timestamp`
- `encounter_type`
- `primary_diagnosis_code`
- `procedure_code`
- `billing_amount`
- `insurance_claim_status`
- `facility_city`
- `facility_country`
- `attending_staff_id`

### EHR Access Logs

Contains electronic health record access activity used for security monitoring and access-pattern analysis.

---

## Medallion Architecture

### Raw Layer

Original synthetic healthcare files are stored in the Fabric Lakehouse Files area.

Source files:

- `patients.csv`
- `prescriptions.csv`
- `clinical_encounters.json`
- `ehr_access_logs.json`

The Raw layer preserves the original source data and supports traceability and reprocessing.

---

### Bronze Layer

Raw files are ingested into Delta tables while retaining their source-level structure.

Bronze tables:

- `bronze_patients`
- `bronze_prescriptions`
- `bronze_clinical_encounters`
- `bronze_ehr_access_logs`

Technical metadata includes:

- `batch_id`
- `source_file_name`
- `ingestion_timestamp`
- `row_hash`

The Bronze layer provides:

- Batch traceability
- Source-file tracking
- Schema visibility
- Rerunnable ingestion
- Delta Lake storage

---

### Silver Layer

PySpark notebooks clean, standardize, validate, and deduplicate Bronze records.

Silver processing includes:

- Data-type conversion
- String trimming and normalization
- Null handling
- Date and timestamp parsing
- Boolean standardization
- Duplicate detection
- Business-rule validation
- Valid and invalid record separation

Silver tables:

- `silver_patients`
- `silver_prescriptions`
- `silver_clinical_encounters`
- `silver_ehr_access_logs`

### Silver Clinical Encounter Processing

![Silver Clinical Encounters](silver_clinical_encounter.png)

This notebook output demonstrates the transformation and validation of clinical-encounter records before Gold reporting.

---

### Quarantine Layer

Records that fail validation are preserved in Quarantine tables rather than being silently removed.

Quarantine tables:

- `quarantine_patients`
- `quarantine_prescriptions`
- `quarantine_clinical_encounters`
- `quarantine_ehr_access_logs`

The Quarantine layer supports:

- Data-quality investigation
- Validation-failure analysis
- Auditability
- Reprocessing
- Record reconciliation

---

## Gold Layer

The Gold layer contains business-ready reporting tables created from validated Silver data.

Implemented Gold tables:

- `gold_claim_status_summary`
- `gold_daily_encounter_summary`
- `gold_diagnosis_summary`
- `gold_encounter_type_summary`
- `gold_facility_performance`
- `gold_patient_encounter_summary`
- `gold_patient_encounters`

### Gold Transformation Notebook

![Gold Transformation Notebook](gold_notebook.png)

The Gold notebook creates reporting-ready tables for claims, encounter, diagnosis, facility, patient, and billing analysis.

Gold transformations use Delta overwrite logic:

- The first successful run creates the table
- Subsequent runs replace the previous reporting output
- `overwriteSchema` supports compatible schema updates

---

## Gold Reporting Tables

### Claim Status Summary

`gold_claim_status_summary` provides metrics such as:

- Total encounters
- Total billing amount
- Average billing amount
- Unique patients
- Unique facilities

Claim-status categories include:

- Approved
- Denied
- Paid
- Pending

### Encounter Type Summary

`gold_encounter_type_summary` provides:

- Total encounters
- Total billing amount
- Average billing amount
- Unique patients
- Unique providers
- Unique facilities

Encounter categories include:

- Inpatient
- Outpatient
- Emergency

### Gold Lakehouse Output

![Gold Layer Tables](03_gold_layer_tables.png)

Gold summary tables contain category-level analytical results rather than individual source records.

For example:

```text
gold_claim_status_summary
→ One row per claim-status category

gold_encounter_type_summary
→ One row per encounter-type category
```

---

## Fabric Data Pipeline Orchestration

The project uses a master Fabric Data Pipeline to execute the processing layers in sequence.

```text
Bronze Ingestion Pipeline
        ↓
Silver Transformation Pipeline
        ↓
Gold Transformation Pipeline
```

The master orchestration pipeline manages:

- Bronze ingestion
- Silver transformation
- Quarantine processing
- Gold aggregation
- Pipeline dependencies
- End-to-end execution

### Successful Master Pipeline Run

![Successful Master Pipeline](02_master_pipeline_success.png)

The pipeline screenshot demonstrates successful execution of the Bronze, Silver, and Gold processing stages.

---

## Data Quality and Reconciliation

The project uses reconciliation checks to account for Bronze records after Silver processing.

```text
Bronze rows = Silver rows + Quarantine rows
```

This ensures that rejected records are:

- Not silently deleted
- Preserved for investigation
- Traceable to their source
- Included in reconciliation totals

---

## Operational Control Framework

A separate Fabric Control Lakehouse stores operational metadata used for pipeline management.

Implemented control tables:

- `file_tracking_table`
- `audit_table`
- `control_table`

These tables support:

- File-level processing status
- Batch tracking
- Source-file traceability
- Pipeline execution auditing
- Processing timestamps
- Pipeline status management
- Rerunnable processing

Invalid records are handled through Quarantine tables. The project does not use a separate `error_table`.

### Audit Table

![Audit Table](audit_table.png)

The audit table records pipeline and notebook execution details for operational monitoring and traceability.

---

## Power BI Reporting

Power BI consumes Gold reporting tables for healthcare claims, encounter, billing, and facility analysis.

### Claims Overview

![Power BI Claims Overview](powerbi/powerbi_claims_overview.png)

The claims dashboard includes:

- Claim-status breakdown
- Claim-status distribution
- Total billing amount
- Total encounters
- Unique patients by encounter type

### Facility Performance

![Power BI Facility Performance](powerbi/powerbi_facility_performance.png)

The facility dashboard includes:

- Top facilities by billing amount
- Denied claims by facility
- Total encounters by facility
- Claim-status analysis

---

## Repository Structure

```text
microsoft-fabric-healthcare-data-platform/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── notebooks/
│   ├── README.md
│   ├── nb_bronze_patients_ingestion.ipynb
│   ├── nb_bronze_prescriptions_ingestion.ipynb
│   ├── nb_bronze_clinical_encounters_ingestion.ipynb
│   ├── nb_bronze_ehr_access_logs_ingestion.ipynb
│   ├── nb_silver_patients.ipynb
│   ├── nb_silver_prescriptions.ipynb
│   ├── nb_silver_clinical_encounters.ipynb
│   └── nb_gold_transformation.ipynb
│
├── powerbi/
│   ├── README.md
│   ├── powerbi_claims_overview.png
│   └── powerbi_facility_performance.png
│
├── microsoft_fabric_healthcare_data_platform.png
├── 02_master_pipeline_success.png
├── 03_gold_layer_tables.png
├── silver_clinical_encounter.png
├── gold_notebook.png
└── audit_table.png
```

---

## Key Engineering Features

- End-to-end Microsoft Fabric implementation
- Medallion Architecture
- Fabric Lakehouse and OneLake
- Delta Lake tables
- PySpark DataFrame transformations
- Spark SQL processing
- Data-quality validation
- Quarantine handling
- Duplicate detection
- Batch metadata
- Row-hash generation
- Reconciliation checks
- Rerunnable notebooks
- Gold reporting aggregates
- Fabric Data Pipeline orchestration
- Power BI integration
- Audit and control tables

---

## Key Learning Outcomes

Through this project, I implemented and practiced:

- Designing a Medallion Architecture in Microsoft Fabric
- Creating Fabric Lakehouse Delta tables
- Reading and transforming data using PySpark
- Applying data-quality validation rules
- Separating valid and invalid records
- Preserving rejected records in Quarantine tables
- Implementing reconciliation checks
- Creating business-ready Gold tables
- Building rerunnable overwrite-based transformations
- Orchestrating dependent Fabric pipelines
- Preparing Gold tables for Power BI
- Documenting an end-to-end data engineering project in GitHub

---

## Project Scope

This project demonstrates Microsoft Fabric data-engineering concepts using synthetic healthcare data.

The implementation focuses on:

- Architecture
- Pipeline orchestration
- PySpark transformations
- Delta Lake
- Data-quality workflows
- Quarantine processing
- Gold aggregation
- Power BI integration

The validation rules were designed for educational and portfolio demonstration and can be refined for production healthcare requirements.

---

## Disclaimer

This project uses synthetic healthcare data exclusively for educational and portfolio purposes.

It does not contain real patient information, protected health information, production credentials, access keys, or confidential healthcare records.
