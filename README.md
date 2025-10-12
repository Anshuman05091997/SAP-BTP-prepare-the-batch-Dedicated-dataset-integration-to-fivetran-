# SAP BTP to Azure Synapse Integration via Fivetran

## Project Overview

This project demonstrates batch data synchronization from **SAP HANA Cloud (BTP)** to **Azure Synapse Analytics (Dedicated SQL Pool)** using **Fivetran** as the integration platform. The solution was developed as part of a research initiative to understand cloud-to-cloud data integration patterns.

**Project Type:** Research Project for Contractor  
**Duration:** 9 months experience level  
**Role:** Research Student  

---

## Business Use Case

**Problem Statement:**  
Organizations need to replicate HR operational data from SAP HANA Cloud to Azure Synapse Analytics for centralized analytics and reporting without building custom ETL pipelines.

**Solution:**  
Implement an automated batch integration using Fivetran to:
- Extract HR data from SAP HANA Cloud on BTP
- Load data into Azure Synapse Dedicated SQL Pool
- Schedule periodic synchronization
- Enable downstream analytics and reporting

---

## Dataset

**HR Sample Dataset** (under 1MB)

| Table | Description |
|-------|-------------|
| DEPARTMENTS | Department master data |
| JOBS | Job roles and salary ranges |
| EMPLOYEES | Employee master records |
| JOB_HISTORY | Employee job transition history |
| ATTENDANCE | Daily attendance records |
| PAYROLL | Payroll transaction data |

---

## Architecture Components

### Source System
- **Platform:** SAP Business Technology Platform (BTP)
- **Database:** SAP HANA Cloud (Column Store)
- **Schema:** TRAINING
- **Access:** Read-only dedicated user for Fivetran

### Integration Layer
- **Tool:** Fivetran
- **Connector Type:** SAP HANA
- **Sync Mode:** Batch (scheduled)
- **Frequency:** Configurable (6 hours / daily)

### Target System
- **Platform:** Microsoft Azure
- **Service:** Azure Synapse Analytics
- **Resource:** Dedicated SQL Pool (DW100c)
- **Database:** fivetran_demo_pool

---

## Key Features

✅ **Automated Batch Synchronization** - No manual intervention required  
✅ **Schema Replication** - Maintains source table structures  
✅ **Referential Integrity** - Preserves foreign key relationships  
✅ **Incremental Updates** - Efficient data transfer after initial load  
✅ **Scheduled Refresh** - Configurable sync intervals  
✅ **SSL/TLS Security** - Encrypted data in transit  

---

## Prerequisites

- Azure subscription (Azure for Students is sufficient)
- Fivetran trial account
- SAP HANA Cloud instance on BTP with internet accessibility
- HR sample dataset (CSV files)
- Basic knowledge of SQL and cloud platforms

---

## High-Level Implementation Flow

```
Phase A: Source Preparation (SAP HANA)
  └─ Create schema and tables
  └─ Import CSV data
  └─ Create read-only user
  └─ Validate data

Phase B: Destination Preparation (Azure Synapse)
  └─ Create Synapse workspace
  └─ Create dedicated SQL pool
  └─ Configure network access
  └─ Create Fivetran user

Phase C: Fivetran Configuration
  └─ Add Azure Synapse destination
  └─ Add SAP HANA connector
  └─ Select tables for sync
  └─ Execute initial batch load

Phase D: Validation & Scheduling
  └─ Verify data in Synapse
  └─ Configure sync schedule
  └─ Create reporting views (optional)
```

---

## Project Deliverables

1. **Configured SAP HANA Schema** - TRAINING schema with 6 tables
2. **Azure Synapse Workspace** - Dedicated SQL Pool for analytics
3. **Fivetran Pipeline** - Automated connector with scheduled sync
4. **Data Validation Scripts** - SQL queries to verify data integrity
5. **Documentation** - Architecture and implementation guide

---

## Sync Performance Metrics

| Metric | Value |
|--------|-------|
| Total Tables | 6 |
| Data Volume | < 1 MB |
| Initial Sync Time | ~2-5 minutes |
| Incremental Sync | ~30-60 seconds |
| Sync Frequency | 6 hours (configurable) |

---

## Network Security

### Firewall Configuration
- **SAP HANA Cloud:** Allowlist Fivetran egress IPs
- **Azure Synapse:** Add client IPs and Fivetran IPs

### Authentication
- **Connection Encryption:** SSL/TLS enabled on both ends
- **Credential Management:** Dedicated service accounts with minimal privileges
- **Password Policy:** Strong passwords enforced

---

## Limitations & Assumptions

- Dataset size is small (< 1MB) - suitable for batch processing
- No real-time requirements - scheduled batch sync is acceptable
- Tables have primary keys for incremental updates
- Network connectivity between SAP BTP and Fivetran is stable
- Azure Synapse Dedicated SQL Pool runs continuously (not paused)

---

## Future Enhancements

- Implement data quality checks and validations
- Add data transformation logic in Synapse
- Create Power BI dashboards for HR analytics
- Set up monitoring and alerting for sync failures
- Optimize Fivetran sync frequency based on data change patterns

---

## Troubleshooting Guide

### Common Issues

**Issue 1: HANA Connection Fails**
- Verify host, port, and credentials
- Check Fivetran IPs in HANA Cloud allowlist
- Confirm user has SELECT privileges on TRAINING schema

**Issue 2: Synapse Connection Fails**
- Validate Synapse endpoint and port (1433)
- Check firewall rules include Fivetran IPs
- Ensure dedicated SQL pool is Online (not paused)

**Issue 3: No Tables in Synapse After Sync**
- Confirm tables are selected in Fivetran connector
- Check sync logs for errors
- Verify correct destination schema name

**Issue 4: Data Import Issues in HANA**
- Ensure CSV files have correct encoding (UTF-8)
- Verify delimiter is comma and header row is enabled
- Check column mapping matches table structure

---

## Technologies Used

| Category | Technology |
|----------|-----------|
| Source Database | SAP HANA Cloud (BTP) |
| Target Database | Azure Synapse Analytics |
| Integration Platform | Fivetran |
| Data Format | CSV (import), Column Store (HANA) |
| Security | SSL/TLS, Firewall Rules |
| Cloud Platforms | SAP BTP, Microsoft Azure |

---

## Learning Outcomes

Through this research project, I gained hands-on experience in:

1. **SAP BTP Ecosystem** - HANA Cloud database management and security
2. **Azure Synapse Analytics** - Dedicated SQL Pool provisioning and configuration
3. **Cloud Integration Patterns** - Batch data replication using SaaS tools
4. **Data Pipeline Design** - Source-to-target mapping and scheduling
5. **Network Security** - Firewall configuration and SSL/TLS setup
6. **Troubleshooting** - Debugging connectivity and sync issues

---

## References

- SAP HANA Cloud Documentation
- Azure Synapse Analytics Documentation
- Fivetran Connector Documentation
- Cloud Integration Best Practices

---

## Project Status

✅ **Completed** - All phases implemented and validated successfully

---

**Author:** Research Student  
**Organization:** Contractor Research Initiative  
**Experience Level:** 9 months  
**Last Updated:** October 2025

