# WannaCry: Anatomy of a Ransomware Worm That Took Down the World

**Author:** Abraham Esau Estrada Cerna
**Context:** Originally written for the *IT Security* course at Universidad TecMilenio, expanded here for portfolio purposes.

---

## Why This Case

Almost a decade later, WannaCry is still one of the best case studies for understanding why cybersecurity can't be solved with technology alone. It's a technically unremarkable attack — a patched exploit, a legacy protocol — that turned into a global incident purely because thousands of organizations failed to apply basic controls. That gap between "how easy this was to prevent" and "how big the blast radius ended up being" is what makes it worth breaking down.

## Timeline

WannaCry surfaced on **May 12, 2017** and spread at a pace that was unusual even by ransomware standards. Within hours, thousands of machines were already compromised across dozens of countries. Reuters reported infections in roughly 100 countries that same day; BBC later put the number closer to 150. Post-incident estimates put the total at over 300,000 infected machines worldwide.

The next day, **May 13**, researcher Marcus Hutchins discovered the malware was trying to resolve a domain that hadn't been registered yet. Registering it accidentally triggered an internal **kill switch** that sharply slowed the spread. Months later, both the US and UK governments attributed the attack to the Lazarus Group, linked to North Korea.

## Threat Actors

- **Lazarus Group** — attributed as the party behind the attack.
- **NSA** — originally developed EternalBlue, the exploit at the core of the attack.
- **Shadow Brokers** — leaked EternalBlue publicly, putting it within reach of any threat actor.
- **Notable victims** — UK's NHS, Telefónica, FedEx, Renault, Nissan, Deutsche Bahn, and hundreds of other organizations worldwide.

## How It Worked

Unlike most ransomware at the time, WannaCry didn't rely on phishing to spread. It behaved like a **worm**: it scanned networks for open port 445 and used **EternalBlue** to exploit **CVE-2017-0144**, a vulnerability in SMBv1 that allowed unauthenticated remote code execution. Once inside, it encrypted the victim's files, dropped the ransom note (300 USD in Bitcoin, rising to 600 USD if not paid in time), and kept scanning the network to propagate automatically.

The root cause wasn't the exploit alone — it was the combination of a *known* vulnerability (Microsoft had shipped the **MS17-010** patch two months earlier), SMBv1 left enabled, and flat, unsegmented networks that let the worm move laterally unopposed.

## Exploited Vulnerabilities

| Category | Vulnerability | Why It Mattered |
|---|---|---|
| Logical | Unpatched systems missing MS17-010 | Enabled the *Exploitation* stage — without it, EternalBlue had nothing to exploit |
| Configuration | SMBv1 enabled + lack of network segmentation | Allowed massive lateral movement between machines |
| Organizational / human | Immature risk and asset management (NHS case) | Didn't cause the initial infection, but explains why the blast radius and recovery time were so large |
| **Physical** | **Lack of physical access controls and operational continuity at critical sites** | See below |

## Mapping to the Cyber Kill Chain

| Stage | How It Applied to WannaCry |
|---|---|
| Reconnaissance | Mass identification of Windows machines running SMBv1 without MS17-010 applied |
| Weaponization | EternalBlue packaged with the WannaCry ransomware payload |
| Delivery | Network/IP scanning for open port 445 — no phishing email involved |
| Exploitation | Unauthenticated remote code execution via CVE-2017-0144 |
| Installation | Automatic install and replication to other machines on the network |
| Command & Control | Attempted resolution of a hardcoded domain that, once registered, acted as a kill switch |
| Actions on Objectives | File encryption and deployment of the ransom note |

## Mitigation Strategies

### Logical / Technical
1. **Vulnerability and patch management** — a formal process to prioritize and deploy critical updates. This alone would have closed the door EternalBlue walked through.
2. **Network segmentation** — VLANs, internal firewalls, and ACLs to contain lateral movement.
3. **Deprecating legacy protocols** — SMBv1 should have been retired years before the attack.
4. **Immutable, air-gapped backups**, disconnected from the network and tested regularly for restoration.
5. **EDR and continuous monitoring** — real-time detection of mass file encryption or lateral movement.
6. **Principle of least privilege** — limiting the blast radius of any single compromised account.
7. **Security awareness and incident response readiness** — continuity plans and rehearsed IR procedures.

### Physical
This is the piece the original analysis was missing, and fair callout — for an attack like WannaCry, where the end goal was operational disruption rather than data theft, physical security matters on two fronts:

- **Physical access control to server rooms and critical endpoints.** This is especially relevant in environments like NHS hospitals, where medical devices often run on legacy Windows builds that are hard to patch without interrupting care. Restricting who can plug in a USB device or walk up to a terminal removes an additional attack surface that patching alone doesn't cover.
- **Physical redundancy for critical infrastructure** — geographically separate DR sites, with air-gapped backups stored off-network and off-site, to guarantee operational continuity if the primary site gets encrypted or becomes unreachable.
- **Physical protection of network hardware** (switches, firewalls) to prevent tampering that could undermine segmentation controls or open up new lateral movement paths.

In organizations like the NHS, where legacy medical equipment couldn't always be patched without disrupting patient care, these physical controls would have served as a fallback layer exactly when patching wasn't an immediate option.

## Lessons Learned

The biggest takeaway from this case is that the worst incidents rarely come from sophisticated tradecraft — they come from organizations underestimating known vulnerabilities or deferring basic controls. WannaCry wasn't a single point of failure. It was the sum of an unapplied patch, a protocol that should have been deprecated years earlier, unsegmented networks, immature risk management, and yes, insufficient physical controls in critical environments.

Effective security isn't a product you buy — it's a continuous process that combines people, process, and technology, and no single one of those can fully compensate for gaps in the other two.

## References

- Cybersecurity and Infrastructure Security Agency (CISA). (2023). *#StopRansomware guide*. https://www.cisa.gov/resources-tools/resources/stopransomware-guide
- Cybersecurity and Infrastructure Security Agency (CISA). (2017). *Indicators associated with WannaCry ransomware* (Alert TA17-132A). https://www.cisa.gov/news-events/alerts/2017/05/12/indicators-associated-wannacry-ransomware
- BBC News Mundo. (2017, May 15). *Qué es "WannaCry", el "ransomware" que provocó un ciberataque de alcance global (y cómo protegerte)*. https://www.bbc.com/mundo/noticias-39903218
- Microsoft Security Response Center. (2017, May 13). *Customer guidance for WannaCrypt attacks*. https://www.microsoft.com/en-us/msrc/blog/2017/05/customer-guidance-for-wannacrypt-attacks
- Reuters. (2017, May 13). *Cyberattack hits businesses, hospitals, and schools worldwide*. https://www.reuters.com/article/business/resumen-un-ciberataque-golpea-a-empresas-hospitales-y-escuelas-a-nivel-mundial-idUSL8N1IF07H/
- National Audit Office. (2017, October 27). *Investigation: WannaCry cyber attack and the NHS*. https://www.nao.org.uk/wp-content/uploads/2017/10/Investigation-WannaCry-cyber-attack-and-the-NHS.pdf
