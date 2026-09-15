# Fund Qubic Core — Base Services Reserve

> [!IMPORTANT]
> **DRAFT.** This proposal is being circulated for team discussion and computor feedback ahead of the 2026-09-23 vote. Numbers, scope, and wording may still change before the final submission.

## Proposal
Send **190 billion QUBIC** to the identity `XQCLNHCEHTKQZDBAHJFVVTRMWFACMAZOBAEDQHEITGGEWZDIBRAIYWPGEONG` to lock in a **15-month base-services runway** for Qubic Core Technology, covering **October 2026 through December 2027**.

> Option 0: no

> Option 1: yes, 190 billion QUBIC

## Why a separate "base services" proposal?

The goal is a **sustainable finance structure** for QCT: predictable spending that survives market volatility and single-vote outcomes.

The CCF treasury is approaching its lower operating range. To de-risk continuity of the Qubic protocol and its user-facing applications, this proposal secures a **minimum baseline** for QCT operations through the end of 2027 — independent of, and in addition to, the normal quarterly operational proposal (see [companion Q4 2026 operational proposal](./2026-09-15-Q4-2026-proposal.md)).

Objective: guarantee that even under adverse funding conditions, **the public apps (Explorer, Wallet, Proposals, Docs) stay online, the RPC/API keeps serving, the long time archive is available and the basic test-net infrastructure keeps running**.

This proposal covers **only** what the running services actually cost. Normal QCT operations (Core team, Integration team, active Testing, Management, Client/Frontend development, BD, incubation) are funded through the separate quarterly operational proposals.

## Scope — what "base services" covers

| Category | Included | Notes |
|---|---|---|
| **explorer.qubic.org** | ✅ | Public block/tx explorer |
| **wallet.qubic.org (+ mobile)** | ✅ | Web + Android/iOS wallet apps |
| **proposals.qubic.org** | ✅ | Quorum-proposal frontend |
| **docs.qubic.org** | ✅ | Documentation site |
| **RPC / API (rpc.qubic.org)** | ✅ | The public RPC endpoint that all apps depend on |
| **Archiver** | ✅ | Historical tx data feed for Explorer / RPC |
| **Basic test net setup** | ✅ | Minimum test net kept running (advanced regression is in the operational proposal) |
| **Test nodes** | ✅ | A small set of test nodes (Core Lite / Bob) run publicly for reference |
| **Tiny tools** | ✅ | GitHub bots, public stats / calculations |
| **Domains + hosting** | ✅ | qubic.org and related required domains, server capacity |
| **DDoS protection** | ✅ | Anti-DDoS on public endpoints |
| **E-mail (Google Workspace)** | ✅ | Team e-mail |
| **Monitoring / alerting** | ✅ | Uptime + error alerting for the endpoints above |
| **Basic infrastructure maintenance** | ✅ | Minimum engineer time to keep the above running + apply critical fixes |
| Core team salaries | ❌ | Not covered — funded via operational proposals |
| Integration team | ❌ | Not covered — funded via operational proposals |
| Active Testing team | ❌ | Not covered — funded via operational proposals |
| Client / Frontend development | ❌ | Not covered — funded via operational proposals |
| Management, BD, incubation | ❌ | Not covered — funded via operational proposals |
| CI/CD (GitHub Actions) | ❌ | Free tier |
| Neuraxon / AI research infra | ❌ | Not part of base |

If both this proposal and the Q4 operational proposal are approved, they combine into full-team Q4 operations. If only this proposal is approved, its user-facing apps, and the RPC/API all keep running for 15 months — but active development, advanced testing, and community driven updates pause.

## Cost break down (base services only)

| Line | Monthly (USD) | Description |
|------|--------------|-------------|
| Infrastructure (all-in) | ~$3,000 | Servers / hosting for Explorer / Wallet / Proposals / Docs / RPC / archiver / test net + test nodes; DDoS; domains; Google Workspace e-mail; monitoring; GitHub bots + tiny tools |
| Basic infrastructure maintenance | ~$2,000 | Engineer time for monitoring, critical fixes, seamless-transition support |
| **Total** | **~$5,000** |  |

### Reserve total (15 months, Oct 2026 – Dec 2027)

| | Amount |
|---|--------|
| Monthly base | ~$5,000 |
| Duration | 15 months (Oct 2026 – Dec 2027) |
| Reserve subtotal | ~$75,000 |
| Valuation | 400 USD/bln |
| **Reserve request** | **190 billion QUBIC (~$76,000)** |
| Headroom vs subtotal | ~$1,000 (~1% buffer) |

### Feasibility vs. CCF balance

- CCF balance today: ~828.5B QU (~$331K at 400 USD/bln).
- This proposal is **190B QU (~23% of current CCF)** — leaves ample CCF headroom for the companion Q4 operational proposal and other ecosystem needs.

## Governance

- Funds sit in the QCT treasury and are drawn down monthly at the base-services rate. Any unspent QU at the end of 2027 either rolls forward or returns to CCF.
- Base-services spend appears in the existing monthly QCT financial reports, tagged so the community can distinguish base-services spend from operational spend.
- @j0et0m receives the funds upon approval and is responsible for reporting all expenditures.
- Independent community-team access to salary details, service invoices, and transaction history is preserved, same as with the operational proposals.
- **This proposal does not replace the quarterly operational proposals.** QCT will continue to request Q4 2026, Q1 2027, Q2 2027, Q3 2027, and Q4 2027 operational budgets as separate proposals for the delta above base services.

## Companion documents

- [Q4 2026 operational proposal](./2026-09-15-Q4-2026-proposal.md) — 3-month full-team delta above base services (funded separately).
- [Q4 org report](../org-reports/2026-09-15-org-report.md) — team roster going into Q4.
- Monthly financial reports: [github.com/qubic/qct/financial-reports](https://github.com/qubic/qct/financial-reports).
