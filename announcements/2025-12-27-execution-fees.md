# 📢 Qubic Ecosystem Update: Execution Fees Are Coming

**Attention Builders & Developers!**

Execution Fees have completed testnet testing and are now in **mainnet testing phase**—with full activation planned for **January 14, 2025**. This brings organic, usage-based burn from smart contracts to the Qubic network.

---

## How Execution Fees Work

Unlike simple fixed fees, Qubic's execution fees are **execution time dependent**—contracts pay based on actual computational resources consumed.

### The Flow:
1. **Execution** — Computors execute your contract's procedures and measure execution time
2. **Accumulation** — Execution times are accumulated over a 676-tick phase
3. **Consensus** — Computors broadcast their measured values via special QuorumTx transactions
4. **Quorum Value** — The ⅔ percentile is computed to reach consensus on the fee
5. **Deduction** — The agreed fee is deducted from the contract's fee reserve

### Phase-Based System
```
Phase n-1          Phase n              Phase n+1
(676 ticks)        (676 ticks)          (676 ticks)
    │                  │                    │
    └── Fees computed ─┘── Fees deducted ───┘
```

Each computor sends their accumulated execution fee value once per phase (676 ticks), making consensus efficient while maintaining accuracy.

---

## Key Details for Builders

| Aspect | Detail |
|--------|--------|
| **Fee Basis** | Actual execution time (not fixed per call) |
| **Multiplication Factor** | Computor-defined via quorum |
| **Functions vs Procedures** | Only procedures are charged; functions are free |
| **Nested Contracts** | Counts toward the calling SC |
| **Empty Reserve** | Contract procedures won't execute if reserve is depleted |

### Reserve Management
- **Initial Reserve** — Funded during contract IPO (locked $QUBIC)
- **Top-ups** — Contracts can burn $QUBIC to add to their reserve
- **QUTIL Enhancement** — New `qpi.burn()` procedure coming to allow burning QU for another contract (donation burns)
- **Infrastructure Support** — SupplyWatcher can burn for infrastructure contracts (e.g. Quorum Proposals, Computor controlled Fund)

---

## 🆕 Contract Reserve Tracker

Monitor your contract's health in real-time:

🔗 **[contracts.qubic.tools](https://contracts.qubic.tools)**

### What You'll See:
- **Current reserve balances** for all active contracts
- **Burns** (+Reserve) — $QUBIC added to contract reserves  
- **Deductions** (-Reserve) — Execution fees per phase
- **Per-contract activity logs** with timestamps

> ⚠️ **Current Status:** Mainnet testing phase. Fee deductions are being **logged but not yet subtracted**. The viewer shows what balance *would be* if fees were active per epoch.

---

## Timeline

| Date | Milestone |
|------|-----------|
| ✅ Oct 29 | Preparation & fixing known issues |
| ✅ Nov 5 | Execution fee aggregation implementation |
| ✅ Nov 19 | Communication (QuorumTx) implementation |
| ✅ Dec 3 | Quorum value computation |
| ✅ Dec 10 | Implementation finalized |
| ✅ Dec 17 | Extensive testing |
| ✅ Dec 17+ | **Running on mainnet without deduction** (current) |
| 🎯 Jan 14 | **Go Live** — Full activation |

---

## What This Means for Your Contracts

### Build Efficiently
Execution time matters now. Procedures that return early save fees. Optimize your code for real savings.

### Plan for Sustainability  
Monitor your reserve via the tracker. If it approaches zero, arrange for top-ups or shareholder fee allocation.

### No User Fees
End users still transact for free. The contract's reserve pays execution costs. A contract may ask users for fees on certain actions.

---

## 🔧 Action Required: Builders

**Your smart contracts will soon pay for execution time. Now is the time to prepare.**

### What to do:
1. **Check current fee estimates** at [contracts.qubic.tools](https://contracts.qubic.tools) — see what your contract would be paying
2. **Review your procedures** — execution time matters; procedures that return early save fees
3. **Optimize if needed** — inefficient code = higher costs from your reserve
4. **Plan reserve sustainability** — ensure your contract has sufficient reserve or a top-up strategy

> 💡 Functions remain free. Only procedures are charged based on execution time.

---

## 🔧 Action Required: Computors

**You control the fee multiplier. Get familiar with this new setting during the testing phase.**

### The Execution Fee Multiplier
Each computor can set their own multiplier that converts raw execution time into fees. The network reaches consensus using the ⅔ percentile of all computor values.

### CLI Commands ([qubic-cli](https://github.com/qubic/qubic-cli)):

**Set your multiplier:**
```bash
-setexecutionfeemultiplier <NUMERATOR> <DENOMINATOR>
```
Sets the multiplier to `NUMERATOR / DENOMINATOR`. Requires valid seed and node ip/port.

**Check current multiplier:**
```bash
-getexecutionfeemultiplier
```
Returns the current multiplier value. Requires valid seed and node ip/port.

### What to do:
1. **Update your qubic-cli** to the latest version
2. **Experiment with different multiplier values** during the testing phase
3. **Monitor the effects** at [contracts.qubic.tools](https://contracts.qubic.tools)
4. **Share your observations** in Discord to help determine optimal defaults

> ⚠️ After testing, the core team will add a default value to the code based on community feedback.

---

## Technical Resources

- **Documentation:** [github.com/qubic/core/blob/main/doc/execution_fees.md](https://github.com/qubic/core/blob/main/doc/execution_fees.md)
- **Reserve Tracker:** [contracts.qubic.tools](https://contracts.qubic.tools)

---
