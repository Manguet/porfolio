---
title: "WPScan: Auditing WordPress Security from the Command Line"
description: "A practical guide to WPScan, the reference open source tool for detecting WordPress vulnerabilities: installation, essential commands, and interpreting results."
pubDate: 2026-02-10
tags: ["Security", "WordPress", "WPScan"]
lang: en
slug: wpscan-audit-wordpress
---

## Introduction: WordPress and the Attack Surface

WordPress powers roughly 43% of all websites worldwide. This popularity makes it a prime target for attackers: abandoned plugins, outdated themes, and weak admin passwords are exploited en masse by automated bots.

Before manually auditing a WordPress site, I always start with WPScan to get a quick map of the attack surface. It is an open source tool written in Ruby, maintained by the security community, with a regularly updated vulnerability database.

## What WPScan Does

WPScan is a WordPress-specific security scanner. It can:

- Detect the WordPress version and exposed files
- List installed plugins and themes (active or not)
- Identify known vulnerabilities (CVEs) for each component
- Enumerate users
- Test passwords via brute force

It uses the [WPScan Vulnerability Database](https://wpscan.com/), a community and professional database listing thousands of WordPress vulnerabilities. The free API allows 25 requests per day — enough for occasional audits.

## Installation

### Via RubyGems

WPScan requires Ruby 2.7+. On Debian/Ubuntu:

```bash
sudo apt install ruby ruby-dev libcurl4-openssl-dev make zlib1g-dev
sudo gem install wpscan
wpscan --version
```

### Via Docker (recommended)

The Docker method avoids dependency issues and guarantees using the latest version:

```bash
docker pull wpscanteam/wpscan
docker run -it --rm wpscanteam/wpscan --url https://target.example.com
```

## Essential Commands

### Basic Scan

```bash
wpscan --url https://target.example.com
```

Detects the WordPress version, common exposed files (`readme.html`, `xmlrpc.php`, `wp-cron.php`) and security headers.

### Vulnerable Plugin Enumeration

```bash
wpscan --url https://target.example.com \
  --enumerate vp \
  --api-token YOUR_API_TOKEN
```

`vp` = *vulnerable plugins*. With an API token, WPScan cross-references each detected plugin against the vulnerability database and displays the corresponding CVEs with their CVSS scores.

### Theme Enumeration

```bash
wpscan --url https://target.example.com \
  --enumerate vt \
  --api-token YOUR_API_TOKEN
```

`vt` = *vulnerable themes*. Premium themes abandoned by their authors are a frequent source of unpatched vulnerabilities.

### User Enumeration

```bash
wpscan --url https://target.example.com --enumerate u
```

WPScan exploits several vectors to find usernames: the WordPress REST API, author pages, RSS feeds. An `admin` username still active is a red flag.

### Password Testing (Brute Force)

```bash
wpscan --url https://target.example.com \
  --usernames admin \
  --passwords /usr/share/wordlists/rockyou.txt \
  --max-threads 10
```

**Warning**: this command must only be used with the explicit authorisation of the site owner. It generates significant traffic and may trigger protections (IP ban, account lockout).

## Interpreting Results and Prioritising

WPScan produces a structured report. The main points to watch:

**WordPress version not masked**: WPScan displays the detected version. If it is below the current stable version, updating is a priority.

**Plugins with CVEs**: each vulnerability shows the CVSS score, the type (XSS, SQLi, CSRF...) and a link to the details. I prioritise scores above 7 and vulnerabilities exploitable without authentication.

**`xmlrpc.php` enabled**: this file is a vector for brute force and DDoS attacks. Disable it if it is not in use.

**`readme.html` accessible**: reveals the WordPress version. Delete it or restrict access.

**Enumerable users**: rename `admin`, enable two-factor authentication, and install a login attempt limiting plugin (e.g. Limit Login Attempts Reloaded).

## Limitations: Passive vs Active Scanning

WPScan operates primarily in **passive detection** mode: it analyses HTTP responses, public metadata, and accessible files. It does not actively test the exploitation of vulnerabilities.

To go further:

- **OWASP ZAP** or **Burp Suite** for active injection tests
- **Metasploit** to validate the exploitability of specific vulnerabilities (in a controlled environment)
- Manual code review of custom plugins

WPScan is therefore an excellent **first filter**, but does not replace a full audit.

## Conclusion

WPScan is an essential tool in the toolkit of any developer or auditor working with WordPress. In just a few minutes, it gives a clear picture of a site's security posture and identifies the most urgent vulnerabilities to address.

Incorporating it into a monthly maintenance routine — alongside WordPress, plugin, and theme updates — is a good practice accessible to everyone, even without deep security expertise.
