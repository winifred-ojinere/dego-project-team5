# dego-project-team5

# Credit Application Governance Analysis

DEGO 2606 | Nova SBE | Team 5

---

## Executive Summary

NovaCred's credit application dataset (502 records) was audited for data quality, algorithmic bias, and GDPR/AI Act compliance. The audit uncovered significant issues across all three dimensions.

On data quality, we identified duplicate records, missing values across multiple fields, invalid financial values, and formatting inconsistencies in emails and SSNs. On fairness, female applicants face a Disparate Impact ratio of **0.775** — below the four-fifths (0.80) legal threshold — and applicants under 30 face a DI ratio of **0.638**, both statistically significant. ZIP code was identified as a likely proxy variable for gender. On privacy, **497 SSNs** are stored in plain text, no consent mechanism exists, and all credit decisions are fully automated with no human oversight — a critical violation of both GDPR Art. 22 and the EU AI Act.

Immediate remediation is required before NovaCred can claim regulatory compliance.

---

## Video Presentation
[Watch our presentation here] https://youtu.be/PdL5YLIPXGo

## Data Quality Findings

The raw dataset contained **504 records** across a nested JSON structure. After cleaning, **502 unique records** were retained.

### Duplicates
Two application IDs appeared more than once: `app_001` and `app_042`. One pair was fully identical (Joseph Lopez); the other (Stephanie Nguyen) had one complete and one incomplete version. Duplicates were resolved by retaining the most complete record per ID.

### Missing Values
Several fields contained missing data. The most affected were `decision.interest_rate` and `decision.approved_amount` (expected for rejected applications), but `applicant_info.ssn` was missing in 4 records and `applicant_info.email` in several others.

### Invalid Values (Accuracy)
Rule-based plausibility checks flagged 4 records with impossible financial values:
- 2 records with **negative credit history months** (logically impossible)
- 1 record with a **negative savings balance**
- 1 record with a **debt-to-income ratio above 1.0** (debt exceeds income)

These records were flagged but not removed, as further validation is needed to confirm whether they are entry errors.

### Validity (Format Checks)
- **10 emails** did not match the expected `x@x.x` format — missing `@` symbols, incomplete domains, or blank values
- **SSN format** (XXX-XX-XXXX) was structurally valid for most records, but 4 SSN fields were entirely missing

### Data Type Inconsistencies
Some fields exhibited type mismatches across records (e.g., income stored as string in some entries), consistent with the nested JSON origin of the data. These were resolved during the cleaning pipeline.

| Issue | Count | % Affected |
|---|---|---|
| Duplicate application IDs | 2 | 0.4% |
| Invalid email format | 10 | 2.0% |
| Missing SSN | 4 | 0.8% |
| Negative credit history months | 2 | 0.4% |
| Negative savings balance | 1 | 0.2% |
| Extreme debt-to-income ratio (>1.0) | 1 | 0.2% |

Full analysis in `notebooks/01-data-quality.ipynb`.

---

## Bias Analysis

### Gender — Disparate Impact

Approval rates were compared between male and female applicants using the cleaned dataset (387 records with binary gender and non-missing decisions).

| Group | Total | Approved | Approval Rate |
|---|---|---|---|
| Male | 194 | 131 | 67.5% |
| Female | 193 | 101 | 52.3% |

**Disparate Impact (Female / Male) = 0.775** — below the four-fifths (0.80) threshold, indicating potential adverse impact against female applicants.

A chi-square test confirmed the association is statistically significant (χ² = p = 0.0032, Cramér's V = 0.150), meaning the gap is unlikely to be due to random variation. A logistic regression controlling for income, credit history, debt-to-income ratio, and savings balance showed the gender effect persists even after accounting for financial differences — the disparity is not fully explained by financial profiles.

Among approved applicants, females also received **lower average approved amounts**, suggesting the disadvantage extends beyond the approval decision itself.

### Age — Disparate Impact

Applicants were grouped into three age bands. Using 30–50 as the reference group:

| Age Group | Approval Rate | DI vs 30–50 | Flags Four-Fifths Rule? |
|---|---|---|---|
| < 30 | 39.7% | 0.638 | ✅ Yes |
| 30–50 | 62.3% | 1.000 | — |
| 50+ | 59.7% | 0.959 | No |

The under-30 group faces the most severe disparity (DI = 0.638). A chi-square test confirmed statistical significance (χ² = 10.80, p = 0.00451, Cramér's V = 0.179). Among approved applicants, younger applicants also receive the lowest average approved amounts.

### Age × Gender Interaction

Approval rates broken down by both age and gender reveal compounding disadvantage:

| Age Group | Female | Male | Gap (M − F) |
|---|---|---|---|
| < 30 | 35.7% | 47.6% | 11.9 pp |
| 30–50 | 57.5% | 64.4% | 6.9 pp |
| 50+ | 54.2% | 76.0% | 21.8 pp |

The gender gap is present across all age groups but is largest for the 50+ group (21.8 percentage points). The most disadvantaged subgroup overall is women under 30 (35.7% approval rate). Fairness interventions should account for these intersectional effects rather than treating age and gender as independent.

### ZIP Code Proxy Analysis

ZIP code approval rates vary substantially across areas, and a Cramér's V test showed a meaningful association between ZIP code and gender distribution. This suggests **ZIP code may act as a proxy for gender** — re-introducing indirect discrimination even if gender is formally excluded from the model. ZIP-based features should be treated with caution in any production model.

Full analysis in `notebooks/02-bias-analysis.ipynb`.

---

## Privacy Assessment

We identified **8 fields** containing personally identifiable information stored without protection.

| Field | PII Type | Risk Level |
|---|---|---|
| full_name | Direct identifier | High |
| email | Direct identifier | High |
| ssn | Sensitive national identifier | Critical |
| ip_address | Online identifier | Medium |
| date_of_birth | Quasi-identifier | Medium |
| zip_code | Quasi-identifier | Medium |
| gender | Special category (Art. 9) | High |
| spending_behavior | Behavioral data (Gambling, Alcohol, Adult Entertainment) | Medium-High |

### GDPR Compliance Gaps

| Article | Issue | Severity |
|---|---|---|
| Art. 5(1)(c): Data Minimization | IP addresses and sensitive spending categories not necessary for credit scoring | Medium |
| Art. 5(1)(e): Storage Limitation | No data retention policy — 502 records stored indefinitely | High |
| Art. 6(1): Lawful Basis | No consent field exists in the dataset — lawful basis undocumented | High |
| Art. 9: Special Categories | Gender collected without documented exception; spending data can reveal health/addiction status | High |
| Art. 17: Right to Erasure | 497 SSNs in plain text — erasure requests cannot be fulfilled | Critical |
| Art. 22: Automated Decisions | All decisions are fully automated — rejection reason is `algorithm_risk_score` with no human review | Critical |
| Art. 25: Privacy by Design | PII stored unprotected, no encryption or access controls evident | High |
| Art. 35: Impact Assessment | No DPIA conducted despite high-risk automated credit scoring | High |

### EU AI Act Classification

Credit scoring is classified as **high-risk** under Annex III, Section 5(b) of the EU AI Act. NovaCred currently fails all seven compliance requirements: risk management, data governance, technical documentation, audit trails, transparency, human oversight, and accuracy testing.

### Pseudonymization Demonstration

SHA-256 hashing with a salt was applied to SSN, full name, and email fields across all 502 records. This prevents casual re-identification and supports duplicate detection while preserving referential integrity. Note: pseudonymized data remains personal data under GDPR — this is not equivalent to anonymization and does not remove GDPR obligations.

Full analysis in `notebooks/03-privacy-demo.ipynb`.

---

## Governance Recommendations

1. **Decision audit trail** — Log every credit decision with timestamp, applicant ID, features used, model version, output, and confidence score (AI Act Art. 12)
2. **Human-in-the-loop** — Flag borderline cases for mandatory manual review with documented override authority (GDPR Art. 22, AI Act Art. 14)
3. **Informed consent mechanism** — Add a consent tracking field; inform applicants about AI use and their right to request human review (GDPR Art. 6, Art. 22)
4. **Data retention policy** — Enforce a 7-year maximum retention limit, followed by full anonymization or deletion (GDPR Art. 5(1)(e))
5. **Data minimization** — Remove IP addresses after fraud checks are complete; aggregate sensitive spending categories to prevent inference of protected characteristics (GDPR Art. 5(1)(c))
6. **Quarterly bias audits** — Compute disparate impact ratios for gender, age, and ZIP quarterly; a DI below 0.80 triggers automatic model review (AI Act Art. 10)

---

## Repository Structure

```
dego-project-team5/
├── README.md
├── data/
│   ├── raw_credit_applications.json
│   └── cleaned_credit_applications.csv
├── notebooks/
│   ├── 01-data-quality.ipynb
│   ├── 02-bias-analysis.ipynb
│   └── 03-privacy-demo.ipynb
├── src/
│   └── fairness_utils.py
└── presentation/
    └── [video link or file]
```

---

## Individual Contributions

| Name | Role | Contributions |
|---|---|---|
| Oliver Mourant | Product Lead | README, video coordination, presentation structure, repository management |
| Nicolas Oteri | Data Engineer | Data loading, JSON flattening, cleaning pipeline, duplicate resolution, `01-data-quality.ipynb` |
| Margarida Gonçalves Alves Rodrigues | Data Scientist | Bias metrics, fairness analysis, statistical testing, proxy analysis, `02-bias-analysis.ipynb` |
| Winifred Ojinere | Governance Officer | Privacy & GDPR analysis, EU AI Act classification, pseudonymization demo, governance recommendations, `03-privacy-demo.ipynb` |

