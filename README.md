# SysAdmin Homelab Bootcamp

A 12-unit hands-on journey to build and manage IT infrastructure from scratch.  
This bootcamp simulates a small-to-medium business environment using a homelab with VMware, Windows Server 2019, Windows 10 Pro, and Ubuntu 20.04 LTS.

---

## 🎯 Objectives
- Gain practical experience with **Windows Server administration** (AD DS, DNS, DHCP, GPO, File/Print, Backup)
- Learn **basic Linux server management** with Ubuntu
- Deploy **monitoring** (Zabbix) and **helpdesk/ITSM** (GLPI)
- Explore **networking fundamentals** aligned with **CCNA** topics
- Produce professional deliverables: runbooks, checklists, configuration files, and documentation

---

## 📚 Curriculum

### Unit 1–2: Homelab Setup & Active Directory
- Build 3 VMs (Windows Server 2019, Windows 10 Pro, Ubuntu 20.04 LTS)
- Install **Active Directory Domain Services** and create `lab.local`
- Create basic OU/Users/Groups
- Deliverables:
  - Runbook: *Install AD DS & create a new domain*
  - OU diagram and screenshot proof of functioning domain

### Unit 3: DNS + DHCP (AD Integrated)
- Configure DNS forwarders and reverse lookup
- Install & authorize DHCP, create scope & options (003 gateway, 006 DNS)
- Deliverables:
  - Documentation: *AD Integrated DNS & DHCP setup*
  - Test: Windows 10 receives IP via DHCP, resolves FQDN

### Unit 4: Join Domain & Account Standards
- Properly join Windows 10 into domain
- Define naming conventions (users, groups, OU structure)
- Deliverables:
  - Checklist: *Onboard Windows 10 into domain*
  - Naming convention document

### Unit 5: Group Policy (GPO)
- Learn GPO concepts (scope, inheritance, GPMC)
- Create sample GPOs: password policy, map drive, block USB, deploy printers
- Deliverables:
  - Runbook: *Deploy GPO & verify with gpresult/Event Viewer*
  - Screenshots: before/after GPO effects

### Unit 6: File/Print Server + Backup
- Configure NTFS & Share permissions
- Setup Print Server and deploy via GPO
- Use Windows Server Backup for basic file/VM backup
- Deliverables:
  - SOP: *Restore deleted/overwritten files*
  - End-user KB: *Connect printer via domain*

### Unit 7: Linux Fundamentals (Ubuntu)
- Learn user/group management, services, logs, firewall, SSH
- Configure static IP/DNS
- Deliverables:
  - Quick reference sheet of essential Linux commands

### Unit 8: System Monitoring (Zabbix)
- Deploy Zabbix server on Ubuntu (package or container-based)
- Install Zabbix Agent on Windows and Linux
- Deliverables:
  - Dashboard with monitored hosts
  - Test alert (CPU threshold)

### Unit 9: Helpdesk/ITSM with GLPI
- Install GLPI and explore ITSM modules
- Define ticketing workflow (ticket → classify → SLA → KB)
- Deliverables:
  - 5 sample tickets (printer, Wi-Fi, email, account, software)
  - 3 end-user KB articles

### Unit 10–11: Networking Fundamentals (CCNA oriented)
- Study subnetting, VLAN, Trunk, Inter-VLAN, static routing, NAT, ACL
- Lab practice with Packet Tracer/GNS3
- Deliverables:
  - Packet Tracer file (.pkt) with VLAN/Trunk/ACL config
  - Configuration tables & diagrams

### Unit 12: Review & Final Project
- Consolidate runbooks (AD, DNS/DHCP, GPO, File/Print, Zabbix, GLPI, CCNA labs)
- Write final report (4–6 pages): *Small Office IT Infrastructure Design*
- Deliverables:
  - GitHub repo `infra-labs/` containing screenshots, .pkt files, GPO exports, DHCP/DNS configs, SOPs, KBs

---

## 📂 Repository Structure
```
sysadmin-homelab-bootcamp/
│── unit01-ADDS/
│── unit02-DNS-DHCP/
│── unit03-Accounts/
│── unit04-GPO/
│── unit05-FilePrint-Backup/
│── unit06-Ubuntu/
│── unit07-Zabbix/
│── unit08-GLPI/
│── unit09-Networking/
│── unit10-Review/
│── docs/
│   ├── runbooks/
│   ├── checklists/
│   └── diagrams/
└── README.md
```

---

## 🛠️ Tech Stack
- **Hypervisor**: VMware Workstation Pro
- **OS**: Windows Server 2019, Windows 10 Pro, Ubuntu 20.04 LTS
- **Services**: AD DS, DNS, DHCP, GPO, File/Print, Backup
- **Tools**: Zabbix, GLPI, Cisco Packet Tracer / GNS3

---

## 📜 License
MIT License
