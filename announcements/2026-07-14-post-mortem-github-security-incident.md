# Post-Mortem — 2026-07-13 GitHub Organization Security Incident

**Incident date:** 2026-07-13

**Post-mortem published:** 2026-07-14

**Status:** ✅ Contained · ✅ Audited · ✅ Remediated

**Author:** @j0et0m, on behalf of the QCT team

**Distribution:** Public

---

## Executive summary

On **2026-07-13**, a compromised contributor account was used to inject a malicious GitHub Actions workflow into one of our repositories. The workflow exfiltrated a broad-scope personal access token belonging to a QCT developer. The attacker used that token to escalate their repository access to admin on eight qubic repositories and attempted further pushes.

We detected the incident, contained it, blocked the compromised accounts, audited every repository in the organization for injected code, rotated all credentials in scope, and reset the artefacts that could not be otherwise verified. **The audit is closed. No malicious code remains in any qubic repository.** The GitHub organization is once again safe to use.

**Some users had to take action:** Network Guardians and Web Wallet users who unlocked during a specific window on the day of the incident. Those actions were communicated on the day. This document is the full record of what happened, what we did, what we're changing to prevent it happening again, and what we chose not to commit to and why.

---

## Timeline (UTC)

| Time | Event |
|---|---|
| ~04:17 | A batch of 8 repository-level admin invitations was created via the GitHub API using a stolen developer token. |
| ~05:30 | Incident detected. Initial analysis begins. |
| ~06:30 | First counter-actions after analysis: coordination of available developers, compromised accounts blocked. |
| ~08:00 | Formal incident response task force stood up. |
| ~08:00 | Warnings to developers and direct communication to affected users. |
| ~09:00 | Access to potentially compromised Web Wallet blocked. |
| ~10:00 | Fixed Web Wallet deployed. |
| ~11:48 | Public incident communication published to all users. |
| 03:00 – 11:00 | Window during which Web Wallet users who unlocked their wallets may have been at risk of seed exposure. |
| ~22:00 | Re-check of all actions taken. |
| Same day | Full audit of Core, Integration, and Client repositories completed. All injected artefacts removed. Docker image (Lite) reset. All GitHub secrets rotated. Auto-publish workflows disabled org-wide. Docker push tokens, FTP credentials revoked. GitHub support engaged. |
| 2026-07-14 (this doc) | Post-mortem published. Developers cleared to resume normal use of the repositories. |

---

## Attack chain

The attacker moved through six distinct stages. Each stage is described below; the corresponding preventive control that would have blocked it (or shortened the response) is called out in the *Preventive actions* section.

**1. Initial foothold — token stolen from a compromised contributor server**
A **server** belonging to a qubic contributor was compromised by an external attacker. The contributor had **stored a GitHub access token on that server**. That token was extracted by the attacker. **The contributor's GitHub account itself was not compromised.** The stolen token had write access to two qubic repositories.

**2. Workflow injection**
Using the stolen token's write access, the attacker committed a malicious GitHub Actions workflow file (`.github/workflows/ci-test.yml`) to one of the affected repositories. Because the workflow was allowed to run against organization-level secrets, it had access to sensitive credentials at execution time.

**3. Secret exfiltration**
The workflow ran and exfiltrated a **broad-scope classic personal access token (PAT)** that had been stored as a GitHub organization secret. The token had the `repo` scope, which grants administrative control over any repository the token owner had access to — effectively an **admin-level token at the scale of the organization's repositories**.

**4. Privilege escalation**
Using the stolen admin PAT, the attacker used the GitHub API to create eight repository-level admin invitations for the compromised contributor account. The invitations were accepted, granting repository-level admin rights on eight qubic repositories.

**5. Malicious pushes**
The attacker attempted protected-branch overrides and malicious pushes. Some pushes succeeded before detection.

**6. Downstream artefact impact**
Two release artefacts were pushed with malicious code before containment:
- **Core-Lite Docker image**: confirmed pushed with malicious code. Image reset as part of remediation.
- **Web Wallet**: confirmed pushed with malicious code. Two independent factors materially limited user impact:
  - **Delayed-deploy cache purge**: Web Wallet deployments require a cache purge step before the new build is served to most users. The attacker was unable to trigger a proper cache purge, so **most users never received the compromised Web Wallet build**.
  - **Browser mixed-content blocking**: the attacker's exfiltration payload attempted to send seed data via **HTTP** from a page served over **HTTPS**. Modern browsers block active mixed-content requests of this kind by default. Any user who did receive the compromised build would still have had the seed exfiltration attempt blocked at the browser level.
- **No more affected downstream artefacts**: no more downstream artefacts/applications were affected

---

## Root cause

**A broad-scope classic PAT was stored as a GitHub organization secret.**

That single credential-storage decision is the reason the incident escalated to organization scale. Once that PAT was in an org secret and a workflow could reach it, every other security control in place — branch protection, restrictions on who can invite organization members, workflow review — was operating at a lower privilege than the credential itself. **The stolen token was admin-scoped, which means an attacker holding it could disable or bypass those other controls at will.** No amount of downstream hardening protects against a leaked admin credential that outranks the hardening.

The direct human error was storing an admin-scoped PAT in a GitHub organization secret. Every one of the preventive actions committed below has been designed with this root cause in mind: prevent an admin credential from being available for exfiltration in the first place, and constrain the workflow environment so no future secret at any scope can be exfiltrated the same way.

---

## Impact — final

### Actions users needed to take (during the incident)

| Group | Action | Status |
|---|---|---|
| **Network Guardians** | Rotate seeds; migrate Guardian identity. | Communicated on the day; users were asked to act immediately. |
| **Web Wallet users who unlocked between 03:00 – 11:00 UTC** on 2026-07-13 | Generate a new identity in a fresh environment; move funds to it; stop using the exposed seed. | Communicated on the day. |
| **Developers building on Qubic** | Pause pulling / installing / deploying code from qubic repositories until audit closed. | Communicated on the day; resume-safe status confirmed once the audit closed. |

### What did not happen (confirmed after the audit)

- **No injected code remains in any qubic repository.** Core, Integration, and Client repositories all clean after full audit.
- **No reports of stolen funds** have been received at time of publishing this post-mortem.
- **The Core-Bob Docker image was not affected** by the incident.
- **The Qubic Core protocol and on-chain state** were not compromised.
- **The mobile wallet applications** were not compromised.

### What we do know with high confidence

- **The list of credentials in scope** for the exfiltration workflow is enumerable. Every one was rotated, and to be safe every other credential in the same organizational scope was rotated as well.
- **Web Wallet seed exfiltration is not believed to have succeeded** for any user, for two independent reasons: (a) the delayed-deploy cache purge was not completed by the attacker, so most users never received the compromised build; (b) the exfiltration payload used HTTP from an HTTPS page, which modern browsers block by default even for users who did receive the compromised build.
- **Core-Lite Docker image was pushed with malicious code** — this is confirmed and the image was reset.
- **Web Wallet had malicious code pushed** — confirmed, and reset with a clean build deployed at ~10:00 UTC. The ~4-hour window during which a user who forced a reload could have received the tampered Web Wallet is between the initial malicious push and the clean redeploy.

---

## Response — what worked

- **The incident was detected within hours of the first malicious action** — first detection at ~05:30 UTC, first counter-actions from ~06:30 UTC.
- **The response was coordinated.** An incident-response plan already existed, an emergency chat, and a task tracker on a third-party tool (deliberately not on GitHub, so the response wouldn't depend on the compromised platform). The formal incident response task force was stood up at ~08:00 UTC. The plan was sufficient for this class of incident.
- **User communication was fast and specific.** Warnings to developers and direct communication to affected users went out at ~08:00 UTC. Public communication published at ~11:48 UTC. Guardians and Web Wallet users received actionable instructions with specific time windows the same day, before the audit completed.
- **The compromised Web Wallet was blocked and replaced within hours.** Access to the potentially compromised Web Wallet was blocked at ~09:00 UTC; the fixed Web Wallet was deployed at ~10:00 UTC.
- **We reset rather than assumed.** The Core-Lite Docker image and the Web Wallet build were reset. Even for artefacts where the direct evidence of harm was limited (see mixed-content blocking above), reset was the safer posture.
- **Auto-publish was killed org-wide during the response.** No further tainted artefacts could ship during the audit window.
- **Every credential in scope of the exfiltration was rotated.** GitHub secrets, Docker push tokens, FTP credentials. NPM and NuGet were not affected — those credentials had never been stored in GitHub in the first place, which limited the credential blast radius by design.

## Response — what didn't work

- **A broad-scope classic PAT was stored as an organization-level secret.** This is the root cause of the escalation (see *Root cause* section). Without that PAT, the exfiltration would not have granted the attacker anything actionable at organization scope, and no other security control we had in place could have compensated once the token was in an attacker's hands.
- **A workflow file could be added and run against organization secrets with no additional review gate.** A standard PR review passed the workflow as "just another file change." Workflows that reference `secrets.*` need additional scrutiny.
- **No real-time alert existed on admin-level actions.** Repository-level admin invitations were created and accepted before an alarm sounded. Detection depended on subsequent anomaly-spotting rather than immediate signal. Time-to-detection was hours; the target is minutes.

---

## Preventive actions

Each committed action is mapped to the stage of the attack it directly blocks. Ownership is the QCT team; target windows below are the outer bound — we aim to ship faster where possible.

### 🟢 Ship this week

| # | Action | Blocks | Notes |
|---|---|---|---|
| 1 | **Enforce 2FA for the entire organization.** Previous policy: privileged users only. New policy: every member, no exceptions. | Stage 1 | Codifies the change made during containment. |
| 2 | **CODEOWNERS on `.github/workflows/`** in every repository. Every workflow file change now requires review by a named workflow owner. | Stage 2 | Highest-leverage single change on this list. |
| 3 | **Mandatory `permissions:` block** in every workflow file, least-privilege by default. | Stage 2 & 3 | Template + lint rule. |
| 4 | **Enable GitHub Advanced Security features** on every public qubic repository — secret scanning, push protection, code scanning, Dependabot. | Stages 2 & 3 | Free for public repositories. |

### 🟡 Ship this month

| # | Action | Blocks | Notes |
|---|---|---|---|
| 5 | **`actionlint` + `zizmor` in CI** for every workflow-file PR. Static analysis catches classes of malicious workflow before merge. | Stage 2 | Free OSS tooling. |
| 6 | **Formalize and publish the incident response runbook.** The existing simple guide + emergency chat + third-party task tracker worked for this incident — it will be written up as the standing plan, with rotation for on-call security lead. The same plan applies to non-GitHub incidents. | Response readiness | Institutionalizes what already worked. |
| 7 | **Quarterly access review** — automated audit of who has write access to what, with default-revoke for stale accounts. | Stage 1 | Reduces standing attack surface over time. |

### 🔵 Ship this quarter

| # | Action | Blocks | Notes |
|---|---|---|---|
| 8 | **Environment-scoped secrets with approval gates for every publish path.** Building on the existing per-repository credentials and main-branch protection, publish/deploy secrets will require reviewer approval to unlock. | Stage 6 (artefact tampering) | Public-repo environments include approval gates on Free. |
| 9 | **Auto-publish policy tightened for the Web Wallet.** Wallet applications (mobile, browser extension) already had no-auto-publish policy in place. The Web Wallet was operating on an older policy that permitted automated publish under some conditions — that policy is being brought in line with the other wallet apps: **no automated publish for any wallet application, ever**. Some tooling automation will be built to enforce the difference between wallet and non-wallet repos programmatically. | Stage 6 | Wallet apps carry direct user-fund risk — different bar than other repos. |

---

## Already-in-place controls (validated by this incident)

The following controls were already in place before the incident and functioned correctly. They are called out because they are part of the reason the blast radius was as contained as it was:

- **Fine-grained PATs only.** By policy, no contributor is permitted to add a classic broad-scope PAT as a credential in any qubic system. Only fine-grained PATs are allowed. The direct violation of this policy — a classic admin-scope PAT stored as a GitHub organization secret — is what enabled the escalation. Enforcement will be tightened; the policy itself is not new.
- **Webhook-based alerts on organization events** (member changes, permission changes) are wired to Discord. Alerts contributed to detection during the incident.
- **Only organization owners can invite new organization members.** This is why the attacker escalated via **repository-level admin invitations from a stolen developer PAT** rather than by inviting themselves directly to the organization. The control worked; the attacker had to work around it via a different path.
- **Onboarding / offboarding rules for contributors are documented.** Enforcement will be tightened as part of the quarterly access review (preventive action #7).
- **A written incident response guide, an emergency chat, and a third-party task tracker** were in place before this incident and drove the response. They are being formalized further (see preventive action #6).
- **NPM and NuGet credentials were never stored in GitHub secrets.** This out-of-band credential-storage decision meant the exfiltration workflow could not reach them, limiting the credential blast radius by design.

---

## Under evaluation / not committing at this time

We deliberately chose not to commit to the following. Each choice is explained honestly:

| Considered | Decision | Rationale |
|---|---|---|
| Migrate CI / bot identities to GitHub App tokens | **Not needed at this time** | We do not have cross-repository automation; all workflows are local to their repository. GitHub App tokens are a strong pattern for cross-repo bot identities — we don't have those. |
| OIDC federation for external publishes | **Not adopted** | We are not adopting this pattern in this cycle. Will reconsider if publish-side risk grows. |
| Hardware key / passkey required for high-privilege accounts | **Not adopted** | 2FA org-wide is the committed baseline. Hardware keys remain optional. |
| Audit-log streaming to SIEM | **Not adopted** | Requires GitHub Enterprise. Existing webhook alerts cover the real-time detection use case. Would reconsider if compliance requirements change. |
| Container image signing (Sigstore / Cosign) | **Not adopted at this time** | Downstream verification story is not yet mature enough for the added complexity. |
| Annual independent security audit | **Not yet** | Deferred to a future budget cycle. |

Being explicit about what we chose not to do is part of the honest record. If community feedback surfaces a strong argument for any of these, we will revisit.

---

## Open questions

One question remains partially open at the time this document is published. It does not block the "safe to use" declaration but matters for future prevention:

1. **How was the contributor's server initially compromised?** We know the mechanism: an external attacker compromised a server belonging to the contributor, on which the contributor had stored a GitHub access token. What we do not yet know is the vector by which that server was compromised in the first place. Standing recommendation to every contributor: **do not store GitHub tokens on general-purpose servers.** Use fine-grained, short-lived, environment-scoped credentials where possible, and treat any long-lived token as sensitive as an SSH private key.

The other question we had at the time of the initial internal report — *"was the exfiltration workflow able to reach any secret we did not enumerate?"* — is closed. The list of credentials in scope was fully enumerable, and every one was rotated. Anything in doubt was rotated to be safe.

If new information about the initial-vector question surfaces, this document will be updated.

---

## For users — quick recap

- **If you are a Network Guardian**: you should have rotated your seed on 2026-07-13. If you have not, do it now.
- **If you used the Web Wallet between 03:00 – 11:00 UTC on 2026-07-13** and forced a reload during that window: you were asked to generate a new identity and move funds. Even though we now have high confidence that seed exfiltration did not succeed for any user (browser mixed-content blocking + incomplete cache purge — see *Downstream artefact impact*), if you were in that window and have not moved yet, it is still the safer choice to do so.
- **All other users**: no direct action required. The audit is closed; the repositories and services are safe to use again.
- **Developers building on Qubic**: **repositories are cleared for normal use as of this publication.** If you built or deployed anything based on qubic artefacts pulled between the initial malicious push and full remediation on 2026-07-13, verify against the current released versions before promoting to production.

---

## Thanks — and apology

This incident should not have happened. It happened. Our job is to fix it, be honest about it, and reduce the chance of the same class of incident happening again. The commitments in *Preventive actions* are the concrete version of that.

Thank you to the QCT team — every member picked up part of the response and moved on it the same day. Thank you to the wider Qubic community for taking the action items seriously and for the patience during the audit. Thank you to GitHub support for responsiveness on the day.

We are sorry for the disruption and for any user impact. We are here, we own it, we're changing what needs changing.

— @j0et0m, on behalf of QCT
