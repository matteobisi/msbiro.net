---
title: "Your Products Are Already in Scope of the EU Cyber Resilience Act: A Practical Roadmap for Executives"
date: 2026-08-04T09:00:00Z
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
description: "The EU Cyber Resilience Act (CRA, Regulation 2024/2847) is vague, broad, and already running. A DevSecOps team leader's practical roadmap helps European C-level leaders understand scope, meet the September 2026 reporting deadline, and reach December 2027 full compliance without panic."
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

If you lead a company that sells software, hardware, or connected devices into the European Union, the Cyber Resilience Act (CRA, Regulation (EU) 2024/2847) already applies to you. It entered into force on 10 December 2024, and it does not need to be transposed into national law. It is live law.

The data confirms this. The OpenSSF 2026 CRA Awareness and Readiness Report, published by The Linux Foundation, surveyed 843 organizations and analyzed over 12,000 open source projects. The findings are not reassuring:

| Finding | Share of respondents | What it means for your company |
| --- | --- | --- |
| Unfamiliar with the CRA | 66% | You are not alone in being unprepared, but the deadline does not wait |
| Never assessed whether the CRA applies | 41% | You cannot plan a path you have not confirmed you are on |
| Cannot identify December 2027 as the full compliance date | 66% | Missed deadlines translate directly into enforcement risk |
| Unaware that fines can reach EUR 15 million or 2.5% of global turnover | 56% | The cost of non-compliance is structural, not symbolic |

This is not a niche compliance topic. It is a market access requirement that most of the industry is still ignoring.

I write this as a DevSecOps team leader. My job is to turn regulation into engineering reality, so I look at the CRA the way I look at any production risk: what is the deadline, what is the obligation, and what is the smallest practical path to compliance. This article is my attempt to give you that path, in plain language, without the legal fog.

---

## The Three Dates That Decide Your Compliance

The CRA is not a directive to be interpreted differently in each member state. It is a regulation, which means it applies uniformly across the EU. It covers what the law calls products with digital elements (PDEs): hardware, software, and the backend services they depend on, made available on the EU market in the course of commercial activity.

Three consequences follow, and each one lands in a different part of your business. From December 2027, non-compliant products cannot be placed on the EU market at all, because the CRA is a CE-marking regulation. The obligations apply to every economic operator, not just the manufacturer: importers and distributors carry their own due diligence duties, and a claim of "the manufacturer told us it was compliant" is not a defence. And the exposure is financial as well as operational, with the fines noted above, plus recalls and corrective orders from market surveillance authorities.

The CRA does not arrive all at once. It is being enforced in phases, and each phase has a distinct purpose.

| Date | What happens | Who it hits first |
| --- | --- | --- |
| 10 December 2024 | The CRA entered into force. Transition periods started. | Everyone, but softly |
| 11 June 2026 | The rules for notifying conformity assessment bodies apply. | Manufacturers of Important Class I and II products |
| 11 September 2026 | Vulnerability and incident reporting becomes mandatory through the ENISA Single Reporting Platform (SRP). | Every manufacturer of products already on the market |
| 11 December 2027 | Full application: essential security requirements, SBOM, CE marking, EU Declaration of Conformity, technical documentation, and market surveillance. | Every manufacturer, importer, and distributor |

The date that should be on your board calendar is not 2027. It is 11 September 2026. That is when vulnerability reporting becomes mandatory for products that are already on the market, and it is only weeks away. From that date, if an actively exploited vulnerability affects a product you have placed on the EU market, the reporting clock starts the moment you become aware, not when a CVE is published.

The reporting obligation in Article 14 of the CRA has three steps, and the first one is brutal.

| Step | Deadline | Content |
| --- | --- | --- |
| Early warning | Within 24 hours of becoming aware | Report to the relevant CSIRT and to ENISA through the SRP |
| Full notification | Within 72 hours | A complete vulnerability notification, unless already covered by the early warning |
| Final report | Within 14 days after a fix or mitigation is available | For actively exploited vulnerabilities; one month for severe incidents |

Two details make this harder than it looks. First, the obligation applies to products already on the market. Article 69(3) is explicit: if you shipped a product in 2020 and it is still in use, and it develops an actively exploited vulnerability, you must report it from September 2026. Your compliance date is not tied to your release cycle. It is tied to the market.

Second, the clock starts at internal awareness. Your engineering team discovering a vulnerable dependency in the morning can start it before anyone has been notified. To meet this, you do not need a better legal team. You need component-level visibility and a tested escalation path.

The small enterprise exemption is narrower than it sounds. Microenterprises and small enterprises are exempt from the fine for missing the 24-hour early warning window, but they are not exempt from reporting. The obligation still applies.

By 11 December 2027, every product placed on the EU market must demonstrate conformity with the essential cybersecurity requirements of Annex I, carry CE marking, and be supported by an EU Declaration of Conformity and technical documentation. Market surveillance authorities can then test products, demand documentation, order fixes or recalls, and impose the fines described above.

Two parts of Annex I deserve your attention before anything else. Products must ship without known exploitable vulnerabilities, use secure default configurations, implement appropriate access control, and minimize the attack surface. Manufacturers must also identify and document their components, run a coordinated vulnerability disclosure process, fix vulnerabilities, make patches available, and declare a support period of at least five years unless the product lifecycle is shorter. The support period end date must be communicated at the point of purchase, which is a transparency obligation with direct product management implications.

---

## What the CRA Demands of Your Products

Security by design is now a legal requirement, not a best practice. Legal requirements must be demonstrated. Your development process must produce reviewable artefacts: threat models, risk assessments, and documented decisions. An internal Information Security Management System is not enough, because the CRA is product-centric.

Do not rely on ISO 27001 or IEC 62443. They are not a basis for CRA conformity. The harmonized standards under Mandate M/606 are still being developed, so treating existing standards as sufficient creates a false sense of readiness.

Your conformity pathway depends on your product category. Most products self-certify under Module A: the manufacturer assesses the product, produces the documentation, and affixes the CE mark. No third party is required.

| Class | Examples | Conformity pathway |
| --- | --- | --- |
| Default | Most hardware and software | Self-certification (Module A) |
| Important Class I | Password managers, standalone browsers, VPNs, network management systems | Self-certification when harmonized standards are applied; otherwise a notified body is required |
| Important Class II | Operating systems, firewalls, routers, microprocessors, ICS software | Notified body assessment is always required |
| Critical (Annex IV) | Smart cards, smart meter gateways, hardware security modules, secure elements | Notified body assessment is always required, with no self-certification |

Capacity is the practical problem. The rules for notified bodies only applied from June 2026, and harmonized standards are still settling. If your product needs a notified body, start identifying candidates early; the Built to Last guide and industry readiness data both point to a tight window before December 2027. The classification exercise starts with Implementing Regulation (EU) 2025/2392, which defines the Important and Critical categories. If you have not classified each product line, you cannot plan.

Your supply chain is part of the deal. When open source ends up in a commercial product, the manufacturer carries the obligation for it. Manufacturers are responsible for the full product, including third-party and open source dependencies.

This changes dependency economics. The 2026 report found organizations maintain 86 private forks on average, costing about USD 258,000 in labour per release cycle. Under the CRA, private forks become technical debt that complicates the required chain of provenance. The report also found CVE discoveries surged 394% year-over-year in Q1 2026, with high-severity vulnerabilities up 811%. The vulnerability burden on your components is growing, and it will land on your reporting obligations.

The strategic answer is upstream investment. Contributing fixes to the projects you depend on strengthens your supply chain and your compliance posture at once. Fund and support the projects you rely on; their security becomes your security.

---

## A Practical Roadmap for the Next 18 Months

This is the part I want you to take to your leadership team. The CRA feels vague until you sequence it. Here is the roadmap I would execute, as a DevSecOps team, with the practices and tooling that make each step concrete.

The CRA sets outcome requirements. It does not mandate specific tools. The tooling column below lists commonly used open source options that teams already run in CI pipelines; treat them as examples, not requirements.

**Phase 1, now to September 2026: know your scope and your exposure.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Classify every product line and record the conformity pathway | Implementing Regulation (EU) 2025/2392; CRA Annexes III and IV | Product inventory and legal review |
| Assign cybersecurity ownership and reporting responsibility | CRA Article 13 (manufacturer obligations) | RACI and named PSIRT owner |
| Generate SBOMs in CI for every product | SPDX (ISO/IEC 5962) or CycloneDX, machine-readable | Syft, Trivy |
| Monitor vulnerability sources against those SBOMs | EUVD, NVD, OSV, CISA KEV, GitHub Advisories | OSV-Scanner, Grype, Trivy |
| Prepare registration on the ENISA Single Reporting Platform | ENISA SRP (operational by 11 September 2026) | ENISA SRP portal |

**Phase 2, before September 2026: make the 24-hour clock survivable.**

| Action | Practice or standard | Example tooling |
| --- | --- | --- |
| Publish a coordinated vulnerability disclosure process and security contact | CRA Annex I Part II; SECURITY.md; ISO/IEC 29147 and 30111 as CVD practice references | SECURITY.md in every product repo |
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

None of these steps requires a large compliance department. Most of them are good engineering that a DevSecOps team can implement directly. What they require is leadership commitment and a sequence.

---

## Do Developers Need to Do Anything?

The answer depends on your role. If you publish open source on your personal account without monetizing it, the CRA does not obligate you directly. Non-commercial open source is out of scope.

My own case is a good test. [Apple Container TUI (actui)](https://github.com/matteobisi/apple-container-tui) is a keyboard-first terminal UI for managing Apple Container on macOS, released under MIT on my personal GitHub. It started as a proof of concept of spec-driven development, and I maintain it alone. Do I need to do something?

Not directly. I do not charge for it, sell support beyond cost, or collect donations that exceed what I spend, so I fall outside the CRA's scope. The same applies to my contributions to CNCF open source projects. Those are non-commercial contributions to code owned by a foundation, and the CRA treats foundations as open source software stewards, with lighter obligations than manufacturers.

But there is a catch. Manufacturers who integrate open source must do due diligence on components and report vulnerabilities upstream. The result is that maintainers outside the CRA are now asked for documentation, security contacts, and patches they were never asked for before.

This is why I treat actui as if it were in scope. The repository has a published SECURITY.md, OpenSSF Scorecard, Dependabot, branch protection, and releases that ship with an SBOM and provenance attestations. Not because the CRA demands it of me. Because commercial users will ask for it, and being ready is cheaper than answering ad hoc requests from every company that integrates the code.

The practical answer for any developer is simple. Publish a SECURITY.md with a clear contact and disclosure process. Document your security practices. And be honest about your monetization status, because as soon as you charge for the product or build a business around the project, the CRA applies and the project becomes a product.

---

## A Final Word

The 2026 report shows that most of the industry is waiting. The organizations that move now will be the exception, and in a CE-marking regime, being the exception is a market advantage. Every company that cannot demonstrate conformity becomes a procurement risk for its customers, and customers will start asking the questions the CRA makes standard: what product category, what conformity pathway, what support period, what coordinated disclosure process.

My experience as a DevSecOps leader is that regulation like this is best met with an engineering mindset. You break the legal text into obligations, the obligations into artefacts, and the artefacts into a delivery plan. You test the plan, not the theory. You simulate the incident before the regulator asks. That is the approach my team applies to the CRA, and it is the same approach we apply to every security requirement that has a deadline.

If your organization needs help turning this regulation into a working programme, a conversation with a DevSecOps team that has done this before is a reasonable place to start. The clock is running, but the path is clear.

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
- [Built to Last: Understanding the European Cyber Resilience Act, OpenSSF](https://openssf.org/resources/built-to-last-understanding-the-european-cyber-resilience-act-cra/)
- [OpenSSF EU Cyber Resilience Act public policy page](https://openssf.org/public-policy/eu-cyber-resilience-act/)
- [OpenSSF Scorecard](https://securityscorecards.dev/)

**SBOM formats and vulnerability data sources**

- [SPDX specification (ISO/IEC 5962)](https://spdx.dev/)
- [CycloneDX specification](https://cyclonedx.org/)
- [OSV vulnerability database](https://osv.dev/)
- [CISA Known Exploited Vulnerabilities catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [NVD (National Vulnerability Database)](https://nvd.nist.gov/)

*This article is for informational and educational purposes only. It does not constitute legal advice and does not create an advisory relationship. The CRA's implementing acts, delegated acts, harmonized standards, and Commission guidance are still evolving. Tools and standards named above are examples of common practice, not CRA-mandated products or certifications. Verify the current status of any regulatory reference with qualified legal counsel before acting on it in a commercial context.*