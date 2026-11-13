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

### Attack surface and exposure

Exposure is how reachable an asset is to an attacker. Criticality describes what is lost if an asset is compromised. Exposure describes who is in a position to attempt that compromise, and by what paths. The attack surface is the set of all the ways in: network services, accounts, and connections to other organizations.

Internet-facing systems are the most exposed. Remote access gateways such as virtual private network (VPN) concentrators, web applications, email, and file transfer services can be reached by any attacker without first compromising something else. Their logs also show a steady stream of scanning and failed attempts from sources that have nothing to do with a targeted attack. A requirement for an internet-facing asset is therefore usually more useful when it targets evidence that an attempt succeeded, such as the service starting an unexpected process or a new account appearing on the system, than when it targets individual attempts. Attempts are still worth a requirement when they show knowledge of the organization, such as sign-in failures against many valid account names from one source.

Internal does not mean unexposed. Exposure depends on where the attacker is, and after one workstation is compromised, the attacker is inside. On a network with little internal segmentation, a database server that every workstation can reach is exposed to whoever controls any workstation. Internal exposure is described by what can reach the asset: which network segments, which accounts, and which management tools.

Exposure is not always direct. An asset with no internet connectivity can still be reached through a system that has it. If an internet-facing application server holds a service account with access to an internal database, an attacker who compromises the application server has a path to the database. When assessing exposure, follow the systems, accounts, and trust relationships that lead to the asset, not only whether the asset itself can be reached from the internet.

Identity is part of the attack surface. An asset behind a strong network boundary is still exposed if an account that can administer it is weakly protected. The accounts that add the most exposure are those without multi-factor authentication (MFA), service accounts with long-lived credentials that are shared across systems, privileged accounts that sign in to many systems, and accounts that are no longer used but remain enabled. Identity exposure lets a requirement be scoped to specific accounts rather than specific hosts, for example, a service account signing in interactively or from a host it has never used.

Third-party access extends the attack surface to organizations whose security the detection team cannot observe or control. Vendors with remote support access, managed service providers, and partners with application programming interface (API) credentials can all reach internal systems, and a compromise on their side becomes access to the organization. When vendor access is granted for a defined purpose, with specific accounts, source addresses, systems, and maintenance windows, the expected pattern is known in advance. That makes it a good subject for a requirement, because activity outside the pattern can be defined precisely. Where vendor access is broad or undocumented, record that as a gap, because no requirement can separate expected vendor activity from misuse without a defined pattern to compare against.

Exposure shapes a requirement's scope and priority as criticality does, and it also changes what the requirement looks for, because the likely path to an asset determines which behaviors are worth detecting on it. The two are assessed together: an exposed path to a critical asset is the case that most needs a requirement. Combining them into a final priority is covered in Lesson 6.

Exposure also changes more often than criticality. A firewall rule change, a new cloud resource, or a newly published service can expose an asset that was internal the day before. Exposure can also close when a service is disabled, or vendor access is revoked. Later sections cover where current exposure data comes from and when an exposure-driven requirement should be retired.

### Vulnerability state as a detection input

Vulnerability state is what is known about the weaknesses on an asset: which known vulnerabilities are present, whether they are being exploited elsewhere, and whether a fix has been applied. For detection, it helps answer a question the threat model cannot: of everything an attacker might try, which known weaknesses are present here? A vulnerability that is present, exposed, and not yet remediated marks a place where a requirement may be the only control until the fix arrives.

Three public sources describe vulnerability state from different angles. Each answers one question well, and as published, none of them knows anything about the environment.

#### CISA KEV: confirmed exploitation

The KEV catalog lists vulnerabilities that CISA has determined were exploited in the wild. CISA adds an entry when a vulnerability has a Common Vulnerabilities and Exposures (CVE) identifier, there is reliable evidence of active exploitation, and there is a clear remediation action. Each entry also records whether the vulnerability is known to be used in ransomware campaigns.

For a detection requirement, a KEV entry is the strongest evidence of exploitation among the three sources, because it replaces a prediction with a fact: attackers have already used this vulnerability. That is evidence about the vulnerability, not about the risk to a particular asset. If a KEV vulnerability is present on an asset that cannot be remediated quickly, that asset is a strong candidate for a requirement.

KEV has three limits. It is not complete, and it trails the attackers: an entry appears only after exploitation is confirmed, so absence from the catalog does not mean exploitation is not happening. It is a yes-or-no record, so it says nothing about how widely a vulnerability is exploited or against whom. And it covers only vulnerabilities with a CVE identifier and a remediation action, which leaves out misconfigurations and most flaws in internally developed software.

#### EPSS: likelihood of exploitation

EPSS is a model maintained by the Forum of Incident Response and Security Teams (FIRST) that estimates, for each published CVE, the probability that exploitation activity will be observed in the next 30 days. Scores range from 0 to 1 and are recalculated daily. Each score is published with a percentile that ranks it against all other scored CVEs. The two numbers are easy to confuse: because most CVEs score near zero, a high percentile can correspond to a low probability.

EPSS is useful for the large set of vulnerabilities that are present in the environment but not in KEV. A vulnerability on an exposed asset whose EPSS score is high, or rising, is a candidate for a requirement before any exploitation is confirmed.

EPSS has limits that matter for detection. The probability describes exploitation activity observed by the organizations that contribute data to the model, anywhere on the internet. It does not describe activity against a particular organization, and the model has no knowledge of an asset's exposure or criticality. The activity it predicts is attempted exploitation, not successful compromise. A low score is an estimate, not an assurance. And because scores change daily, a decision made on last month's score may no longer hold. FIRST states that EPSS is not a risk score.

#### CVSS: severity, not risk

CVSS, also maintained by FIRST, describes the technical characteristics of a vulnerability and produces a severity score from 0 to 10. Version 4.0 names scores by the metric groups used: CVSS-B is a Base score alone, and CVSS-BTE adds Threat and Environmental metrics, which account for exploit maturity and for the organization's own environment. The scores published by vendors and vulnerability databases are usually Base scores only. The same caution applies to version 3.1 scores, which remain common in scanner output.

A Base score describes how bad exploitation could be in general, with no reference to whether anyone is exploiting the vulnerability or where the vulnerable system sits. The CVSS documentation is explicit that a Base score measures severity and should not be used alone to assess risk. A requirement driven by Base score alone can lead a team to spend effort on critical-rated vulnerabilities on isolated systems while overlooking medium-rated ones on exposed systems that attackers are using.

The part of CVSS that helps a detection engineer most is the vector string, not the number. The vector records how the vulnerability is reached and what exploitation gives the attacker, and that indicates what behavior a requirement should target. A vulnerability with a network attack vector and no privileges required can be attempted by anyone who can reach the service, so the requirement concerns what the service does after exploitation. A vulnerability with a local attack vector cannot be reached over the network. It is exploited by someone who can already run code on the system, or by a user who is persuaded to open a malicious file. In the first case, the requirement concerns a change in privilege by an account already present on the host. In the second, it concerns what the application does after it opens the file.

#### Using the three together

| Source | Question it answers | What it does not show |
| --- | --- | --- |
| CISA KEV | Has this vulnerability been exploited? | How widely, or whether unconfirmed exploitation is occurring |
| EPSS | How likely is exploitation activity in the next 30 days? | Whether attempts will succeed, or whether they will reach this organization |
| CVSS Base Score | How severe is exploitation, and how is it carried out? | Whether anyone is exploiting the vulnerability|

None of the three shows whether the vulnerability is present in the environment, whether the affected asset is exposed, or how much that asset matters. The organization's own data provides those answers. A vulnerability most clearly warrants a requirement when it is present on an asset that is exposed or critical, and it cannot be remediated before an attacker is likely to use it.

Vulnerability state also has an end date. Once a vulnerability is remediated, a requirement written because of it may no longer be needed, which the requirement expiry section covers. How to weigh these inputs against threat relevance, feasibility, and effort is covered in Lesson 6.

## Examples

TBD

## Operational Considerations

TBD

## References

TBD
