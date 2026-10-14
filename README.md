# SecurityOrganizer 🛡️

> **Continuous External Attack Surface Management (EASM) & Asset Governance Engine.**

[![Production Site](https://img.shields.io/badge/Production-securityorganizer.com-06b6d4?style=flat-square&logo=googlechrome&logoColor=white)](https://securityorganizer.com)
[![Hosting](https://img.shields.io/badge/Hosting-GitHub%20Pages-22c55e?style=flat-square&logo=github)](https://pages.github.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-slate.svg?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Beta%20Waitlist-emerald?style=flat-square)](#roadmap)

---

## 📑 Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Problem Statement & Solution](#2-problem-statement--solution)
- [3. Core Platform Capabilities](#3-core-platform-capabilities)
- [4. System Architecture](#4-system-architecture)
- [5. Database Schema (Target MVP)](#5-database-schema-target-mvp)
- [6. Repository File Layout](#6-repository-file-layout)
- [7. Local Development & Deployment](#7-local-development--deployment)
- [8. Custom Domain & DNS Mapping](#8-custom-domain--dns-mapping)
- [9. Product Roadmap](#9-product-roadmap)
- [10. Security & Ethical Standards](#10-security--ethical-standards)
- [11. License](#11-license)

---

## 1. Executive Summary

**SecurityOrganizer** is an External Attack Surface Management (EASM) and digital asset governance solution created to eliminate organization blind spots. Modern agile teams deploy services, staging subdomains, cloud buckets, and internal APIs rapidly across distributed clouds. 

SecurityOrganizer acts as an automated, non-intrusive adversarial reconnaissance engine—continuously discovering exposed infrastructure, cataloging certificates, checking port exposures, and scoring posture risk before bad actors can exploit misconfigurations.

---

## 2. Problem Statement & Solution

| The Problem | The SecurityOrganizer Solution |
| :--- | :--- |
| **Shadow IT & Stale Subdomains** | Automated Certificate Transparency (CT) log analysis and DNS brute-forcing identify every rogue or forgotten host. |
| **Silent SSL Outages & Cipher Risks** | Proactive alerts 90, 60, 30, and 7 days prior to certificate expiration, alongside weak protocol (TLS 1.0/1.1) detection. |
| **Accidentally Exposed Services** | Non-invasive banner grabbing scans for leaked database, cache, or debug ports (Redis `6379`, Postgres `5432`, SSH `22`, etc.). |
| **Overpriced Enterprise Tooling** | A lightweight, fast-loading dashboard and flexible pricing accessible to startups, SMBs, and solo founders without $30k contracts. |

---

## 3. Core Platform Capabilities

* **Zero-Agent Discovery:** Operates 100% externally. Requires no server daemons, no software installs, and zero privileged access.
* **Continuous DNS & Drift Tracking:** Monitors for orphaned DNS records that could allow subdomain takeovers.
* **Security Posture Score (0–100):** Real-time weighted formula calculating risk based on misconfigurations, exposed services, and cipher weaknesses.
* **Instant Multi-Channel Alerts:** Dispatches actionable findings straight to Slack, Discord, email, or custom Webhook endpoints.
* **One-Click Compliance Exports:** Generates auditor-friendly reports mapped to SOC 2, ISO 27001, and GDPR evidence collection requirements.

---

## 4. System Architecture

```
                    ┌────────────────────────────┐
                    │    securityorganizer.com   │
                    └──────────────┬─────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         ▼                         ▼                         ▼
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│  Asset & Surface │      │  Certificate &   │      │    Exposure &    │
│    Discovery     │      │  Cipher Auditing │      │ Service Auditing │
└────────┬─────────┘      └────────┬─────────┘      └────────┬─────────┘
         │                         │                         │
         └─────────────────────────┼─────────────────────────┘
                                   │
                                   ▼
                   ┌───────────────────────────────┐
                   │ Threat Engine & Risk Scoring  │
                   └───────────────┬───────────────┘
                                   │
                     ┌─────────────┴─────────────┐
                     ▼                           ▼
          ┌─────────────────────┐     ┌─────────────────────┐
          │  Notification Hub   │     │  Reporting Engine   │
          │ (Slack, Email, Web) │     │ (PDF / CSV Exports) │
          └─────────────────────┘     └─────────────────────┘
```

### Technology Matrix
- **Static Portal & Waitlist (Current):** Semantic HTML5, Tailwind CSS, Vanilla JavaScript (ES6+), Font Awesome 6.
- **Hosting & Edge Delivery:** GitHub Pages Anycast CDN with automatic Let's Encrypt SSL/TLS.
- **Backend & Worker Layer (Production Target):** Next.js / TypeScript, Supabase (PostgreSQL, Row Level Security, Auth), Node.js / Go workers.
- **Billing Infrastructure:** Stripe / Lemon Squeezy for recurring subscriptions.

---

## 5. Database Schema (Target MVP)

The target backend runs on PostgreSQL with Row Level Security (RLS) to enforce tenant isolation:

```sql
-- Organizations / Teams
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Monitored Root Assets
CREATE TABLE monitored_domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    org_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
    domain_name VARCHAR(255) NOT NULL,
    scan_frequency VARCHAR(50) DEFAULT 'daily',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Discovered Subdomains & Endpoints
CREATE TABLE discovered_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id UUID REFERENCES monitored_domains(id) ON DELETE CASCADE,
    subdomain VARCHAR(255) NOT NULL,
    ip_address INET,
    is_live BOOLEAN DEFAULT TRUE,
    last_seen TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Certificate Tracking
CREATE TABLE certificates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID REFERENCES discovered_assets(id) ON DELETE CASCADE,
    issuer VARCHAR(255),
    valid_from TIMESTAMP WITH TIME ZONE,
    valid_to TIMESTAMP WITH TIME ZONE,
    tls_version VARCHAR(50),
    days_to_expire INT,
    last_audited TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Security Findings
CREATE TABLE findings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID REFERENCES discovered_assets(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    severity VARCHAR(20) CHECK (severity IN ('LOW', 'MEDIUM', 'HIGH', 'CRITICAL')),
    description TEXT,
    remediation TEXT,
    status VARCHAR(50) DEFAULT 'OPEN',
    detected_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

---

## 6. Repository File Layout

```text
securityorganizer/
├── .github/
│   └── workflows/              # GitHub Actions CI/CD workflows
├── CNAME                       # Maps GitHub Pages to securityorganizer.com
├── .gitignore                  # Excludes local editor/OS debris
├── index.html                  # Responsive SaaS promotional portal & live demo
├── LICENSE                     # MIT Open Source License
└── README.md                   # Complete project documentation & guide
```

---

## 7. Local Development & Deployment

### Quick Local Preview
No compiler or Node dependencies required to run the promotional portal:

```bash
# Clone the repository
git clone https://github.com/<your-username>/securityorganizer.git
cd securityorganizer

# Launch lightweight preview server with Python
python3 -m http.server 8080
```
Open your browser at `http://localhost:8080`.

### Pushing Updates to GitHub
```bash
git add .
git commit -m "docs: sync project documentation in README"
git push origin main
```

---

## 8. Custom Domain & DNS Mapping

To host on **`securityorganizer.com`** via GitHub Pages:

### 1. Repository Settings
1. Navigate to **Settings** > **Pages** in this GitHub repository.
2. Select Source: `Deploy from a branch` -> `main` / `/(root)`.
3. Set **Custom domain** to: `securityorganizer.com`.

### 2. Registrar DNS Configuration
Add the following DNS records in your domain registrar dashboard:

| Record Type | Name / Host | Target / Value | Description |
| :--- | :--- | :--- | :--- |
| **A** | `@` | `185.199.108.153` | GitHub Anycast Node 1 |
| **A** | `@` | `185.199.109.153` | GitHub Anycast Node 2 |
| **A** | `@` | `185.199.110.153` | GitHub Anycast Node 3 |
| **A** | `@` | `185.199.111.153` | GitHub Anycast Node 4 |
| **CNAME** | `www` | `<your-username>.github.io` | Subdomain redirect |

*Wait 15–30 minutes for DNS propagation, then check **Enforce HTTPS** under repository Pages settings.*

---

## 9. Product Roadmap

- [x] High-converting landing page with simulated domain audit interface
- [x] Custom domain mapping (`securityorganizer.com`) via GitHub Pages
- [x] Responsive layout with dark-mode cybersecurity palette
- [ ] Direct database webhook integration for waitlist lead capture
- [ ] Authentication portal and dashboard wireframes (Next.js & Supabase)
- [ ] Asynchronous worker microservice for live DNS & TLS health checks
- [ ] Automated Slack / Webhook alert triggers on certificate drift
- [ ] PDF executive report export engine for SOC 2 audits

---

## 10. Security & Ethical Standards

* **Ethical Probing Only:** SecurityOrganizer is committed to non-destructive testing. The scanning engine only conducts passive information harvesting and safe protocol handshakes.
* **Ownership Verification:** Production scanning requires domain control verification via DNS TXT record or well-known challenge tokens prior to deep discovery sweeps.
* **Zero Host Invasiveness:** No binary execution, credential testing, or exploit attempts are ever executed against target infrastructure.

---

## 11. License

This project is licensed under the [MIT License](LICENSE).
```