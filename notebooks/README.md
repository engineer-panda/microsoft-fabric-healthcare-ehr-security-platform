# Microsoft Fabric Notebooks

This folder contains the Microsoft Fabric notebooks used to implement the Bronze, Silver, Quarantine, and Gold processing layers for the Healthcare Analytics and EHR Security Platform.

## Bronze Ingestion Notebooks

- `nb_bronze_patients_ingestion.ipynb`
- `nb_bronze_prescriptions_ingestion.ipynb`
- `nb_bronze_clinical_encounters_ingestion.ipynb`
- `nb_bronze_ehr_access_logs_ingestion.ipynb`

These notebooks ingest Raw healthcare source files into Bronze Delta tables and add technical metadata such as:

- `batch_id`
- `source_file_name`
- `ingestion_timestamp`
- `row_hash`

## Silver Transformation Notebooks

- `nb_silver_patients.ipynb`
- `nb_silver_prescriptions.ipynb`
- `nb_silver_clinical_encounters.ipynb`
- `nb_silver_ehr_access_logs.ipynb`

These notebooks perform:

- Data-type conversion
- String normalization
- Date and timestamp parsing
- Null validation
- Duplicate detection
- Business-rule validation
- Valid and invalid record separation
- Quarantine processing
- Bronze-to-Silver reconciliation

## Gold Transformation Notebook

- `nb_gold_transformation.ipynb`

This notebook creates business-ready Gold reporting tables for:

- Claim-status analysis
- Encounter-type analysis
- Diagnosis analysis
- Facility performance
- Patient encounter analysis
- Billing analysis

## Processing Flow

```text
Raw Files
   ↓
Bronze Delta Tables
   ↓
Silver Validation and Cleaning
   ↙                  ↘
Validated Records    Quarantine Records
   ↓
Gold Reporting Tables
