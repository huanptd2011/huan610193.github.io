📂 DFIR - Core Arsenal (Windows + Linux + Cloud)

├── 📂 02. Disk & File Systems (Ổ đĩa)
│   ├── 📂 Windows (NTFS/FAT)
│   │   ├── MBR vs VBR (Boot sectors).md
│   │   ├── FAT (12/16/32) - Directory entries.md
│   │   ├── NTFS Master File Table ($MFT).md
│   │   ├── NTFS Attributes ($DATA, $INDEX_ROOT, $BITMAP).md
│   │   ├── Alternate Data Streams (ADS).md
│   │   ├── $LogFile & $UsnJrnl (Journaling).md
│   │   ├── Volume Shadow Copy (VSS).md
│   │   └── BitLocker Decryption (Recovery keys, TPM).md
│   └── 📂 Linux (EXT4/XFS)
│       ├── EXT4 Superblock & Inodes.md
│       ├── Journal (JBD2) Recovery.md
│       ├── LUKS (Linux Unified Key Setup).md
│       └── Swap Partition Analysis.md
│
├── 📂 03. Windows Artifacts (Dấu vết Windows)
│   ├── 📂 Registry Forensics
│   │   ├── Registry Hives (SYSTEM, SAM, SECURITY, SOFTWARE, NTUSER.DAT).md
│   │   ├── USB Device History (USBSTOR, MountedDevices).md
│   │   ├── UserAssist (Program execution tracking).md
│   │   ├── MRU Lists (Recent Docs, RunMRU).md
│   │   ├── Network History & Wi-Fi Profiles.md
│   │   └── Windows Services & Drivers installation.md
│   ├── 📂 Event Logs (.evtx)
│   │   ├── Security Log (4624 Logon, 4672 Privileges, 4688 Process creation).md
│   │   ├── System Log (Shutdowns, Driver failures).md
│   │   ├── PowerShell Operational Log.md
│   │   └── Sysmon Logs (Event IDs 1-Process, 3-Network, 7-ImageLoad).md
│   └── 📂 File System Artifacts
│       ├── Prefetch (.pf) - Application startup.md
│       ├── JumpLists (Recent Office/Explorer files).md
│       ├── LNK Files (Shortcuts with MACB times).md
│       ├── ShellBags (Folder view preferences).md
│       ├── Browser History (Chrome/Edge/Firefox - SQLite).md
│       ├── Zone.Identifier (Mark of the Web).md
│       └── Recycle Bin ($Recycle.bin) analysis.md
│
├── 📂 04. Linux Artifacts (Dấu vết Linux)
│   ├── Shell History (.bash_history, .zsh_history).md
│   ├── Cron Jobs & Systemd Timers (Persistence).md
│   ├── Syslog & Journald (journalctl).md
│   ├── Auth Logs (SSH logins, sudo usage).md
│   ├── Lastlog, Wtmp, Btmp (Login records).md
│   ├── RPM/Dpkg Package History (Installed software).md
│   └── Auditd Logs (System call tracking).md
│
├── 📂 05. Memory Forensics (RAM)
│   ├── Memory Acquisition (WinPmem, LiME, DumpIt).md
│   ├── Volatility Framework (Profiles, basic plugins).md
│   ├── Process Lists & Network Connections (PsList, Netscan).md
│   ├── Malware Injection Detection (Malfind, HollowFind).md
│   └── Extracting Passwords (Mimikatz in memory, LSA Secrets).md
│
├── 📂 06. Network Forensics (Mạng)
│   ├── PCAP acquisition (Tcpdump, Wireshark).md
│   ├── TCP/IP Protocol Analysis (Handshakes, Fragments).md
│   ├── HTTP/HTTPS Decryption (SSL keys).md
│   ├── DNS Forensics (DNSCache, DGA detection).md
│   ├── SMTP & Email Header Analysis (SPF, DKIM).md
│   ├── C2 Detection (Beaconing, Data exfiltration patterns).md
│   └── Network Proxy & Firewall logs (Squid, pfSense).md
│
├── 📂 07. Data Recovery & Carving (Cứu dữ liệu)
│   ├── File Signatures (Magic Bytes database).md
│   ├── Carving Tools (Foremost, Scalpel, PhotoRec).md
│   ├── Unformatting & Partition Recovery (TestDisk).md
│   └── Deleted File Recovery (NTFS Undelete, EXT undeletion).md
│
├── 📂 08. Cloud & SaaS Forensics (Mây)
│   ├── 📂 AWS
│   │   ├── CloudTrail (API call logging).md
│   │   ├── S3 Access Logs & Public Bucket detection.md
│   │   └── IAM & EC2 User Data logs.md
│   ├── 📂 Azure
│   │   ├── Azure Active Directory Sign-ins & Audit Logs.md
│   │   └── Blob Storage Access logs.md
│   ├── 📂 M365 / Google Workspace
│   │   ├── M365 Unified Audit Log (Mail, SharePoint).md
│   │   └── Google Admin Logs (Gmail, Drive).md
│   └── 📂 Containers (Docker/K8s)
│       ├── Docker Daemon Logs & Container Metadata.md
│       └── Kubernetes Audit Logs (Kube-Apiserver).md
│
├── 📂 09. Anti-Forensics & Evasion (Kẻ tấn công xóa dấu vết)
│   ├── Data Wiping & Encryption (Ransomware).md
│   ├── Timestomping (Changing MACB times).md
│   ├── Log Tampering & Wiping (Clearing Event Logs).md
│   ├── Obfuscated Command Lines (PowerShell encoded).md
│   └── Living off the Land (LOLBins - using system tools).md
│
├── 📂 10. Incident Response (IR - Xử lý sự cố)
│   ├── IR Playbooks (Phishing, Ransomware, Insider).md
│   ├── Triage Collection (KAPE, Cyber Triage).md
│   ├── Timeline Creation (Plaso/Log2Timeline).md
│   ├── Live Response (Velociraptor, OSQuery).md
│   ├── YARA Scanning for IOCs.md
│   └── MITRE ATT&CK Mapping (Techniques & Tactics).md
│
├── 📂 11. Tools (Bộ công cụ chính)
│   ├── 📂 Commercial (EnCase, FTK, Magnet AXIOM).md
│   ├── 📂 Open Source (TSK/Autopsy, Volatility, Wireshark).md
│   ├── 📂 Collection & Parsing (KAPE, Plaso, CyberChef).md
│   └── 📂 Scripting (Python - pyevtx, python-registry; PowerShell).md
│
├── 📂 12. Cheat Sheets (Tra cứu nhanh)
│   ├── Windows Artifact Locations (USN, MFT, Registry hives).md
│   ├── Volatility Commands (Windows/Linux).md
│   ├── Event Log Critical IDs (4624, 4688, 7045, Sysmon 1,3,7).md
│   ├── Linux Forensic Commands (mount, lsof, dmesg).md
│   └── AWS/Azure CLI Forensic Queries.md
│
└── 📂 13. Writeups & Case Studies (Học từ thực tế)
    ├── 📂 Ransomware Incidents (LockBit, Conti)
    ├── 📂 APT Intrusions (APT29, Lazarus)
    ├── 📂 Insider Threats
    └── 📂 DFIR Challenges (CyberDefenders, BlueTeamLabs)