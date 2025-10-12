# Visual Diagrams - SAP BTP to Azure Synapse Integration

This document contains various visual representations of the project architecture and data flow.

---

## 1. High-Level Architecture (Mermaid)

```mermaid
graph TB
    subgraph "Phase A: Source System (SAP BTP)"
        A1[SAP BTP Cockpit]
        A2[HANA Cloud Instance]
        A3[HANA Database Explorer]
        A4[TRAINING Schema]
        A5[6 HR Tables]
        
        A1 --> A2
        A2 --> A3
        A3 --> A4
        A4 --> A5
    end
    
    subgraph "Phase C: Integration Platform (Fivetran)"
        C1[SAP HANA Connector]
        C2[Sync Engine]
        C3[Scheduler]
        C4[Azure Synapse Destination]
        
        C1 --> C2
        C2 --> C3
        C3 --> C4
    end
    
    subgraph "Phase B: Target System (Microsoft Azure)"
        B1[Azure Portal]
        B2[Synapse Workspace]
        B3[Synapse Studio]
        B4[Dedicated SQL Pool]
        B5[Replicated Tables]
        
        B1 --> B2
        B2 --> B3
        B3 --> B4
        B4 --> B5
    end
    
    A5 -->|Extract<br/>via JDBC/SSL| C1
    C4 -->|Load<br/>via TDS/SSL| B4
    
    style A2 fill:#0078d4,color:#fff
    style B2 fill:#0078d4,color:#fff
    style C2 fill:#00b388,color:#fff
```

---

## 2. Data Flow Timeline (Mermaid)

```mermaid
gantt
    title Project Implementation Timeline
    dateFormat YYYY-MM-DD
    section Phase A: HANA Setup
    Access Database Explorer      :a1, 2025-01-01, 10m
    Create TRAINING Schema        :a2, after a1, 5m
    Create 6 Tables              :a3, after a2, 10m
    Import CSV Data              :a4, after a3, 15m
    Validate Data                :a5, after a4, 5m
    Create Service Account       :a6, after a5, 5m
    
    section Phase B: Synapse Setup
    Create Synapse Workspace     :b1, 2025-01-01, 10m
    Create SQL Pool              :b2, after b1, 15m
    Create Service Account       :b3, after b2, 5m
    Configure Firewall           :b4, after b3, 5m
    
    section Phase C: Fivetran Config
    Add Synapse Destination      :c1, after b4, 10m
    Add HANA Connector           :c2, after c1, 10m
    Select Tables                :c3, after c2, 5m
    Initial Sync                 :c4, after c3, 5m
    
    section Phase D: Validation
    Verify Data in Synapse       :d1, after c4, 5m
    Configure Schedule           :d2, after d1, 3m
    Create Reporting Views       :d3, after d2, 10m
```

---

## 3. Component Interaction Diagram (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    participant Admin as Administrator
    participant BTP as SAP BTP
    participant HANA as HANA Cloud
    participant FT as Fivetran
    participant Azure as Azure Portal
    participant Synapse as Synapse Pool
    
    Note over Admin,Synapse: SETUP PHASE
    
    Admin->>BTP: Login to BTP Cockpit
    Admin->>HANA: Open Database Explorer
    Admin->>HANA: Create TRAINING schema
    Admin->>HANA: Create 6 tables
    Admin->>HANA: Import CSV files
    Admin->>HANA: Create TRAINING_USER
    
    Admin->>Azure: Login to Azure Portal
    Admin->>Azure: Create Synapse Workspace
    Admin->>Synapse: Create SQL Pool (DW100c)
    Admin->>Synapse: Create fivetran_user
    Admin->>Synapse: Configure firewall
    
    Note over Admin,Synapse: INTEGRATION PHASE
    
    Admin->>FT: Login to Fivetran
    Admin->>FT: Add Synapse destination
    FT->>Synapse: Test connection
    Synapse-->>FT: Connection OK
    
    Admin->>FT: Add HANA connector
    FT->>HANA: Test connection
    HANA-->>FT: Connection OK
    
    Admin->>FT: Select TRAINING schema tables
    Admin->>FT: Start initial sync
    
    Note over Admin,Synapse: SYNC PHASE
    
    FT->>HANA: Connect & discover schema
    HANA-->>FT: Return table metadata
    FT->>HANA: Extract all rows (SELECT)
    HANA-->>FT: Return data
    FT->>FT: Transform data
    FT->>Synapse: Create schema & tables
    FT->>Synapse: Bulk insert data
    Synapse-->>FT: Load complete
    FT-->>Admin: Sync successful
    
    Note over Admin,Synapse: VALIDATION PHASE
    
    Admin->>Synapse: Run validation queries
    Synapse-->>Admin: Return row counts
    Admin->>FT: Configure schedule (6 hours)
```

---

## 4. Table Relationships (Mermaid)

```mermaid
erDiagram
    DEPARTMENTS ||--o{ EMPLOYEES : "has"
    JOBS ||--o{ EMPLOYEES : "assigned to"
    EMPLOYEES ||--o{ JOB_HISTORY : "has history"
    EMPLOYEES ||--o{ ATTENDANCE : "records"
    EMPLOYEES ||--o{ PAYROLL : "receives"
    JOBS ||--o{ JOB_HISTORY : "in history"
    
    DEPARTMENTS {
        int DEPT_ID PK
        string DEPT_NAME
    }
    
    JOBS {
        string JOB_ID PK
        string JOB_TITLE
        int MIN_SALARY
        int MAX_SALARY
    }
    
    EMPLOYEES {
        int EMP_ID PK
        string FIRST_NAME
        string LAST_NAME
        int DEPT_ID FK
        string JOB_ID FK
        date HIRE_DATE
        int ANNUAL_SALARY
    }
    
    JOB_HISTORY {
        int EMP_ID FK
        string JOB_ID FK
        date START_DATE
        date END_DATE
    }
    
    ATTENDANCE {
        int EMP_ID FK
        date WORK_DATE
        string STATUS
    }
    
    PAYROLL {
        int EMP_ID FK
        date PAY_DATE
        decimal GROSS_PAY
    }
```

---

## 5. Sync Process Flow (Mermaid)

```mermaid
flowchart TD
    Start([Scheduled Sync Triggered]) --> CheckType{First<br/>Sync?}
    
    CheckType -->|Yes - Initial| InitialPhase[Initial Batch Sync]
    CheckType -->|No - Ongoing| IncrementalPhase[Incremental Sync]
    
    InitialPhase --> ConnectHANA1[Connect to HANA Cloud]
    ConnectHANA1 --> DiscoverSchema[Discover TRAINING Schema]
    DiscoverSchema --> GetMetadata[Get Table Structures]
    GetMetadata --> ExtractAllData[Extract All Rows from 6 Tables]
    ExtractAllData --> Transform1[Map Schema & Transform Data]
    Transform1 --> ConnectSynapse1[Connect to Synapse]
    ConnectSynapse1 --> CreateSchema[Create Destination Schema]
    CreateSchema --> CreateTables[Create 6 Tables]
    CreateTables --> BulkLoad[Bulk Insert All Rows]
    BulkLoad --> UpdateState1[Update Sync State]
    UpdateState1 --> Complete1[Initial Sync Complete]
    
    IncrementalPhase --> ConnectHANA2[Connect to HANA Cloud]
    ConnectHANA2 --> CheckChanges[Check for Changes Since Last Sync]
    CheckChanges --> HasChanges{Changes<br/>Detected?}
    
    HasChanges -->|Yes| ExtractDelta[Extract Changed Rows Only]
    HasChanges -->|No| NoAction[No Action Needed]
    
    ExtractDelta --> Transform2[Transform Delta Data]
    Transform2 --> ConnectSynapse2[Connect to Synapse]
    ConnectSynapse2 --> ApplyChanges[Apply Inserts/Updates/Deletes]
    ApplyChanges --> UpdateState2[Update Sync State]
    UpdateState2 --> Complete2[Incremental Sync Complete]
    
    Complete1 --> Schedule[Wait for Next Schedule]
    Complete2 --> Schedule
    NoAction --> Schedule
    Schedule --> End([End - Next Sync in 6 Hours])
    
    style InitialPhase fill:#ffeb3b
    style IncrementalPhase fill:#4caf50
    style Complete1 fill:#00b388
    style Complete2 fill:#00b388
    style NoAction fill:#9e9e9e
```

---

## 6. Network Security Architecture (ASCII)

```
NETWORK SECURITY ARCHITECTURE

┌─────────────────────────────────────────────────────────────────────────┐
│                          SECURITY PERIMETER                              │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐                           ┌──────────────────────────┐
│   SAP BTP Environment    │                           │   Azure Environment      │
│                          │                           │                          │
│  ┌────────────────────┐  │                           │  ┌────────────────────┐  │
│  │ HANA Cloud         │  │                           │  │ Synapse Workspace  │  │
│  │ Firewall           │  │                           │  │ Firewall           │  │
│  │                    │  │                           │  │                    │  │
│  │ Allowed IPs:       │  │                           │  │ Allowed IPs:       │  │
│  │ ✓ Admin IP         │  │                           │  │ ✓ Admin IP         │  │
│  │ ✓ Fivetran IPs     │  │                           │  │ ✓ Fivetran IPs     │  │
│  │ ✗ All Others       │  │                           │  │ ✗ All Others       │  │
│  │                    │  │                           │  │                    │  │
│  └────────┬───────────┘  │                           │  └────────┬───────────┘  │
│           │              │                           │           │              │
│           ▼              │                           │           ▼              │
│  ┌────────────────────┐  │                           │  ┌────────────────────┐  │
│  │ SSL/TLS Layer      │  │◄─────────────┐   ┌───────┼─►│ SSL/TLS Layer      │  │
│  │ Port: 443          │  │              │   │       │  │ Port: 1433         │  │
│  └────────┬───────────┘  │              │   │       │  └────────┬───────────┘  │
│           │              │              │   │       │           │              │
│           ▼              │              │   │       │           ▼              │
│  ┌────────────────────┐  │              │   │       │  ┌────────────────────┐  │
│  │ Authentication     │  │              │   │       │  │ Authentication     │  │
│  │ User: TRAINING_    │  │              │   │       │  │ User: fivetran_    │  │
│  │       USER         │  │              │   │       │  │       user         │  │
│  │ Type: READ-ONLY    │  │              │   │       │  │ Type: DB_OWNER     │  │
│  └────────┬───────────┘  │              │   │       │  └────────┬───────────┘  │
│           │              │              │   │       │           │              │
│           ▼              │              │   │       │           ▼              │
│  ┌────────────────────┐  │              │   │       │  ┌────────────────────┐  │
│  │ TRAINING Schema    │  │              │   │       │  │ sap_hana_training  │  │
│  │ - 6 Tables         │  │              │   │       │  │ - 6 Tables         │  │
│  └────────────────────┘  │              │   │       │  └────────────────────┘  │
└──────────────────────────┘              │   │       └──────────────────────────┘
                                          │   │
                                          │   │
                                ┌─────────┴───┴────────┐
                                │  Fivetran Cloud      │
                                │  Platform            │
                                │                      │
                                │  ┌────────────────┐  │
                                │  │ SAP HANA       │  │
                                │  │ Connector      │  │
                                │  └────────────────┘  │
                                │         │            │
                                │         ▼            │
                                │  ┌────────────────┐  │
                                │  │ Sync Engine    │  │
                                │  │ - Encrypted    │  │
                                │  │ - Monitored    │  │
                                │  └────────────────┘  │
                                │         │            │
                                │         ▼            │
                                │  ┌────────────────┐  │
                                │  │ Azure Synapse  │  │
                                │  │ Destination    │  │
                                │  └────────────────┘  │
                                │                      │
                                │  Egress IPs:         │
                                │  - Must be allowed   │
                                │    in both firewalls │
                                └──────────────────────┘

SECURITY CONTROLS:
├─ SSL/TLS encryption on all connections
├─ Service accounts with minimal privileges
├─ IP allowlisting (deny all except specific IPs)
├─ Strong password policies enforced
├─ Audit logging enabled on both sides
└─ Schema isolation (separate from production data)
```

---

## 7. Data Journey Visualization (Mermaid)

```mermaid
graph LR
    subgraph CSV["CSV Files (Local)"]
        CSV1[departments.csv]
        CSV2[jobs.csv]
        CSV3[employees.csv]
        CSV4[job_history.csv]
        CSV5[attendance.csv]
        CSV6[payroll.csv]
    end
    
    subgraph HANA["SAP HANA Cloud"]
        H1[DEPARTMENTS]
        H2[JOBS]
        H3[EMPLOYEES]
        H4[JOB_HISTORY]
        H5[ATTENDANCE]
        H6[PAYROLL]
    end
    
    subgraph FT["Fivetran Processing"]
        F1[Extract]
        F2[Transform]
        F3[Load]
    end
    
    subgraph Synapse["Azure Synapse"]
        S1[departments]
        S2[jobs]
        S3[employees]
        S4[job_history]
        S5[attendance]
        S6[payroll]
    end
    
    CSV1 -->|Import| H1
    CSV2 -->|Import| H2
    CSV3 -->|Import| H3
    CSV4 -->|Import| H4
    CSV5 -->|Import| H5
    CSV6 -->|Import| H6
    
    H1 & H2 & H3 & H4 & H5 & H6 -->|SELECT| F1
    F1 --> F2
    F2 --> F3
    F3 -->|INSERT| S1 & S2 & S3 & S4 & S5 & S6
    
    style CSV fill:#fff4e6
    style HANA fill:#e3f2fd
    style FT fill:#e8f5e9
    style Synapse fill:#f3e5f5
```

---

## 8. Phase-by-Phase Implementation (ASCII)

```
PROJECT IMPLEMENTATION PHASES

╔═══════════════════════════════════════════════════════════════════════╗
║                        PHASE A: SOURCE PREPARATION                     ║
║                           (SAP HANA on BTP)                           ║
╚═══════════════════════════════════════════════════════════════════════╝

┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Step 1    │ ───► │   Step 2    │ ───► │   Step 3    │ ───► │   Step 4    │
│             │      │             │      │             │      │             │
│   Access    │      │   Create    │      │   Create    │      │   Import    │
│   Database  │      │   Schema    │      │   Tables    │      │   CSV Data  │
│   Explorer  │      │             │      │   (6)       │      │   (6 files) │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
                                                                        │
                                                                        ▼
┌─────────────┐      ┌─────────────┐                          ┌─────────────┐
│   Step 6    │ ◄─── │   Step 5    │ ◄─────────────────────── │   Step 4    │
│             │      │             │                          │   Complete  │
│   Create    │      │   Validate  │                          └─────────────┘
│   Service   │      │   Data      │
│   Account   │      │             │
└─────────────┘      └─────────────┘

Result: ✓ HANA schema ready with 6 populated tables


╔═══════════════════════════════════════════════════════════════════════╗
║                   PHASE B: DESTINATION PREPARATION                     ║
║                      (Azure Synapse Analytics)                        ║
╚═══════════════════════════════════════════════════════════════════════╝

┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Step 7    │ ───► │   Step 8    │ ───► │   Step 9    │ ───► │   Step 10   │
│             │      │             │      │             │      │             │
│   Create    │      │   Create    │      │   Create    │      │   Configure │
│   Synapse   │      │   SQL Pool  │      │   Service   │      │   Network   │
│   Workspace │      │   (DW100c)  │      │   Account   │      │   Firewall  │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘

Result: ✓ Synapse SQL pool ready to receive data


╔═══════════════════════════════════════════════════════════════════════╗
║                     PHASE C: FIVETRAN CONFIGURATION                    ║
║                         (Integration Setup)                            ║
╚═══════════════════════════════════════════════════════════════════════╝

┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Step 11   │ ───► │   Step 12   │ ───► │   Step 13   │ ───► │   Step 14   │
│             │      │             │      │             │      │             │
│   Add       │      │   Add       │      │   Select    │      │   Run       │
│   Synapse   │      │   HANA      │      │   6 Tables  │      │   Initial   │
│   Destination│     │   Connector │      │   to Sync   │      │   Sync      │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘

Result: ✓ Data replicated from HANA to Synapse


╔═══════════════════════════════════════════════════════════════════════╗
║                  PHASE D: VALIDATION & SCHEDULING                      ║
║                    (Ongoing Operations Setup)                          ║
╚═══════════════════════════════════════════════════════════════════════╝

┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Step 15   │ ───► │   Step 16   │ ───► │   Step 17   │
│             │      │             │      │             │
│   Verify    │      │   Schedule  │      │   Create    │
│   Data in   │      │   Syncs     │      │   Reporting │
│   Synapse   │      │   (6 hours) │      │   Views     │
└─────────────┘      └─────────────┘      └─────────────┘

Result: ✓ Automated pipeline operational


═══════════════════════════════════════════════════════════════════════
                          PROJECT COMPLETE ✓
═══════════════════════════════════════════════════════════════════════
```

---

## 9. Technology Stack Visualization (Mermaid)

```mermaid
graph TD
    subgraph "Presentation Layer"
        P1[Power BI]
        P2[Excel]
        P3[Custom Apps]
    end
    
    subgraph "Analytics Layer (Azure)"
        A1[Synapse Studio]
        A2[SQL Scripts]
        A3[Reporting Views]
        A4[Dedicated SQL Pool DW100c]
    end
    
    subgraph "Integration Layer (SaaS)"
        I1[Fivetran Dashboard]
        I2[SAP HANA Connector]
        I3[Sync Scheduler]
        I4[Azure Synapse Destination]
    end
    
    subgraph "Data Layer (SAP BTP)"
        D1[HANA Database Explorer]
        D2[TRAINING Schema]
        D3[Column Store Tables]
        D4[HANA Cloud Instance]
    end
    
    subgraph "Source Layer"
        S1[CSV Files]
        S2[departments.csv]
        S3[jobs.csv]
        S4[employees.csv]
        S5[job_history.csv]
        S6[attendance.csv]
        S7[payroll.csv]
    end
    
    S1 --> S2 & S3 & S4 & S5 & S6 & S7
    S2 & S3 & S4 & S5 & S6 & S7 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4
    D4 --> I2
    I2 --> I3
    I3 --> I4
    I4 --> A4
    A4 --> A3
    A3 --> A2
    A2 --> A1
    A1 --> P1 & P2 & P3
    
    style I1 fill:#00b388
    style I2 fill:#00b388
    style I3 fill:#00b388
    style I4 fill:#00b388
    style D4 fill:#0078d4
    style A4 fill:#0078d4
```

---

## 10. Sync Frequency Options (ASCII)

```
BATCH SYNC FREQUENCY OPTIONS

┌─────────────────────────────────────────────────────────────┐
│                   SYNC FREQUENCY MATRIX                      │
└─────────────────────────────────────────────────────────────┘

Option 1: EVERY 6 HOURS (Recommended for this project)
├─ 00:00 ────► Sync ────► 06:00 ────► Sync ────► 12:00
│                                                   │
│                                                   ▼
└─ 24:00 ◄──── Sync ◄──── 18:00 ◄──── Sync ◄──── 12:00

Pros: Fresh data 4x per day
Cons: Uses more Fivetran quota


Option 2: DAILY (Once per day)
└─ 02:00 AM ────► Sync ────► [Next day] 02:00 AM ────► Sync

Pros: Lower cost, less resource usage
Cons: Data could be 24 hours old


Option 3: CUSTOM (As needed)
└─ Configured based on business requirements

Examples:
  - Every hour during business hours
  - Multiple times during peak season
  - Once per week for static data


CURRENT PROJECT SETTING: Every 6 hours ⏰
RATIONALE: Balances freshness with cost for demo project
```

---

## 11. Error Handling Flow (Mermaid)

```mermaid
flowchart TD
    Start([Sync Triggered]) --> Connect{Can Connect<br/>to Source?}
    
    Connect -->|No| Error1[Connection Error]
    Connect -->|Yes| Extract{Can Extract<br/>Data?}
    
    Error1 --> Log1[Log Error]
    Log1 --> Retry1{Retry<br/>Attempt < 3?}
    Retry1 -->|Yes| Start
    Retry1 -->|No| Alert1[Send Alert]
    Alert1 --> End1([Sync Failed])
    
    Extract -->|No| Error2[Extraction Error]
    Extract -->|Yes| Load{Can Load<br/>to Target?}
    
    Error2 --> Log2[Log Error]
    Log2 --> Retry2{Retry<br/>Attempt < 3?}
    Retry2 -->|Yes| Extract
    Retry2 -->|No| Alert2[Send Alert]
    Alert2 --> End2([Sync Failed])
    
    Load -->|No| Error3[Load Error]
    Load -->|Yes| Validate{Data<br/>Valid?}
    
    Error3 --> Log3[Log Error]
    Log3 --> Retry3{Retry<br/>Attempt < 3?}
    Retry3 -->|Yes| Load
    Retry3 -->|No| Alert3[Send Alert]
    Alert3 --> End3([Sync Failed])
    
    Validate -->|No| Rollback[Rollback Changes]
    Validate -->|Yes| Success[Update Sync State]
    
    Rollback --> Alert4[Send Alert]
    Alert4 --> End4([Sync Failed - Rolled Back])
    
    Success --> End5([Sync Successful])
    
    style Error1 fill:#f44336,color:#fff
    style Error2 fill:#f44336,color:#fff
    style Error3 fill:#f44336,color:#fff
    style Success fill:#4caf50,color:#fff
    style End5 fill:#4caf50,color:#fff
```

---

## 12. Project Success Metrics Dashboard (ASCII)

```
╔═══════════════════════════════════════════════════════════════╗
║              PROJECT SUCCESS METRICS DASHBOARD                 ║
╠═══════════════════════════════════════════════════════════════╣
║                                                               ║
║  SOURCE SYSTEM (SAP HANA)                                     ║
║  ┌───────────────────────────────────────────────────────┐   ║
║  │ Schema: TRAINING                            ✓ Online  │   ║
║  │ Tables: 6                                   ✓ Loaded  │   ║
║  │ Total Rows: ~500                            ✓ Valid   │   ║
║  │ Data Size: < 1 MB                                     │   ║
║  └───────────────────────────────────────────────────────┘   ║
║                                                               ║
║  INTEGRATION PLATFORM (FIVETRAN)                              ║
║  ┌───────────────────────────────────────────────────────┐   ║
║  │ Connector Status: Active                    ✓ Running │   ║
║  │ Last Sync: 2 hours ago                      ✓ Success │   ║
║  │ Next Sync: In 4 hours                                 │   ║
║  │ Sync Frequency: Every 6 hours                         │   ║
║  │ Rows Synced Today: 0 (no changes)                     │   ║
║  │ Initial Load: 2-5 minutes                   ✓ Complete│   ║
║  └───────────────────────────────────────────────────────┘   ║
║                                                               ║
║  TARGET SYSTEM (AZURE SYNAPSE)                                ║
║  ┌───────────────────────────────────────────────────────┐   ║
║  │ SQL Pool: fivetran_demo_pool                ✓ Online  │   ║
║  │ Schema: sap_hana_training                   ✓ Created │   ║
║  │ Tables: 6                                   ✓ Loaded  │   ║
║  │ Total Rows: ~500                            ✓ Match   │   ║
║  │ Data Freshness: 2 hours old                           │   ║
║  └───────────────────────────────────────────────────────┘   ║
║                                                               ║
║  DATA QUALITY                                                 ║
║  ┌───────────────────────────────────────────────────────┐   ║
║  │ Row Count Match:             100%           ✓ Pass    │   ║
║  │ Primary Keys Valid:          100%           ✓ Pass    │   ║
║  │ Foreign Keys Valid:          100%           ✓ Pass    │   ║
║  │ Null Values Expected:        Yes            ✓ Pass    │   ║
║  │ Data Types Preserved:        Yes            ✓ Pass    │   ║
║  └───────────────────────────────────────────────────────┘   ║
║                                                               ║
║  PROJECT STATUS: ✓ SUCCESSFULLY COMPLETED                     ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## 13. Before & After Comparison (ASCII)

```
BEFORE vs AFTER PROJECT IMPLEMENTATION

╔════════════════════════════════════════╗  ╔════════════════════════════════════════╗
║           BEFORE PROJECT               ║  ║           AFTER PROJECT                ║
╠════════════════════════════════════════╣  ╠════════════════════════════════════════╣
║                                        ║  ║                                        ║
║  Data Location:                        ║  ║  Data Location:                        ║
║  └─ SAP HANA only                      ║  ║  ├─ SAP HANA (source)                  ║
║                                        ║  ║  └─ Azure Synapse (analytics)          ║
║                                        ║  ║                                        ║
║  Data Access:                          ║  ║  Data Access:                          ║
║  └─ HANA Database Explorer only        ║  ║  ├─ HANA Database Explorer             ║
║                                        ║  ║  ├─ Synapse Studio                     ║
║                                        ║  ║  ├─ Power BI                           ║
║                                        ║  ║  └─ Any SQL client                     ║
║                                        ║  ║                                        ║
║  Analytics Capability:                 ║  ║  Analytics Capability:                 ║
║  └─ Limited to HANA                    ║  ║  └─ Full Azure ecosystem               ║
║                                        ║  ║                                        ║
║  Data Refresh:                         ║  ║  Data Refresh:                         ║
║  └─ Manual effort required             ║  ║  └─ Automated every 6 hours            ║
║                                        ║  ║                                        ║
║  Integration:                          ║  ║  Integration:                          ║
║  └─ None                               ║  ║  └─ Fivetran pipeline                  ║
║                                        ║  ║                                        ║
║  Reporting:                            ║  ║  Reporting:                            ║
║  └─ Manual SQL queries                 ║  ║  └─ Automated dashboards possible      ║
║                                        ║  ║                                        ║
║  Data Freshness:                       ║  ║  Data Freshness:                       ║
║  └─ Real-time in HANA only             ║  ║  └─ Within 6 hours in Synapse          ║
║                                        ║  ║                                        ║
║  Scalability:                          ║  ║  Scalability:                          ║
║  └─ Limited by HANA instance           ║  ║  └─ Scalable DWU in Synapse            ║
║                                        ║  ║                                        ║
╚════════════════════════════════════════╝  ╚════════════════════════════════════════╝

TRANSFORMATION ACHIEVED: From siloed data → Integrated analytics platform
```

---

## Legend & Symbols

```
DIAGRAM SYMBOLS USED

┌─────┐
│ Box │  = Component or System
└─────┘

───►     = Data Flow Direction
◄───     = Reverse Direction

║        = Vertical Border
═        = Horizontal Border (emphasis)

✓        = Success / Completed
✗        = Failed / Blocked
⏰        = Scheduled / Timer
🔒       = Secured / Encrypted

[Component]  = Mermaid syntax for nodes
-->          = Mermaid syntax for flow
```

---

**Document Purpose:** Visual reference for project architecture and data flows  
**Diagram Types:** ASCII (text-based) and Mermaid (rendered graphics)  
**Usage:** Present in interviews, documentation, and portfolio  

---

**Created for:** Research Student (9 months experience)  
**Project:** SAP BTP to Azure Synapse Integration via Fivetran  
**Last Updated:** October 2025

