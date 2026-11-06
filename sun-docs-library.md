# Sun Documentation Library (filibeto.org/sun/lib/)

The solarisUsersGroup archive at https://www.filibeto.org/sun/lib/ is a community mirror of Sun's documentation, frozen around 2007. It is the best free source for **Solaris 8/9/10-era BluePrints** — Sun's official engineering best-practice papers. Use it for deep-dive procedures on legacy systems (Solaris 10 SPARC, V880-era hardware, JumpStart, DiskSuite/VxVM); always sanity-check against man pages for anything that might have changed in later releases.

## Site map

Directories have browsable Apache listings at `https://www.filibeto.org/sun/lib/<dir>/`:

| Directory | Contents |
|---|---|
| `blueprints/` | ~120 Sun BluePrints PDFs, named by Sun part number (`817-xxxx.pdf`). The library index page (`/sun/lib/`) maps titles to files — open it to find the link for a title below. |
| `certification/` | Solaris 7/8/9/10 certification paths, cram sessions, and SA courseware in HTML (`sol_8_admin_1.html`, `sol_8_admin_2.html`, `sol_8_tcp_admin.html`, `shell_programming.html`) |
| `development/` | Sun WorkShop 6, Forte Developer 7, Sun ONE Studio 8 compiler docs, GCC 3.3 manuals |
| `hardware/` | Architecture/datasheet PDFs: `sun_ultrasparc_iii.pdf` (UltraSPARC III — the V880's CPU), `sun_netra_family.pdf`, `v20z-v40z-arch-wp.pdf`, `1100z-2100z-arch-wp.pdf`, `SF15K_JTF_clientes.pdf` (Sun Fire 15K), `u5u10-arch052001.pdf` (Ultra 5/10), `t1125_users_guide.pdf`, `lsi_scsihab.pdf`, `flashprom.pdf` |
| `books/` | `basic_unix_85321.pdf` |
| `security/` | `wp-s8security.pdf` (Solaris 8 security whitepaper), `_security_sol11c.pdf`, `improve_by_breakin.html` (hardening by pen-testing), `smartspoof-en.pdf`, `osdetect-perl.txt` |
| `misc/` | Sun-themed comics and INFOWORLD issues — historical fun, not admin material |

## BluePrints by task (titles as listed on the index page)

### Security hardening
- Solaris OE Minimization for Security (+ the *Updated for Solaris 9* edition) — the canonical "install less" methodology
- Solaris OE Network Settings for Security (+ updated for 9) — ndd hardening values
- Auditing in the Solaris 8 OE; Kerberos Network Security; IPsec — A Secure Deployment Option
- Building/Deploying/Configuring OpenSSH for Solaris; RBAC and Secure Shell
- Securing Systems with Host-Based Firewalls; Building Secure N-Tier Environments
- Solaris 10 privilege model set: Limiting Service Privileges, Privilege Debugging, Privilege Bracketing, Restricting Service Administration, Two-Person Rule via RBAC
- Automating Centralized File Integrity Checks (BART); Integrating BART and the Solaris Fingerprint Database; The Solaris Fingerprint Database
- Securing Sun Enterprise 10000 System Service Processors; Computer Forensics When Investigating System Attacks
- *Book:* Securing Systems with the Solaris Security Toolkit (JASS)

### Boot, install, patching
- Configuring Boot Disks; Configuring Boot Disks with Solstice DiskSuite Software
- Using Live Upgrade 2.0 with a Logical Volume Manager; Patching Mirrored Systems with the Solaris Live Upgrade Software
- Solaris Patch Management: Recommended Strategy
- Building a JumpStart Infrastructure; Customizing the JumpStart Framework for Installation and Recovery; Network Installations Without a Local Boot Server; Configuring JumpStart Servers for Sun x86-64
- Building a Bootable DVD to Deploy a Solaris Flash Archive; Creating a Customized Boot CD/DVD (x86); GRUB and Solaris on x86 — A Guide to Creating a Customized Boot DVD; Configuring Multiboot Environments on Sun x64

### Storage and volume management
- Veritas VxVM Storage Management; VxVM Private Regions: Mechanics & Internals; Toward a Reference Configuration for VxVM Managed Boot Disks; Automatic Encapsulation of the Root Disk
- Solaris Volume Manager Performance Best Practices
- Scrubbing Disks Using the Solaris OE Format Program (secure disk erasure)
- The Sun StorEdge T3 Array: Installation, Configuration, Monitoring; RAID 6 on the Sun StorageTek 6140; Using iSCSI Multipathing in Solaris 10; Increasing SAN Productivity; Hardware Replication Challenges

### Clustering and HA
- Implementation Guide: Sun Cluster 3.0 (Parts I and II); Understanding the Benefits of Oracle RAC on Sun Cluster
- Enterprise Network Design Patterns: High Availability; Architecting Availability and Disaster Recovery Solutions; Planning to Fail

### Virtualization and consolidation
- The Sun BluePrints Guide to Solaris Containers (book); Solaris Containers Technology Architecture Guide; Solaris Containers — What They Are and How to Use Them; Self-Balancing Solutions with Solaris Containers; Working with Solaris Containers and the Solaris Service Manager
- Slicing and Dicing Servers: A Guide to Virtualization and Containment Technologies
- Beginners Guide to LDoms; LDOMS IO Best Practices — Data Reliability with Logical Domains; Understanding the Sun xVM Hypervisor Architecture
- Consolidation: Consolidation in the Data Center (book), Consolidation Methodology, Web Consolidation on the Sun Fire T1000, Consolidating the Sun Store onto T2000s, Application and Database Server Consolidation on the X4600, Consolidating Legacy Applications onto Sun x64 Servers

### Performance and tuning
- Static Performance Tuning; Maximizing Performance of a Gigabit Ethernet NIC Interface; Multithreading in the Solaris OE
- Solaris OE and Oracle RDBMS Performance Tuning; Dynamic Reconfiguration and Oracle 9i Dynamically Resizable SGA
- Developing and Tuning Applications on UltraSPARC T1 (Niagara) systems; Using the Cryptographic Accelerator of the UltraSPARC T1

### Networking and naming
- Using NTP to Control and Synchronize System Clocks, Parts I–III; Using NTP on the Sun Fire 15K/12K
- Policy-Based Networks; Enterprise QoS using Solaris Bandwidth Manager 1.6
- Ethernet Autonegotiation Best Practices (the classic reference for forcing vs auto-negotiating)
- LDAP in the Solaris OE: Deploying Secure Directory Services (book); Understanding the NIS to LDAP (N2L) Service Architecture; LDAP Triggers

### Hardware operations (legacy Sun Fire / Enterprise)
- Sun Fire 15K/12K Server Preferred Practices; Sun Fire Midrange Servers Auto Diagnosis and Recovery (+ Updated Best Practices for Firmware 5.18.x); Predictive Fault Monitoring in Sun Fire Servers
- Securing Sun Enterprise 10000 SSPs; Sun Enterprise 10000 Floating Tape Library Solution
- Automating Initial Setup and Management of Sun Fire V20z/V40z; Updating Firmware on V20z/V40z; Remote Monitoring of Sun x64 Systems using ipmitool and ipmievd; Differentiating X4100/X4200 from the M2 models
- Service Provisioning with Resource Management; Using Solaris Resource Manager with Sun Ray

### Sun Ray and desktop
- Metropolitan Area Sun Ray Services; Sun Ray Deployment on Shared Networks; Sizing Sun Ray Servers Running Windows Applications; Running Windows Applications with SunPCi IIpro; Desktop Architecture Selection Guide; The Managed Desktop Factory (Sun VDI)

## Certification courseware (Solaris 10 track)

Canonical SA study path, mirrored in `certification/`:
- SA-100-S10 UNIX Essentials; SA-200-S10 Intermediate System Administration; SA-202-S10 Advanced System Administration; SA-300-S10 Network Administration
- Exams: CX-310-200 (SCSA Part I), CX-310-202 (SCSA Part II), CX-310-302 (SCNA)
- Solaris 7/8/9 cram sessions and practice tests are in the same directory.

## Caveats

- Snapshot is from ~2007: no Solaris 10 update releases past early 2007, no Solaris 11, no illumos. Prefer man.omnios.org (see [omnios-manpages.md](omnios-manpages.md)) and Oracle docs for anything newer.
- BluePrint PDFs are stored under Sun part numbers; the human-readable index at `/sun/lib/` is the way to find a specific title.
- Era-specific security advice (ndd hardening lists, SSH config) predates modern best practices; treat as historical baseline for legacy boxes only.
