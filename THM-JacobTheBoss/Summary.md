# TryHackMe: Jacob the Boss — Writeup
Unauthenticated JBoss RCE → Command Injection in a Custom SUID Binary → Root

| Attribute | Details |
|---|---|
| **Room name** | Jacob the Boss |
| **Difficulty** | Medium |
| **OS** | Linux (CentOS 7) |
| **Primary service** | JBoss Application Server 5.0.0.GA ("Morpheus") |
| **Author / Creator** | TryHackMe |
| **Date completed** | Nov 10, 2026 |

This room drops you in front of a single Linux host running a severely outdated JBoss Application Server 5.0.0.GA (released 2008, end-of-life since 2013) with its management interfaces wide open to the internet. The path to root is short and realistic — it mirrors a huge number of real internet-facing JBoss/WildFly compromises from 2015–2018:

- Recon — Nmap finds JBoss's HTTP Invoker/JMX ports exposed alongside a Dotclear blog on port 80.
- Initial foothold — The jmx-console and web-console management applications require no authentication at all. JexBoss abuses this to deploy a malicious WAR through the JMX MainDeployer service, yielding a shell as the low-privileged service account jacob.
- Privilege escalation — A custom, intentionally-vulnerable SUID binary (/usr/bin/pingsys) builds a shell command from unsanitized user input, allowing classic OS command injection and a root shell.

## Flags
  | Flag |	Location |	Value |
  |---|---|---|
  | **User flag** |	/home/jacob/user.txt	| f4d491f280de360cc49e26ca158***** |
  | **Root flag** |	/root/root.txt	| 29a5641eaa0c01abe5749608c82***** |

## High-level attack path
```text
Internet
   │
   ▼
Nmap discovers JBoss 5.0.0.GA on :8080/:8083/:1099 and an Apache/Dotclear blog on :80
   │
   ▼
jmx-console & web-console reachable with zero authentication (CWE-306)
   │
   ▼
JexBoss deploys a malicious WAR via jboss.system:service=MainDeployer → RCE as "jacob"
   │
   ▼
Local enumeration finds a root-owned SUID binary: /usr/bin/pingsys
   │
   ▼
/usr/bin/pingsys builds "ping -c 4 <input>" and hands it to a shell (CWE-78)
   │
   ▼
Inject ";/bin/bash" → shell inherits the binary's root EUID → root
```
