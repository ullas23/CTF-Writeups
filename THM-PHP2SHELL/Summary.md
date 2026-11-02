>[!CAUTION]
># PHP2Shell

**Platform:** TryHackMe  
**Topic:** PHP Internals | Memory Corruption | ASLR Bypass | Remote Code Execution  
**Target:** PHP 8.4.26 with Apache prefork and mod_php  
**Status:** Completed

## Overview

PHP2Shell explores vulnerabilities in PHP's session upload-progress mechanism that can lead to memory disclosure and Remote Code Execution (RCE).

The exploitation chain involves a dangling pointer resulting from an out-of-memory (OOM) bailout. By manipulating subsequent HTTP requests and PHP's internal memory structures, an attacker can disclose process memory, bypass Address Space Layout Randomization (ASLR), and ultimately achieve arbitrary command execution as the web server user.

## The PHP2Shell Disclosure

On **21 October 2026**, the PHP Project publicly disclosed two critical vulnerabilities in the PHP runtime that could allow unauthenticated attackers to achieve Remote Code Execution (RCE) through specially crafted HTTP requests. Collectively known as **PHP2Shell**, the vulnerabilities are tracked as **CVE-2026-109618** and **CVE-2026-109619**, with each receiving a CVSS v4.0 score of **9.2 (Critical)**.

What makes PHP2Shell particularly concerning is that the vulnerabilities exist within PHP's underlying runtime, rather than in any specific application, framework, or CMS. An attacker does not need valid credentials or an exploitable application-level vulnerability. Under vulnerable configurations, specially crafted HTTP requests can trigger the flaws before application code is executed.

The earliest affected execution path has been traced back to **PHP 5.4.0 (2012)**, exposing a broad range of PHP releases to these vulnerabilities. The research and coordinated disclosure involved security researchers from **Wiz Research, OpenAI, and Calif**, working with the PHP security team to investigate and remediate the issues.

In response, the PHP Project released emergency security updates across its supported branches:

| PHP Branch | Fixed Release |
|---|---|
| PHP 8.5 | 8.5.12 |
| PHP 8.4 | 8.4.27 |
| PHP 8.3 | 8.3.36 |
| PHP 8.2 | 8.2.35 |

The disclosure was preceded by premature circulation of vulnerability information, and reports of exploitation attempts emerged around the disclosure period, including activity reportedly observed before the official patches were released.

PHP2Shell highlights a significant security concern: vulnerabilities deep within a language runtime can undermine the security of every application built on top of it, regardless of how securely that application was developed.

**This writeup explores the underlying vulnerabilities, their exploitation mechanics, and the practical process of achieving Remote Code Execution in the TryHackMe PHP2Shell lab.**

## Vulnerabilities

PHP2Shell involves two critical vulnerabilities in the PHP runtime:

- **[CVE-2026-109618](https://nvd.nist.gov/vuln/detail/CVE-2026-109618):** **Memory Disclosure** &rarr; Out-of-memory (OOM) corruption leading to memory disclosure and Remote Code Execution.
- **[CVE-2026-109619](https://nvd.nist.gov/vuln/detail/CVE-2026-109619):** **Remote Code Execution** &rarr; Use-after-free caused by stale global state, potentially leading to Remote Code Execution.

Both vulnerabilities are rated CVSS v4.0: 9.2 (Critical).

## Repository Contents

| File | Description |
|---|---|
| `research-document.md` | Detailed technical explanation of the exploit, vulnerabilities and underlying PHP mechanisms. |
| `php2shell-writeup.pdf` | My TryHackMe walkthrough, including exploitation steps and terminal screenshots. |
| `exploit.py` | Python-based exploitation script. |

## Exploitation Summary

The exploitation process involves:

1. Triggering an out-of-memory condition during PHP session initialization.
2. Exploiting a dangling pointer to disclose heap memory.
3. Extracting PHP library addresses to bypass ASLR.
4. Manipulating PHP internal data structures to redirect execution.
5. Achieving RCE through PHP's `system()` function.

## Proof of Exploitation

Successful command execution was verified through the following output:

```text
PROOF: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The room was completed by successfully retrieving the TryHackMe flag.

## Learning Outcomes

- PHP session handling and upload-progress internals.
- Out-of-memory conditions and dangling pointer vulnerabilities.
- Heap memory disclosure and ASLR bypass.
- PHP internal data structures and destructor manipulation.
- Remote Code Execution through runtime-level vulnerabilities.

## References

- [TryHackMe – PHP2Shell](https://tryhackme.com/)
- [CVE-2026-109618](https://nvd.nist.gov/vuln/detail/CVE-2026-109618)
- [CVE-2026-109619](https://nvd.nist.gov/vuln/detail/CVE-2026-109619)
- [PHP Project – Official Security Advisory](https://www.php.net/archive/2026.php)
- [Wiz Research – PHP2Shell Technical Analysis](https://www.wiz.io/blog/php2shell-php-rce)
- [Calif – PHP2Shell Research](https://calif.io/php2shell/)
- [Sophos – PHP2Shell Security Advisory](https://www.sophos.com/en-us/security-advisories/sophos-sa-20261021-php2shell)

*Disclaimer: This documentation is intended for educational purposes and authorized security testing only.*
