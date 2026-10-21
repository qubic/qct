# Qubic Web Wallet Weak-Entropy Incident Post-Mortem

**Date of incident:** 17–18 October 2026
**Report date:** 21 October 2026
**Affected component:** Qubic Web Wallet (browser seed generation), origin ts-library / wallet
**Root cause:** Predictable 55-character seeds generated from `Math.random()` backed by a 32-bit-seeded PRNG on Apple platforms
**Victims:** 1,405 drained identities
**Total funds moved:** 2,955,249,075,344 QU (~2.955T QU)
**Network status:** Halted 18 Oct ~00:30, restored 20 Oct ~20:00

---

## Summary

Between **23:30 17 Oct** and the network halt at **~00:30 18 Oct 2026**, an attacker drained **1,405 Qubic identities** for a combined **2,955,249,075,344 QU** (~2.955T QU). The funds were swept into **7 collection wallets** and partially forwarded one hop further; **379,039,994,345 QU** reached a deposit address at the exchange **MEXC** before exchanges froze.

The root cause was **not** a flaw in the Qubic protocol, core node, or signature scheme. It was **weak seed entropy in the web wallet**: seeds were generated with JavaScript `Math.random()`, which on Apple platforms (Safari on macOS/iOS, and every browser on iOS, which is forced to use WebKit) is backed by WebKit's `WeakRandom` — a PRNG with only a **32-bit seed**. The entire seed space is therefore brute-forceable. With ~200× RTX 5090 the core devs reproduced the bulk of the keyspace in **~12 hours** using unoptimized code; a dedicated attacker did the same.

The community voted to **revert the hacker transactions** and route recovered assets through an **escrow identity** for victim claims. **2,560,596,538,581 QU** (~2,560.6B) was reverted and made claimable; the **379.04B** that reached MEXC is pending exchange refund.

---

## Discussion of the exploit

### Wallet design and seed generation

A Qubic identity is derived deterministically from a **55-character lowercase seed**:

```
seed (55 chars, a–z) → K12 → subseed → FourQ SchnorrQ keypair → 60-char identity
```

The seed is the **only** secret. Anyone who can reproduce a seed controls the identity and can sign transfers. Seed quality depends entirely on the randomness used to pick the 55 characters.

The web wallet generated each character by drawing from `Math.random()`:

```
for i in 0..55:
    seed[i] = alphabet[ floor(Math.random() * 26) ]
```

This is safe **only if** `Math.random()` is backed by a cryptographically strong, high-entropy source. It is not, on all platforms.

### The randomness problem

`Math.random()` is **not** a CSPRNG by specification. Engines implement it with xorshift128+, which has 128 bits of **internal state** but is **seeded** differently per engine. The security bound is the **seed** entropy, not the state width:

- **V8 (Chrome):** takes a **64-bit** seed from the OS entropy source, then expands it into the two 64-bit state words (`s0 = MurmurHash3(seed)`, `s1 = MurmurHash3(~seed)`, fmix64 finalizer). The 128-bit state is derived deterministically from 64 bits, so brute-forcing from scratch is **2^64**, not 2^128. Infeasible today, but far less than the state width implies.
- **SpiderMonkey (Firefox):** seeds from the OS CSPRNG as well; not brute-forceable from scratch.
- **JavaScriptCore / WebKit (Safari, all iOS browsers):** `WeakRandom` is seeded from a **32-bit** value. The whole seed space is **2^32 ≈ 4.3 billion** — trivially enumerable on commodity GPUs.

Reference: [WebKit `WeakRandom.h`](https://raw.githubusercontent.com/WebKit/WebKit/main/Source/WTF/wtf/WeakRandom.h)

Because the PRNG is seeded from 32 bits, **the derived wallet seed carries at most 32 bits of entropy**, regardless of being 55 characters long. An attacker does not search 26^55 seeds; they search 2^32 PRNG seeds, replay the stream, and read off the wallet seed.

### Affected combinations

| Platform | Browser | Vulnerable |
|---|---|---|
| macOS (MacBook / iMac) | Safari | Yes |
| iOS / iPadOS (iPhone / iPad) | **Any** browser (all use WebKit) | Yes |
| Windows / Linux / Android | Chrome | No (64-bit OS-seeded; 2^64 brute) |
| Windows / Linux / Android | Firefox | No (OS CSPRNG-seeded) |

The common pattern across victims was **Apple devices**, confirmed during the victim survey at ~04:00.

### The attack vector

The reproduction is mechanical. For each of the 2^32 possible PRNG seeds, replay the output stream and slide a 55-character window over it; each window yields a candidate wallet seed, hence an identity to check against the ledger:

```
for seed in 0 .. 2^32-1:
    rng = WeakRandom(seed)
    stream = [ rng.next() for _ in 0..N ]        # more draws = more coverage
    for offset in sliding_window(stream, 55):
        wallet_seed = map_to_alphabet(stream[offset : offset+55])
        pubkey      = derive_identity(wallet_seed)
        if pubkey in funded_ledger:
            mark_vulnerable(wallet_seed, pubkey)
```

**Reproduction cost.** With ~200× RTX 5090, the core devs scanned most of the viable keyspace in **~12 hours** with unoptimized code. The attacker had the same capability. This is the central takeaway: 32-bit seed space is **not** a security boundary against modern GPUs.

### Impact of the exploit

| Measure | Amount (QU) |
|---|---|
| Total moved from victims | 2,955,249,075,344 |
| Traced into collection wallets (hop 1) | 2,939,636,532,926 |
| Reverted / made claimable | 2,560,596,538,581 |
| Reached MEXC (pending refund) | 379,039,994,345 |

Victim distribution (loss per identity):

| Loss bucket | Victims |
|---|---|
| ≥ 10B QU | 61 |
| 1B – 10B QU | 398 |
| 1M – 1B QU | 583 |
| < 1M QU | 363 |

- Victims: **1,405** identities.
- Largest single loss: **212,742,939,086 QU**.
- Top 10 victims accounted for **25.9%** of all funds moved.

**Fund flow.** The 1,412 victim transfers converged on **7 collection wallets** (hop 1). Those wallets largely **held** the funds — only 3 outgoing transactions were made, forwarding **473B QU** one hop further to two downstream addresses:

- `DKYOWQRWAYFDIHPOWVCEJLZFPJHCYWZGDRHQJEFJFDMTPUYVLLPRYMTBHNQG` — **379,039,994,345 QU** → routed to MEXC.
- `RPJPMIGBLLQLNDMUICFSRFAAUAHASCFRTPGDRKAWPBIIHMLMATEMBMAENKMD` — **94,000,000,000 QU**.

Because the collection wallets held rather than laundered, nearly the entire haul was recoverable on-chain once the network halted.

---

## Factors that contributed to the exploit

Several conditions combined to turn a known weakness into a full drain.

### Weak PRNG used for key material — the root enabling condition

Key material was generated from `Math.random()`, a non-cryptographic PRNG. On Apple platforms the backing PRNG is 32-bit-seeded, collapsing effective seed entropy from a nominal 26^55 to **2^32**. Any wallet created on an Apple device was brute-forceable from the moment it was funded.

### Legacy, unreviewed wallet code

The web wallet originated as **pool code written by qubic.li around 2023** for their miners. It was later contributed to the community and adopted as the **official** wallet, but received little security review or upgrade thereafter. A security-sensitive seed-generation path ran for ~2 years without audit.

### Known-but-dismissed entropy concern

The weak-entropy issue was introduced ~2 years ago and had been discussed internally:

- [ts-library PR #5](https://github.com/qubic/ts-library/pull/5/changes)
- [wallet PR #229](https://github.com/qubic/wallet/pull/229)

The internal conclusion was that Chrome and Firefox seed `Math.random()` from the OS RNG, so the wallet "should be secure." **Safari and the Apple ecosystem were not considered**, and iOS's forced use of WebKit for all browsers was missed. The gap sat in production until exploited.

### No defense in depth

Seed generation was the single point of failure. There were no secondary entropy sources mixed into the seed, no hardware-RNG path, and no warning to Apple users, so a single weak primitive translated directly into total loss of funds.

---

## Response

### Containment

The decisive mitigation was the **coordinated network halt** at ~00:30 on 18 Oct, which trapped ~2.8T QU of in-flight funds before they could be dispersed or bridged off-chain, and the rapid exchange freezes that locked the hacker deposit accounts.

### Recovery mechanism

The community voted to **revert all hacker transactions** and hold recovered assets in an **escrow identity** for victim claims, implemented via a coordinated restart. **2,560.6B QU** was reverted and made claimable; the **379.04B QU** that reached MEXC is pending exchange refund paperwork.

### Still-exposed funds

Beyond the 1,405 drained identities, the full keyspace scan found **~2,900 vulnerable identities** in total. **1,391 weak wallets** holding **260,180,436,980 QU** were still funded but not yet drained at halt time. These were prioritized for freeze/migration in the restart, since their seeds are equally reproducible by anyone.

### Permanent fix

All Qubic software has moved to **true hardware randomness** (CSPRNG) for key generation. WebKit also shipped a fix to the JIT `Math.random()` path (2026-09-30, bug 299242), but that does **not** restore entropy — it only aligns engine behavior. The entropy fix must live in the wallet: never derive key material from `Math.random()` on any platform.

---

## Is `Math.random()` on Chrome also weak?

Not brute-forceable today, but **not strong either**. Chrome/V8 seeds xorshift128+ from the OS entropy source with a **64-bit** seed, which it expands via MurmurHash3 into the two 64-bit state words. So the from-scratch brute bound is **2^64**, not the 2^128 the state width suggests. Two caveats:

- xorshift128+ is **predictable**: an observer who sees a handful of outputs can recover the 128-bit state and compute all past/future values. It is not designed to resist an adversary who sees any output.
- The only thing protecting a Chrome-generated wallet is that the attacker **cannot see the outputs** and cannot brute-force 2^64. That is a brittle guarantee for key material, and the margin erodes as hardware improves. 2^32 fell to ~200 GPUs in 12 hours in 2026; 2^64 is ~4.3 billion times larger — infeasible now, but the exact thing that was thought safe about 32-bit once, too.

**ETA to brute-force the Chrome (2^64) case.** By our estimation, a basic scan of the 2^64 seed space takes around **5 years** on a hypothetical **10,000× RTX 5090**. That number is not great, but also not that low — Chrome wallets are not under the same immediate threat the Apple (2^32) wallets were.

Still, "5 years on a 10,000-GPU cluster" is a margin that only shrinks as hardware improves and scanners are optimized, and it rests on 64 seed bits rather than the 128-bit state the generator advertises. It is a reason to act, not to relax. **We therefore recommend migrating to fully secured (hardware-CSPRNG) seeds now**, rather than depending on brute-force cost to stay ahead.

**`Math.random()` is never acceptable for key generation, on any browser.** Use a CSPRNG (`crypto.getRandomValues` / hardware RNG) for anything that protects funds.

---

## Timeline

All times are on **18 October 2026** unless stated otherwise.

**Before the incident**

- **~23:30, 17 Oct** — Attack begins; drains start across Apple-generated wallets.

**18 October 2026**

- **00:00** — Users report to computors and the team that many funds are moving suspiciously.
- **00:30** — Computors coordinate a **network halt** for investigation, trapping ~2.8T QU on-chain.
- **01:00** — Emergency channel established to gather evidence and run survey.
- **01:05** — First money-flow report; an attack is confirmed.
- **01:10** — All exchanges informed; asked to freeze related funds.
- **01:15** — Core devs begin victim survey to find the common pattern.
- **01:52** — Exchanges confirm the hacker accounts are frozen.
- **04:00** — Survey shows most victims used wallets on **Apple devices**.
- **04:30** — Core devs reproduce the attack vector and recover seeds for several drained identities.
- **05:00** — Mass scan of the network for vulnerable identities begins.
- **06:00** — Dedicated victim-support communication channel established.
- **12:00** — Community vote on next steps → **revert hacker transactions**, hold assets in an **escrow identity**.
- **13:00** — Core devs reproduce all **1,405** hacked identities.

**19 October 2026**

- **06:00** — Full scan complete: **>1,500** additional vulnerable identities found (≈2,900 vulnerable in total, including the 1,405 already drained).
- **12:00** — Core devs publish the list; coordinate with computors to bring the network back online.

**20 October 2026**

- **20:00** — Qubic network back online; hacker transactions reverted, **except** the 390B-class deposit that reached MEXC.
- **22:00** — Claim platform released for victims to recover assets.

---

## Conclusion

This incident was a **key-generation failure in client software**, not a protocol break. The Qubic chain, FourQ signatures, and computor consensus performed exactly as designed; the loss came from seeds that never had more than 32 bits of entropy because they were produced by `Math.random()` on Apple platforms. Any 55-character seed built that way was reproducible by anyone with a handful of GPUs and a few hours.

Three things made it worse than a single bad primitive: security-sensitive code inherited from a mining pool and adopted as official without audit; a known entropy concern dismissed because Chrome and Firefox looked fine while Safari/iOS were never checked; and no defense in depth to catch a weak seed before funds depended on it.

The recovery worked because the network could be halted and transactions reverted by computor coordination, and because the attacker held funds in a few traceable wallets rather than dispersing them. That should be read as fortunate, not as a reliable safety net. The durable fixes are the ones already in flight: hardware CSPRNG for all key generation, a security review process for wallet code regardless of its origin, and the hard rule that `Math.random()` is never used for anything protecting funds — on any browser.

---

***This report is informational. Figures are drawn from on-chain data and the ongoing investigation and may be revised as analysis continues. It is not an admission of liability by any party.***
