---
title: "Your Products Are Already in Scope of the EU Cyber Resilience Act: A Practical Roadmap for Executives"
date: 2026-08-01T09:00:00Z
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

If you lead a company that sells software, hardware, or connected devices into the European Union, the Cyber Resilience Act (CRA, Regulation (EU) 2024/2847) already applies to you. It entered into force on 10 December 2024, and it does not need to be transposed into national law. It is live law, and most executives have not yet acted on it.

The data confirms this. The OpenSSF 2026 CRA Awareness and Readiness Report, published by The Linux Foundation, found that 66% of respondents remain unfamiliar with the CRA, 41% of organizations have not yet determined whether it applies to them, only 34% correctly identify December 2027 as the full compliance date, and 56% do not know that fines can reach EUR 15 million or 2.5% of global annual turnover. This is not a niche compliance topic. It is a market access requirement that most of the industry is still ignoring.

I write this as a DevSecOps team leader. My job is to turn regulation into engineering reality, so I look at the CRA the way I look at any production risk: what is the deadline, what is the obligation, and what is the smallest practical path to compliance. This article is my attempt to give you that path, in plain language, without the legal fog.

---

## The Three Dates That Decide Your Compliance

The CRA is not a directive to be interpreted differently in each member state. It is a regulation, which means it applies uniformly across the EU. It covers what the law calls products with digital elements (PDEs): hardware, software, and the backend services they depend on, made available on the EU market in the course of commercial activity.

Three consequences follow, and each one lands in a different part of your business. From December 2027, non-compliant products cannot be placed on the EU market at all, because the CRA is a CE-marking regulation. The obligations apply to every economic operator, not just the manufacturer: importers and distributors carry their own due diligence duties, and a claim of "the manufacturer told us it was compliant" is not a defence. And the financial exposure is real, with fines up to EUR 15 million or 2.5% of global annual turnover, plus corrective measures, recalls, and requests from market surveillance authorities.

The CRA does not arrive all at once. It is being enforced in phases, and each phase has a distinct purpose.

| Date | What happens | Who it hits first |
| --- | --- | --- |
| 10 December 2024 | The CRA entered into force. Transition periods started. | Everyone, but softly |
| 11 June 2026 | The rules for conformity assessment bodies (notified bodies) became operational. | Manufacturers of Important Class I and II products |
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

Second, the 24-hour clock starts when you become aware, internally. That is an internal knowledge threshold. It means your engineering team discovering a vulnerable dependency in the morning can start the clock before anyone has notified you. To meet this, you do not need a better legal team. You need component-level visibility and a tested escalation path.

The small enterprise exemption is narrower than it sounds. Microenterprises and small enterprises are exempt from the fine for missing the 24-hour early warning window, but they are not exempt from reporting. The obligation still applies. The exemption is a protection from a specific penalty, not a pass.

By 11 December 2027, every product placed on the EU market must demonstrate conformity with the essential cybersecurity requirements of Annex I, carry CE marking, and be supported by an EU Declaration of Conformity and technical documentation. Market surveillance authorities can then test products, demand documentation, order fixes or recalls, and impose the fines described above. Two parts of Annex I deserve your attention before anything else. Products must ship without known exploitable vulnerabilities, use secure default configurations, implement appropriate access control, and minimize the attack surface. And manufacturers must identify and document their components, run a coordinated vulnerability disclosure process, fix vulnerabilities and make patches available, and declare a support period of at least five years unless the product lifecycle is shorter. You must communicate the end of support date at the point of purchase, which is a transparency requirement with direct product management implications.

---

## What the CRA Demands of Your Products

For years, security by design was a best practice. The CRA makes it a legal requirement, and that is a real shift: principles can be interpreted loosely, legal requirements have to be demonstrated. Your secure software development lifecycle has to produce artefacts that a reviewer can inspect. Threat models need to be documented. Risk assessments need to be defensible. The CRA is product-centric, not organization-centric: an internal Information Security Management System does not automatically prove product-level conformity.

One common trap is building a compliance programme around ISO 27001 or IEC 62443. Those standards are valuable, but they are not a basis for CRA conformity assessment. The harmonized standards under Mandate M/606 are still being developed by ETSI, CEN, and CENELEC, and until they are finalized, treating existing management-system standards as sufficient creates a risk of duplicated effort and a false sense of readiness.

You also need to know your conformity pathway, because it depends on your product category. Most products fall into the default category and can self-certify under Module A: the manufacturer assesses its own product, generates the technical documentation, produces the EU Declaration of Conformity, and affixes the CE mark. No third party is required.

| Class | Examples | Conformity pathway |
| --- | --- | --- |
| Default | Most hardware and software | Self-certification (Module A) |
| Important Class I | Password managers, standalone browsers, VPNs, network management systems | Self-certification when harmonized standards are applied; otherwise a notified body is required |
| Important Class II | Operating systems, firewalls, routers, microprocessors, ICS software | Notified body assessment is always required |
| Critical (Annex IV) | Smart cards, smart meter gateways, hardware security modules, secure elements | Notified body assessment is always required, with no self-certification |

The practical problem is capacity. The notified body framework only became operational on 11 June 2026, and the harmonized standards landscape is still settling. Organizations whose products require a notified body should be identifying candidates now, because demand is likely to outstrip supply as December 2027 approaches. The European Commission published Implementing Regulation (EU) 2025/2392 to clarify the technical descriptions of the Important and Critical categories, and it is the right starting reference for the classification exercise. If you have not determined which category each product line falls into, you do not know your pathway, and you cannot plan.

Your supply chain is part of the deal as well. Non-commercial open source is outside the CRA's scope, and individual contributors who publish code without monetizing it have no direct obligations. But the commercial use of open source triggers obligations for the manufacturers who integrate it. The CRA makes manufacturers responsible for the full product, including third-party and open source dependencies, and it requires them to notify upstream providers about vulnerabilities and fixes.

This changes the economics of dependency management. The 2026 report found that on average, organizations maintain 86 private forks of open source components, costing approximately USD 258,000 in labour per release cycle, with large organizations facing over 11,000 hours per cycle. Private forks were a workaround; under the CRA they become technical debt that actively complicates the auditable chain of provenance the regulation demands. The report's most striking supply chain finding is that CVE discoveries surged 394% year-over-year in Q1 2026, with high-severity vulnerabilities up 811%. Some of this reflects better automated detection, but the trend is clear: the vulnerability burden on the components you depend on is growing, and it will land on your reporting obligations.

The strategic answer is upstream investment. Contributing fixes back to the projects you depend on amortizes maintenance costs across the community, strengthens the security posture of your supply chain, and is a direct investment in your own compliance. Organizational diversity in a project is a strong predictor of its security posture. The projects you fund are the projects you will be able to defend.

---

## A Practical Roadmap for the Next 18 Months

This is the part I want you to take to your leadership team. The CRA is vague until you make it concrete, and it becomes concrete when you sequence it. Here is the roadmap I would execute, as a DevSecOps team.

**Phase 1, now, before September 2026: know your scope and your exposure.**

- Classify every product line against the CRA's definitions and Implementing Regulation (EU) 2025/2392, and record the conformity pathway for each one.
- Identify who is responsible for cybersecurity within the company, and who will own the reporting process. The CRA assumes a clear point of accountability.
- Start generating SBOMs in CI for every product, in a standard machine-readable format such as SPDX or CycloneDX.
- Begin monitoring vulnerability sources: the EUVD, NVD, OSV, the CISA KEV catalogue, and GitHub advisories.
- Register or prepare to register on the ENISA Single Reporting Platform, since reporting becomes mandatory in September 2026.

**Phase 2, before September 2026: make the 24-hour clock survivable.**

- Stand up a coordinated vulnerability disclosure process with a published security contact and a clear intake path. If you already have a PSIRT, formalize it; if you do not, the CRA is the reason to build one.
- Define the internal escalation path that triggers the early warning, and test it with a simulation, because the first real incident is not the right moment to discover the process.
- Automate SBOM-based vulnerability matching against public databases so that a newly disclosed CVE is triaged against your products in hours, not days.
- Draft the notification templates for the three reporting steps, since writing them under a 24-hour clock is how mistakes happen.

**Phase 3, before December 2027: make the product compliant.**

- Embed the Annex I security by design requirements into the development lifecycle, and produce the documentation that demonstrates them: threat models, risk assessments, and design decisions.
- Build the technical file per Annex VII, including product description, intended use, applied standards, the cybersecurity risk assessment, the design and production process, and vulnerability handling records.
- Run the conformity assessment for each pathway, and engage a notified body early for Class I without harmonized standards, Class II, and critical products.
- Prepare the EU Declaration of Conformity and affix CE marking, and declare the support period with its end date visible at the point of purchase.

**Phase 4, ongoing: make compliance sustainable.**

- Treat the September 2026 reporting obligation and the December 2027 requirements as an operating model, not a project with an end date.
- Decide your upstream contribution strategy, and start moving off unsustainable private forks.
- Prepare for market surveillance readiness: documentation that can be produced on request, and evidence that the product is maintained over its declared support period.

None of these steps requires a large compliance department. Most of them are good engineering that a DevSecOps team can implement directly. What they require is leadership commitment and a sequence.

---

## A Final Word

The 2026 report shows that most of the industry is waiting. The organizations that move now will be the exception, and in a CE-marking regime, being the exception is a market advantage. Every company that cannot demonstrate conformity becomes a procurement risk for its customers, and customers will start asking the questions the CRA makes standard: what product category, what conformity pathway, what support period, what coordinated disclosure process.

My experience as a DevSecOps leader is that regulation like this is best met with an engineering mindset. You break the legal text into obligations, the obligations into artefacts, and the artefacts into a delivery plan. You test the plan, not the theory. You simulate the incident before the regulator asks. That is the approach my team applies to the CRA, and it is the same approach we apply to every security requirement that has a deadline.

If your organization needs help turning this regulation into a working programme, a conversation with a DevSecOps team that has done this before is a reasonable place to start. The clock is running, but the path is clear.

---

## References

- [Regulation (EU) 2024/2847, the Cyber Resilience Act (full text)](https://eur-lex.europa.eu/eli/reg/2024/2847/oj)
- [2026 CRA Awareness and Readiness Report, OpenSSF and The Linux Foundation](https://openssf.org/resources/publications/2026-cra-awareness-and-readiness-report/)
- [Built to Last: Understanding the European Cyber Resilience Act, OpenSSF](https://openssf.org/resources/built-to-last-understanding-the-european-cyber-resilience-act-cra/)
- [OpenSSF EU Cyber Resilience Act public policy page](https://openssf.org/public-policy/eu-cyber-resilience-act/)
- [European Commission CRA policy pages](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act)
- [ENISA Single Reporting Platform (SRP)](https://www.enisa.europa.eu/topics/product-security-and-certification/single-reporting-platform-srp)
- [EC Implementing Regulation (EU) 2025/2392 on product categories](https://eur-lex.europa.eu/)

*This article is for informational purposes and does not constitute legal advice. The CRA's implementing acts, harmonized standards, and Commission guidance are still evolving. Verify the current status of any regulatory reference with qualified legal counsel before acting on it.*