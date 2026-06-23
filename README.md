<div align="center">

# NexusPool

**Mine solo. Keep the whole block. Trust nothing — verify it.**

[![License: MIT](https://img.shields.io/badge/license-MIT-1a7f7a)](https://opensource.org/licenses/MIT)
![Stratum V1 + V2](https://img.shields.io/badge/Stratum-V1%20%2B%20V2-01b2aa)
![Fee 0%](https://img.shields.io/badge/fee-0%25-33e6a4)
![Non-custodial](https://img.shields.io/badge/custody-non--custodial-13b8a6)
![Bitcoin mainnet](https://img.shields.io/badge/network-Bitcoin%20mainnet-f7931a)

A non-custodial **solo &amp; lottery Bitcoin mining pool** by SweetHash — **0% fee**, **100% of the block to the finder**, native **Stratum V1 &amp; V2**, with the **Glass Ledger** cryptographic proof of custody.

**Live on Bitcoin mainnet → [nexuspool.io](https://nexuspool.io)**

</div>

---

## Why NexusPool

- **0% fee, 100% to the finder.** When you solve a block, the entire reward is paid to *your* address by the coinbase. No pool cut, no "donation."
- **Non-custodial by design.** You mine to your own Bitcoin address. The pool never holds, routes, or touches your coins — there is nothing to withdraw and nothing to trust.
- **Native Stratum V1 *and* V2.** Encrypted, authenticated Stratum V2 over Noise, served side by side with Stratum V1 from a single engine — not a V1 pool with a translation shim bolted on.
- **The Glass Ledger.** Every share's journey — *counted → useful → conserved* — is signed by the pool's single authority key and verifiable by anyone, offline, from the ledger alone. We don't ask you to trust us; we hand you the proof.
- **Bitaxe-ready.** Point any Bitaxe (Ultra / Supra / Gamma), NerdAxe / NerdQAxe, or ESP32 miner at the pool with `your-address.worker` — it works out of the box with AxeOS.
- **Built for industrial stability.** Watchdog + crash recovery, OOM caps, a durable block journal with resubmit, intelligent vardiff, and a public-profile hardening layer. *A solved block is never lost to anything that isn't mining itself.*

## The honest part

Solo mining is a lottery, and **no pool can change your odds.** Your probability of finding a block is set by the network — `p = 1 / (D · 2³²)` per hash — and it is identical on every honest pool. What NexusPool changes is everything *around* that hash:

> **We don't change your luck. We prove we didn't lose your work.**

0% fee, 100% kept, your own address, and a cryptographic receipt that every hash you sent was counted, turned into useful work, and conserved if it won.

## Connect a miner

Point any Stratum V1/V2 miner at the pool. Your **username is your payout address**, plus an optional worker name:

```
Host / URL:  nexuspool.io          (stratum+tcp://nexuspool.io:3350)
Port:        3350
Username:    <your-bitcoin-address>.<worker>      e.g.  bc1q…z3.bitaxe1
Password:    x
```

- **One endpoint, both protocols.** Stratum V2 (Noise-encrypted) and Stratum V1 are auto-detected on the same port.
- **AxeOS / Bitaxe:** set the host and port above and use your Bitcoin address as the user — that's the whole setup.
- Live pool stats, your workers, and the Glass Ledger are on the dashboard at **[nexuspool.io](https://nexuspool.io)**.

## The Glass Ledger — proof of custody

Mining pools ask for trust: that your shares were counted, that a found block was really yours, that nothing was skimmed. NexusPool replaces that trust with **signed, append-only receipts**:

- **Counted** — your submitted hashes were recorded.
- **Useful** — they were turned into valid work against the current block template.
- **Conserved** — if you solved a block, it was preserved and submitted, not lost.

Each receipt is signed (BIP340 Schnorr) by the pool's single authority key and is **verifiable offline** from the ledger alone. Pin the authority key shown on the dashboard and audit the whole chain of custody yourself — no trust in the operator required.

## Architecture

NexusPool is a decoupled system with a hard IPC boundary, so its parts fail independently:

| Component | Stack | Role |
|---|---|---|
| **Core** | C11 | The mining engine — Stratum V1/V2 server, `bitcoind` RPC + ZMQ, GBT → coinbase / merkle / witness, **unconditional** block detection with exact 256-bit target math, anti-stale, hardware-aware vardiff, durable block journal + resubmit. |
| **Control** | TypeScript | Observability &amp; product — persistence, per-miner stats, REST API. If it goes down, the Core keeps mining. |
| **Web** | Next.js | The public dashboard and Glass Ledger at nexuspool.io. |

**Guiding principle:** *a solved block is never lost to anything that isn't mining itself.*

## Status

Live on **Bitcoin mainnet** at [nexuspool.io](https://nexuspool.io). Pre-1.0 (v0.6.x), in active hardening. This repository is being opened up progressively — starting with this README.

## License

**MIT.** NexusPool is original, independent work by SweetHash.

---

<div align="center"><sub>Built by <b>SweetHash</b> · <a href="https://nexuspool.io">nexuspool.io</a></sub></div>
