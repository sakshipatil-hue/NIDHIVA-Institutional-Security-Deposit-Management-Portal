<div align="center">

<img src="assets/banner.svg" alt="NIDHIVA: Institutional Security Deposit Management Portal" width="100%"/>

<br/>

**One portal for every security deposit and FDR, from submission to release.**

![Smart India Hackathon 2026](https://img.shields.io/badge/SIH-2026-e8743b?style=for-the-badge)
![Problem Statement](https://img.shields.io/badge/PS-26265-0f2a47?style=for-the-badge)
![Organization](https://img.shields.io/badge/AICTE-Regulation%20Bureau-1f9d8a?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Working%20Prototype-8e5bd9?style=for-the-badge)

[The problem](#-the-problem) · [Our solution](#-our-solution) · [How it works](#-how-it-works) · [Architecture](#-architecture) · [Run it](#-run-it-locally) · [Roadmap](#-roadmap)

</div>

---

## 🎯 The problem

AICTE asks some technical institutions to lock money with it as a guarantee, usually as a bank **Fixed Deposit Receipt (FDR)**. Today these are tracked through scattered records, letters, emails and separate trackers.

That makes it hard to answer simple questions:

| Question an officer needs answered | What goes wrong today |
|---|---|
| Which FDRs are about to mature? | Dates sit in spreadsheets and files, so deadlines get missed. |
| Is this instrument genuine and complete? | Verification is manual, and claimed data is not separated from bank-confirmed data. |
| Which institutions are waiting for a refund? | Requests arrive by email with no status trail. |
| Who changed what, and when? | No single audit history. |

> **Why it keeps getting bigger:** [342 AICTE-approved institutions closed over three years](https://news.careers360.com/342-aicte-approved-technical-institutions-shut-lok-sabha-reply), and every closure is a deposit to locate, verify and release.

---

## 💡 Our solution

**NIDHIVA** is a secure, centralized portal that manages the whole life of every deposit.

<table>
<tr>
<td width="33%" valign="top">

### 📥 Submit
Institutions upload FDR details and documents online instead of sending letters.

</td>
<td width="33%" valign="top">

### 🔍 Verify
Automatic rule checks flag problems, and every record is tagged **submitted** or **bank-confirmed**.

</td>
<td width="33%" valign="top">

### ⏰ Track
Daily maturity monitoring sends reminders and escalates anything overdue.

</td>
</tr>
<tr>
<td valign="top">

### 🔁 Renew
A replacement FDR is linked to the old one, and the full history is kept.

</td>
<td valign="top">

### 📤 Release
Release and refund requests move through officer review to the competent authority.

</td>
<td valign="top">

### 📊 Oversee
A live dashboard and automated reports show every instrument and pending case.

</td>
</tr>
</table>

### What makes it different

- **Trust tagging:** institution-submitted and bank-confirmed data are never mixed up.
- **Rule-based discrepancy engine:** catches date errors, duplicates, missing documents and expired instruments before an officer spends time on them.
- **SLA-based escalation:** alerts cover stalled verifications and unresolved cases, not only maturity dates.
- **Pluggable bank verification:** a mock service today, swappable for real bank APIs once authorized.
- **Full audit trail:** every submission, change and approval is traceable.

---

## 🔄 How it works

### Lifecycle of an FDR

```mermaid
flowchart LR
    A([Institution submits FDR + documents]) --> B{Auto-validation<br/>rule checks}
    B -- issues found --> C[Flagged]
    C -- institution corrects --> A
    B -- clean --> D[Officer scrutiny +<br/>bank verification]
    D --> E([Active<br/>Bank-confirmed])
    E --> F{Maturity monitor<br/>daily scan}
    F -- reminders --> G[Expiring]
    G --> H[Renewal<br/>new FDR linked to old]
    G --> I[Matured / Overdue]
    I -- escalation --> J[Senior authority]
    H --> E
    J --> K[Release request]
    I --> K
    K --> L{Competent authority<br/>approval}
    L --> M([Released + discharge record])

    classDef good fill:#d9f2ee,stroke:#1f9d8a,color:#0f2a47
    classDef warn fill:#fde9dc,stroke:#e8743b,color:#0f2a47
    class E,M good
    class C,I,J warn
```

### Status states

```mermaid
stateDiagram-v2
    [*] --> Submitted
    Submitted --> Flagged: rule check fails
    Flagged --> Submitted: institution corrects
    Submitted --> UnderScrutiny: rule check passes
    UnderScrutiny --> Active: verified
    UnderScrutiny --> Flagged: discrepancy found
    Active --> Expiring: maturity approaching
    Expiring --> Renewed: replacement submitted
    Expiring --> Overdue: maturity passed
    Renewed --> Active
    Overdue --> ReleaseRequested
    Expiring --> ReleaseRequested
    ReleaseRequested --> Released: authority approves
    Released --> [*]
```

### Test scenarios from the problem statement

| # | Scenario | Where to see it in the demo |
|---|---|---|
| 1 | New FDR submission | Institution view → *New deposit* |
| 2 | Validity tracking | Dashboard → *Active* |
| 3 | Approaching maturity | Dashboard → *Expiring* and alerts |
| 4 | Renewal | FDR detail → *Renewal history* |
| 5 | Discrepancy identification | Flagged list with reasons |
| 6 | Authorized release processing | Release request → approval |

---

## 🏗 Architecture

```mermaid
flowchart TB
    subgraph Client["Client (React)"]
        I[Institution view]
        O[Officer / Bureau Head view]
        A[Admin / Auditor view]
    end

    subgraph API["Backend API"]
        AU[Auth + roles]
        FL[FDR lifecycle]
        DM[Documents]
        RE[Rule engine]
        SC[Scheduler]
        NT[Notifications]
        AL[Audit log]
    end

    subgraph Data["Data layer"]
        PG[(PostgreSQL)]
        RD[(Redis queue)]
        S3[(MinIO documents)]
    end

    subgraph Ext["External, pluggable"]
        BK[Bank verification<br/>mock in prototype]
        EM[Email / SMS]
        AI[AICTE systems]
    end

    Client --> API
    API --> Data
    RE --> BK
    NT --> EM
    FL -.-> AI

    classDef ext fill:#fde9dc,stroke:#e8743b,color:#0f2a47
    class BK,EM,AI ext
```

> **Honest note:** bank verification is **simulated** in this prototype behind a clean interface. It is built to be replaced with real bank or NeSL-style verification APIs when formally authorized.

### Tech stack

![React](https://img.shields.io/badge/React-20232a?style=flat-square&logo=react&logoColor=61dafb)
![TypeScript](https://img.shields.io/badge/TypeScript-3178c6?style=flat-square&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-0f172a?style=flat-square&logo=tailwindcss&logoColor=38bdf8)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-dc382d?style=flat-square&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ed?style=flat-square&logo=docker&logoColor=white)

<!-- TODO: remove any badge for a technology you did not actually use. -->

---

## 🖼 Screenshots

<!-- TODO: add real screenshots to /docs/screenshots and update the paths below. -->

| Dashboard | New FDR submission |
|---|---|
| ![Dashboard](docs/screenshots/dashboard.png) | ![Submission](docs/screenshots/submission.png) |

| Discrepancy flags | Release workflow |
|---|---|
| ![Flags](docs/screenshots/flags.png) | ![Release](docs/screenshots/release.png) |

🎥 **Demo video:** <!-- TODO: paste YouTube link -->

---


## 🗺 Roadmap

<!-- TODO: tick only what actually works in your prototype. -->

- [ ] Institution view: FDR submission and document upload
- [ ] Rule-based discrepancy checks
- [ ] Officer scrutiny with submitted vs bank-confirmed tagging
- [ ] Maturity tracking with configurable reminders
- [ ] Renewal linking with history
- [ ] Release and refund approval workflow
- [ ] Dashboard and automated reports
- [ ] Role-based access and audit trail
- [ ] Real bank verification API integration *(needs formal authorization)*
- [ ] Integration with existing AICTE systems *(needs formal authorization)*

---

## 📈 Target impact

> These are **projected design targets**, to be measured in the prototype, not measured results.

| Area | Target |
|---|---|
| Administrative and paperwork cost | 30-40% reduction |
| Manual verification and tracking time | 50-60% reduction |
| Unnoticed maturities | Zero, through alerts at 90/30/15/7/1 days plus overdue escalation |

---

## 📚 References

1. AICTE, *Approval Process Handbook 2024-25 to 2026-27*, [aicte.gov.in](https://aicte.gov.in/sites/default/files/APH%20Final.pdf)
2. Careers360, [*Closures of AICTE-approved institutions (Lok Sabha reply)*](https://news.careers360.com/342-aicte-approved-technical-institutions-shut-lok-sabha-reply)
3. NeSL e-Bank Guarantee platform, cited as a precedent for digital verification of bank instruments (it covers bank guarantees, not FDRs)

---

👥 Team
<div align="center">
	
Project	NIDHIVA
Competition	Smart India Hackathon 2026
Problem Statement	SIH26265
Domain Smart Automation
Team name	555_ATHENA


---

<div align="center">

Built for **Smart India Hackathon 2026** · Problem Statement **26265**

</div>
