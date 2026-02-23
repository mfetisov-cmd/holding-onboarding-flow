# Corporate Entity Graph — Business Requirements

> **Purpose**: Define requirements for a graph-based data model to represent relationships between companies and people.  
> **Status**: Draft for internal discussion  
> **Created**: February 3, 2026  
> **Author**: Product/Data team

---

## 1. Problem Statement

### Current State

Our data model is flat. For each Legal Entity, we store:
- Applicant (admin, verified via IDV)
- Directors (verified via IDV, can get account access)
- Shareholders (names, ownership %)
- UBOs (names, documents, often no IDV)

### Limitations

- No link between Company A and Company B if one owns the other
- No way to see if a person appears across multiple companies in our system
- Cannot trace ownership chains (A → B → C)
- Cannot flag when a person from a fraud case appears in a new application
- Cannot identify sales opportunities from connected companies/people
- Counterparty data from transactions is not linked to company entities
- No unified view for operations teams to see relationships

### Desired State

Graph-based model where companies and people are nodes, relationships are edges.

### Key Capabilities Needed

- **Visual graph UI** for all operations teams (Compliance, Customer Care, Sales, Product)
- **Simple API**: one call returns full graph for any entity
- **Easy CRUD** operations on nodes and edges
- **Traversal queries** without complex SQL

---

## 2. Use Cases

### 2.1 Sales & Growth

| ID | Scenario | Trigger | Action | Value |
|----|----------|---------|--------|-------|
| S1 | **Parent/child company discovery** | Company A opens account; registry shows it's owned by Company B (not our client) | Save Company B as prospect; assign to sales | New lead generation |
| S2 | **Portfolio person targeting** | Person X appears as director/UBO/shareholder across multiple companies | Target Person X for "portfolio onboarding" — open accounts for all their companies | Higher-value deals |
| S3 | **Supplier/counterparty upsell** | Compliance learns Company A's key suppliers during review or from transactions | Approach client: "Bring your suppliers to Vivid, simplify payments" | Organic expansion |
| S4 | **Holding company expansion** | NACE code indicates holding company; client provides list of subsidiaries | Create prospects for each subsidiary; coordinated outreach | Multi-entity deals |
| S5 | **Multi-company onboarding in one flow** | During application, we detect person is director/UBO of multiple companies | Ask about ALL their companies at once; offer to open accounts for all in single flow | More accounts + better CX (no repeated actions) |
| S6 | **Scoring via group** | During application, we detect that this company is part of bigger/richer company group | Assignment of HV flags | Cherry picking in inbound stream |
| S7 | **Scoring via people** | During application, we detect that this company is related to a high-potential individual | Assignment of HV flags | Cherry picking in inbound stream |
| S8 | **Reverse lookup for outbound sales** | Sales wants to approach external Company X | Search tool, find that X has sister company Y already at Vivid | Warm intro, higher conversion |

### 2.2 Compliance & Risk

| ID | Scenario | Trigger | Action | Value |
|----|----------|---------|--------|-------|
| C1 | **Fraud propagation check** | Fraud detected at Company X | Query graph: find all connected companies/people; flag for review | Prevent related fraud |
| C2 | **Bad actor reappearance** | Person was director of closed/fraud company | When they appear in new application → automatic flag for enhanced due diligence | Early warning |
| C3 | **Suspicious counterparty chain** | Transaction monitoring finds suspicious entity | Trace through graph to see if any of our clients are connected | Proactive risk management |
| C4 | **Ownership chain verification** | Complex ownership structure during KYC | Visualize full chain; verify UBOs at the top | Faster, accurate KYC |

### 2.3 Operations Efficiency

| ID | Scenario | Trigger | Action | Value |
|----|----------|---------|--------|-------|
| O1 | **Pre-fill from known entities** | New application; person/company already exists in graph | Auto-populate verified data; skip re-verification where allowed | Faster onboarding |
| O2 | **Cross-reference during review** | Ops reviews application | One-click view of all related entities in system | Better context |
| O3 | **Document reuse** | UBO document already on file from related company | Surface existing docs; reduce client requests | Better CX |

---

## 3. Graph Model

### 3.1 Node Types

| Node Type | Description | Key Attributes |
|-----------|-------------|----------------|
| **Company** | Any legal entity (client or not) | `id`, `name`, `registration_number`, `country`, `legal_form`, `nace_code`, `is_client`, `client_status`, `risk_flags[]` |
| **Person** | Any individual (verified or not) | `id`, `name`, `date_of_birth`, `nationality`, `is_verified` (IDV done), `verification_date`, `risk_flags[]`, `documents[]` |

**Node status examples:**
- Company: `is_client: true/false`, `client_status: active/closed/prospect/fraud`
- Person: `is_verified: true/false`, `risk_flags: ["fraud_association", "pep", "sanctioned"]`

### 3.2 Edge Types (Relationships)

| Edge Type | From → To | Description | Attributes |
|-----------|-----------|-------------|------------|
| **OWNS** | Company → Company | Parent owns subsidiary | `ownership_%`, `since_date`, `source` |
| **SHAREHOLDER_OF** | Person → Company | Person holds shares | `ownership_%`, `since_date`, `source` |
| **UBO_OF** | Person → Company | Ultimate beneficial owner | `control_type` (direct/indirect), `ownership_%`, `source` |
| **DIRECTOR_OF** | Person → Company | Director/representative | `role` (CEO, Managing Director, etc.), `since_date`, `source` |
| **APPLICANT_OF** | Person → Company | Filed the application | `application_date`, `is_admin: true` |
| **TRANSACTS_WITH** | Company → Company | Cash flow relationship | `relationship_type` (supplier, customer, partner), `source`, `last_seen_date` |
| **SISTER_OF** | Company ↔ Company | Companies sharing same parent (derived) | 2 hops via OWNS: A ← Parent → B |

### 3.3 Multi-hop (Second-degree) Relationships

Beyond direct edges, we need to support **traversal queries** — finding entities connected through 2, 3, or more edges.

**Definition:** Second-degree relationship = path from one node to another through 2+ edges.

#### Why Multi-hop Matters

| Pattern | Hops | Example | Value |
|---------|------|---------|-------|
| **Sister companies** | 2 | Company A ← Parent → Company B | Understand company is part of larger group |
| **Person's portfolio** | 2 | Company A ← Person → Company B | Find all companies where person has role |
| **Group size/value** | 2-3 | Client ← Parent → Sisters (multiple) | Score client by group size/revenue |
| **Indirect ownership** | 2-4 | Client ← Parent ← Grandparent ← UBO | Trace true beneficial ownership |
| **Risk propagation** | 2-4 | Fraud company → Person → Other companies | Find connected entities for review |

#### API Requirement

The graph API must support:
- `GET /graph/{entity_id}?depth=N` — return all connected nodes up to N hops
- `GET /graph/{entity_id}/siblings` — return sister companies (same parent)
- `GET /graph/person/{person_id}/portfolio` — return all companies where person has role

### 3.4 Comprehensive Visual Example

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              CORPORATE ENTITY GRAPH                                     │
│                      (All edge types & multi-hop relationships)                         │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│  OWNERSHIP CHAIN (3 levels):                                                            │
│                                                                                         │
│                            ┌──────────────────┐                                         │
│                            │  🏛️ Grandparent  │                                         │
│                            │   Holding SE     │                                         │
│                            └────────┬─────────┘                                         │
│                                     │                                                   │
│                                     │ OWNS 100%                                         │
│                                     ▼                                                   │
│                            ┌──────────────────┐                                         │
│                            │  🏢 Parent GmbH  │                                         │
│                            └────────┬─────────┘                                         │
│                                     │                                                   │
│           ┌─────────────────────────┼─────────────────────────┐                         │
│           │                         │                         │                         │
│           │ OWNS 60%                │ OWNS 100%               │ OWNS 40%                │
│           ▼                         ▼                         ▼                         │
│    ┌────────────┐           ┌────────────┐            ┌────────────┐                    │
│    │ 🏢 Sister A │           │ ✅ CLIENT  │            │ 🏢 Sister B │                    │
│    │  Tech GmbH  │◄─SISTER─►│  Main GmbH │◄──SISTER──►│  Sales AG  │                    │
│    │ (prospect)  │           │            │            │ (prospect) │                    │
│    └────────────┘           └─────┬──────┘            └─────┬──────┘                    │
│                                   │                         │                           │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
│                                   │                         │                           │
│  PEOPLE CONNECTED TO CLIENT:      │           PEOPLE CONNECTED TO SISTER B:             │
│                                   │                         │                           │
│    ┌──────────┐                   │                         │        ┌──────────┐       │
│    │ 👤 John  │───DIRECTOR_OF────►│                         │◄───────│ 👤 Max   │       │
│    │    ✅    │───APPLICANT_OF───►│                         │        │    ✅    │       │
│    │ verified │                   │                    DIRECTOR_OF   │ verified │       │
│    └────┬─────┘                   │                         │        └──────────┘       │
│         │                         │                         │                           │
│         │ DIRECTOR_OF             │                                                     │
│         ▼                         │                                                     │
│    ┌──────────┐                   │                                                     │
│    │🏢 Other  │                   │                                                     │
│    │   Co     │  ◄── John's portfolio: 2 companies                                      │
│    │(prospect)│                   │                                                     │
│    └──────────┘                   │                                                     │
│                                   │                                                     │
│    ┌──────────┐                   │                                                     │
│    │ 👤 Anna  │─────UBO_OF───────►│                                                     │
│    │    ❌    │                   │                                                     │
│    │ not ver. │                   │                                                     │
│    └──────────┘                   │                                                     │
│                                   │                                                     │
│    ┌──────────┐                   │                                                     │
│    │🏢 Corp X │──SHAREHOLDER_OF──►│ (30% ownership)                                     │
│    │ investor │                   │                                                     │
│    └──────────┘                   │                                                     │
│                                   │                                                     │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
│                                   │                                                     │
│  TRANSACTION RELATIONSHIPS:       │                                                     │
│                                   │                                                     │
│    ┌──────────┐                   │                                                     │
│    │🏭Supplier│◄──TRANSACTS_WITH──┤                                                     │
│    │   GmbH   │                   │                                                     │
│    └──────────┘                   │                                                     │
│                                                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│  LEGEND:                                                                                │
│                                                                                         │
│  Nodes:   ✅ = Client    🏢 = Prospect    👤 = Person    🏛️ = Holding    🏭 = Supplier   │
│  Edges:   ───► = Direct relationship    ◄─SISTER─► = Derived (2 hops)                  │
│                                                                                         │
│  Multi-hop examples:                                                                    │
│  • Ownership chain: CLIENT ← Parent ← Grandparent (3 levels)                           │
│  • Sister companies: CLIENT ↔ Sister A, Sister B (2 hops via Parent)                   │
│  • Person portfolio: John → CLIENT + Other Co (2 companies)                            │
│  • Cross-link: Max → Sister B → (sister of) → CLIENT (3 hops)                          │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.5 Source Tracking

Every edge should track where the data came from:

| Source Code | Description |
|-------------|-------------|
| `registry` | Official register (via CreditSafe or direct) |
| `client_provided` | Client told us during onboarding |
| `document` | Extracted from uploaded document |
| `transaction` | Derived from transaction data |
| `compliance_inquiry` | Learned during compliance review |
| `prospect_db` | From prospect/leads database |

---

## 4. Data Sources Mapping

### 4.1 Where does each relationship come from?

| Edge Type | Primary Source | Secondary Sources | When Collected |
|-----------|---------------|-------------------|----------------|
| **OWNS** (Company→Company) | Registry (CreditSafe) | Client documents, Compliance inquiry | Onboarding, Follow-up |
| **SHAREHOLDER_OF** (Person→Company) | Registry (CreditSafe) | Client documents | Onboarding |
| **UBO_OF** (Person→Company) | Client-provided + Registry validation | Documents (shareholder registry) | Onboarding |
| **DIRECTOR_OF** (Person→Company) | Registry (CreditSafe) | Client-provided, Prospect DB | Onboarding, Sales research |
| **APPLICANT_OF** (Person→Company) | Application data | — | Onboarding |
| **TRANSACTS_WITH** (Company→Company) | Transaction monitoring | Compliance inquiry, Client-provided | Ongoing, Follow-up |

### 4.2 Data Sources Detail

| Source | What it provides | Current availability | Notes |
|--------|------------------|---------------------|-------|
| **CreditSafe (Registry)** | Directors, Shareholders, UBOs, Parent companies, Subsidiaries | ✅ Available (full response stored) | Need to expand what we extract and store in graph |
| **Prospect Database** | Companies, Directors | ✅ Available | Need to confirm: can we query by person? |
| **Client Application** | Applicant info, declared UBOs/directors | ✅ Available | Structured data in onboarding forms |
| **Client Documents** | Shareholder registries, Ownership charts, Articles of association | ✅ Available | Extraction capability exists, not implemented yet |
| **Transaction Data** | Counterparty IBANs, Company names | ✅ Available | Need to match/resolve to Company nodes |

### 4.3 Gaps to Address

| # | Gap | How to address |
|---|-----|----------------|
| 1 | **Document ownership extraction not implemented** | Build automated extraction from shareholder registries, org charts |
| 2 | **CreditSafe response not fully parsed into graph** | Define which fields to extract; build parser |
| 3 | **Prospect DB query by person unknown** | Confirm with data team |
| 4 | **Person deduplication logic undefined** | Define unique identifier (Name + DOB? National ID?) |

---

## 5. Open Questions for Internal Discussion

### 5.1 Data & Technical

| # | Question | Owner (suggested) | Impact |
|---|----------|-------------------|--------|
| 1 | **Can Prospect DB be queried by person (director name)?** | Data/Engineering | Enables portfolio person targeting (S2) |
| 2 | **What's the unique identifier for a Person across systems?** Name + DOB? National ID? | Data/Compliance | Critical for deduplication — same person in multiple companies |
| 3 | **How to implement document ownership extraction?** | Engineering | Automate extraction from shareholder registries, org charts |
| 4 | **What fields from CreditSafe response should we parse into graph?** | Data/Compliance | Define exactly which relationships to extract |

### 5.2 Product & UX

| # | Question | Owner (suggested) | Impact |
|---|----------|-------------------|--------|
| 5 | **Which team owns the graph UI?** | Product | Compliance tool? Sales tool? Unified? |
| 6 | **Should Sales see compliance flags (fraud, risk)?** | Product/Compliance | Role-based access design |
| 7 | **How should multi-company onboarding flow work (S5)?** | Product/Onboarding | UX for "open accounts for all your companies" |

### 5.3 Process

| # | Question | Owner (suggested) | Impact |
|---|----------|-------------------|--------|
| 8 | **Who maintains the graph data?** Manual edits by Ops? Automated only? | Operations/Data | Data quality ownership |
| 9 | **How do we handle conflicts?** (CreditSafe says X, client says Y) | Compliance | Data reconciliation rules |

### 5.4 Graph Population Strategy

| # | Question | Owner (suggested) | Impact |
|---|----------|-------------------|--------|
| 10 | **What is the "zero point" for graph population?** Only companies touched during onboarding, or also external enrichment (prospect DB, on-demand CreditSafe)? | Product/Data | Defines scope, cost, and S8 feasibility |
| 11 | **Should we capture sister companies during onboarding?** (not just parents/UBOs) | Product/Compliance | Enables reverse lookup (S8) via client-triggered approach |

---

## 6. Implementation Phases

*Pending team feedback on Sections 1–5*

---

## Appendix: Reference

- **North Data** (northdata.com) — European corporate graph visualization, reference for desired UX
- **Current data model** — See `/Users/mfetisov/vivid work/VIVIDMINE_KNOWLEDGE_BASE.md`, Section 5

---

*This document is a draft for internal discussion. Please add comments and questions.*
