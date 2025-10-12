# Quick Reference Guide

## One-Page Project Summary

**Project:** Batch Dataset Preparation and Integration to Fivetran  
**Path:** SAP HANA Cloud (BTP) → Fivetran → Azure Synapse Analytics  
**Role:** Research Student for Contractor  
**Experience:** 9 months  

---

## 30-Second Summary

Built an automated data integration pipeline that replicates HR data from SAP HANA Cloud to Azure Synapse using Fivetran. Configured 6 tables for batch synchronization every 6 hours, implementing security best practices and data validation.

---

## Core Components

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Source** | SAP HANA Cloud (BTP) | Operational HR database |
| **Integration** | Fivetran | Automated ETL platform |
| **Target** | Azure Synapse (DW100c) | Analytics data warehouse |

---

## Data Tables (6 Total)

1. **DEPARTMENTS** - Department master data
2. **JOBS** - Job titles and salary ranges
3. **EMPLOYEES** - Employee records
4. **JOB_HISTORY** - Job change history
5. **ATTENDANCE** - Daily attendance logs
6. **PAYROLL** - Payroll transactions

---

## Implementation Phases

```
A. HANA Setup (30-45 min)
   → Create schema → Create tables → Import CSVs → Create user

B. Synapse Setup (30-45 min)
   → Create workspace → Create SQL pool → Configure firewall

C. Fivetran Config (20-30 min)
   → Add destination → Add connector → Select tables → Sync

D. Validation (15-20 min)
   → Verify data → Schedule syncs → Create views
```

---

## Key Credentials

**SAP HANA:**
- User: `TRAINING_USER`
- Privilege: SELECT on TRAINING schema (read-only)
- Connection: SSL/TLS, port 443

**Azure Synapse:**
- User: `fivetran_user`
- Privilege: db_owner (for demo)
- Connection: SSL/TLS, port 1433

---

## Security Measures

✅ SSL/TLS encryption on all connections  
✅ Firewall IP allowlisting (Fivetran egress IPs)  
✅ Service accounts with minimal privileges  
✅ Strong password policies  
✅ Schema isolation  

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Data Volume | < 1 MB |
| Initial Sync | 2-5 minutes |
| Incremental Sync | 30-60 seconds |
| Sync Frequency | Every 6 hours |
| Tables | 6 |
| Total Rows | ~500 |

---

## Technical Skills Demonstrated

- SAP HANA Cloud administration
- Azure Synapse provisioning
- Fivetran connector configuration
- Database schema design
- Data import/export
- SQL query writing
- Network security configuration
- Service account management
- Data validation
- Documentation

---

## Common Interview Questions - Quick Answers

**Q: What challenge did you face?**  
A: Firewall configuration - resolved by allowlisting Fivetran IPs

**Q: How did you validate data?**  
A: Row count comparison, join queries, date range checks

**Q: Why batch vs real-time?**  
A: Cost-effective for non-critical data with acceptable 6-hour latency

**Q: How do you handle schema changes?**  
A: Fivetran auto-detects and adds new columns

**Q: What's your biggest learning?**  
A: Cloud integration is simplified with SaaS platforms, but network and security configuration are critical

---

## Architecture in 3 Lines

```
HANA (source) ──[JDBC/SSL]──► Fivetran (ETL) ──[TDS/SSL]──► Synapse (target)
  6 tables                    Batch sync                     6 tables
  READ-ONLY                   Every 6 hours                  DB_OWNER
```

---

## Troubleshooting Checklist

**Connection Fails:**
- [ ] Check firewall rules
- [ ] Verify credentials
- [ ] Confirm SSL/TLS enabled
- [ ] Test with SQL client

**Data Mismatch:**
- [ ] Check sync status
- [ ] Verify primary keys exist
- [ ] Review Fivetran logs
- [ ] Compare row counts

**Sync Slow:**
- [ ] Check network latency
- [ ] Review table sizes
- [ ] Verify indexes exist
- [ ] Monitor resource usage

---

## Key URLs & Tools

**SAP BTP:**
- BTP Cockpit: https://cockpit.btp.cloud.sap/
- HANA Database Explorer: (via Actions menu)

**Azure:**
- Azure Portal: https://portal.azure.com
- Synapse Studio: (via workspace link)

**Fivetran:**
- Dashboard: https://fivetran.com/dashboard

---

## File Organization

```
Project Documentation/
├── README.md                  Main project overview
├── ARCHITECTURE.md            Detailed architecture with diagrams
├── PROJECT_FLOW.md            Step-by-step implementation guide
├── INTERVIEW_GUIDE.md         Interview preparation Q&A
├── VISUAL_DIAGRAMS.md         All diagrams (ASCII + Mermaid)
└── QUICK_REFERENCE.md         This file - quick lookup
```

---

## Sync Process (Simplified)

```
1. Schedule triggers sync (every 6 hours)
2. Fivetran connects to HANA
3. Extracts changed rows (or all rows if first sync)
4. Transforms data (schema mapping)
5. Loads into Synapse
6. Updates sync state
7. Waits for next schedule
```

---

## Data Model Relationships

```
DEPARTMENTS ─┐
             ├──► EMPLOYEES ─┬──► JOB_HISTORY
JOBS ────────┘               ├──► ATTENDANCE
                             └──► PAYROLL
```

---

## Commands Reference (What Was Run)

### HANA Setup:
1. Run SQL: Create schema
2. Run SQL: Create 6 tables
3. Import CSV files (via GUI)
4. Run SQL: Create service user
5. Run SQL: Grant SELECT privileges
6. Run SQL: Validate data

### Synapse Setup:
1. Azure Portal: Create workspace
2. Azure Portal: Create SQL pool
3. Run SQL: Create login
4. Run SQL: Create user
5. Run SQL: Grant db_owner role
6. Azure Portal: Configure firewall

### Fivetran Setup:
1. Web UI: Add destination (Synapse)
2. Web UI: Test connection
3. Web UI: Add connector (HANA)
4. Web UI: Test connection
5. Web UI: Select tables
6. Web UI: Start sync
7. Web UI: Configure schedule

### Validation:
1. Run SQL: Count rows per table
2. Run SQL: Test joins
3. Run SQL: Check date ranges
4. Run SQL: Create views (optional)

---

## Project Success Criteria ✅

- [x] HANA schema created with 6 tables
- [x] CSV data imported successfully
- [x] Synapse workspace and SQL pool provisioned
- [x] Fivetran pipeline established
- [x] Initial sync completed
- [x] Data validated in target
- [x] Scheduled sync configured
- [x] Documentation completed
- [x] Security implemented

---

## Cost Considerations

**SAP HANA Cloud:**
- Free tier or trial for learning
- Small instance size

**Azure Synapse:**
- DW100c = lowest cost tier
- Can pause when not in use
- Azure for Students credits available

**Fivetran:**
- Free trial for development
- Production pricing based on Monthly Active Rows (MAR)

---

## Next Steps After Interview

If project approved:
1. Scale to production data volumes
2. Implement data transformations
3. Build Power BI dashboards
4. Set up monitoring and alerts
5. Document operational procedures
6. Train users on data access

---

## Talking Points Summary

### What I Built
"An automated batch data integration pipeline using Fivetran to replicate HR data from SAP HANA Cloud to Azure Synapse Analytics."

### Why It Matters
"Enables centralized analytics without custom code, reduces manual effort, and provides reliable data synchronization."

### What I Learned
"Hands-on experience with cloud platforms, integration patterns, security best practices, and troubleshooting skills."

### My Approach
"Systematic implementation: prepare source, prepare target, configure integration, validate thoroughly."

### Challenges Overcome
"Network connectivity, firewall configuration, data validation - all resolved through methodical troubleshooting."

---

## 5 Key Achievements

1. ✅ Configured enterprise cloud platforms (SAP BTP, Azure)
2. ✅ Implemented automated data integration pipeline
3. ✅ Applied security best practices (encryption, access control)
4. ✅ Validated data integrity across systems
5. ✅ Documented complete architecture and process

---

## Confidence Boosters

Remember:
- You worked with **real enterprise platforms**
- You implemented **end-to-end integration**
- You solved **actual technical challenges**
- You followed **industry best practices**
- You created **professional documentation**

This is **genuine experience** that demonstrates capability!

---

## One-Sentence Project Description

"Built an automated ETL pipeline using Fivetran to batch-replicate HR data from SAP HANA Cloud to Azure Synapse Analytics with 6-hour sync frequency."

---

## Resources for Further Learning

**SAP BTP:**
- SAP Learning Hub
- SAP Community tutorials
- HANA Cloud documentation

**Azure Synapse:**
- Microsoft Learn modules
- Azure documentation
- Synapse best practices

**Fivetran:**
- Fivetran documentation
- Connector setup guides
- Integration patterns

**General:**
- Data integration patterns
- Cloud security best practices
- ETL vs ELT concepts

---

## Printable Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│                  PROJECT CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────┤
│ Project: SAP BTP to Azure Synapse via Fivetran             │
│ Tables: 6 HR tables (DEPARTMENTS, JOBS, EMPLOYEES, etc.)   │
│ Sync: Every 6 hours, batch mode                            │
│ Security: SSL/TLS, service accounts, IP allowlisting       │
│ Source: SAP HANA Cloud (TRAINING schema)                   │
│ Target: Azure Synapse (fivetran_demo_pool)                 │
│ Time: ~2 hours total implementation                        │
│ Experience: 9 months, research student                     │
├─────────────────────────────────────────────────────────────┤
│ Key Success Factors:                                        │
│ • Firewall configuration critical                          │
│ • Service accounts with minimal privileges                 │
│ • Data validation essential                                │
│ • Documentation for maintainability                        │
└─────────────────────────────────────────────────────────────┘
```

---

**Remember:** You successfully completed a real-world cloud integration project. Be confident and articulate your experience clearly!

Good luck! 🚀

