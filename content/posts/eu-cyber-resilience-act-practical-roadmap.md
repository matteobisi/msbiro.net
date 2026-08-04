---
title: "The EU Cyber Resilience Act: A Practical Roadmap for Executives"
date: 2026-08-05T06:00:00Z
tags: [
  "cybersecurity", "devsecops", "CRA", "eu-regulation", "compliance",
  "sbom", "supply-chain-security", "vulnerability-management",
  "cyber-resilience-act", "leadership", "cto", "eng-security"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A practical roadmap to help European leaders assess whether the EU Cyber Resilience Act applies to their products, prepare for the September 2026 reporting duties, and work towards December 2027 product compliance."
canonicalURL: "https://www.msbiro.net/posts/eu-cyber-resilience-act-practical-roadmap/"
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "EU Cyber Resilience Act practical roadmap for C-level leaders"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

If you lead a company that sells software, hardware, or connected devices into the European Union, you should assess whether the Cyber Resilience Act (CRA, Regulation (EU) 2024/2847) applies to your products. The CRA entered into force on 10 December 2024 and, as a regulation, does not need national transposition. Its main product obligations apply from 11 December 2027, while manufacturer reporting duties start on 11 September 2026.

The [OpenSSF 2026 CRA Awareness and Readiness Report](https://openssf.org/resources/publications/2026-cra-awareness-and-readiness-report/), published by The Linux Foundation, surveyed 843 organizations and analyzed more than 12,000 open source projects. It points to a material readiness gap. It is useful evidence of what surveyed organizations are experiencing, not a measure of the whole industry.

This is not a niche compliance topic. For products within scope, it is a market access requirement. The potential penalties can reach EUR 15 million or 2.5% of worldwide annual turnover, whichever is higher.

I write this as a DevSecOps team leader. My job is to turn regulation into engineering reality, so I look at the CRA the way I look at any production risk: what is the deadline, what is the obligation, and what is the smallest practical path to compliance. This article is my attempt to give you that path, in plain language, without the legal fog.

---

## The Dates That Decide Your Compliance

The CRA is not a directive to be interpreted differently in each member state. It is a regulation, so it applies uniformly across the EU.

It covers what the law calls products with digital elements, or PDEs. These are hardware and software products, including separately marketed components, with an intended direct or indirect logical or physical data connection to a device or network. It also covers some remote data-processing solutions when their absence would stop a product performing an intended function. Before going further, assess the Article 2 exclusions and the exclusion for non-commercial free and open source software.

For products within scope, three consequences follow. From December 2027, they must meet the CRA before they can be placed on the EU market, because the CRA is a CE-marking regulation. Manufacturers carry the primary duties for product design, conformity assessment, documentation, the declaration of conformity, and CE marking. Importers, distributors, and authorized representatives have their own duties to verify, cooperate, and take corrective action. The exposure is financial as well as operational, with the fines noted above, plus corrective orders and recalls from market surveillance authorities.

The CRA does not arrive all at once. It is being enforced in phases, and each phase has a distinct purpose.

| Date | What happens | Who it hits first |
| --- | --- | --- |
| 10 December 2024 | The CRA entered into force and its transition periods began. | Everyone affected by the regulation |
| 11 June 2026 | The rules for notifying authorities and conformity assessment bodies apply. | Authorities and prospective notified bodies |
| 11 September 2026 | Article 14 vulnerability and incident reporting starts through the ENISA Single Reporting Platform (SRP). | Manufacturers |
| 11 December 2027 | The main product obligations apply: essential security requirements, SBOM, CE marking, EU Declaration of Conformity, technical documentation, and market surveillance. | Manufacturers, importers, distributors, and authorized representatives |

The date that should be on your board calendar is not 2027. It is 11 September 2026. From that date, if an actively exploited vulnerability affects a product you have made available on the EU market, the reporting clock starts when you become aware, not when a CVE is published.

Article 14 has three reporting steps. The first has a demanding deadline.

| Step | Deadline | Content |
| --- | --- | --- |
| Early warning | Within 24 hours of becoming aware | Submit through the SRP. The notification is addressed to the relevant CSIRT and made available to ENISA. |
| Full notification | Within 72 hours | Submit a vulnerability or incident notification, unless the relevant information was already supplied in the early warning. |
| Final report | Within 14 days after a corrective measure becomes available | This applies to actively exploited vulnerabilities. The final report for a severe incident is due within one month. |

Reporting can affect legacy products, but continued use by a customer is not the legal test on its own. If you still make a product available or support it, assess its position before September 2026. Your preparation cannot wait for a new release cycle.

The clock also starts at internal awareness. Your engineering team discovering a vulnerable dependency in the morning can start it before anyone outside the company has been notified. To meet the deadline, you need component-level visibility and a tested escalation path.

The small enterprise protection is narrower than it sounds. Microenterprises and small enterprises are protected from an administrative fine specifically for missing the 24-hour early-warning deadline. They are not exempt from reporting, and the other CRA duties still apply where the product is in scope.

By 11 December 2027, each product within scope that is placed on the EU market must demonstrate conformity with the essential cybersecurity requirements of Annex I, carry CE marking, and be supported by an EU Declaration of Conformity and technical documentation. Market surveillance authorities can then test products, demand documentation, order fixes or recalls, and impose the fines described above.

Two parts of Annex I deserve attention first. Products must ship without known exploitable vulnerabilities, use secure default configurations, implement appropriate access control, and minimize the attack surface.

Manufacturers must also identify and document their components, operate a coordinated vulnerability disclosure policy, fix vulnerabilities, make patches available, and declare the support period. That period must match the product's expected lifetime or five years from placing it on the market, whichever is shorter. It must be communicated clearly and accessibly at the time of purchase.

---

## What the CRA Demands of Your Products

Security by design is now a legal requirement, not a best practice. Legal requirements must be demonstrated. The CRA requires a documented cybersecurity risk assessment and technical documentation. Threat models, design reviews, and decision records are practical ways to produce that evidence. An internal Information Security Management System alone is not enough, because the CRA is product-centric.

Do not treat ISO 27001 or IEC 62443 as automatic proof of CRA conformity. They can provide useful evidence, but they do not by themselves create a presumption of conformity. That presumption follows from applicable harmonized standards cited in the Official Journal, or applicable common specifications. The harmonized standards under Mandate M/606 are still developing, so treating existing standards as sufficient creates a false sense of readiness.

Your conformity pathway depends on your product category. Most products self-certify under Module A: the manufacturer assesses the product, produces the documentation, and affixes the CE mark. No third party is required.

| Class | Examples | Conformity pathway |
| --- | --- | --- |
| Default | Most hardware and software | Self-certification (Module A) |
| Important Class I | Password managers, standalone browsers, VPNs, network management systems | Self-certification when applicable harmonized standards or common specifications are applied; otherwise a notified body is required |
| Important Class II | Operating systems, firewalls, routers, microprocessors, ICS software | Notified body assessment is always required |
| Critical (Annex IV) | Smart cards, smart meter gateways, hardware security modules, secure elements | Notified body assessment is always required, with no self-certification |

Capacity is the practical problem. The rules that allow notified bodies to be designated apply from June 2026, and harmonized standards are still settling. If your product needs a notified body, start identifying candidates early. Start classification with CRA Annexes III and IV, then use Implementing Regulation (EU) 2025/2392 for technical descriptions of the listed categories. Without a classification for each product line, you cannot plan.

Your supply chain is part of the deal. When open source ends up in a commercial product, the manufacturer remains responsible for the conformity of the full product, including relevant third-party and open source components. That means identifying and documenting components, managing vulnerabilities, and maintaining a coordinated vulnerability disclosure policy.

This changes dependency economics. Private forks can make provenance, maintenance, and vulnerability management harder to demonstrate. You need a clear view of the components in your product, who maintains them, and how you will assess and act on a vulnerability report.

The strategic answer is upstream investment. Contributing fixes to the projects you depend on strengthens your supply chain and your compliance posture at once. Fund and support the projects you rely on; their security becomes your security.

---

## A Practical Roadmap for the Next 18 Months

This is the part to take to your leadership team. The CRA feels vague until you turn it into an ordered plan. Here is the roadmap I would use as a DevSecOps team, with the practices and tooling that make each step concrete.

The CRA sets outcomes, not tool choices. The tooling column below lists commonly used open source options for CI pipelines. Treat them as examples, not requirements.

**Phase 1, now to September 2026: know your scope and your exposure.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Classify every product line and record the conformity pathway | CRA Annexes III and IV; Implementing Regulation (EU) 2025/2392 technical descriptions | Product inventory and legal review |
| Assign cybersecurity ownership and reporting responsibility | CRA Article 13 (manufacturer obligations) | RACI and named PSIRT owner |
| Generate SBOMs in CI for every product | SPDX (ISO/IEC 5962) or CycloneDX, machine-readable | Syft, Trivy |
| Monitor vulnerability sources against those SBOMs | EUVD, NVD, OSV, CISA KEV, GitHub Advisories | OSV-Scanner, Grype, Trivy |
| Prepare registration on the ENISA Single Reporting Platform | ENISA SRP (operational by 11 September 2026) | ENISA SRP portal |

**Phase 2, before September 2026: make the 24-hour clock survivable.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Establish a coordinated vulnerability disclosure policy and a security contact | CRA Annex I Part II; ISO/IEC 29147 and 30111 as CVD practice references | A published SECURITY.md is a strong practical implementation |
| Formalize or create a Product Security Incident Response Team (PSIRT) | CRA Article 14 reporting path | On-call rota and intake channel |
| Define and test the internal escalation path that starts the 24-hour clock | Tabletop exercise before first real incident | Runbook and simulated disclosure drill |
| Automate SBOM-based vulnerability matching | Continuous matching against OSV, NVD, CISA KEV | OSV-Scanner, Grype, Trivy |
| Draft notification templates for early warning, full notification, and final report | ENISA SRP format and CRA Article 14 steps | Pre-approved templates in the PSIRT kit |

**Phase 3, before December 2027: make the product compliant.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Embed security by design; produce threat models, risk assessments, and design decisions | CRA Annex I Part I | SDLC gates and design review records |
| Build and maintain the technical file | CRA Annex VII | Versioned documentation store |
| Run the conformity assessment; engage a notified body early where required | CRA Annex VIII modules (A, B+C, or H); NANDO list of notified bodies | Self-assessment pack or notified body engagement |
| Prepare the EU Declaration of Conformity, CE marking, and support period declaration | CRA Articles 13, 28, and 30 | Product packaging and release checklist |

**Phase 4, ongoing: make compliance sustainable.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Treat reporting and conformity as an operating model, not a one-off project | CRA Articles 13 and 14 | Quarterly compliance review |
| Contribute fixes upstream and reduce private forks | OpenSSF upstream contribution guidance | Patch contribution workflow |
| Keep market surveillance evidence ready on request | Annex VII technical file; declared support period | Audit-ready documentation pack |

None of these steps requires a large compliance department. Most are good engineering that a DevSecOps team can implement directly. What they do require is leadership commitment, clear ownership, and a sensible order.

---

## Do Developers Need to Do Anything?

The answer depends on your role and circumstances. Free and open source software developed or supplied outside a commercial activity is excluded from the CRA. That is a fact-specific test, not a simple question of whether money changes hands.

My own case is a good test. [Apple Container TUI (actui)](https://github.com/matteobisi/apple-container-tui) is a keyboard-first terminal UI for managing Apple Container on macOS, released under MIT on my personal GitHub. It started as a proof of concept of spec-driven development, and I maintain it alone. Do I need to do something?

The facts described for actui, a personal MIT-licensed project with no stated commercial offering, may support an assessment that it is outside a commercial activity. Revisit that assessment if how the project is supplied changes. Donations, cost recovery, support charges, and business use are relevant facts, but none is a bright-line test on its own.

The same care applies to contributions to foundation projects. A foundation is an open source software steward only if it meets the Article 3(14) definition. A qualifying steward has the specific duties in Article 24, but is not automatically the manufacturer of every project it supports.

But there is a catch. Manufacturers who integrate open source remain responsible for the relevant components in their products. Their Article 14 reports go through the SRP, not upstream to maintainers. In practice, commercial users may ask maintainers for documentation, security contacts, and patches that they were never asked for before.

This is why it is sensible to treat a widely used project as if commercial users will ask security questions. A clear security contact, a documented disclosure process, dependency information, and repeatable release evidence make those conversations easier. They are practical choices, not automatic CRA duties for a non-commercial maintainer.

The practical answer for any developer is simple. Publish a clear security contact and disclosure process, such as a SECURITY.md. Document your security practices. If the project is supplied commercially, assess the CRA carefully rather than relying on a single signal such as donations or support revenue.

---

## A Final Word

The 2026 report suggests that many surveyed organizations are still preparing. Organizations that move now can make their procurement conversations easier. In a CE-marking regime, customers will increasingly ask the questions the CRA makes standard: what product category, what conformity pathway, what support period, and what coordinated disclosure process?

My experience as a DevSecOps leader is that regulation like this is best met with an engineering mindset. You break the legal text into obligations, the obligations into artefacts, and the artefacts into a delivery plan. You test the plan, not the theory. You simulate the incident before the regulator asks. That is the approach my team applies to the CRA, and it is the same approach we apply to every security requirement that has a deadline.

If your organization needs help turning this regulation into a working program, a conversation with an experienced DevSecOps team is a reasonable place to start. The clock is running, but the path is clear.

---

## References

**Regulation and official guidance**

- [Regulation (EU) 2024/2847, the Cyber Resilience Act (full text)](https://eur-lex.europa.eu/eli/reg/2024/2847/oj)
- [Implementing Regulation (EU) 2025/2392 on product categories](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32025R2392)
- [European Commission CRA overview](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act)
- [European Commission CRA conformity assessment](https://digital-strategy.ec.europa.eu/en/policies/cra-conformity-assessment)
- [European Commission CRA reporting obligations](https://digital-strategy.ec.europa.eu/en/policies/cra-reporting)
- [Commission guidance on CRA implementation](https://digital-strategy.ec.europa.eu/en/library/commission-publishes-new-guidance-support-timely-cyber-resilience-act-implementation)
- [Commission FAQs on CRA implementation](https://digital-strategy.ec.europa.eu/en/library/cyber-resilience-act-implementation-frequently-asked-questions)
- [ENISA Single Reporting Platform (SRP) FAQ](https://www.enisa.europa.eu/topics/product-security-and-certification/single-reporting-platform-srp)
- [NANDO database of notified bodies](https://webgate.ec.europa.eu/single-market-compliance-space/notified-bodies)

**OpenSSF and industry readiness**

- [2026 CRA Awareness and Readiness Report, OpenSSF and The Linux Foundation](https://openssf.org/resources/publications/2026-cra-awareness-and-readiness-report/)

**SBOM formats and vulnerability data sources**

- [SPDX specification (ISO/IEC 5962)](https://spdx.dev/)
- [CycloneDX specification](https://cyclonedx.org/)
- [OSV vulnerability database](https://osv.dev/)
- [CISA Known Exploited Vulnerabilities catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [NVD (National Vulnerability Database)](https://nvd.nist.gov/)

*This article is for informational and educational purposes only. It does not constitute legal advice and does not create an advisory relationship. The CRA's implementing acts, delegated acts, harmonized standards, and Commission guidance are still evolving. Tools and standards named above are examples of common practice, not CRA-mandated products or certifications. Verify the current status of any regulatory reference with qualified legal counsel before acting on it in a commercial context.*