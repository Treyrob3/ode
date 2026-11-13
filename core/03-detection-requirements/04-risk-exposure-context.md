# Risk & Exposure Context

## Overview

Lesson 3 showed how a detection threat model turns threats into requirements. That answers what an adversary might do. This lesson answers a different question: where in this environment would that activity matter most, and where is detection the only control between the adversary and the outcome?

The same technique does not carry the same risk everywhere. Credential dumping on a lobby kiosk is a different problem than credential dumping on a domain controller. An exploit for a vulnerability patched last month is a different problem than an exploit for a system that cannot be patched until next year. Asset criticality, exposure, and vulnerability state are the context that turns a generic requirement into one scoped to the environment it protects.

This lesson covers where that context comes from, how to use vulnerability intelligence such as the Cybersecurity and Infrastructure Security Agency (CISA) Known Exploited Vulnerabilities (KEV) catalog, the Exploit Prediction Scoring System (EPSS), and the Common Vulnerability Scoring System (CVSS) without being misled by them, and when detection should act as a compensating control for risk the organization cannot remediate in the near term. Combining these inputs into a final priority is covered in Lesson 6.

## Core Content

### Asset Criticality

Asset criticality is how much harm the organization suffers if an asset is compromised. It depends on what the asset does for the business, what data it holds, and what access it grants to other systems. Risk assessment frameworks such as National Institute of Standards and Technology (NIST) Special Publication (SP) 800-30 Rev. 1 treat impact as one of the core factors of risk alongside likelihood. Asset criticality helps a detection engineer estimate the potential impact of compromising a specific system.

Crown-jewel systems are the ones whose compromise is the outcome the organization most fears: payment processing, customer data stores, or the systems that run a core business process. The systems that grant access to the crown jewels matter almost as much. Domain controllers, identity providers, backup infrastructure, and software build pipelines are often more valuable to an attacker than the data itself, because control of them gives access to many other systems.

The same technique warrants different priority on different assets. A new service created on an ordinary workstation may be routine software deployment. The same event on a backup server or a domain controller is more unusual, because those systems usually change less often, and it is far more damaging if missed. A requirement that treats both the same either hides the important alerts among routine ones or under-protects the systems that matter most.

Criticality shapes a requirement in three ways. It narrows **scope**, because a requirement can target a defined set of high-criticality assets rather than the whole environment. It raises **priority**, which Lesson 6 covers in depth. And it changes the **tolerance for alert volume**, because a missed compromise costs more on a critical asset, so each alert is worth more investigation time. On a domain controller, a detection that fires a few extra times a week may be acceptable, while the same rate on each of thousands of workstations would overwhelm triage. Higher criticality does not excuse a noisy detection, because alerts that analysts learn to ignore are not investigated.

Criticality data is often wrong. Asset tiers in a configuration management database (CMDB) are frequently set once and never revisited, and asset owners may not know what depends on their systems. Treat recorded criticality as a starting point, and confirm it with the asset owner before building a requirement around it.

Sometimes the problem is not a wrong rating but a missing asset. An owner leaves the organization, and no one else knows the system exists. A system is assumed to be decommissioned but is still running. A server built quickly during an outage or incident never goes through normal provisioning and never reaches the inventory. These assets are often unpatched and unmonitored, which makes them a likely place for an attacker to gain a foothold without being noticed. A detection requirement scoped to "all critical assets" silently excludes them, so treat inventory gaps as a known limit of any asset-scoped requirement.

## Examples

TBD

## Operational Considerations

TBD

## References

TBD
