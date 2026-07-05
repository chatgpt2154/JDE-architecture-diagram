# JDE Architecture Specification & Implementation Guide

**Version:** 2.0  
**Date:** July 2026  
**Status:** Production Ready  
**Based on:** Upgrade_Build Specification_v0.2

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Architecture Overview](#architecture-overview)
3. [Server Components](#server-components)
4. [Environment Segregation](#environment-segregation)
5. [Network & Firewall Configuration](#network--firewall-configuration)
6. [Software Stack & Requirements](#software-stack--requirements)
7. [Security & Access Controls](#security--access-controls)
8. [Deployment Matrix](#deployment-matrix)
9. [Implementation Checklist](#implementation-checklist)

---

## Executive Summary

This document provides a comprehensive specification for the JD Edwards EnterpriseOne (JDE) infrastructure upgrade. The architecture consists of **11 distinct server types** deployed across **Non-Production**, **Production**, and **Infrastructure** layers, utilizing **Windows Server 2025** as the standardized operating system platform.

### Key Statistics
- **Total Server Types:** 11
- **Total Server Instances:** ~16 (including dev clients)
- **Primary Database:** Microsoft SQL Server 2022
- **Application Server:** WebLogic 14.1.2.0.0
- **Operating System:** Windows x64 (64-bit)
- **JDK Versions:** 17.0.14 / 1.8.0_491

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         JDE INFRASTRUCTURE LAYERS                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  DEVELOPMENT LAYER                                                 │    │
│  │  • 3x Development Clients (CNC-1, DEV-2)                          │    │
│  │  • Microsoft Visual Studio C++ 2022                               │    │
│  │  • JDK 17.0.14 / 1.8.0_491                                        │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  INFRASTRUCTURE SUPPORT LAYER                                      │    │
│  │  • One-Click Provisioning Server (1x)                             │    │
│  │  • Deployment Server (1x) - Oracle DB 19.0                        │    │
│  │  • Apache Load Balancer (1x)                                      │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────────┐        ┌──────────────────────────┐         │
│  │  NON-PRODUCTION LAYER    │        │  PRODUCTION LAYER        │         │
│  │                          │        │                          │         │
│  │  • Enterprise Server     │        │  • Enterprise Server     │         │
│  │    (4 CPU, 16GB RAM)     │        │    (8 CPU, 16GB RAM)     │         │
│  │                          │        │                          │         │
│  │  • WebLogic JAS (1x)     │        │  • WebLogic JAS (2x)     │         │
│  │  • WebLogic AIS (1x)     │        │  • WebLogic AIS (2x)     │         │
│  │                          │        │  • Batch Server          │         │
│  │  • Database Server       │        │    (6 CPU, 16GB RAM)     │         │
│  │    (SQL Server 2022)     │        │                          │         │
│  │                          │        │  • Database Server       │         │
│  │                          │        │    (SQL Server 2022)     │         │
│  └──────────────────────────┘        └──────────────────────────┘         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Server Components

### 1. One-Click Provisioning Server

**Purpose:** Automated infrastructure provisioning and configuration management

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | 2 |
| **Memory** | 30 GB |
| **Storage** | 100 GB (C:) |
| **OS** | Windows x64 2025 |
| **Inbound Ports** | 445, 3000, 3389, 5150, 5985, 8998-8999, 7000-7001, 14501-14502 |
| **Key Role** | Infrastructure automation, environment setup |
| **Service Accounts** | IT to provide |
| **Remarks** | High memory for provisioning workloads |

**Installation Requirements:**
- Remote Desktop Protocol (RDP) enabled
- Windows Management Framework (WMF) 5.1+
- PowerShell 5.0 or higher
- Network access to all target servers

---

### 2. Deployment Server

**Purpose:** Central deployment engine and orchestration platform

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | 2 |
| **Memory** | 16 GB |
| **Storage** | C: 100 GB, D: 400 GB |
| **OS** | Windows x64 2025 |
| **Database** | Oracle Database 19.0 (E1local) |
| **DB Client** | SQL Server 2022 (SSMS) |
| **Compiler** | Microsoft Visual Studio C++ 2022 |
| **JDK** | 17.0.14 / 1.8.0_491 |
| **Windows SDK** | 10.0.26100.x |
| **Inbound Ports** | 445, 3389, 5150, 5985, 6017-6022, 14501-14510 |
| **Service Accounts** | Same as ENT servers |
| **Remarks** | Dual-drive for deployment artifacts & logs |

**Installation Requirements:**
- Oracle 19.0 database with E1local schema
- SQL Server Management Studio (SSMS)
- Microsoft Visual Studio C++ 2022 runtime and compiler
- 400 GB D: drive for deployment artifacts, rollback packages

---

### 3. Database Server (Non-Production)

**Purpose:** SQL Server 2022 for testing, staging, and development workloads

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | DBA-defined |
| **Memory** | DBA-defined |
| **Storage** | DBA-defined |
| **OS** | Windows x64 2025 |
| **Database** | Microsoft SQL Server 2022 |
| **Inbound Ports** | 445, 3389, 5150, 5985, <NEW_DB_PORT>, 14501-14510 |
| **DB Client** | None (database role) |
| **Service Accounts** | As-Is |
| **Remarks** | Configuration to be provided by DBA team |

**Important Notes:**
- Database port to be determined by DBA (must be documented in firewall rules)
- Standard SQL Server ports: 1433 (default), or custom if configured
- Password restrictions: Only `_ @ ~ + - *` special characters allowed
- Backup strategy must be defined and tested before go-live

---

### 4. Database Server (Production)

**Purpose:** SQL Server 2022 for production transactional workloads

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | DBA-defined |
| **Memory** | DBA-defined |
| **Storage** | DBA-defined |
| **OS** | Windows x64 2025 |
| **Database** | Microsoft SQL Server 2022 |
| **Inbound Ports** | 445, 3389, 5150, 5985, <NEW_DB_PORT>, 14501-14510 |
| **Service Accounts** | As-Is |
| **Remarks** | HA/DR configuration required |

**Critical Requirements:**
- High-availability setup (clustering, mirroring, or Always On)
- Comprehensive backup and recovery plan
- Database port must be unique and not conflict with other services
- Dedicated storage with RAID configuration for data files
- Transaction log on separate disk for performance

---

### 5. Enterprise Server (Non-Production)

**Purpose:** JDE application server for testing and validation

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | 4 |
| **Memory** | 16 GB |
| **Storage** | C: 100 GB, D: 120 GB, E: 10 GB |
| **OS** | Windows x64 2025 |
| **DB Client** | SQL Server 2022 (SSMS) |
| **Compiler** | Microsoft Visual Studio C++ 2022 |
| **JDK** | 17.0.14 / 1.8.0_491 |
| **Windows SDK** | 10.0.26100.x |
| **Inbound Ports** | 445, 3389, 5150, 5985, 6017-6022, 14501-14510 |
| **Service Accounts** | As-Is |

**Storage Allocation:**
- **C: 100 GB** — Operating system and system files
- **D: 120 GB** — JDE application files and program binaries
- **E: 10 GB** — Temporary files and caching

---

### 6. Enterprise Server (Production)

**Purpose:** JDE application server for production workloads

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | 8 |
| **Memory** | 16 GB |
| **Storage** | C: 100 GB, D: 120 GB, E: 10 GB |
| **OS** | Windows x64 2025 |
| **DB Client** | SQL Server 2022 (SSMS) |
| **Compiler** | Microsoft Visual Studio C++ 2022 |
| **JDK** | 17.0.14 / 1.8.0_491 |
| **Windows SDK** | 10.0.26100.x |
| **Inbound Ports** | 445, 3389, 5150, 5985, 6017-6022, 14501-14510 |
| **Service Accounts** | As-Is |

**Key Differences from Non-Prod:**
- **Higher CPU:** 8 cores (vs. 4 in non-prod) for production load handling
- Same memory and storage to maintain consistency

---

### 7. Batch Server (Production)

**Purpose:** Scheduled batch job processing and report generation

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **CPU Cores** | 6 |
| **Memory** | 16 GB |
| **Storage** | C: 100 GB, D: 100 GB, E: 10 GB |
| **OS** | Windows x64 2025 |
| **DB Client** | SQL Server 2022 (SSMS) |
| **Compiler** | Microsoft Visual Studio C++ 2022 |
| **JDK** | 17.0.14 / 1.8.0_491 |
| **Windows SDK** | 10.0.26100.x |
| **Inbound Ports** | 445, 3389, 5150, 5985, 6017-6022, 14501-14510 |
| **Service Accounts** | As-Is |

**Storage Allocation:**
- **C: 100 GB** — Operating system
- **D: 100 GB** — Batch job outputs and reports
- **E: 10 GB** — Temporary processing files

---

### 8. WebLogic Server (JAS) - Java Application Server

**Purpose:** Hosts Java-based application components and web services

| Specification | Details |
|---|---|
| **Quantity** | 3 total (1 non-prod, 2 prod) |
| **CPU Cores** | 4 per instance |
| **Memory** | 16 GB per instance |
| **Storage** | C: 100 GB, D: 100 GB |
| **OS** | Windows x64 2025 |
| **WebLogic Version** | 14.1.2.0.0 |
| **JDK** | 17.0.14 |
| **Inbound Ports** | 445, 3389, 5150, 5985, 8998-8999, <WLS_PORTS>, 14501-14520 |
| **Web Client** | Edge (145.*, 139.*) |
| **Service Accounts** | As-Is |

**Configuration:**
- **Non-Production Instance (1x):**
  - Development and testing of Java components
  - Load balanced with prod instances
  
- **Production Instances (2x):**
  - Active-active cluster configuration
  - Behind Apache Load Balancer
  - SSL certificates required

**Port Notes:**
- WebLogic ports must be **> 1024**
- Typical range: 8998-8999 (or custom as per policy)
- Admin console port: typically 7001-7002
- **Important:** Cannot use special characters `$` or `!` in WebLogic passwords

---

### 9. WebLogic Server (AIS) - Application Integration Server

**Purpose:** Handles application integrations and external system communications

| Specification | Details |
|---|---|
| **Quantity** | 3 total (1 non-prod, 2 prod) |
| **CPU Cores** | 4 per instance |
| **Memory** | 8 GB per instance |
| **Storage** | C: 100 GB, D: 100 GB |
| **OS** | Windows x64 2025 |
| **WebLogic Version** | 14.1.2.0.0 |
| **JDK** | 17.0.14 |
| **Inbound Ports** | 445, 3389, 5150, 5985, <WLS_PORTS>, 14501-14520 |
| **Web Client** | Edge (145.*, 139.*) |
| **Service Accounts** | As-Is |

**Configuration:**
- **Non-Production Instance (1x):**
  - Integration testing environment
  
- **Production Instances (2x):**
  - Active-active cluster
  - Handles B2B, EDI, and third-party integrations
  - Behind Apache Load Balancer

---

### 10. Development Client

**Purpose:** Developer workstations for application development and testing

| Specification | Details |
|---|---|
| **Quantity** | 3 total (CNC: 1, DEV: 2) |
| **CPU Cores** | 2 minimum |
| **Memory** | 8 GB minimum |
| **Storage** | C: 100 GB, D: 100 GB |
| **OS** | Windows (2025, 2022, 11) |
| **Compiler** | Microsoft Visual Studio C++ 2022 |
| **JDK** | 17.0.14 / 1.8.0_491 |
| **Windows SDK** | 10.0.26100.x |
| **DB Client** | Microsoft SQL Server 2022 (SSMS) |
| **Inbound Ports** | 445, 3389, 5150, 5985, 6017-6022 |

**Allocation:**
- **CNC Client (1x):** Change Notification Center workstation
- **DEV Clients (2x):** General development workstations

---

### 11. Apache Load Balancer

**Purpose:** Distributes incoming requests across WebLogic clusters

| Specification | Details |
|---|---|
| **Quantity** | 1 |
| **Role** | Load balancing, SSL termination, failover |
| **Backend Targets** | JAS (2x prod), AIS (2x prod) |
| **Configuration** | Round-robin or weighted distribution |
| **SSL Certificates** | Required for new web components |
| **High Availability** | Configuration recommended |

**Responsibilities (TBD):**
- Firewall settings for load balancer communications
- Port forwarding rules for JAS/AIS backends
- SSL certificate management and renewal

---

## Environment Segregation

### Non-Production Environment

**Purpose:** Development, testing, integration, and staging

**Components:**
```
Non-Prod Database Server (SQL Server 2022)
         ↓
Non-Prod Enterprise Server (4 CPU, 16 GB)
         ↓
WebLogic JAS (1 instance, 4 CPU, 16 GB)
         ↓
WebLogic AIS (1 instance, 4 CPU, 8 GB)
         ↓
Development Clients (2x DEV workstations)
```

**Characteristics:**
- Allows code changes and testing
- Mirrors production architecture but with reduced resources
- Integrated with CI/CD pipelines
- Data refresh from production recommended

---

### Production Environment

**Purpose:** Live transactional workloads and critical business processes

**Components:**
```
Production Database Server (SQL Server 2022, HA/DR)
         ↓
Production Enterprise Server (8 CPU, 16 GB)
         ↓
Batch Server (6 CPU, 16 GB) ← Scheduled jobs
         ↓
Apache Load Balancer
    ↙              ↘
WebLogic JAS (2x)  WebLogic AIS (2x)
(4 CPU, 16 GB)     (4 CPU, 8 GB)
```

**Characteristics:**
- Change-controlled environment
- High availability and disaster recovery
- Monitoring and alerting enabled
- Backup and restore procedures tested
- Service accounts restricted (As-Is)

---

### Infrastructure Layer

**Purpose:** Support services for provisioning, deployment, and development

**Components:**
```
One-Click Provisioning Server (2 CPU, 30 GB)
         ↓
Deployment Server (2 CPU, 16 GB, Oracle DB 19.0)
         ↓
CNC Development Client (1x for change management)
```

---

## Network & Firewall Configuration

### Port Matrix by Server Type

#### One-Click Provisioning Server
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB (file sharing, RDP)
3000                TCP          Web UI
3389                TCP          Remote Desktop Protocol (RDP)
5150                TCP          JDE Enterprise Server
5985                TCP          Windows Remote Management (WinRM)
8998-8999           TCP          Custom services
7000-7001           TCP          Application services
14501-14502         TCP          Management agents
```

#### Deployment Server
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
6017-6022           TCP          SQL Server Agent
14501-14510         TCP          Management agents
```

#### Database Servers (Non-Prod & Prod)
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
1433                TCP          SQL Server (default, may vary)
<NEW_DB_PORT>       TCP          Custom SQL Server port (DBA-defined)
14501-14510         TCP          Management agents
```

#### Enterprise Servers (Non-Prod & Prod)
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
6017-6022           TCP          SQL Server connectivity
14501-14510         TCP          Management agents
```

#### Batch Server
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
6017-6022           TCP          SQL Server connectivity
14501-14510         TCP          Management agents
```

#### WebLogic Servers (JAS & AIS)
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
8998-8999           TCP          WebLogic JAS application
<WLS_PORTS>         TCP          Custom WebLogic ports
7001-7002           TCP          WebLogic Admin Console
14501-14520         TCP          Management agents
```

#### Development Clients
```
Port Range          Protocol     Purpose
────────────────────────────────────────────────────────────
445                 TCP          SMB
3389                TCP          RDP
5150                TCP          JDE Enterprise Server
5985                TCP          WinRM
6017-6022           TCP          SQL Server connectivity
```

### Firewall Policy Recommendations

**Inbound:**
- Restrict traffic to specific source IP ranges
- Use IP whitelisting for critical ports (DB access, RDP)
- Implement rate limiting for RDP (prevent brute force)

**Outbound:**
- Allow DNS (53) to all servers
- Allow NTP (123) for time synchronization
- Allow specific outbound connections for software updates

**Inter-Zone Communication:**
- Non-Prod ↔ Non-Prod: Allow all (internal)
- Prod ↔ Prod: Allow specific inter-server ports
- Non-Prod ↔ Prod: Restrict to defined APIs only (prevent data leakage)

---

## Software Stack & Requirements

### Operating System Baseline

| Component | Version | Requirement |
|---|---|---|
| **OS** | Windows x64 2025 | All servers except dev clients |
| **Dev Client OS** | Windows 2025/2022/11 | Latest critical patches |
| **Windows SDK** | 10.0.26100.x | Development servers |
| **PowerShell** | 5.1+ | Remote management |
| **.NET Framework** | 4.8+ | JDE compatibility |

### Database & Clients

| Component | Version | Purpose |
|---|---|---|
| **SQL Server** | 2022 | Production & non-prod databases |
| **Oracle Database** | 19.0 (E1local) | Deployment Server only |
| **SSMS** | SQL Server 2022 | Database administration |
| **SQL JDBC** | Latest | WebLogic DB connectivity |

### Development & Compilation

| Component | Version | Purpose |
|---|---|---|
| **Visual Studio C++** | 2022 | Compilation & development |
| **JDK** | 17.0.14 / 1.8.0_491 | Java applications, WebLogic |
| **Windows SDK** | 10.0.26100.x | Development tools |

### Application Server

| Component | Version | Deployment |
|---|---|---|
| **WebLogic** | 14.1.2.0.0 | JAS & AIS servers |
| **JAS Instances** | 3 total | 1 non-prod, 2 prod |
| **AIS Instances** | 3 total | 1 non-prod, 2 prod |

### Web Client

| Browser | Version | Support |
|---|---|---|
| **Microsoft Edge** | 145.*, 139.* | Primary browser |
| **Compatibility** | HTML5, JavaScript ES6+ | Modern standards |

---

## Security & Access Controls

### Password Policy

#### Windows System Accounts
**Allowed Special Characters:** `_ @ ~ # % * + ( ) { } [ ] . ?`
**Disallowed:** Any other special characters

```
Example Valid Passwords:
  Abc@123#Valid
  Prod_Pass+2026
  User~Password[123]
  
Example Invalid Passwords:
  Pass$123 ($ not allowed)
  Pwd!2026 (! not allowed)
  Test&Pass (& not allowed)
```

#### SQL Server Database Users
**Allowed Special Characters:** `_ @ ~ + - *`
**Disallowed:** Other special characters including `$`

```
Example Valid Passwords:
  DBPass@123+Auth
  Prod_User-Pass*2026
  
Example Invalid Passwords:
  DB$Pass123 ($ not allowed)
  Pass#123 (# not allowed)
```

#### WebLogic Server
**Restrictions:** Cannot contain `$` or `!`

```
Example Valid Passwords:
  WLSAdmin@123_Pass
  WebLogic+Prod-2026*Auth
  
Example Invalid Passwords:
  WLS$Admin123 ($ not allowed)
  Password! (! not allowed)
```

### Service Accounts

| Server Type | Service Account | Notes |
|---|---|---|
| One-Click Provisioning | IT to provide | Custom provisioning account |
| Enterprise Servers | As-Is | JDE application service account |
| WebLogic Servers | As-Is | WebLogic domain service account |
| Batch Server | As-Is | Batch job execution account |
| Development Clients | As-Is | Individual developer accounts |

**Security Note:** Admin service account access to be provided to NDVER CNC for change management and approvals.

### SSL/TLS Configuration

**Requirement:** SSL certificates must be provided for new web components

- **Certificate Source:** Obtain from Certificate Authority (CA)
- **Format:** PEM or PKCS12 (WebLogic compatible)
- **Deployment:** Load Balancer, WebLogic servers
- **Renewal:** Automated or manual process to be documented
- **Certificate Pinning:** Consider for critical services

---

## Deployment Matrix

### Server Deployment Summary

```
╔════════════════════════╦════════╦═════════╦═════════╦════════════════════╗
║ Server Type            ║ Qty    ║ Non-Prod║ Prod    ║ Notes              ║
╠════════════════════════╬════════╬═════════╬═════════╬════════════════════╣
║ One-Click Provisioning ║ 1      ║    —    ║    1    ║ Infrastructure     ║
║ Deployment Server      ║ 1      ║    —    ║    1    ║ Central engine     ║
║ Database Server        ║ 2      ║    1    ║    1    ║ SQL Server 2022    ║
║ Enterprise Server      ║ 2      ║    1    ║    1    ║ JDE App Server     ║
║ Batch Server           ║ 1      ║    —    ║    1    ║ Prod batch jobs    ║
║ WebLogic JAS           ║ 3      ║    1    ║    2    ║ Java App Server    ║
║ WebLogic AIS           ║ 3      ║    1    ║    2    ║ Integration Server ║
║ Apache Load Balancer   ║ 1      ║    —    ║    1    ║ HA/Failover        ║
║ Development Client     ║ 3      ║    2    ║    1    ║ CNC + DEV machines ║
╠════════════════════════╬════════╬═════════╬═════════╬════════════════════╣
║ TOTALS                 ║ 17     ║    6    ║   11    ║                    ║
╚════════════════════════╩════════╩═════════╩═════════╩════════════════════╝
```

### Resource Allocation Summary

| Metric | Non-Prod | Prod | Infrastructure | Total |
|---|---|---|---|---|
| **Total CPU Cores** | 12 | 27 | 2 | **41** |
| **Total Memory (GB)** | 66 | 72 | 30 | **168** |
| **Total Storage (GB)** | ~440 | ~450 | ~400 | **~1,290** |

---

## Implementation Checklist

### Phase 1: Infrastructure Preparation
- [ ] Verify hardware meets CPU, memory, and storage requirements
- [ ] Prepare DBA specifications for database servers (CPU, memory, storage)
- [ ] Plan network IP address allocation and VLAN segregation
- [ ] Design firewall rules based on port matrix
- [ ] Procure SSL certificates for web components
- [ ] Document all custom port assignments (especially database ports)

### Phase 2: Base Operating System Installation
- [ ] Install Windows Server 2025 x64 on all servers
- [ ] Apply latest Windows updates and security patches
- [ ] Configure disk partitions per specification (C:, D:, E: drives)
- [ ] Install required Windows features (RDP, WinRM, .NET Framework)
- [ ] Join servers to domain (if applicable)
- [ ] Configure NTP for time synchronization

### Phase 3: Database Tier
- [ ] Install SQL Server 2022 on Non-Prod and Prod database servers
- [ ] Install Oracle 19.0 (E1local schema) on Deployment Server
- [ ] Create database user accounts with approved special characters only
- [ ] Configure backup and recovery procedures
- [ ] Implement HA/DR for production database
- [ ] Test failover procedures

### Phase 4: Application Tier
- [ ] Install JDK 17.0.14 / 1.8.0_491 on all applicable servers
- [ ] Install Microsoft Visual Studio C++ 2022 on development/compilation servers
- [ ] Install Windows SDK 10.0.26100.x
- [ ] Install WebLogic 14.1.2.0.0 on JAS and AIS servers
- [ ] Configure WebLogic domain (non-prod and prod)
- [ ] Deploy SSL certificates to WebLogic servers
- [ ] Configure cluster mode for JAS and AIS (active-active)

### Phase 5: Load Balancing & Networking
- [ ] Install and configure Apache Load Balancer
- [ ] Define backend server pools (JAS prod, AIS prod)
- [ ] Configure load balancing algorithm (round-robin or weighted)
- [ ] Set up health checks for backend servers
- [ ] Deploy SSL certificates to load balancer
- [ ] Test failover scenarios

### Phase 6: Support Services
- [ ] Set up One-Click Provisioning Server with automation tools
- [ ] Configure Deployment Server with deployment artifact storage
- [ ] Install required clients (SSMS, etc.)
- [ ] Set up monitoring and alerting

### Phase 7: Development Environment
- [ ] Set up 3 development clients (CNC: 1, DEV: 2)
- [ ] Install required development tools per specification
- [ ] Configure network access to development resources
- [ ] Set up user accounts and permissions

### Phase 8: Security & Compliance
- [ ] Apply password policies per specification
- [ ] Create and assign service accounts
- [ ] Configure RBAC (Role-Based Access Control)
- [ ] Enable audit logging on critical servers
- [ ] Document all administrative procedures
- [ ] Conduct security review

### Phase 9: Testing & Validation
- [ ] Validate all ports and firewall rules
- [ ] Test database connectivity from application servers
- [ ] Test WebLogic clustering and failover
- [ ] Verify load balancer failover scenarios
- [ ] Conduct end-to-end integration testing
- [ ] Performance testing and baseline establishment

### Phase 10: Production Cutover
- [ ] Schedule maintenance window
- [ ] Execute final validation checks
- [ ] Migrate production data
- [ ] Enable monitoring and alerting
- [ ] Verify disaster recovery procedures
- [ ] Document lessons learned

---

## Important Notes & Reminders

### Machine Naming Convention
- **Maximum length:** 15 characters (lowercase alphanumeric only)
- **Special characters allowed:** Hyphen "-" only
- **Pattern example:** `jde-prod-ent-001`, `jde-nonprod-db-01`

### Database Port Assignment
- **Status:** TBD pending DBA review
- **Default SQL Server port:** 1433 (if using default)
- **Custom port:** Must be documented in firewall rules
- **WebLogic ports:** Must be > 1024

### License Requirements
- [ ] Microsoft Visual Studio C++ 2022 licenses (provide by organization)
- [ ] WebLogic Server 14.1.2.0.0 (Oracle license)
- [ ] SQL Server 2022 (Microsoft license)
- [ ] Oracle Database 19.0 (Oracle license)
- [ ] Windows Server 2025 (Microsoft license)

### Outstanding Decisions (TBD)
1. **Database ports:** DBA to provide custom SQL Server ports
2. **WebLogic ports:** Specific port ranges for JAS/AIS
3. **Load balancer:** Firewall settings for LB-to-backend communications
4. **Hostnames:** Final naming scheme (currently: "<Names to be decided by>")
5. **Service accounts:** Specific AD service account names (IT to provide)

### Support & Escalation
- **Database Issues:** DBA Team
- **Network/Firewall:** IT Security & Infrastructure
- **WebLogic/Application:** JDE Application Team
- **Development:** Development Lead / CNC

---

## Document Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | June 2026 | Initial | Initial specification from upgrade plan |
| 2.0 | July 2026 | Updated | Added detailed architecture, port matrix, checklist |

---

**Next Step:** Review this document with all stakeholders and confirm outstanding decisions (marked as TBD) before proceeding to detailed design and implementation.
