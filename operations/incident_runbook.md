# QCT Incident Response Runbook

**Owner:** QCT

**Version:** 1.0

**Last updated:** 2026-07-14

**Status:** Standing document

---

## Purpose

This runbook is the standing plan for responding to a security or availability incident affecting Qubic. It exists so that on the worst day, no one has to invent the process. Read it once before you need it; keep it findable when you do.

**Scope.** Any event that materially threatens the integrity, availability, or user trust of Qubic infrastructure. Non-exhaustive list of triggers:

- Unauthorized access to a QCT-controlled GitHub repository, package registry, or CI/CD credential
- Compromise of a contributor account, machine, or long-lived credential with Qubic access
- Malicious or tampered artefact released to users (Docker image, wallet binary, browser wallet, package)
- Live exploitation or credible imminent threat against the Qubic Core protocol or a critical service
- Loss or suspected loss of user funds attributable to a Qubic-controlled surface
- Prolonged outage of a Qubic-controlled service that users depend on (RPC, wallet, Guardian coordination, etc.)

If in doubt, **treat it as an incident**. Over-response is cheap. Under-response is not.

---

## Roles

Roles are functions, not people. One person may hold more than one role at once. The important thing is that every role is **explicitly held by a named individual** during a live incident, and everyone in the response knows who holds what.

| Role | Responsibility |
|---|---|
| **Incident Lead** | Owns the response end-to-end. Makes containment / communication calls. The single point of accountability. Not necessarily the most technical person in the room — the coordinator. |
| **Technical Lead** | Owns the technical investigation, containment actions, and remediation. Coordinates the contributors doing the actual work. |
| **Communications Lead** | Owns all public and user communication (Discord, blog, email, integrator outreach). Nothing is published to users without their sign-off. |
| **Scribe** | Maintains the running task tracker and timeline in the third-party tool. Every action taken and every decision made is captured with a timestamp. |
| **Contributors** | Everyone else pulled in to execute specific tasks. Report status to the Technical Lead. |

By default the QCT Lead holds the role of the incident lead. The role can be delegated and rotate at any time. If the incident lead is not reachable an ad-hoc incident lead can step in.

---

## Communication channels

Do **not** depend on any Qubic-controlled platform for internal incident communication — if that platform is what got compromised, the response cannot happen there.

| Channel | Purpose |
|---|---|
| **Emergency chat** (third-party, off-platform) | Real-time internal coordination. Standing channel; membership is the QCT security on-call group. |
| **Task tracker** (third-party, off-platform) | Running task list, ownership, status, and timeline. Not on GitHub. |
| **Discord `#announcements`** | Public user-facing communication. Communications Lead only. |
| **Direct outreach** | Integrators, exchanges, Guardians, and other affected parties who need earlier or more specific communication than a public post. (Include BD, Community Management, Marketing representative as early as possible) |
| **GitHub Issues / PRs** | For remediation code changes only. Not for incident coordination. |

The specific tools (chat + task tracker) are named in the internal team playbook; they are deliberately not listed here because this document is a template and the specific vendors may change. Every QCT security on-call member is added to both on onboarding.

---

## The response — phases

The response is organized into six phases. They are broadly sequential but overlap in practice; the Incident Lead runs them in parallel where possible.

### Phase 1 — Detect & declare (target: within minutes)

**Objective:** get from "something looks wrong" to "we are formally responding" as fast as possible.

1. Anyone who notices a credible signal (webhook alert, user report, contributor observation) escalates to the on-call Incident Lead via the emergency chat.
2. The Incident Lead makes the declaration call. **Bias to declaring.** If the initial signal turns out to be benign, standing down is cheap.
3. On declaration:
   - Post the declaration in the emergency chat with a one-line summary and severity estimate.
   - Assign the four roles (Incident Lead, Technical Lead, Communications Lead, Scribe). Named individuals, not teams.
   - Open the task tracker for the incident. First entries: the initial signal, the declaration timestamp, and the assigned roles.

### Phase 2 — Contain (target: within 1 hour of declaration)

**Objective:** stop the bleeding. Prevent the impact from growing while investigation continues.

Standard containment actions — apply the ones relevant to the incident:

- **Compromised account / token** → block the account; revoke the token; force session logout.
- **Compromised repository or workflow** → disable the workflow; make the repository private or archive it; rotate any secrets it could reach.
- **Compromised release artefact** → pull the artefact; block distribution; disable auto-publish across the org for the duration.
- **Compromised credential of any kind** → revoke, rotate, and enumerate everything that credential could reach. **Assume worse than you know.**
- **Live protocol / service exploit** → follow the specific service's containment procedure (documented per service).

**Rotate broadly, not narrowly.** If you have to think about whether a credential is in scope, rotate it. The cost of rotating a credential that turned out to be safe is much less than the cost of missing one that wasn't.

### Phase 3 — Communicate to affected users (target: as soon as containment permits user-actionable information)

**Objective:** any user who needs to take action must know **what** to do, **when** to do it by, and **why** — before they read about it from someone else.

Order of user communication:

1. **Users with acute risk of loss** (e.g., unlock-window Web Wallet users, Guardians whose seeds may be exposed). Direct action instructions with a specific time window. Do not wait for full audit.
2. **Users who must pause a normal activity** (e.g., developers pulling from qubic repos). Clear pause instruction; commit to publishing an all-clear.
3. **Everyone else** — a general acknowledgement that an incident is in progress, with a commitment to a full report.

The Communications Lead owns the wording. The Incident Lead signs off. **No one else publishes.**

### Phase 4 — Audit (target: same day for the initial pass; ongoing until closed)

**Objective:** answer "are we clean?" with high confidence.

Standard audit checklist — apply what is relevant:

- **All repositories** in the affected scope (Core, Integration, Client): review recent commits, workflow files, and merge history for injected code.
- **All organization secrets** in the affected scope: enumerate, verify each is legitimate, rotate anything in doubt.
- **All release artefacts** pushed during the compromise window: verify hashes / rebuild from source, or reset outright.
- **All integration credentials** (Docker, NPM, NuGet, FTP, cloud provider, DNS, etc.) that the attacker could have reached: audit for unauthorized use, rotate.
- **Organization audit logs**: enumerate every action performed by suspicious identities during the window.
- **Downstream integrations** (partner exchanges, integrators, hosted services): notify and coordinate their own audits if their surface intersects ours.

Audit findings go into the task tracker with named owners and status. The audit is not closed until every checklist item is either **verified clean** or **remediated**.

### Phase 5 — Remediate (target: parallel with audit; complete before all-clear)

**Objective:** everything found in Phase 2 (containment) and Phase 4 (audit) is fully cleaned up, not just paused.

- Injected code removed from every affected repository, verified by review.
- Compromised artefacts reset or rebuilt from verified-clean source.
- Every credential in scope rotated, documented in the task tracker.
- Containment blocks (disabled workflows, blocked accounts, paused publish paths) reviewed: keep those that should be permanent; carefully re-enable those that are safe to restore.

Nothing that was rotated or reset during containment is restored to its old state. Fresh values only.

### Phase 6 — Close & report (target: within 48 hours for containment/audit close; within a week for public post-mortem)

**Objective:** the incident is officially over, users are informed, and lessons are captured.

1. **Internal close.** Incident Lead formally closes the incident in the task tracker. All tasks status = done or explicitly deferred with an owner.
2. **All-clear communication.** Communications Lead publishes the all-clear: what was resolved, what users no longer need to worry about, what (if anything) users should still do.
3. **Public post-mortem.** Communications Lead + Technical Lead publish the post-mortem within a week. Contents:
   - Timeline (UTC).
   - Attack chain / failure chain.
   - Root cause.
   - Impact (final).
   - What worked / what didn't.
   - Preventive actions committed, with owner and target date.
   - What was deliberately not committed to, with rationale.
   - Open questions.
4. **Internal retrospective.** Within two weeks of close, run a QCT internal retrospective. Update this runbook with anything learned. Track preventive actions to completion.

---

## Severity levels

Used to set response urgency and communication scope.

| Severity | Definition | Response |
|---|---|---|
| **SEV-1 — Critical** | Active exploitation with user-fund risk, protocol integrity risk, or org-wide credential compromise. | Full response, all roles assigned, public communication within hours. |
| **SEV-2 — High** | Compromise of a QCT-controlled surface without confirmed user-fund risk, but with credible potential for escalation. | Full response, all roles assigned, communication scoped to affected parties. |
| **SEV-3 — Moderate** | Isolated compromise (e.g., single non-critical repository, single credential with narrow scope), fully contained, no user impact. | Streamlined response, Incident Lead may hold multiple roles, communication may be internal-only depending on findings. |
| **SEV-4 — Low** | Suspicious activity investigated and confirmed benign, or minor issue with no user impact. | Log and close. No public communication required unless there is a reporting obligation. |

The Incident Lead sets and may adjust severity as the picture develops. **Start high, downgrade later** — the opposite direction is much more expensive.

---

## Decision principles

When the situation is unclear and the clock is running, default to these:

1. **Bias to declaring.** A false alarm costs an hour of team time. A missed incident costs users.
2. **Bias to over-warning users.** Users can be told "we over-corrected, you're fine" later. They cannot un-lose funds.
3. **Bias to rotating credentials.** Rotating an unaffected credential is a nuisance. Missing an affected one is another incident.
4. **Bias to resetting rather than verifying.** If an artefact might have been tampered with and you cannot verify it in the time available, reset it.
5. **Communicate what you know AND what you don't.** Say "we do not yet know" when you do not know. Users trust honest gaps more than confident guesses.
6. **The response is not on GitHub.** No incident coordination happens on a platform that could be the one that is compromised.
7. **One voice out.** Public communication comes from the Communications Lead only, cleared by the Incident Lead. No side-channel updates from other team members during the response.

---

## Standing prerequisites

Things that need to be **already in place before an incident**, checked quarterly:

- QCT security on-call rotation defined; every member has access to the emergency chat and the task tracker.
- Emergency chat channel exists; membership is current.
- Third-party task tracker exists; template incident board exists.
- List of integrators / exchanges / key partners with direct contact points is current.
- Documented containment procedures for each critical service exist and are accessible off-GitHub (e.g., how to pull a Docker image, how to block a Web Wallet deployment, how to rotate each specific credential class).
- Every QCT contributor has completed basic security hygiene onboarding (2FA, credential storage, phishing awareness).

The Incident Lead on-call reviews this list at the start of every rotation.

---

## After-action

Every incident, regardless of severity, produces:

1. **Task tracker entries** for every action taken and every decision made, with timestamps.
2. **A written internal report** (SEV-1): timeline, attack chain, impact, response actions, artefacts (internal-only fields like token IDs, commit hashes, account names).
3. **A public post-mortem** (SEV-1 and SEV-2): the sanitised version of the internal report, safe for external distribution.
4. **Preventive-action commitments** with named owners and target dates, tracked to completion.
5. **A runbook update** if anything about the response revealed a gap in this document.

---

## Related documents

- Per-service containment procedures — maintained in the internal QCT documentation (off-GitHub).
- Contributor security hygiene onboarding — maintained in the internal QCT documentation.

---

## Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-07-14 | 1.0 | Initial version, drafted following the 2026-07-13 GitHub organization security incident. Formalizes the response process that worked during that incident. | @j0et0m |
