# Research Report: Common Network Security Threats

**Task:** Task 4 – Research Report: Common Network Security Threats  
**File:** `network_security_threats_report.md`  
**Focus:** How attacks work, real-world impact, and practical mitigation strategies

---

## 1. Introduction

Network security threats matter because modern organizations depend on interconnected networks for applications, cloud services, communication, authentication, payments, and critical infrastructure. An attack against a network can affect the **confidentiality, integrity, or availability (CIA)** of systems and data. Threats such as DoS/DDoS, Man-in-the-Middle attacks, IP spoofing, and DNS poisoning exploit different weaknesses in network communication, but they can be combined to cause much larger incidents. For example, spoofed source addresses can support reflection-based DDoS attacks, while DNS or ARP manipulation can redirect users into attacker-controlled systems. Effective network security therefore requires layered controls including encryption, authentication, network filtering, secure DNS, segmentation, monitoring, patching, and incident-response planning.

---

## 2. DoS/DDoS Attacks

### 2.1 What is DoS/DDoS?

**Definition:** A Denial-of-Service (DoS) attack attempts to prevent authorized users from accessing a system or service, while a Distributed Denial-of-Service (DDoS) attack performs the attack using numerous hosts.

NIST defines DoS as the prevention of authorized access to resources or the delaying of system operations. NIST defines DDoS as a denial-of-service technique that uses numerous hosts.

### 2.2 How the attack works

A DoS attack may originate from one system, while a DDoS attack normally uses many compromised or controlled systems, commonly called a **botnet**.

A simplified DDoS flow is:

```text
                    ┌───────────────┐
                    │   Attacker    │
                    └───────┬───────┘
                            │
                     Controls botnet
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
     ┌─────────┐       ┌─────────┐       ┌─────────┐
     │  Bot 1  │       │  Bot 2  │  ...  │  Bot N  │
     └────┬────┘       └────┬────┘       └────┬────┘
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                    ┌───────────────┐
                    │ Target Server │
                    └───────────────┘
                            │
                       Service outage
```

Attackers can generate large volumes of TCP, UDP, ICMP, or HTTP traffic. Another important method is **reflection/amplification**, where the attacker sends requests with a spoofed victim IP address to third-party services that generate larger responses toward the victim.

MITRE ATT&CK identifies Network Denial of Service as **T1498**, with Direct Network Flood (**T1498.001**) and Reflection Amplification (**T1498.002**) as sub-techniques.

### 2.3 Real-world example: Dyn DNS attack (2016)

On October 21, 2016, the DNS infrastructure company **Dyn** was hit by a major DDoS attack. The attack generated traffic from a very large number of IP addresses and disrupted DNS resolution. Because Dyn provided DNS services for many well-known websites, users experienced difficulty reaching services including Twitter, Spotify, Reddit, and other major websites.

The incident demonstrated an important network-security principle: attacking a critical shared infrastructure provider can create a much larger downstream impact than attacking one website directly. Reporting linked much of the traffic to the **Mirai IoT botnet**, which had compromised poorly secured Internet-connected devices.

### 2.4 Impact

DDoS attacks primarily affect **availability**, but the business consequences can include:

- Website and API downtime
- Loss of revenue and customer trust
- Service-level agreement violations
- Increased infrastructure and mitigation costs
- Disruption of DNS and other shared services
- Operational distraction while security teams respond
- Potential use of the attack as a diversion for another intrusion

### 2.5 Mitigation strategies

#### 1. Use upstream DDoS protection and CDN-based filtering

For attacks larger than the organization's Internet connection, filtering must happen upstream. Organizations can use an ISP, CDN, or specialized DDoS mitigation provider to absorb and filter malicious traffic before it reaches the origin network.

#### 2. Apply network filtering and rate controls

Firewalls, routers, WAFs, and load balancers can restrict unnecessary ports/protocols and apply rate limits. Traffic baselines can also help identify abnormal packet rates and request patterns.

#### 3. Build redundancy and an incident-response plan

Critical services should use redundant infrastructure, multiple network paths where appropriate, autoscaling, resilient DNS, and tested business-continuity procedures. Organizations should also maintain an escalation path with their ISP/DDoS provider so mitigation can begin quickly.

---

## 3. Man-in-the-Middle (MITM) Attacks

### 3.1 What is a MITM attack?

**Definition:** A Man-in-the-Middle (MITM), also called **Adversary-in-the-Middle (AiTM)**, occurs when an attacker positions themselves between communicating parties so that traffic can be observed, redirected, or modified while the victims believe they are communicating directly.

MITRE ATT&CK tracks this behavior under **T1557 – Adversary-in-the-Middle**. Common methods include ARP cache poisoning, DNS/name-resolution poisoning, DHCP spoofing, and rogue/evil-twin Wi-Fi networks.

### 3.2 How the attack works

A typical MITM attack can follow these stages:

```text
Normal communication:

Client  ───────────────────────►  Server


MITM communication:

Client  ───────►  Attacker  ───────►  Server
                    │
                    ├── Read traffic
                    ├── Modify traffic
                    └── Forward traffic
```

On a local network, an attacker may use ARP poisoning to make the victim associate the attacker's MAC address with another device's IP address. Another approach is an **evil-twin Wi-Fi access point**, where the attacker creates a wireless network designed to look like a legitimate hotspot.

MITRE notes that adversaries can use their position to sniff traffic, manipulate transmitted data, or capture authentication material.

### 3.3 Real-world example: Lenovo Superfish (2015)

In 2015, security researchers discovered that Lenovo laptops had been shipped with **Superfish** software. The software installed a root certificate and intercepted HTTPS traffic so that it could inspect encrypted web connections and inject advertising.

WIRED reported that the design could allow an attacker to exploit the installed certificate mechanism and perform a MITM attack against affected users. This demonstrated how weakening certificate validation can turn encrypted communications into interceptable traffic.

### 3.4 Impact

MITM attacks can result in:

- Theft of usernames, passwords, and session tokens
- Exposure of confidential information
- Manipulation of data in transit
- Session hijacking
- Redirection to malicious websites
- Malware delivery
- Financial fraud
- Loss of trust in network communications

### 3.5 Mitigation strategies

#### 1. Enforce strong encryption and certificate validation

Use HTTPS/TLS for web applications and secure protocols such as SSH instead of insecure alternatives. Do not ignore browser certificate warnings, and use properly managed certificates.

#### 2. Secure local networks and wireless access

Use modern Wi-Fi security such as WPA2/WPA3, disable unnecessary legacy protocols, protect network-management interfaces, and prevent unauthorized devices from joining sensitive networks.

#### 3. Use MFA and network segmentation

Multi-factor authentication reduces the value of stolen passwords. Network segmentation limits what an attacker can reach even if they successfully obtain a position inside a network.

---

## 4. IP Spoofing

### 4.1 What is IP spoofing?

**Definition:** IP spoofing is the creation of network packets with a forged source IP address so that the packet appears to originate from another system.

The attacker changes the source address in the packet header. The destination may therefore believe that the packet came from a trusted host or from the victim itself.

```text
Actual source:

Attacker ───────────────► Target
10.0.0.50                  10.0.0.20


Spoofed source:

Attacker ───────────────► Target
claims: 10.0.0.99          sees source = 10.0.0.99
```

IP spoofing is especially important in DDoS attacks because it can make attack traffic harder to attribute and can enable reflection/amplification attacks.

### 4.2 How the attack works

An attacker creates packets containing a forged source address. Since basic IP routing primarily uses the **destination address** to determine where to forward traffic, the source address can be manipulated unless network controls validate it.

Spoofing can be used for:

- Reflection/amplification DDoS
- Bypassing weak source-address trust rules
- Hiding the true origin of traffic
- Sending traffic that appears to come from another system

SANS explains that IP spoofing exploits the fact that network routing traditionally does not authenticate the packet's claimed origin.

### 4.3 Real-world example: GitHub Memcached DDoS (2018)

In February 2018, GitHub experienced a record-scale DDoS attack that peaked at approximately **1.35 Tbps**. The attack abused publicly accessible **memcached** servers to amplify traffic.

The attackers used spoofed source IP addresses so that memcached responses were directed toward GitHub. This is a classic example of how **IP spoofing + reflection + amplification** can turn relatively small attacker requests into a very large traffic flood.

WIRED reported that GitHub was able to mitigate the attack by rerouting and filtering the traffic.

### 4.4 Impact

IP spoofing can cause:

- Large-scale DDoS and reflection attacks
- Difficulty tracing the true source of traffic
- Abuse of trusted-source filtering
- False attribution of malicious activity
- Network instability
- Increased incident-response workload

### 4.5 Mitigation strategies

#### 1. Implement ingress and egress filtering (BCP 38/BCP 84)

Network operators should validate that packets leaving or entering a network use source addresses that are legitimate for that network. NIST identifies BCP 38 and related anti-spoofing techniques as important controls.

#### 2. Avoid trusting source IP addresses as the only authentication mechanism

Applications should use cryptographic authentication, certificates, API tokens, or MFA rather than assuming that a packet is trustworthy merely because its source IP appears familiar.

#### 3. Disable exposed reflection/amplification services

Organizations should not expose unnecessary UDP services or caching infrastructure to the public Internet. Services such as memcached should be properly configured, access-controlled, patched, and monitored.

---

## 5. DNS Poisoning / DNS Spoofing

### 5.1 What is DNS poisoning?

**Definition:** DNS poisoning is the insertion of false DNS information into a resolver's cache or DNS infrastructure so that a domain name resolves to an incorrect IP address.

**DNS spoofing** is a broader term that can describe forged or manipulated DNS responses. DNS hijacking can also occur when an attacker gains control of DNS settings or records and redirects users.

Because DNS translates human-readable domain names into IP addresses, manipulating DNS can redirect users without directly compromising the destination server.

### 5.2 How the attack works

A simplified DNS poisoning attack is:

```text
User
  │
  │  "What is example.com?"
  ▼
DNS Resolver
  │
  │  Attacker injects forged response
  ▼
False DNS Record
  │
  ▼
Attacker-controlled IP
  │
  ▼
Fake / malicious website
```

An attacker may attempt to race a legitimate DNS response with a forged response or exploit weaknesses in DNS software. DNS hijacking can instead involve compromising a registrar, DNS account, router, or authoritative DNS configuration.

NIST's current **SP 800-81r3 Secure Domain Name System (DNS) Deployment Guide**, published in 2026, emphasizes protecting the integrity, authenticity, availability, and confidentiality of DNS services and recommends DNSSEC for protecting the integrity and authenticity of authoritative DNS information.

### 5.3 Real-world example: WikiLeaks DNS hijacking (2017)

In 2017, WIRED reported that visitors to **WikiLeaks.org** were redirected to a defacement page after attackers apparently compromised its DNS configuration. The incident was notable because the attackers did not need to compromise WikiLeaks' web servers directly; manipulating DNS redirected visitors before they reached the intended server.

This example illustrates the broader risk of DNS manipulation: the attacker can attack the **name-resolution layer** rather than the application server itself.

> **Important distinction:** DNS hijacking, DNS spoofing, and DNS cache poisoning are related but not identical. Hijacking commonly involves unauthorized control of DNS configuration or records, while cache poisoning specifically attempts to insert false records into a resolver's cache.

### 5.4 Impact

DNS attacks can cause:

- Redirection to phishing websites
- Credential theft
- Malware distribution
- Website defacement
- Interruption of legitimate services
- Email redirection
- Loss of trust in domain infrastructure
- Potential interception of sensitive traffic

### 5.5 Mitigation strategies

#### 1. Deploy DNSSEC

DNSSEC uses digital signatures to provide data-origin authentication and integrity for DNS records. Proper DNSSEC validation helps prevent forged DNS data from being accepted as authentic.

#### 2. Secure DNS administration

Protect registrar and DNS-provider accounts with MFA, strong unique credentials, least privilege, and registrar/domain locks where available. Monitor DNS records for unexpected changes.

#### 3. Harden and monitor DNS resolvers

Keep DNS software patched, use secure randomization and validation mechanisms, restrict recursive DNS access, log DNS activity, and use protective/encrypted DNS where appropriate. Organizations should monitor for unusual changes in DNS answers and suspicious domains.

---

## 6. Comparison Table

| Threat | Attack Vector | Who Is at Risk? | Difficulty to Execute | Ease of Mitigation |
|---|---|---|---|---|
| **DoS/DDoS** | Flooding traffic, botnets, reflection/amplification | Websites, APIs, DNS providers, cloud services, networks | **Medium–High** for large attacks; attack tools can lower the barrier | **Medium** – requires upstream capacity and filtering for major attacks |
| **MITM** | ARP poisoning, rogue Wi-Fi, DNS manipulation, DHCP spoofing, weak encryption | Users on local/public networks, enterprises, applications | **Medium** | **Medium–High** when strong TLS, certificate validation, MFA, and segmentation are used |
| **IP Spoofing** | Forged source IP addresses in packets | Internet services, networks, DDoS targets, organizations relying on IP trust | **Medium** | **Medium** – filtering is effective but requires network-wide cooperation |
| **DNS Poisoning/Spoofing** | Forged DNS responses, compromised DNS accounts, resolver weaknesses | Websites, users, DNS resolvers, enterprises, email systems | **Medium–High** depending on method | **Medium–High** with DNSSEC, secure administration, patching, and monitoring |

### Relative risk summary

- **Highest availability risk:** DoS/DDoS
- **Highest direct confidentiality risk:** MITM
- **Strong enabler for reflection attacks:** IP spoofing
- **Strong redirection/phishing risk:** DNS poisoning/spoofing

These threats can also be chained together. For example, IP spoofing can support a reflection-based DDoS attack, while DNS manipulation can help establish an adversary-in-the-middle position.

---

## 7. Conclusion: Three Key Takeaways for a Network Administrator

### 1. Do not rely on a single security control

Network security should use defense in depth: firewalls, filtering, encryption, MFA, segmentation, secure DNS, endpoint security, monitoring, and tested incident-response procedures should work together.

### 2. Protect the protocols that control communication

DNS, IP, ARP, DHCP, and routing protocols are fundamental to network operation. Weaknesses at these layers can undermine otherwise secure applications. Administrators should use DNSSEC where appropriate, anti-spoofing controls, secure network configurations, modern wireless security, and strong authentication.

### 3. Prepare for attacks before they happen

Monitoring and incident response are as important as prevention. Organizations should establish traffic baselines, detect abnormal network behavior, maintain relationships with ISPs/DDoS providers, keep systems patched, test recovery procedures, and regularly review network architecture.

---

## 8. References

1. **NIST – Distributed Denial of Service (DDoS) Glossary**  
   https://csrc.nist.gov/glossary/term/distributed_denial_of_service

2. **NIST – DoS Glossary**  
   https://csrc.nist.gov/glossary/term/DoS

3. **NIST SP 800-189 – Resilient Interdomain Traffic Exchange: BGP Security and DDoS Mitigation**  
   https://csrc.nist.gov/pubs/sp/800/189/final

4. **NIST – Advanced DDoS Mitigation Techniques**  
   https://www.nist.gov/programs-projects/advanced-ddos-mitigation-techniques

5. **NIST SP 800-81r3 – Secure Domain Name System (DNS) Deployment Guide (2026)**  
   https://www.nist.gov/publications/secure-domain-name-system-dns-deployment-guide-5

6. **MITRE ATT&CK – Network Denial of Service (T1498)**  
   https://attack.mitre.org/techniques/T1498/

7. **MITRE ATT&CK – Adversary-in-the-Middle (T1557)**  
   https://attack.mitre.org/techniques/T1557/

8. **MITRE ATT&CK – Network Sniffing (T1040)**  
   https://attack.mitre.org/techniques/T1040/

9. **MITRE ATT&CK – Name Resolution Poisoning and SMB Relay (T1557.001)**  
   https://attack.mitre.org/techniques/T1557/001/

10. **SANS Institute – Introduction to IP Spoofing**  
    https://www.sans.org/white-papers/959

11. **SANS Institute – Address Resolution Protocol Spoofing and Man-in-the-Middle Attacks**  
    https://www.sans.org/white-papers/474/

12. **SANS Institute – DNS Spoofing by The Man In The Middle**  
    https://www.sans.org/white-papers/1567

13. **WIRED – What We Know About Friday's Massive East Coast Internet Outage (Dyn DDoS, 2016)**  
    https://www.wired.com/2016/10/internet-outage-ddos-dns-dyn/

14. **Krebs on Security – DDoS on Dyn Impacts Twitter, Spotify, Reddit**  
    https://krebsonsecurity.com/2016/10/ddos-on-dyn-impacts-twitter-spotify-reddit/

15. **WIRED – A 1.3-Tbps DDoS Hit GitHub, the Largest Yet Recorded (2018)**  
    https://www.wired.com/story/github-ddos-memcached/

16. **WIRED – Lenovo's Response to Its Dangerous Adware Is Astonishingly Clueless (Superfish, 2015)**  
    https://www.wired.com/2015/02/lenovo-superfish/

17. **WIRED – Hacker Lexicon: What Is DNS Hijacking? (WikiLeaks example, 2017)**  
    https://www.wired.com/story/what-is-dns-hijacking/

---

## Submission Checklist

- [x] Introduction included
- [x] DoS/DDoS: mechanism, real-world example, impact, and 3 mitigations
- [x] MITM: mechanism, real-world example, impact, and 3 mitigations
- [x] IP Spoofing: mechanism, real-world example, impact, and 3 mitigations
- [x] DNS Poisoning/Spoofing: mechanism, real-world example, impact, and 3 mitigations
- [x] Comparison table included
- [x] Conclusion with 3 key administrator takeaways
- [x] 4+ credible references included
- [x] NIST, MITRE ATT&CK, and SANS sources included
- [x] WIRED/Krebs real-world reporting included
- [x] No demo video required
