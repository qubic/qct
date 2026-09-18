# Qubic Vottun Bridge Incident Post-Mortem  
  
Date: 2026-09-17   
Affected component: Vottun Bridge smart contract (Qubic side) and Vottun bridge middleware (off-chain, closed-source)   
Contract source: [src/contracts/VottunBridge.h](https://github.com/qubic/core/blob/main/src/contracts/VottunBridge.h)   
Bridge contract (Qubic): [ZAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAZUQI](https://explorer.qubic.org/network/address/ZAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAZUQI)   
wQUBIC token (Ethereum): [0xa989edfee575425904514d4090846a5afd58f225](https://etherscan.io/token/0xa989edfee575425904514d4090846a5afd58f225)  
  
## Summary  
  
On August 20, 2026, between 23:34 and 23:41 UTC, an attacker drained approximately **319.3 billion QUBIC** from the Qubic side of the Vottun Bridge in three escalating withdrawals of 40 million, ~10.6 billion, and ~308.7 billion QUBIC. The entire attack took place on the Qubic side; nothing was deposited, burned, or touched on Ethereum.  
  
The attacker funded a fresh Qubic address from an exchange, used the bridge contract's **public** `createOrder` procedure to create Ethereum=>Qubic withdrawal orders naming that address as the destination, and then - by means that cannot be explained with public available information - had each order completed by a Vottun-controlled **manager** identity. Two different manager identities signed the three completions. No wQUBIC was ever deposited on Ethereum for any of the three orders, and nothing in the system checked: the Qubic-side contract has **no mechanism whatsoever to verify that an Ethereum-side deposit actually happened** before releasing QUBIC. It trusts the manager's signature alone.  
  
As a result, the bridge is no longer backed. Roughly **325 billion wQUBIC** remain in circulation on Ethereum across 137 holders, while only about **4.2 billion QUBIC** remain in the bridge contract on Qubic. Approximately 98.7% of the wQUBIC supply is currently unbacked.  
  
The bridge smart contract is deployed on Qubic and its source is public. The middleware that holds the manager keys and drives the contract is developed and operated exclusively by Vottun and is closed-source. Vottun has stated that this middleware was compromised and has provided a private forensic report to core developers and a few members of the community. **The report cannot be independently verified, because the middleware is entirely closed-source.** The community therefore cannot verify how the manager signatures were obtained, and cannot rule out other explanations such as leakage or theft of private keys by other means. This report treats the root cause on the operator side as unconfirmed.  
  
This report discusses the exploit, its impact, and the factors that contributed to the incident.  
  
## Discussion of the exploit  
  
### Bridge design  
  
The Vottun Bridge connects QUBIC on the Qubic network with wQUBIC, an ERC-20 token on Ethereum. The Qubic-side smart contract custodies the QUBIC that backs wQUBIC. Cross-chain transfers are driven by an off-chain middleware operated by Vottun: when a user deposits or burns wQUBIC on Ethereum, the middleware is expected to observe that event and then instruct the Qubic contract to release the corresponding QUBIC.  
  
The contract has two privilege tiers:  
  
- **Multisig admins** - three identities, with a 2-of-3 approval threshold. Admins can only act through proposals (`createProposal` / `approveProposal`). Proposal types are: replace an admin, add a manager, remove a manager, withdraw accumulated fees, and change the threshold.  
- **Managers** - up to three identities, elected by the multisig. Once elected, a manager has **full, unilateral authority** to complete or refund any order. No further approval is required.  
  
An Ethereum=>Qubic transfer (releasing QUBIC to a user) is a two-step process on the Qubic side:  
  
1. `createOrder` - a **public** procedure with **no caller restriction**. We have verified this in the contract source: the procedure checks only that the amount is at or above `minimumOrderAmount`, that the invocation reward covers the 1% fee (0.5% operator + 0.5% network), that the supplied Ethereum address is non-zero, and - for Ethereum=>Qubic orders - that `lockedTokens` is sufficient. It never checks `isManager` or `isAdmin`. Any identity can therefore create an order for any amount and any Qubic destination, and for Ethereum=>Qubic orders the contract immediately reserves the amount from `lockedTokens`.  
2. `completeOrder` - a **manager-only** procedure. It checks `isManager(invocator)`, transfers the reserved amount to the order's `qubicDestination`, and marks the order completed.  
  
A third procedure, `transferToContract`, exists only for the opposite direction (Qubic=>Ethereum): it is how a user deposits their QUBIC into the contract after creating an order. For Ethereum=>Qubic orders the contract explicitly rejects it and refunds whatever was attached (guard `[QVB-22]` in the source). It plays no functional role in the attack, but the attacker called it anyway; see below.  
  
**Neither `createOrder` nor `completeOrder` contains any check that a corresponding deposit or burn of wQUBIC occurred on Ethereum.** The contract has no oracle, no proof submission, no Ethereum event hash, and no second signer. The question "did the user actually lock wQUBIC on the other side?" is answered nowhere on-chain; it is delegated entirely to the operator's closed-source middleware, and the contract simply trusts whatever a manager signs. In other words, the only thing standing between the custodied QUBIC and any Qubic address is a single signature from a manager identity.  
  
### The attack  
  
The attacker operated from the Qubic address `SQDKIHJHJROAKCGKYGJROORTAQCAKZRKHKNXZYSTCDFOTVPNZIBLEXAGXHLD` (hereafter `SQDKI...`). All order creations came from this address; all completions came from Vottun-controlled manager identities. All times UTC.  
  
**Step 0 - Funding.** The attacker purchased `67,111,161` QUBIC on Bitget and withdrew it to `SQDKI...`. ([tx dxvlyifo...](https://explorer.qubic.org/network/tx/dxvlyifoyybwcgluwkyrrbpqzhkbsobdozdmwwezzabedsooovgwrqncmhpd)) This was enough to pay the 1% fee on the first two orders; the third order's fee was paid out of already-stolen funds.  
  
**Step 1 - Probe: 40 million QUBIC (Order 785).**  
  
- **23:34:44** - `SQDKI...` calls `createOrder` (Ethereum=>Qubic) for `40,000,000` QUBIC, paying a `400,000` QUBIC fee. Order ID 785 is created and 40M is reserved from `lockedTokens`. ([tx yqfuptss...](https://explorer.qubic.org/network/tx/yqfuptssrgxyhbceokttvsfcaxyapvqqcwewgaywlbisagokoakqwvabgyab))  
- **23:35:07** - `SQDKI...` calls `transferToContract` on Order 785 with 40M attached. The contract rejects it (wrong direction) and refunds the 40M. ([tx annxkrbi...](https://explorer.qubic.org/network/tx/annxkrbibghtbhihaevubfiboaofstmmhjnmgsauobpociybxsfcgcddpano))  
- **23:35:09** - Manager `NXVNLCDJVBAPYGASOCATQDJLOLFAUKSFTWYFVZKTLCFUAUPHFKGKIUTEABUD` calls `completeOrder` on Order 785. `40,000,000` QUBIC is released to `SQDKI...`. No wQUBIC was deposited on Ethereum. ([tx ojjokfaf...](https://explorer.qubic.org/network/tx/ojjokfafkbrgzfluujuagkpdhmabyahndiqegjjgfgdpcbizuuvwzanfjvqk))  
  
**Step 2 - Escalation: ~10.6 billion QUBIC (Order 786).**  
  
- **23:39:52** (tick 75,452,051) - `SQDKI...` calls `createOrder` for `10,571,116,199` QUBIC, paying a `105,711,160` QUBIC fee. Order ID 786 is created. ([tx hsmvolrz...](https://explorer.qubic.org/network/tx/hsmvolrzhrqoudfauoiyiesgeptgnayetebjxjjcbafqpipjdctsrtjdwcvl))  
- **23:39:58** (tick 75,452,063) - Manager `WGHJGJYOZGCAKDNGFIYREONXFSRCUYMEDVLPHOFXLDSFNYNIOBOFNXEGTBRI` calls `completeOrder` on Order 786. `10,571,116,199` QUBIC is released to `SQDKI...`. ([tx lwjndbtn...](https://explorer.qubic.org/network/tx/lwjndbtnccqofetuvzouvfojhdndeatmuawzajqcocvcousdfdvxctwccghb))  
- **23:40:12** - `SQDKI...` calls `transferToContract` on Order 786 with `10,571,116,199` attached; rejected and refunded. ([tx thbtgajl...](https://explorer.qubic.org/network/tx/thbtgajlaqgkqewvskxfzpctnsocwlfpbbfdrhquqcbyaejqfjtkvzggjfhc))  
  
**Step 3 - Main drain: ~308.7 billion QUBIC (Order 787).**  
  
- **23:41:24** (tick 75,452,243) - `SQDKI...` calls `createOrder` for `308,708,125,375` QUBIC, paying the ~3.09 billion QUBIC fee out of the proceeds of Order 786. Order ID 787 is created; `lockedTokens` is reduced by the full amount. ([tx uavhnbme...](https://explorer.qubic.org/network/tx/uavhnbmezqvzvajhalesvgylhzgdvlwzmurhmvwycabxqtnihpduxinajgfe))  
- **23:41:28** (tick 75,452,251) - Manager `WGHJ...` calls `completeOrder` on Order 787. `308,708,125,375` QUBIC is released to `SQDKI...`. ([tx egdnpdho...](https://explorer.qubic.org/network/tx/egdnpdhonoqymbcvfkbgelfjnbhgnfspcnzzwarwlgmwalakzrobnbjaobuf))  
- Shortly after - `SQDKI...` calls `transferToContract` on Order 787 with `308,708,125,375` attached; rejected and refunded. <!-- TODO: add tx link and timestamp -->  
  
From the first `createOrder` to the last `completeOrder`, the attack took under seven minutes. Each completion followed its order by two to six seconds. No bug in the contract logic was exploited: every check the contract performs passed, because the completions were signed by keys the contract was designed to trust. The exploit is **fabricated Ethereum=>Qubic orders completed by compromised manager authority**, enabled by a contract that never verifies the Ethereum side.   
  
The attacker did not need the multisig admin keys, and did not need a manager key to *create* the orders - `createOrder` is public and was called from the attacker's own address. Only the three `completeOrder` calls required manager authority, and manager authority alone is sufficient to drain the contract.  
  
**On the `transferToContract` calls - worth flagging.** For Ethereum=>Qubic orders these calls are functionally meaningless at the contract level: guard `[QVB-22]` rejects them and refunds the attached QUBIC, so on-chain they changed nothing. Yet the attacker made one for every order - before completion for Order 785, after completion for Orders 786 and 787 - each time attaching the *exact* order amount (40M, then 10,571,116,199, then 308,708,125,375). Attaching hundreds of billions of QUBIC to a call you expect to be refunded is not something you do by accident, and it is not something a naive attacker spraying transactions would do. It is deliberate, and it is the single most revealing behavior in the whole sequence. However, we do not have a confirmed explanation, hence that is exactly why it deserves to be called out. Several readings are possible (they are all hypothesis that would need to be validated analizing the middleware):  
  
- **They knew the middleware's expected call pattern.** A legitimate Qubic=>Ethereum deposit is `createOrder` => `transferToContract` => `completeOrder`. If Vottun's middleware watches the chain for that pattern - or reconciles orders by matching a `transferToContract` against each order - then emitting the call, even though the contract rejects it, could be what made the middleware treat the order as "funded" and proceed to sign `completeOrder`. Under this reading the redundant call was not redundant to the *middleware*; it was the trigger. That would mean the attacker understood (or successfully guessed) the middleware's internal logic, not just the public contract - which points toward deep access to, or knowledge of, the closed-source backend rather than a simple stolen key.  
  
- **They were bypassing an off-chain check.** A near-variant of the above: the middleware may gate `completeOrder` on some signal that a `transferToContract` for that order exists, without validating direction or the refund. The attacker may have discovered that firing the call - regardless of outcome - satisfied that gate.  
  
- **They were blending in.** Every honest order the middleware ever processed may carry a `transferToContract`. Reproducing it makes the malicious orders look, to a casual observer scanning the explorer or to a naive pattern-matching monitor, like ordinary bridge traffic. This is easier to explain but does not account for why the call appears *after* completion on two of the three orders, where it can no longer influence anything.  
  
- **They were probing / imitating without full understanding.** The attacker may simply have copied the three-step shape of previous on-chain orders from these managers without knowing the contract would reject the deposit for this direction.  
  
These are not equally comforting. The first two would mean the attacker knew how the middleware behaved internally, which narrows the field of who could have done this and arises questions about how the keys and the backend were accessed. The last is the most benign. **The way to distinguish them is on-chain and available to anyone:** examine the managers' *legitimate* historical orders. If genuine EVM=>Qubic completions were routinely preceded by a (rejected) `transferToContract`, then the call is the middleware's own fingerprint and the attacker was reproducing it - evidence the drain ran through the middleware's normal path. If legitimate orders never carried such a call, the attacker added it themselves, and the "knew the internals" readings gain weight. We flag this as an open item for the joint review;  
  
### How the manager signatures were obtained  
  
The completions were signed by two distinct Vottun-controlled manager identities: `NXVN...` for Order 785 and `WGHJ...` for Orders 786 and 787. Neither is the manager hard-coded at contract initialization; both were added by Vottun after deploying the smart contract.  
  
Vottun has stated that its middleware infrastructure was compromised by an external attacker and that the attacker used the compromised middleware to issue the completion transactions. Vottun has shared a private forensic report with core developers and a small number of community members. That report is Vottun's property and is not reproduced here.  
  
At a high level, the report describes an intrusion into the bridge backend and attributes the drain to that intrusion. It does not, however, provide evidence the community can independently check. The middleware is closed-source, and the community has no access to its code, its logs, its database, or its key store. Every claim about what happened inside the backend - how it was entered, what the attacker could reach, and how signatures were produced - rests on artifacts that only Vottun holds.  
  
What the community *can* verify is the on-chain record:  
  
- All three orders were created by the attacker's own address `SQDKI...`, not by any bridge-controlled identity.  
- All three orders were completed by Vottun-controlled managers, and **two different manager identities** were used within four minutes of each other. Whoever executed this attack could obtain signatures from more than one manager key.  
- Every completion followed its order by a few seconds, with no Ethereum-side deposit for any of them.  
- The funds were released to `SQDKI...` and subsequently moved onward.  
  
The use of two managers is consistent with an attacker driving a system that holds multiple manager keys, rather than possessing one stolen key. It is also consistent with two stolen keys. The on-chain record confirms *that* two Vottun-controlled managers signed the drain; it does not show *how* those signatures were obtained, and Vottun's account does not yet establish it either. The following therefore remain unconfirmed:  
  
- Whether the `NXVN...` and `WGHJ...` private keys were used through the middleware, or were exfiltrated by other means (insider access, backup or configuration leak, etc.).  
- Whether the middleware, when it signs `completeOrder`, performs any check against Ethereum at all - and if so, how it was bypassed three times in a row.  
- Whether the keys were ever held in an HSM or equivalent, or were stored in plaintext on the middleware host.  
- Who had access to the middleware and its key material at the time of the incident.  
- Why three fabricated withdrawals over seven minutes, growing by more than four orders of magnitude, were not detected promptly.  
  
Until Vottun provides evidence that can be independently verified - or makes the relevant middleware components available for review - the community considers the root cause on the operator side an **open question (?)**.  
  
****Impact of the exploitß****  

|  | Amount |
| ---------------------------------------- | ----------------------------------- |
| QUBIC withdrawn - Order 785 (probe) | 40,000,000 |
| QUBIC withdrawn - Order 786 (escalation) | 10,571,116,199 |
| QUBIC withdrawn - Order 787 (main drain) | 308,708,125,375 |
| QUBIC withdrawn - total | 319,319,241,574 |
| QUBIC remaining in bridge contract | ~4,200,000,000 |
| wQUBIC in circulation on Ethereum | ~325,000,000,000 |
| wQUBIC holders on Ethereum | 137 |
| Unbacked wQUBIC | ~320,800,000,000 (~98.7% of supply) |
  
  
- All stolen QUBIC was released to `SQDKIHJHJROAKCGKYGJROORTAQCAKZRKHKNXZYSTCDFOTVPNZIBLEXAGXHLD`.  
- wQUBIC holders on Ethereum can no longer redeem their tokens 1:1 for QUBIC through the bridge. The bridge is effectively insolvent.  
- The Qubic network itself, its consensus, and the QUBIC token supply were **not** affected. No QUBIC was minted or created; funds were moved out of the contract that custodied them.  
- Wallets, exchanges, and other contracts on Qubic are unaffected.  
  
## Factors that contributed to the exploit  
  
The following section examines the design, operational, and governance factors relevant to this incident. We are documenting them so the community can identify specific areas where its security practices should be strengthened.  
  
### No cross-check of the Ethereum-side deposit - the root enabling condition  
  
This is the single most important factor. The bridge's entire security model rests on one assumption: that a manager will only complete an Ethereum=>Qubic order if a matching wQUBIC deposit or burn really happened on Ethereum. **Nothing enforces that assumption.** Not the contract, not an oracle, not a second signer, and - as far as the community can tell - not any independent monitor.  
  
Concretely:  
  
- `createOrder` accepts an Ethereum=>Qubic order from anyone, for any amount, with only a fee check. It does not ask for, and cannot verify, any evidence of an Ethereum-side event. The attacker created all three orders from a fresh address funded with 67M QUBIC from an exchange.  
- `completeOrder` releases the funds on a manager signature alone. It does not ask for, and cannot verify, any evidence of an Ethereum-side event.  
- The only place the Ethereum side is supposedly checked is inside Vottun's closed-source middleware, whose behavior nobody outside Vottun can inspect and which, by Vottun's own account, was compromised.  
  
A bridge that cannot tell a real redemption from a fabricated one has no security beyond the honesty and integrity of its signer. Had the contract required even a minimal proof - a relayed Ethereum event hash checked by an independent party, a second signature from a party that watches the Ethereum chain, or a delay long enough for an external reconciliation job to compare Ethereum deposits against pending Qubic orders - all three orders would have failed, because no Ethereum-side deposit existed for any of them.  
  
### Single manager signature with unilateral release authority  
  
The contract's multisig protects only *who becomes a manager*. Once elected, a manager can complete any pending order with a single signature. There is no threshold of independent signers on the release path and no second party that must confirm a large release. A single compromised manager signature per order was therefore sufficient to drain the entire contract.  
  
### Publicly callable `createOrder`  
  
Because `createOrder` is open to any caller, the attacker did not need to forge a burn event, manipulate the middleware's Ethereum observer, or hold any bridge key to set up the withdrawals. They simply paid the 1% fee from their own address, created an order for the desired amount and destination, and had it completed. Restricting order creation to the middleware would not by itself have fixed the bridge - a compromised middleware could still create orders - but it would have removed one degree of freedom from the attacker and made externally-originated orders immediately distinguishable from legitimate ones.  
  
### No withdrawal limits, delays, or circuit breakers  
  
The contract allowed a single order for ~97% of custodied funds to be created and completed within four seconds. There were no per-order caps, per-epoch volume limits, mandatory delays between `createOrder` and `completeOrder`, or automatic pause triggers on anomalous volume. Any one of these would have limited the loss or given operators time to react.  
  
### No detection across three escalating withdrawals  
  
The attacker did not go straight for the full balance. They ran a 40 million QUBIC probe, waited four and a half minutes, ran a ~10.6 billion QUBIC escalation, waited another minute and a half, and only then took ~308.7 billion. Each step was on-chain and public. Each was an Ethereum=>Qubic completion with no matching deposit on Ethereum - an unambiguous signal. The first two steps together gave a window of roughly six and a half minutes in which any alert on `completeOrder` events, any reconciliation of Qubic releases against Ethereum deposits, or any anomaly rule on outflow volume would have fired. No monitoring on either the operator's side or the community's side flagged anything.  
  
### Closed-source, third-party operated middleware  
  
The off-chain component that holds the manager keys and drives the contract is closed-source and run exclusively by Vottun. The community had no visibility into:  
  
- how the manager keys were generated, stored, or accessed;  
- what infrastructure the middleware ran on and who had access to it;  
- whether the middleware actually verified Ethereum-side deposits before signing `completeOrder`, and how;  
- whether any monitoring or alerting existed.  
  
This lack of transparency also means that, after the fact, the community cannot verify the operator's account of what happened. The private forensic report provided by Vottun does not fill this gap. The security of a bridge holding hundreds of billions of QUBIC depended on operational practices that nobody outside the operator could review.  
  
## Timeline  
  
  
**Before the incident**  
  
- Bridge deployed and operated by Vottun.  
- Manager identities `NXVN...` and `WGHJ...` operate normally, processing routine bridge orders.  
- Attacker withdraws `67,111,161` QUBIC from Bitget to `SQDKI...`. ([tx](https://explorer.qubic.org/network/tx/dxvlyifoyybwcgluwkyrrbpqzhkbsobdozdmwwezzabedsooovgwrqncmhpd)) **August 20, 2026 23:31:02**  
  
**2026-08-20**  
  
- **23:34:44** - `SQDKI...` calls `createOrder` for `40,000,000` QUBIC (Order 785). ([tx](https://explorer.qubic.org/network/tx/yqfuptssrgxyhbceokttvsfcaxyapvqqcwewgaywlbisagokoakqwvabgyab))  
- **23:35:07** - `SQDKI...` calls `transferToContract` on Order 785; rejected, 40M refunded. ([tx](https://explorer.qubic.org/network/tx/annxkrbibghtbhihaevubfiboaofstmmhjnmgsauobpociybxsfcgcddpano))  
- **23:35:09** - Manager `NXVN...` calls `completeOrder` on Order 785. `40,000,000` QUBIC released. No alert fires. ([tx](https://explorer.qubic.org/network/tx/ojjokfafkbrgzfluujuagkpdhmabyahndiqegjjgfgdpcbizuuvwzanfjvqk))  
- **23:39:52** - Tick `75,452,051`. `SQDKI...` calls `createOrder` for `10,571,116,199` QUBIC (Order 786). ([tx](https://explorer.qubic.org/network/tx/hsmvolrzhrqoudfauoiyiesgeptgnayetebjxjjcbafqpipjdctsrtjdwcvl))  
- **23:39:58** - Tick `75,452,063`. Manager `WGHJ...` calls `completeOrder` on Order 786. `10,571,116,199` QUBIC released. No alert fires. ([tx](https://explorer.qubic.org/network/tx/lwjndbtnccqofetuvzouvfojhdndeatmuawzajqcocvcousdfdvxctwccghb))  
- **23:40:12** - `SQDKI...` calls `transferToContract` on Order 786; rejected, refunded. ([tx](https://explorer.qubic.org/network/tx/thbtgajlaqgkqewvskxfzpctnsocwlfpbbfdrhquqcbyaejqfjtkvzggjfhc))  
- **23:41:24** - Tick `75,452,243`. `SQDKI...` calls `createOrder` for `308,708,125,375` QUBIC (Order 787). ([tx](https://explorer.qubic.org/network/tx/uavhnbmezqvzvajhalesvgylhzgdvlwzmurhmvwycabxqtnihpduxinajgfe))  
- **23:41:28** - Tick `75,452,251`. Manager `WGHJ...` calls `completeOrder` on Order 787. `308,708,125,375` QUBIC released. ([tx](https://explorer.qubic.org/network/tx/egdnpdhonoqymbcvfkbgelfjnbhgnfspcnzzwarwlgmwalakzrobnbjaobuf))  
- **Shortly after** - `SQDKI...` calls `transferToContract` on Order 787; rejected, refunded. ([tx](https://explorer.qubic.org/network/tx/vqdiheleucbxsdcwvzggojibibzfvrmneuwrbgbsgbmvwvoniuuftcjfdzyd))  
  
**Following days**  
  
- Community becomes aware of the incident. Vottun informs the community that its middleware was compromised and halts bridge operations.  
- Vottun shares a private forensic report with core developers and a small number of community members. Its contents cannot be independently verified by the community.  
  
**2026-09-17**  
  
- This post-mortem is published.  
  
## Conclusion  
  
Bridges concentrate risk. In this case, hundreds of billions of QUBIC were held under the control of a handful of manager identities, operated by a third party through infrastructure that nobody else could inspect, with no on-chain mechanism to check that a release was backed by a real deposit on Ethereum. When those identities were compromised - by whatever means - the contract did exactly what it was designed to do. The attacker funded a wallet with 67 million QUBIC from an exchange, probed with 40 million, escalated to 10 billion, and nothing noticed; seven minutes after the first order, 319 billion was gone.  
  
The Qubic community did not build or operate the Vottun Bridge, but the community carries the consequences. We are publishing this report so that holders, builders, and operators have an honest account of what happened, what is known, and what remains unknown. We will update this document if Vottun provides evidence that substantiates its account of the compromise, or if there are developments in tracing or recovering the stolen funds.  
  
  
  
***This report is provided for informational purposes only and does not constitute an admission of liability. It is based on on-chain data, the public contract source, and information provided by third parties that the community has not been able to independently verify. Nothing in this report creates, expands, or modifies any legal duty or obligation.***
