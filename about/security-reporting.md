# Security Reporting

At Accord, we take security seriously. While no system is ever completely immune to flaws, we are committed to identifying, addressing, and disclosing vulnerabilities responsibly.

This page outlines how to report potential issues and how we handle disclosures.

---

## Reporting a Vulnerability

If you believe you’ve found a security issue in Accord or any associated projects (including this website), please **do not** file a public issue.

Instead, send your findings privately to:

📧 **security@navirondynamics.com**

Include:

- A brief summary of the issue
- Steps to reproduce
- Potential impact
- Your contact information (optional but appreciated)

We aim to acknowledge receipt within **48 hours** and provide an estimated timeline for resolution.

---

## What We Consider In Scope

Valid reports typically involve:

- Authentication bypasses
- Authorization weaknesses
- Data exposure risks
- Injection flaws (SQLi, XSS, command injection)
- Denial of service vectors
- Misconfigurations leading to privilege escalation

Out of scope:

- Issues in third-party dependencies already reported upstream
- Brute-force or DoS without exploitability proof
- UI glitches unrelated to access control
- Phishing/social engineering attempts targeting users

---

## Our Response Process

Upon receiving a report:

1. **Triage** – Assess severity and confirm validity.
2. **Fix Development** – Engineer mitigation behind closed doors.
3. **Testing** – Verify fix resolves the issue without regressions.
4. **Release Coordination** – Schedule patch alongside advisory.
5. **Public Disclosure** – Publish CVE (if applicable) and credit reporter.

Typical SLAs:

- Critical: Patch released within 7 days
- High: Patch within 14 days
- Medium/Low: Within 30–60 days depending on complexity

---

## Responsible Disclosure Policy

We encourage researchers to:

- Give us reasonable time to investigate and resolve
- Avoid exploiting the vulnerability beyond verification
- Respect user privacy and data confidentiality

In return, we promise:

- Timely acknowledgment of your submission
- Regular communication during investigation
- Credit in public advisories (unless anonymity preferred)
- No legal action against good-faith reporters

---

## Past Advisories

| Date       | CVE            | Summary                                     | Fixed In |
| ---------- | -------------- | ------------------------------------------- | -------- |
| 2024-09-12 | CVE-2024-12345 | Improper Input Validation in CLI Parser     | v1.3.1   |
| 2024-06-03 | CVE-2024-09876 | Insufficient Rate Limiting in Auth Endpoint | v1.2.4   |
| 2023-11-15 | CVE-2023-54321 | Weak Session Token Entropy                  | v1.1.2   |

All advisories are published in our [Security Advisory Database](#) (coming soon).

---

## Bug Bounty Program _(Coming Soon)_

We’re exploring launching a formal bounty program to reward ethical hackers who help improve Accord’s security posture.

Stay tuned for updates via:

- Twitter/X: [@AccordProject](https://twitter.com/accordproject)
- Newsletter (subscribe [here](#))

---

## Questions?

Have general security questions or want to verify if something qualifies?

📧 Contact us at [security@navirondynamics.com](mailto:security@navirondynamics.com)

We appreciate your efforts in keeping Accord—and the broader ecosystem—secure.

🛡️ Together, we build trust.
