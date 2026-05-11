---
title: "Web Security Audit: My Method for Covering the OWASP Top 10"
description: "A practical overview of my web application security audit methodology: the 10 OWASP vulnerabilities, the tools I use, and how to prioritise remediation."
pubDate: 2026-03-20
tags: ["Security", "OWASP", "Audit"]
lang: en
slug: audit-securite-owasp-top10
---

## Introduction: Why Run a Security Audit

A website in production is never static. Dependencies evolve, configurations drift, new features introduce new attack vectors. Yet security is often treated as a checkbox at launch, then forgotten.

Regular application security audits detect vulnerabilities before they are exploited. This is not reserved for large enterprises: a poorly maintained WordPress site, an API without proper authentication, a contact form vulnerable to injection — easy targets are everywhere.

Here is the methodology I have developed through my audits, structured around the OWASP Top 10.

## The OWASP Top 10

The OWASP (Open Web Application Security Project) regularly publishes the Top 10, a list of the most critical web vulnerabilities. The 2021 version includes:

1. **A01 — Broken Access Control**: unauthorised access to protected resources
2. **A02 — Cryptographic Failures**: sensitive data poorly encrypted or transmitted in plaintext
3. **A03 — Injection**: SQL, NoSQL, LDAP, OS commands — anything that concatenates unfiltered data
4. **A04 — Insecure Design**: lack of threat modelling from the outset
5. **A05 — Security Misconfiguration**: default configurations, missing headers, verbose error messages
6. **A06 — Vulnerable and Outdated Components**: dependencies with known, unpatched CVEs
7. **A07 — Identification and Authentication Failures**: weak passwords, poorly managed sessions
8. **A08 — Software and Data Integrity Failures**: unsecured CI/CD pipelines, unsafe deserialisation
9. **A09 — Security Logging and Monitoring Failures**: absence of actionable logs for incident detection
10. **A10 — Server-Side Request Forgery (SSRF)**: the application makes requests to internal resources at an attacker's request

## My Methodology: Passive First, Active Second

I systematically divide my audits into two phases.

### Passive Phase (Reconnaissance)

Without sending any abnormal requests, I collect as much information as possible:

- Analysis of **HTTP headers** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **TLS configuration** verification (version, ciphers, certificate)
- Reading `robots.txt` and HTML metadata
- Searching for exposed files (`.git`, `.env`, `backup.zip`, `phpinfo.php`)
- Consulting CVE databases for identified technologies

### Active Phase (Testing)

Once reconnaissance is complete, I move to active testing — always with the client's written consent:

- Fuzzing GET/POST parameters to detect injections
- Authentication and session management tests
- Access control checks (IDOR, privilege escalation)
- CORS and CSRF tests

## The Tools in My Arsenal

### HTTP Headers and TLS

[SecurityHeaders.com](https://securityheaders.com) gives an immediate score on HTTP header quality. [SSL Labs](https://www.ssllabs.com/ssltest/) performs an in-depth analysis of TLS configuration and detects obsolete ciphers or legacy protocols (TLS 1.0/1.1).

### OWASP ZAP

OWASP ZAP (Zed Attack Proxy) is the Swiss army knife of web auditing. It intercepts traffic, performs automated scans, and enables advanced manual testing. I use it primarily in "Active Scan" mode on test environments:

```bash
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://target.example.com \
  -r zap_report.html
```

### Nikto

Nikto is a fast web vulnerability scanner, ideal for detecting common configuration issues: exposed sensitive files, revealed server versions, dangerous HTTP options.

```bash
nikto -h https://target.example.com -output nikto_report.txt
```

### WPScan for WordPress

For WordPress sites, WPScan is essential. It detects vulnerable plugins, outdated themes, and enumerable users:

```bash
wpscan --url https://target.example.com --enumerate vp,vt,u
```

## Vulnerability Prioritisation

Not all vulnerabilities are equal. I use the **CVSS** (Common Vulnerability Scoring System) to prioritise:

- **Critical (9–10)**: fix immediately (SQL injection, RCE, authentication bypass)
- **High (7–8.9)**: fix within the week (stored XSS, IDOR, SSRF)
- **Medium (4–6.9)**: plan for the next sprint (missing headers, information disclosure)
- **Low (0–3.9)**: note and fix during the next maintenance window

I also factor in **exploitability**: a theoretical vulnerability that is difficult to exploit in real conditions is less urgent than a trivial flaw on a critical endpoint.

## Report and Follow-up

The audit report is structured in three parts:

1. **Executive summary**: overall risk level, strengths, critical points — readable by a non-technical stakeholder
2. **Vulnerability details**: description, proof of concept (screenshots), CVSS score, precise recommendations
3. **Remediation plan**: prioritised list of actions, ranked by effort and impact

I systematically offer a **re-test** two to four weeks after delivering the report, to verify that corrections have been properly applied and have not introduced new vulnerabilities.

## Conclusion

An OWASP Top 10 audit is not a guarantee of invulnerability, but it is a solid foundation for identifying the most common and most exploited risks. Security is a continuous process, not a final state.

If you would like to have your web application or WordPress site audited, feel free to get in touch. I work in Charente-Maritime and remotely across France.
