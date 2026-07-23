📂 Web Exploitation - Battle-Tested Arsenal (v1.0)
│
├── 📂 00. War Room (Thay cho Inbox - Ghi chép khi đang "chiến đấu")
│   ├── Quick Notes.md                 (Cờ lệnh, payload lạ, output đẹp)
│   ├── Target Scope.md                (Domain, IP, Subnet, Cloud Accounts)
│   ├── TODO (Pentest Checklist).md    (Các mục cần test trong ngày)
│   └── Interesting Endpoints.md       (API, admin panels, backup files phát hiện)
│
├── 📂 01. Reconnaissance (Thu thập thông tin - Chiếm 60% thời gian)
│   ├── Passive Recon (OSINT)
│   │   ├── Subdomain Enumeration (Amass, Subfinder).md
│   │   ├── Technology Stack (Wappalyzer, BuiltWith).md
│   │   ├── Archive History (Wayback, Gau, Katana).md
│   │   ├── GitHub/Code Leakage (Tokens, Keys, .env).md
│   │   └── Cloud Bucket Enumeration (AWS S3, Azure Blob).md
│   ├── Active Recon (Tương tác nhẹ)
│   │   ├── Port Scanning (Nmap, Naabu).md
│   │   ├── Web Crawling & Spidering (Burp Spider, ZAP).md
│   │   ├── Directory/File Brute-force (ffuf, dirb).md
│   │   ├── Parameter Discovery (Arjun, ParamSpider).md
│   │   └── API Endpoint Discovery (Swagger, GraphQL introspection).md
│   └── Fingerprinting
│       ├── WAF Detection (Cloudflare, AWS WAF).md
│       ├── Load Balancer & CDN Detection.md
│       └── CMS Detection (WordPress, Joomla, Drupal).md
│
├── 📂 02. Fuzzing & Input Validation (Tìm kiếm điểm chết)
│   ├── HTTP Fuzzing Basics (Burp Intruder, ffuf).md
│   ├── Header Injection (Host, X-Forwarded-For, Referer).md
│   ├── Parameter Pollution (HPP - HTTP Parameter Pollution).md
│   ├── Content-Type Manipulation (multipart, JSON, XML).md
│   ├── Encoding Bypass (Double URL, Unicode, UTF-16).md
│   ├── GraphQL Fuzzing (Aliases, Batches).md
│   └── Rate Limit Testing (Bypass via IP rotation).md
│
├── 📂 03. Authentication & Session (Cổng vào - Khó nhất)
│   ├── 📂 JWT (JSON Web Tokens)
│   │   ├── Algorithm Confusion (None, HS256→RS256).md
│   │   ├── Kid Injection (Key ID Path Traversal).md
│   │   ├── Weak Secret Brute-force.md
│   │   └── JWT in Cookies vs Headers (CSRF risks).md
│   ├── 📂 OAuth 2.0 & OpenID Connect
│   │   ├── Redirect_URI Manipulation (Open Redirect).md
│   │   ├── CSRF on OAuth Flows (State bypass).md
│   │   ├── Scope Escalation (Token swapping).md
│   │   └── Authorization Code Interception.md
│   ├── 📂 SAML
│   │   ├── Signature Wrapping (XML Canonicalization).md
│   │   └── SAML Replay Attacks.md
│   ├── Session Management
│   │   ├── Session Fixation.md
│   │   ├── Weak Session ID (Predictable RNG).md
│   │   └── Session Expiration Bypass.md
│   └── MFA Bypass
│       ├── OTP Brute-force & Timing Attacks.md
│       ├── MFA Stage Skipping (Param manipulation).md
│       └── Social Engineering (MFA Fatigue).md
│
├── 📂 04. Injection Attacks (Lỗ hổng kinh điển - Vẫn còn sống)
│   ├── SQL Injection (Classic)
│   │   ├── Union-based, Error-based, Boolean Blind.md
│   │   ├── Time-based Blind (Heavy queries).md
│   │   ├── Second-order SQLi.md
│   │   └── WAF Bypass for SQLi (Comments, Case, Hex).md
│   ├── NoSQL Injection (MongoDB, Elasticsearch)
│   │   ├── Operator Injection ($ne, $gt, $where).md
│   │   └── JavaScript Injection in NoSQL.md
│   ├── Command Injection (OS CMD)
│   │   ├── Classic (ping, whoami).md
│   │   ├── Blind Command Injection (Time-based).md
│   │   └── Filter Bypass (IFS, newline, env).md
│   ├── LDAP Injection.md
│   └── XPath Injection.md
│
├── 📂 05. Cross-Site Scripting (XSS) - Chiến trường Client-side
│   ├── Reflected vs Stored vs DOM-based XSS.md
│   ├── XSS Contexts (HTML, Attribute, JavaScript, CSS).md
│   ├── CSP Bypass (Nonce reuse, JSONP, Trusted Types).md
│   ├── XSS via SVG / MathML / Web Components.md
│   ├── XSS via PostMessage & DOM Clobbering.md
│   ├── XSS to CSRF / Session Hijacking.md
│   └── XSS Polyglots (One payload fits all).md
│
├── 📂 06. Server-Side Advanced Attacks (Đỉnh cao - RCE)
│   ├── Server-Side Template Injection (SSTI)
│   │   ├── Jinja2 / Twig (Python/PHP).md
│   │   ├── Freemarker / Velocity (Java).md
│   │   ├── ERB (Ruby).md
│   │   └── Sandbox Escape via Template Engine.md
│   ├── Deserialization (Object Injection)
│   │   ├── Java (ysoserial, Gadget Chains).md
│   │   ├── .NET ViewState / LosFormatter.md
│   │   ├── PHP (Phar, Session, Native).md
│   │   ├── Python Pickle / PyYAML.md
│   │   └── Node.js (node-serialize, payload).md
│   ├── Server-Side Request Forgery (SSRF)
│   │   ├── Classic SSRF (Internal IP, Localhost).md
│   │   ├── Blind SSRF (Out-of-band).md
│   │   ├── SSRF to AWS IMDS (IMDSv1/v2 bypass).md
│   │   ├── SSRF via Gopher / Dict / FTP protocols.md
│   │   └── SSRF via DNS Rebinding (TOCTOU).md
│   └── XML External Entity (XXE)
│       ├── Classic XXE (Read files).md
│       ├── Blind XXE (OOB - Out-of-band).md
│       ├── XXE via SVG / DOCX uploads.md
│       └── XXE to SSRF / DoS.md
│
├── 📂 07. Access Control & Logic (Lỗi logic - Tiền thưởng lớn)
│   ├── IDOR (Insecure Direct Object References)
│   │   ├── Numeric / UUID / Hash ID enumeration.md
│   │   └── Horizontal vs Vertical Privilege Escalation.md
│   ├── Mass Assignment (Auto-binding from JSON/Form).md
│   ├── Business Logic Flaws
│   │   ├── Currency / Quantity Manipulation.md
│   │   ├── Referral / Coupon Abuse (Race).md
│   │   ├── Workflow Bypass (Step skipping).md
│   │   └── Email Verification Bypass.md
│   └── Race Conditions (TOCTOU, Concurrency).md
│
├── 📂 08. API Security (REST, GraphQL, gRPC, WebSocket)
│   ├── 📂 REST API
│   │   ├── Swagger / OpenAPI Exposure.md
│   │   ├── API Key Leakage (URL, Logs).md
│   │   └── Rate Limiting Bypass (X-Forwarded-For).md
│   ├── 📂 GraphQL
│   │   ├── Introspection Queries (Leaking schema).md
│   │   ├── GraphQL Injection (via resolvers).md
│   │   ├── GraphQL Batching (DoS & Rate limit bypass).md
│   │   └── GraphQL Aliases for IDOR.md
│   ├── 📂 gRPC
│   │   ├── Protobuf Reflection (Service discovery).md
│   │   └── gRPC Metadata Injection.md
│   └── 📂 WebSocket
│       ├── Origin Validation Bypass.md
│       ├── WebSocket Injection (SQLi in frames).md
│       └── WebSocket DoS (Large frames).md
│
├── 📂 09. Cloud & Infrastructure (AWS, GCP, Azure, K8s)
│   ├── AWS (S3, EC2, Lambda, IAM)
│   │   ├── S3 Bucket Public Listing & Injection.md
│   │   ├── IAM Privilege Escalation (PassRole).md
│   │   ├── EC2 User Data Injection (SSRF).md
│   │   └── Cognito Identity Pools (JWT misuse).md
│   ├── GCP (Cloud Functions, Storage, IAM).md
│   ├── Azure (Blob, AD, Functions).md
│   ├── Kubernetes (K8s)
│   │   ├── Kubelet API Exposure (SSRF).md
│   │   ├── RBAC Misconfiguration (ClusterRole).md
│   │   └── Secrets Exposure via etcd.md
│   └── Serverless (AWS Lambda environment variables).md
│
├── 📂 10. Cache Poisoning & Request Smuggling (Desync)
│   ├── Web Cache Poisoning (Unkeyed Headers).md
│   ├── Cache Deception (Path Confusion).md
│   ├── HTTP Request Smuggling (CL.TE, TE.CL).md
│   ├── HTTP/2 Request Smuggling (Pseudo-headers).md
│   ├── Smuggling to XSS / SSRF / Cache Poisoning.md
│   └── CDN Cache Poisoning (Edge).md
│
├── 📂 11. Payloads & Snippets (Kho đạn dược)
│   ├── 📂 SQLi
│   │   ├── Generic UNION payloads.md
│   │   └── Time-based sleep payloads.md
│   ├── 📂 XSS
│   │   ├── Classic alert() payloads.md
│   │   ├── Exfiltration (fetch, BeEF).md
│   │   └── CSP Bypass payloads.md
│   ├── 📂 SSTI
│   │   ├── Jinja2 RCE chain.md
│   │   └── Freemarker RCE chain.md
│   ├── 📂 SSRF
│   │   └── Gopher payload generator (Python).md
│   ├── 📂 Deserialization
│   │   ├── ysoserial command generator.md
│   │   └── PHP Phar payload.md
│   └── 📂 Misc
│       ├── JWT Algorithm Confusion script.md
│       └── Prototype Pollution payloads.md
│
├── 📂 12. Tools Configuration (Cấu hình & Workflow)
│   ├── 📂 Burp Suite
│   │   ├── Extensions (Turbo Intruder, Autorize, Logger).md
│   │   ├── Collaborator Setup.md
│   │   └── Match-and-Replace Rules.md
│   ├── 📂 Other Tools
│   │   ├── ffuf / gobuster (Custom wordlists).md
│   │   ├── Nuclei (Custom templates).md
│   │   ├── SQLMap (Tamper scripts).md
│   │   ├── ZAP Automation Scripts.md
│   │   └── Caido / Fiddler.md
│   ├── 📂 Browser
│   │   ├── DevTools Tips (Network, Console, Sources).md
│   │   └── Security Extensions (HackBar, Wappalyzer).md
│   └── 📂 Environment
│       ├── Dockerized Labs (Juice Shop, WebGoat).md
│       └── Custom Proxy Chains (Tor, VPN).md
│
├── 📂 13. Writeups & Labs (Học từ thực chiến)
│   ├── 📂 PortSwigger Academy (Tất cả các lab)
│   ├── 📂 Bug Bounty Reports (HackerOne, Bugcrowd, Intigriti)
│   ├── 📂 HTB / TryHackMe (Web Machines)
│   ├── 📂 CTF Web (DreamHack, PwnFunction)
│   └── 📂 Real-World 0-day Analysis (CVE)
│
├── 📂 14. Cheat Sheets (Bỏ túi - Tra cứu 1 giây)
│   ├── HTTP Status Codes & Headers Quickref.md
│   ├── WAF Bypass Strings (SQLi, XSS).md
│   ├── JWT Common Attacks Quickref.md
│   ├── OAuth Flows Cheatsheet.md
│   ├── GraphQL Pentesting Commands.md
│   ├── ffuf / Nuclei Common Flags.md
│   └── Reverse Shell One-Liners (Web edition).md
│
├── 📂 15. Report Writing (Kết thúc - Đưa lên khách hàng)
│   ├── Bug Bounty Report Template.md
│   ├── Pentest Report Template (Executive + Technical).md
│   ├── Proof of Concept (PoC) Format.md
│   ├── Risk Rating (CVSS 3.1) Guide.md
│   └── Remediation Advice Templates.md
│
╞═══════════════════════════════════════════════════════════╡
║         🚀 CHUYÊN SÂU (Zero-day / Research Track)         ║
╚═══════════════════════════════════════════════════════════╝
│
├── 📂 16. Gadget Hunting & Chain Exploitation
│   ├── Java Commons Collections Analysis.md
│   ├── .NET ViewState Gadget Discovery.md
│   ├── PHP Phar Deserialization Chain Building.md
│   ├── Node.js Prototype Pollution to RCE (Chain).md
│   └── Spring4Shell / Log4Shell Technical Deep-dive.md
│
├── 📂 17. Browser & JavaScript Deep Dive
│   ├── V8 Engine Exploitation (Client-side).md
│   ├── Shadow DOM & Trusted Types Bypass.md
│   ├── WebAssembly Memory Corruption (WASM).md
│   └── WebRTC IP Leakage & Exploitation.md
│
└── 📂 18. Conference Talks & Academic Papers
    ├── BlackHat / DEFCON Web Security Talks.md
    ├── USENIX / NDSS Web Security Papers.md
    └── Google Project Zero Writeups.md