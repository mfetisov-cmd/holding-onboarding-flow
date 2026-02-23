# CRS Data Collection Requirements

> **Purpose**: Summary of new data fields needed for CRS compliance, by role  
> **Status**: Confirmed with compliance team  
> **Created**: February 23, 2026  
> **Context**: ACT-231 (Extend onboarding flow to capture missing CRS data)

---

## What's New to Collect

| Who | New Fields | When Required |
|-----|-----------|---------------|
| **Every Legal Entity** | Tax residence(s), TIN(s), Entity classification | Always at onboarding |
| **Every Retail client** | Tax residence(s), TIN(s), Place of birth | Always at onboarding |
| **Each UBO of Passive NFE** | Tax residence(s), TIN(s), Place of birth, Nature of control | Only when entity = Passive NFE |

> **Initial rollout**: All new fields are **optional** during onboarding. Prepares for mandatory CRS reporting.  
> **Scope**: New clients only (not retroactive for existing accounts).

---

## Detail by Role

### A. Legal Entity (SME Account Holder)

| Field | Already Collected? | CRS Requirement |
|-------|--------------------|-----------------|
| Legal name | Yes | Mandatory |
| Registered address | Yes | Mandatory |
| Jurisdiction(s) of tax residence | **No — NEW** | Mandatory. Multiple allowed |
| TIN(s) per jurisdiction | **No — NEW** | Mandatory. E.g., German Steuernummer, French SIREN |
| Entity classification (FI / Active NFE / Passive NFE) | **No — NEW** | Mandatory. Drives controlling person reporting |

### B. Controlling Persons (UBOs) — Only If Entity Is Passive NFE

| Field | Already Collected? | CRS Requirement |
|-------|--------------------|-----------------|
| Full name | Yes | Mandatory |
| Residential address | Partially | Mandatory — verify completeness |
| Jurisdiction(s) of tax residence | **No — NEW** | Mandatory. Multiple allowed per person |
| TIN(s) per jurisdiction | **No — NEW** | Mandatory |
| Date of birth | Yes | Mandatory |
| Place of birth | **No — NEW** | Mandatory (Luxembourg CRS requirement) |
| Nature of control | **No — NEW** | "Ownership" vs "control by other means" |

### C. Applicant / Directors — Not Required by CRS

| Role | CRS Data Needed? | Reason |
|------|-------------------|--------|
| Applicant | No (unless also UBO of Passive NFE) | Not a CRS reporting concept |
| Director | No (unless also UBO of Passive NFE) | Not a CRS reporting concept |
| Shareholder/Member | No (unless also UBO of Passive NFE) | Not a CRS reporting concept |

### D. Individual / Retail Account Holder

| Field | Already Collected? | CRS Requirement |
|-------|--------------------|-----------------|
| Full name | Yes | Mandatory |
| Residential address | Yes | Mandatory |
| Jurisdiction(s) of tax residence | **No — NEW** | Mandatory. Multiple allowed |
| TIN per jurisdiction | **No — NEW** | Mandatory |
| Date of birth | Yes | Mandatory |
| Place of birth | **No — NEW** | Mandatory (Luxembourg CRS requirement) |

---

## CRS Entity Classification Definitions

| Classification | Definition | Example |
|----------------|------------|---------|
| **Financial Institution (FI)** | Bank, investment fund, insurance company, custodial institution | Currently out of VMSA risk appetite |
| **Active NFE** | ≥50% of gross income from active sources (sales, services) OR ≥50% of assets used in active business | Trading companies, SaaS, retailers, holding companies managing active subsidiaries |
| **Passive NFE** | Does not qualify as Active and is not an FI. Income mainly from dividends, interest, royalties, rental income | Dormant holdings, IP holding entities, SPVs, family offices, investment vehicles |

> **Key nuance**: Income from actively managed operating businesses = active income. Shareholdings in active subsidiaries = active assets (not passive).

---

## CRS Account-Level Data (Always Required)

| Field | Source |
|-------|--------|
| Account number / IBAN | System-generated |
| Name & identifying number of EMI (Vivid Money SA) | Static |
| Account balance or value at year-end (or closure date) | System-generated |

---

## CRS Account Holder Types (Organization)

| Code | Description |
|------|-------------|
| CRS101 | Passive NFE with one or more controlling person that is a Reportable Person |
| CRS102 | CRS Reportable Person (entity itself) |
| CRS103 | Passive NFE that is itself a CRS Reportable Person |

---

## Sources

- [CRS Luxembourg Circular ECHA-4](https://impotsdirects.public.lu/content/dam/acd/fr/legislation/legi17/ECHA4.pdf)
- [CRS Manual (e-file.lu)](https://www.e-file.lu/wiki/CRS_Manual)
- Notion ACT-231: Extend onboarding flow to capture missing CRS data
- Andrejs' comment on CRS reporting requirements
