# Project Implementation Flow

## Step-by-Step Implementation Guide

This document outlines the complete implementation process for the SAP BTP to Azure Synapse integration project using Fivetran.

---

## Prerequisites Checklist

Before starting the implementation:

- [ ] Azure account (Azure for Students is acceptable)
- [ ] Fivetran trial account created
- [ ] SAP HANA Cloud instance on BTP (internet accessible)
- [ ] HR sample dataset downloaded (departments.csv, jobs.csv, employees.csv, job_history.csv, attendance.csv, payroll.csv)
- [ ] Basic understanding of SQL and cloud platforms

---

## Phase A: Source System Preparation (SAP HANA on BTP)

### Step 1: Access HANA Database Explorer

**Objective:** Connect to SAP HANA Cloud database for configuration

**Actions:**
1. Log in to SAP BTP Cockpit
2. Navigate to HANA Cloud instance
3. Click Actions → Open in SAP HANA Database Explorer
4. Authenticate with administrator credentials

**Expected Outcome:** Successfully connected to HANA Database Explorer interface

---

### Step 2: Create Schema

**Objective:** Create isolated schema for HR data

**Actions:**
1. Open SQL Console in Database Explorer
2. Run SQL query to create schema named `TRAINING`
3. Refresh schema list to verify creation

**Expected Outcome:** TRAINING schema appears in the database catalog

**Validation:** Schema visible in left navigation panel

---

### Step 3: Create Table Structures

**Objective:** Define relational data model for HR dataset

**Tables Created:**
1. **DEPARTMENTS** - Department master data table
2. **JOBS** - Job roles and salary ranges table
3. **EMPLOYEES** - Employee master records table
4. **JOB_HISTORY** - Job transition history table
5. **ATTENDANCE** - Daily attendance log table
6. **PAYROLL** - Payroll transaction table

**Actions:**
1. Open SQL Console
2. Run SQL queries to create all 6 tables with appropriate columns
3. Define primary keys for each table
4. Set up foreign key relationships between tables
5. Use COLUMN TABLE type for HANA optimization

**Expected Outcome:** 6 empty tables created in TRAINING schema with defined relationships

**Validation:** All tables visible under TRAINING schema in catalog

---

### Step 4: Import CSV Data

**Objective:** Load sample HR data from CSV files into HANA tables

**Process for Each Table:**

**4.1 Import DEPARTMENTS**
- Right-click TRAINING schema → Import Data
- Select source: Local file → departments.csv
- Target table: TRAINING.DEPARTMENTS
- Configure: delimiter=comma, header row=yes, encoding=UTF-8
- Execute import

**4.2 Import JOBS**
- Repeat import process for jobs.csv
- Target table: TRAINING.JOBS

**4.3 Import EMPLOYEES**
- Repeat import process for employees.csv
- Target table: TRAINING.EMPLOYEES

**4.4 Import JOB_HISTORY**
- Repeat import process for job_history.csv
- Target table: TRAINING.JOB_HISTORY

**4.5 Import ATTENDANCE**
- Repeat import process for attendance.csv
- Target table: TRAINING.ATTENDANCE

**4.6 Import PAYROLL**
- Repeat import process for payroll.csv
- Target table: TRAINING.PAYROLL

**Expected Outcome:** All 6 tables populated with data from CSV files

**Troubleshooting:**
- If import fails, verify CSV encoding is UTF-8
- Check delimiter settings match CSV format
- Ensure header row option is enabled
- Verify column mapping matches table structure

---

### Step 5: Validate Source Data

**Objective:** Confirm data loaded correctly in HANA

**Validation Queries:**
1. Run SQL query to count rows in EMPLOYEES table
2. Run SQL query to get maximum date from ATTENDANCE table
3. Run SQL query to get maximum date from PAYROLL table

**Expected Outcome:**
- EMPLOYEES table contains multiple rows
- ATTENDANCE has recent dates
- PAYROLL has recent dates

**Success Criteria:** All queries return valid data

---

### Step 6: Create Read-Only Service Account

**Objective:** Create dedicated user for Fivetran with minimal privileges

**Actions:**
1. Open SQL Console
2. Run SQL query to create user `TRAINING_USER` with strong password
3. Run SQL query to grant SELECT privileges on TRAINING schema to this user
4. Document username and password securely

**Expected Outcome:** TRAINING_USER created with read-only access to TRAINING schema

**Security Notes:**
- Use strong password meeting SAP security requirements
- Grant only SELECT privilege (no INSERT, UPDATE, DELETE)
- Credentials will be used in Fivetran configuration

---

## Phase B: Destination System Preparation (Azure Synapse)

### Step 7: Create Synapse Workspace

**Objective:** Provision Azure Synapse Analytics workspace

**Actions:**
1. Log in to Azure Portal
2. Search for "Synapse Analytics"
3. Click Create
4. Fill in configuration:
   - Subscription: Select your subscription
   - Resource group: Create new or select existing
   - Workspace name: Choose unique name
   - Region: Select closest region
   - Data Lake Storage: Auto-create or select existing
5. Review configuration
6. Click Create
7. Wait for deployment to complete (5-10 minutes)
8. Navigate to created workspace

**Expected Outcome:** Synapse workspace successfully deployed

**Validation:** Workspace appears in Azure Portal resources

---

### Step 8: Create Dedicated SQL Pool

**Objective:** Provision compute resource for data warehouse

**Actions:**
1. Inside Synapse workspace, click "New Dedicated SQL Pool"
2. Configure:
   - Pool name: `fivetran_demo_pool`
   - Performance level: DW100c (smallest tier)
   - Collation: Default (SQL_Latin1_General_CP1_CI_AS)
3. Review settings
4. Click Create
5. Wait for provisioning to complete (10-15 minutes)
6. Open Synapse Studio
7. Navigate to Data → Workspace → SQL Databases
8. Verify `fivetran_demo_pool` status is "Online"

**Expected Outcome:** Dedicated SQL pool running and accessible

**Cost Consideration:** DW100c is lowest tier, suitable for demo/development

---

### Step 9: Create Fivetran Service Account

**Objective:** Create database user for Fivetran with necessary privileges

**Actions:**
1. In Synapse Studio, connect to `fivetran_demo_pool`
2. Open new SQL script
3. Run SQL query to create login `fivetran_user` with strong password
4. Run SQL query to create database user for this login
5. Run SQL query to add user to `db_owner` role (for demo; use minimal privileges in production)
6. Document credentials securely

**Expected Outcome:** fivetran_user created with database owner privileges

**Security Notes:**
- Use strong password
- For production, grant only necessary privileges (CREATE TABLE, INSERT, UPDATE, DELETE)
- Credentials will be used in Fivetran destination configuration

---

### Step 10: Configure Network Access

**Objective:** Allow Fivetran to connect to Synapse SQL pool

**Actions:**
1. Navigate to Synapse workspace → Networking settings
2. Add firewall rule for your client IP (for admin access)
3. Note: Fivetran IPs will be added later during connector setup
4. Enable "Allow Azure services and resources to access this workspace" (optional)
5. Save firewall configuration

**Expected Outcome:** Network firewall configured for access

**Documentation:**
- Note down Synapse server endpoint (e.g., yourworkspace.sql.azuresynapse.net)
- Note down database name: fivetran_demo_pool
- Port: 1433 (default SQL Server port)

---

## Phase C: Fivetran Configuration

### Step 11: Configure Azure Synapse Destination

**Objective:** Set up target system in Fivetran

**Actions:**
1. Log in to Fivetran dashboard
2. Navigate to Destinations
3. Click "Add Destination"
4. Select "Azure Synapse Analytics"
5. Fill in connection details:
   - Host: Synapse endpoint (from Step 10)
   - Port: 1433
   - Database: fivetran_demo_pool
   - User: fivetran_user
   - Password: (from Step 9)
   - Connection security: Enable SSL/TLS
6. Click "Save & Test Connection"
7. Review test results
8. If connection fails, check firewall rules
9. If needed, add Fivetran egress IPs to Azure firewall
10. Retry until connection succeeds
11. Complete destination setup wizard

**Expected Outcome:** Azure Synapse destination successfully configured

**Troubleshooting:**
- Verify endpoint URL is correct
- Check firewall allows Fivetran IPs
- Ensure SQL pool is Online (not Paused)
- Verify credentials are correct

---

### Step 12: Configure SAP HANA Connector

**Objective:** Set up source system connection in Fivetran

**Actions:**
1. In Fivetran dashboard, navigate to Connectors
2. Click "Add Connector"
3. Search for and select "SAP HANA"
4. Fill in connection details:
   - Host: HANA Cloud SQL endpoint (from BTP Cockpit)
   - Port: HANA SQL port (typically 443 or custom)
   - Database: Tenant database name (if required)
   - User: TRAINING_USER
   - Password: (from Step 6)
   - Use SSL/TLS: Enable
5. Click "Test Connection"
6. Review test results
7. If connection fails, check HANA Cloud firewall
8. Navigate to HANA Cloud instance → Allowed Connections
9. Add Fivetran egress IP addresses to allowlist
10. Retry connection test until successful
11. Click "Save"

**Expected Outcome:** SAP HANA connector successfully configured

**Troubleshooting:**
- Verify HANA Cloud instance is running
- Check endpoint and port are correct
- Ensure TRAINING_USER has SELECT privileges
- Confirm Fivetran IPs are allowlisted in HANA Cloud
- Verify SSL/TLS is enabled

---

### Step 13: Select Schema and Tables

**Objective:** Choose which data to replicate from HANA to Synapse

**Actions:**
1. In the SAP HANA connector configuration, navigate to Schema tab
2. Fivetran displays available schemas
3. Locate and expand TRAINING schema
4. Select tables for synchronization:
   - Toggle ON: DEPARTMENTS
   - Toggle ON: JOBS
   - Toggle ON: EMPLOYEES
   - Toggle ON: JOB_HISTORY
   - Toggle ON: ATTENDANCE
   - Toggle ON: PAYROLL
5. Review table list
6. Click "Save" or "Save & Sync"

**Expected Outcome:** 6 tables selected for replication

**Notes:**
- Only selected tables will be synced
- Table selection can be modified later
- Primary keys are automatically detected

---

### Step 14: Execute Initial Batch Sync

**Objective:** Perform first-time data load from HANA to Synapse

**Actions:**
1. In connector dashboard, click "Sync Now" or "Start Initial Sync"
2. Fivetran begins sync process:
   - Connects to SAP HANA
   - Extracts data from 6 tables
   - Creates destination schema in Synapse
   - Creates tables with appropriate structure
   - Loads data into Synapse
3. Monitor sync progress in Fivetran dashboard
4. Watch for completion status
5. Review sync summary:
   - Total rows synced per table
   - Sync duration
   - Any warnings or errors

**Expected Outcome:** Initial sync completes successfully with all data loaded

**Sync Process Details:**
- Duration: Typically 2-5 minutes for small dataset
- Fivetran creates schema named similar to `sap_hana_training`
- Table names converted to lowercase
- Fivetran adds metadata columns (_fivetran_synced, _fivetran_deleted)

**Troubleshooting:**
- If sync fails, check connector logs
- Verify both source and destination connections are active
- Ensure no network interruptions
- Check for any table-specific errors

---

## Phase D: Validation and Scheduling

### Step 15: Verify Data in Synapse

**Objective:** Confirm data successfully loaded into target system

**Actions:**
1. Open Synapse Studio
2. Connect to fivetran_demo_pool
3. Navigate to Data → Workspace → SQL Databases → fivetran_demo_pool
4. Expand Tables
5. Locate Fivetran-created schema (e.g., sap_hana_training)
6. Expand schema to view tables
7. Open new SQL script
8. Run validation queries:
   - Count rows in employees table
   - Count rows in attendance table
   - Get latest date from payroll table
   - Perform join query between employees and departments
9. Compare row counts with source HANA tables
10. Verify data integrity

**Expected Outcome:**
- All 6 tables present in Synapse
- Row counts match source tables
- Relationships preserved
- Data values correct

**Success Criteria:**
- employees table has correct row count
- attendance table has correct row count
- payroll has recent dates
- Join queries work correctly

---

### Step 16: Configure Sync Schedule

**Objective:** Set up automated batch refresh cadence

**Actions:**
1. In Fivetran dashboard, open SAP HANA connector
2. Navigate to Setup or Schedule tab
3. Configure sync frequency:
   - Option 1: Every 6 hours
   - Option 2: Daily at specific time
   - Option 3: Custom schedule
4. Select desired frequency (e.g., Every 6 hours)
5. Save configuration
6. Fivetran will now automatically run syncs on schedule

**Expected Outcome:** Automated sync schedule activated

**Sync Behavior:**
- Fivetran checks for changes at scheduled intervals
- Only changed data is transferred (incremental sync)
- Primary keys used to identify updates
- Synapse data stays up-to-date automatically

**Recommendations:**
- For slowly changing data: Daily sync
- For frequently updated data: Every 6 hours
- Monitor sync history to optimize frequency

---

### Step 17: Create Reporting Views (Optional)

**Objective:** Create simplified views for analytics and BI tools

**Actions:**
1. In Synapse Studio, connect to fivetran_demo_pool
2. Open new SQL script
3. Run SQL query to create new schema for analytics (e.g., `analytics`)
4. Run SQL queries to create views:
   - **employee_dim**: Clean employee dimension view
   - **attendance_daily**: Daily attendance fact view
   - **payroll_monthly**: Monthly payroll summary view
5. These views hide Fivetran metadata columns
6. Views provide clean interface for reporting tools

**Expected Outcome:** Reporting views created in analytics schema

**Benefits:**
- Simplified data model for BI tools
- Hide technical columns
- Apply business logic and transformations
- Stable interface even if source schema changes

**Next Steps:**
- Connect Power BI Desktop to Synapse
- Use views as data source for reports
- Build dashboards and analytics

---

## Implementation Summary

### Overall Process Flow

```
Phase A: Source Preparation (HANA)
  ├─ Step 1: Access Database Explorer
  ├─ Step 2: Create TRAINING schema
  ├─ Step 3: Create 6 tables
  ├─ Step 4: Import CSV data
  ├─ Step 5: Validate data
  └─ Step 6: Create service account

Phase B: Destination Preparation (Synapse)
  ├─ Step 7: Create Synapse workspace
  ├─ Step 8: Create SQL pool
  ├─ Step 9: Create service account
  └─ Step 10: Configure network

Phase C: Fivetran Configuration
  ├─ Step 11: Add Synapse destination
  ├─ Step 12: Add HANA connector
  ├─ Step 13: Select tables
  └─ Step 14: Run initial sync

Phase D: Validation & Scheduling
  ├─ Step 15: Verify data
  ├─ Step 16: Schedule syncs
  └─ Step 17: Create views (optional)
```

---

## Key Accomplishments

✅ SAP HANA Cloud configured with HR dataset  
✅ Azure Synapse Analytics workspace and SQL pool provisioned  
✅ Fivetran integration pipeline established  
✅ Batch data synchronization automated  
✅ Data validation completed  
✅ Scheduled refresh configured  
✅ Optional reporting views created  

---

## Technical Skills Demonstrated

1. **SAP HANA Administration**
   - Schema and table creation
   - Data import from CSV
   - User and privilege management
   - Database Explorer proficiency

2. **Azure Cloud Services**
   - Synapse workspace provisioning
   - Dedicated SQL pool management
   - Firewall configuration
   - Synapse Studio operations

3. **Fivetran Integration**
   - Connector configuration
   - Destination setup
   - Schema mapping
   - Sync scheduling

4. **Data Validation**
   - SQL query execution
   - Data integrity verification
   - Row count reconciliation
   - Relationship testing

5. **Security Implementation**
   - Service account creation
   - Privilege management
   - SSL/TLS configuration
   - IP allowlisting

---

## Common Challenges & Solutions

### Challenge 1: HANA Connection Timeout
**Solution:** Added Fivetran egress IPs to HANA Cloud allowlist

### Challenge 2: CSV Import Errors
**Solution:** Verified UTF-8 encoding and correct delimiter settings

### Challenge 3: Synapse Firewall Blocking
**Solution:** Added Fivetran IPs and client IPs to firewall rules

### Challenge 4: Sync Not Detecting Changes
**Solution:** Ensured tables have primary keys for incremental updates

### Challenge 5: SQL Pool Paused
**Solution:** Manually resumed SQL pool before testing connection

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Source tables | 6 |
| Total data volume | < 1 MB |
| Initial sync duration | 2-5 minutes |
| Incremental sync duration | 30-60 seconds |
| Sync frequency | Every 6 hours |
| Network latency | Varies by region |

---

## Project Timeline

| Phase | Duration | Complexity |
|-------|----------|------------|
| Phase A (HANA Setup) | 30-45 minutes | Medium |
| Phase B (Synapse Setup) | 30-45 minutes | Medium |
| Phase C (Fivetran Config) | 20-30 minutes | Easy |
| Phase D (Validation) | 15-20 minutes | Easy |
| **Total** | **~2 hours** | **Medium** |

*Note: Timeline assumes familiarity with platforms and no major troubleshooting required*

---

## Lessons Learned

1. **Firewall Configuration is Critical**: Most connection issues were firewall-related
2. **Primary Keys Are Essential**: Required for efficient incremental syncs
3. **CSV Encoding Matters**: UTF-8 encoding prevented import issues
4. **Service Accounts Best Practice**: Dedicated users with minimal privileges improve security
5. **Validation Is Important**: Always verify data after sync to catch issues early
6. **Documentation Helps**: Keeping track of credentials and endpoints saved time

---

## Future Enhancements

### Short-term (1-3 months)
- Add data quality validation rules
- Implement error alerting via email
- Create Power BI dashboards for HR analytics

### Medium-term (3-6 months)
- Add more HR data sources
- Implement data transformations in Synapse
- Optimize sync frequency based on usage patterns

### Long-term (6-12 months)
- Migrate to real-time CDC if needed
- Add data governance and lineage tracking
- Implement automated testing framework

---

## Resources & References

### Official Documentation
- SAP HANA Cloud Administration Guide
- Azure Synapse Analytics Documentation
- Fivetran Connector Documentation
- SAP BTP Cockpit User Guide

### Useful Tools
- SAP HANA Database Explorer
- Azure Portal
- Synapse Studio
- Fivetran Dashboard

### Learning Resources
- SAP BTP tutorials
- Azure learning paths
- Fivetran integration guides
- SQL optimization techniques

---

## Conclusion

This project successfully demonstrated end-to-end batch data integration between SAP HANA Cloud and Azure Synapse Analytics using Fivetran. The implementation covered source preparation, destination configuration, connector setup, and automated scheduling.

**Key Takeaways:**
- Cloud-to-cloud integration is simplified using SaaS platforms like Fivetran
- Proper planning and configuration reduce troubleshooting time
- Security best practices (service accounts, SSL/TLS) are essential
- Validation ensures data integrity throughout the pipeline

The solution provides a foundation for expanding to more complex data integration scenarios and larger datasets.

---

**Project Status:** ✅ Successfully Completed  
**Implementation Date:** October 2025  
**Implemented By:** Research Student (9 months experience)  
**Organization:** Contractor Research Initiative

