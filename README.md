# dego-project-team5

# Credit Application Governance Analysis

DEGO 2606 | Nova SBE | Team 5

## Executive Summary

*To be updated with findings*

## Data Quality Findings

*To be updated*

## Bias Analysis

*To be updated*

## 

## Privacy Assessment

*We identified 7 PII fields stored without protection, including SSNs in plain text. The dataset has no consent tracking mechanism, no data retention policy, and no audit trail for automated credit decisions.*



*### PII Fields Identified*



*| Field | PII Type | Risk Level |*

*|-------|----------|------------|*

*| full\_name | Direct identifier | High |*

*| email | Direct identifier | High |*

*| ssn | Sensitive national identifier | Critical |*

*| ip\_address | Online identifier | Medium |*

*| date\_of\_birth | Quasi-identifier | Medium |*

*| zip\_code | Quasi-identifier | Medium |*

*| gender | Special category (Art. 9) | High |*

*| spending\_behavior | Behavioral data | Medium-High |*



*### GDPR Compliance Gaps*



*| Article | Issue | Severity |*

*|---------|-------|----------|*

*| Art. 5(1)(c) — Data Minimization | IP addresses and granular spending data not necessary for credit scoring | Medium |*

*| Art. 5(1)(e) — Storage Limitation | No data retention policy — records stored indefinitely | High |*

*| Art. 6(1) — Lawful Basis | No documented lawful basis — consent field missing | High |*

*| Art. 9 — Special Categories | Gender collected and used without documented exception | High |*

*| Art. 17 — Right to Erasure | SSNs in plain text — erasure requests unfulfillable | Critical |*

*| Art. 22 — Automated Decisions | Fully automated decisions with no human oversight | Critical |*

*| Art. 25 — Privacy by Design | PII stored unprotected, no encryption or access controls | High |*

*| Art. 35 — Impact Assessment | No DPIA conducted despite high-risk credit scoring | High |*



*### EU AI Act*



*Credit scoring is classified as high-risk under Annex III, Section 5(b) of the EU AI Act. NovaCred fails to meet all seven compliance requirements: risk management, data governance, technical documentation, audit trails, transparency, human oversight, and accuracy testing.*



*### Pseudonymization*



*SHA-256 hashing with a salt was applied to SSN, name, and email fields. This prevents casual identification and supports duplicate detection, but pseudonymized data remains personal data under GDPR — it is not true anonymization.*



*Full analysis in `notebooks/03-privacy-demo.ipynb`.*



## Governance Recommendations

1. *\*\*Decision audit trail\*\*: Log every credit decision with timestamp, applicant ID, features used, model version, output, and confidence score (AI Act Art. 12)*

2. *\*\*Human-in-the-loop\*\*: Flag borderline cases for manual review with override authority (GDPR Art. 22, AI Act Art. 14)*

3. *\*\*Informed consent\*\*: Add a consent tracking field, inform applicants about AI use and their right to human review (GDPR Art. 6, Art. 22)*

4. *\*\*Data retention policy\*\*: 7-year maximum retention, then full anonymization or deletion (GDPR Art. 5(1)(e))*

5. *\*\*Data minimization\*\*: Remove IP addresses after fraud checks, aggregate spending categories to prevent inference of protected characteristics (GDPR Art. 5(1)(c))*
6. 
7. *\*\*Quarterly bias audits\*\*: Compute disparate impact ratios for gender and age regularly; DI below 0.8 triggers automatic review (AI Act Art. 10)*



## Individual Contributions

|Name|Role|Contributions|
|-|-|-|
|\[Your name]|Product Lead|...|
|...|Data Engineer|...|
|...|Data Scientist|...|
|Winifred Ojinere|Governance Officer|Privacy \& GDPR analysis, EU AI Act classification, pseudonymization demo, governance recommendations|



