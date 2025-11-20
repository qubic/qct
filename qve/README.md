# QVE — Qubic Vulnerabilities and Exposures

**QVE (Qubic Vulnerabilities and Exposures)** is the central registry for documenting and tracking security vulnerabilities, weaknesses, and exposures found within the Qubic ecosystem.  

This folder is part of the **QCT (Qubic Core Technology)** repository and serves as a transparent, structured place to share and track security-related findings.

---

## Purpose

The goal of QVE is to:
- Provide a **centralized registry** of reported vulnerabilities and exposures.
- Facilitate **transparency** and **collaboration** between developers, researchers, and the community.
- Help ensure that known issues are properly **tracked, prioritized, and mitigated**.

---

## Bounties for White Hats

We value the efforts of security researchers and white-hat hackers who help keep Qubic secure.
While there is no formal bug bounty program (yet), the QCT team is exploring ways to reward meaningful vulnerability reports that protect the community.

Potential rewards may include:

- Monetary bounties for high-severity vulnerabilities.
- Recognition in the QVE registry and community channels.
- Early access to security patches or Qubic roadmap updates.

Note: Bounties, if granted, are evaluated on a case-by-case basis depending on severity, reproducibility, and potential impact.

---

## Contribution Guidelines

We welcome responsible disclosure and collaboration.
If you discover a vulnerability:

1. Privately report it to the QCT team first (via [Discord](https://discord.gg/qubic), contact `@dkat.` or `@j0et0m`).
2. Work with the QCT team to validate and mitigate the issue.
3. Once it is safe to disclose, create a QVE entry following the template below.

---

## Structure

### QVE Entries

Each vulnerability or exposure is documented as a separate file inside this folder.
Suggested format: `QVE-YYYY-NNN.md` or `QVE-YYYY-NNN.pdf`

Where:
- `YYYY` = Year of discovery
- `NNN` = Incremental number (e.g., 001, 002, ...)

### Audit Reports

Comprehensive security assessments and penetration test reports from external security firms are stored in the **[audits/](audits/)** subfolder.
See the [audits README](audits/README.md) for more information about available audit reports

---

## Vulnerability Report Template

When adding a new QVE entry, please follow this template:

```markdown
# QVE-YYYY-NNN: [Short Title]

**Date Reported:** YYYY-MM-DD  
**Status:** Open | Mitigated | Resolved  
**Severity:** Low | Medium | High | Critical  

## Description
Brief description of the vulnerability or exposure.

## Impact
Explain the potential consequences if exploited.

## Affected Components
List the components, repositories, or modules that are affected.

## Steps to Reproduce
Provide reproducible steps if possible.

## Mitigation / Fix
Describe current mitigation steps or link to the fix/patch.

## References
- Links to commits, issues, discussions, or external references.
```


