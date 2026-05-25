# Compliance Checklists

## GDPR Checklist (`/legal/gdpr-checklist.md`)

### Lawful Basis
- [ ] Every processing activity has an identified lawful basis (Art. 6)
- [ ] Special category data has an additional Art. 9 basis (explicit consent, vital interests, etc.)
- [ ] Legitimate interests assessed via LIA (Legitimate Interests Assessment) where relied upon
- [ ] No "consent by default" — consent is unbundled, specific, informed, unambiguous
- [ ] Consent withdrawal is as easy as giving it

### Privacy Notices
- [ ] Notice provided at point of collection (or within 1 month if indirect collection)
- [ ] Notice includes: identity, contact, DPO, purposes, legal bases, recipients, retention, rights, right to complain to SA
- [ ] Language is plain — no legalese beyond what is necessary
- [ ] Cookie consent banner meets ePrivacy requirements (no pre-ticked boxes, reject as easy as accept)

### Data Subject Rights
- [ ] Right to access: respond within 1 month; mechanism in place
- [ ] Right to erasure: criteria assessed; mechanism in place; cascades to sub-processors
- [ ] Right to portability: for consent/contract data; machine-readable format available
- [ ] Right to rectification: correction mechanism exists
- [ ] Right to object: to legitimate interest processing and direct marketing (unconditional)
- [ ] Right to restrict processing: mechanism in place
- [ ] Automated decision-making/profiling: if Art. 22 applies, human review option available
- [ ] Response within 1 calendar month (extensible to 3 for complex requests)

### Third Parties & Processors
- [ ] DPA signed with every processor (anyone processing personal data on your behalf)
- [ ] Sub-processor list maintained and kept current
- [ ] Mechanism to notify controller of new sub-processors with objection window
- [ ] Joint controller agreement where applicable

### International Transfers
- [ ] No personal data transferred outside EEA without a transfer mechanism
- [ ] Transfer mechanism documented per data flow: adequacy decision / SCCs / BCRs / derogation
- [ ] Post-Schrems II transfer impact assessment (TIA) completed where SCCs used

### Security
- [ ] Encryption at rest and in transit
- [ ] Access controls and least privilege enforced
- [ ] Breach detection process documented
- [ ] Breach notification to SA within 72 hours (and to individuals if high risk)
- [ ] Regular security testing (pen tests, vulnerability scanning)

### Accountability & Documentation
- [ ] RoPA maintained (Article 30) — see references/document-structures.md
- [ ] DPIA conducted for high-risk processing (mandatory for large-scale special category, systematic profiling, biometrics, etc.)
- [ ] Privacy by design and by default applied to new features
- [ ] Staff data protection training conducted

### Governance
- [ ] DPO appointed (mandatory if: public authority, large-scale special category processing, large-scale systematic monitoring)
- [ ] DPO contact published on privacy policy
- [ ] DPO notified to supervisory authority (if required by national law)
- [ ] Lead supervisory authority identified (if operating across multiple EU member states)


## CCPA/CPRA Checklist (`/legal/ccpa-checklist.md`)

### Applicability Confirmation
- [ ] For-profit business with California residents as consumers
- [ ] Meets at least one threshold: >$25M revenue / 100,000+ consumers-households-devices / 50%+ revenue from selling/sharing

### Required Disclosures
- [ ] Privacy policy includes: categories collected, purposes, categories sold/shared, retention periods, consumer rights, how to exercise rights
- [ ] "Do Not Sell or Share My Personal Information" link in footer (if selling/sharing)
- [ ] Privacy policy updated at least every 12 months

### Consumer Rights
- [ ] Right to know: categories + specific pieces; respond within 45 days (extendable 45 more)
- [ ] Right to delete: honored within 45 days; communicated to service providers
- [ ] Right to opt-out of sale/sharing: "Do Not Sell or Share" link or Global Privacy Control (GPC) honored
- [ ] Right to correct inaccurate personal information
- [ ] Right to limit use of sensitive personal information (use only for primary purpose)
- [ ] Right to non-discrimination: no denying service or charging more for exercising rights
- [ ] Two-step verification for deletion requests (confirm intent + identity)

### Sensitive Personal Information
- [ ] Sensitive PI categories identified: SSN, financial account, precise geolocation, racial/ethnic origin, religious beliefs, genetic data, biometric, health, sex life, union membership, contents of communications
- [ ] "Limit the Use of My Sensitive Personal Information" link if sensitive PI used beyond primary purpose

### Service Providers & Contractors
- [ ] Written contract with every service provider and contractor (restricts their use of data to stated purpose)
- [ ] Contract prohibits combining data with other sources (unless permitted)
- [ ] Service providers not counted as "sale" if contract compliant

### Data Security
- [ ] Reasonable security measures in place (CCPA creates private right of action for data breaches)
- [ ] Sensitive PI subject to heightened security

### CPRA-Specific (additions since 2023)
- [ ] Retention periods disclosed per category
- [ ] Data minimization principle applied
- [ ] Automated decision-making: consumer right to opt-out if applicable
- [ ] Annual cybersecurity audit if processing sensitive PI at risk


## Singapore PDPA Checklist (`/legal/pdpa-sg-checklist.md`)

### Consent & Collection
- [ ] Notification of purpose given before or at time of collection
- [ ] Consent obtained (or valid exception applies: legitimate interests, contractual necessity, life-threatening emergency, etc.)
- [ ] Consent not bundled with terms of service in a way that makes it non-voluntary
- [ ] No excessive collection beyond stated purpose
- [ ] Deemed consent by notification: 30-day advance notice given for new uses

### Purpose Limitation & Access
- [ ] Data used only for purposes notified at collection (or reasonably related)
- [ ] Individuals can access and correct their personal data on request
- [ ] Response to access/correction request within reasonable time

### Care & Protection
- [ ] Reasonable security arrangements to protect personal data
- [ ] Processor (data intermediary) agreements in place
- [ ] Staff handling personal data trained

### Retention & Disposal
- [ ] Data not retained longer than necessary for its purpose
- [ ] Secure disposal process documented

### Breach Notification
- [ ] Breach assessment process: determine if likely to cause significant harm OR affects 500+ individuals
- [ ] If both conditions met: notify PDPC within 3 calendar days of assessment
- [ ] Notify affected individuals if breach likely to cause significant harm
- [ ] Breach response and communication templates prepared

### DPO Appointment
- [ ] Data Protection Officer (DPO) appointed
- [ ] DPO contact details published on website
- [ ] DPO registered with PDPC (if applicable)
- [ ] DPO accessible for data protection queries from public

### Do Not Call (DNC) Registry
- [ ] Before sending marketing messages to Singapore numbers: check against DNC Registry
- [ ] Telemarketing calls: check Do Not Call Registry before calling
- [ ] "Clear and unambiguous" opt-out provided in each marketing message
- [ ] Opt-outs honored within 30 days

### Data Portability (if applicable to business customer data)
- [ ] Data portability request process established
- [ ] Machine-readable format available for portable data

### Cross-Border Transfers
- [ ] Personal data only transferred to countries with comparable protection, OR
- [ ] Contractual safeguards equivalent to PDPA obligations in place


## APAC Compliance Comparison Table

| Requirement | Singapore PDPA | Thailand PDPA | Japan APPI | Australia APPs | NZ Privacy Act | South Korea PIPA | India DPDP |
|---|---|---|---|---|---|---|---|
| Lawful basis required | Consent / LI | Yes (6 bases) | Purpose specification | Collection notice | Collection notice | Consent (strict) | Consent / Legitimate use |
| DPO mandatory | Yes | Large-scale | No | No | No | Yes (Privacy Officer) | Significant Data Fiduciaries |
| Breach notification to authority | 3 days (if 500+ or significant harm) | 72 hours | To PPC + individuals | OAIC (NDB scheme) | If serious harm | 72 hours | To Data Protection Board |
| Breach notification to individuals | If significant harm | If high risk | Yes | If serious harm | If serious harm | Yes | Yes |
| Cross-border transfer restriction | Comparable protection or contractual | Adequate protection or consent | Consent or adequate protection | Reasonable steps | Yes | Consent or contractual safeguards | Government-whitelisted countries only |
| Data subject access right | Yes | Yes | Yes | Yes (APPs 12-13) | Yes | Yes | Yes |
| Right to deletion/erasure | Via withdrawal of consent | Yes | Via opt-out | Limited | Yes | Yes | Yes (Right to Erasure) |
| Data portability | Business data portability | No | No | No | No | No | Yes |
| Registration requirement | DPO with PDPC | No | No | No | No | PIPC notification | Data Fiduciary registration (TBD) |
| Max penalty | S$1M or 10% turnover | THB 5M or 2% global revenue | JPY 100M | A$50M or 30% of turnover | NZD 10,000 (individual) | 3% of total sales | INR 250 crore (significant DF) |
