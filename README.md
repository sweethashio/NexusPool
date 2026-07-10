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
- **Native Stratum V1 *and* V2.** Encrypted, authenticated Stratum V2 over Noise, served side by side with Stratum V1 from a single engine — not a V1 pool with a translation shim bolted on. Spec-conformant on **both standard and extended** V2 channels.
- **The Glass Ledger.** Every share's journey — *counted → useful → conserved* — is signed by the pool's single authority key and verifiable by anyone, offline, from the ledger alone. We don't ask you to trust us; we hand you the proof.
- **Bitaxe-ready.** Point any Bitaxe (Ultra / Supra / Gamma), NerdAxe / NerdQAxe, or ESP32 miner at the pool with `your-address.worker` — it works out of the box with AxeOS, on V1 or native V2.
- **See your payout before you win.** Payout Preflight rebuilds the exact coinbase that would pay you — byte-for-byte, the same construction used for every real block — so you can verify the destination before a block is ever found.
- **Built for industrial stability.** Watchdog + crash recovery, OOM caps, a durable block journal with resubmit, hardware-aware per-rig vardiff, and a public-profile hardening layer. *A solved block is never lost to anything that isn't mining itself.*

## The honest part

Solo mining is a lottery, and **no pool can change your odds.** Your probability of finding a block is set by the network — `p = 1 / (D · 2³²)` per hash — and it is identical on every honest pool. What NexusPool changes is everything *around* that hash:

> **We don't change your luck. We prove we didn't lose your work.**

0% fee, 100% kept, your own address, and a cryptographic receipt that every hash you sent was counted, turned into useful work, and conserved if it won.

## Connect a miner

Point any Stratum V1/V2 miner at the pool. Your **username is your payout address**, plus an optional worker name. One endpoint serves both protocols on the same port — V2 (Noise-encrypted) and V1 are auto-detected.

**Stratum V1 / AxeOS:**

```
Host / URL:  nexuspool.io          (stratum+tcp://nexuspool.io:3350)
Port:        3350
Username:    <your-bitcoin-address>.<worker>      e.g.  bc1q…z3.bitaxe1
Password:    x
```

**Native Stratum V2** (Noise-encrypted, authenticated). Use the full connection string — it carries the pool's authority key so your client can pin the pool's identity:

```
stratum2+tcp://nexuspool.io:3350/9amd6GUzTaGXASESCa75c9Rx3vWYihRyLUAE3Vrmqwgm3T9jtxN
```

- **One endpoint, both protocols.** V2 and V1 share port `3350` and are auto-detected; standard **and** extended V2 channels are supported.
- **Pin the authority key.** The key embedded in the V2 string is the pool's single signing identity — the same one shown on the dashboard and used for the Glass Ledger. Pin it and every job and receipt is verifiably ours.
- **AxeOS / Bitaxe:** set the host and port above and use your Bitcoin address as the user — that's the whole setup.
- Live pool stats, your workers, and the Glass Ledger are on the dashboard at **[nexuspool.io](https://nexuspool.io)**.

## Verify, don't trust

NexusPool is built so you can check it rather than believe it. Everything below is public and non-custodial:

- **[Glass Ledger](https://nexuspool.io/glass-ledger)** — signed, offline-verifiable proof that your work was *counted → useful → conserved*.
- **[Payout Preflight](https://nexuspool.io/payout-preflight)** — enter your address and see the exact coinbase that would pay you, before you win.
- **[Pool statistics](https://nexuspool.io/pool)** — live pool hashrate, blocks, and top-miner leaderboards. A **Donate** option is in the footer for anyone who wants to support the project.
- **[Status](https://nexuspool.io/status)** — forward-only availability history for each component of the pool.
- **[Changelog](https://nexuspool.io/changelog)** — every release, in plain language.
- **[Terms of Use](https://nexuspool.io/terms)** — NexusPool stated plainly: free, non-custodial software — a communication protocol, not a product, service, or investment; SweetHash never holds your funds or keys; and solo mining is a lottery with no guarantee of finding a block.
- **Your fleet is yours.** Public stats and leaderboards mask addresses — other miners cannot enumerate your workers or your hashrate.

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
| **Core** | C11 | The mining engine — Stratum V1/V2 server, `bitcoind` RPC + ZMQ, GBT → coinbase / merkle / witness, **unconditional** block detection with exact 256-bit target math, anti-stale, hardware-aware per-rig vardiff, durable block journal + resubmit. |
| **Control** | TypeScript | Observability &amp; product — persistence, per-miner stats, rig telemetry, REST API. If it goes down, the Core keeps mining. |
| **Web** | Next.js | The public dashboard, Glass Ledger, and tools at nexuspool.io. |

**Guiding principle:** *a solved block is never lost to anything that isn't mining itself.* The public deployment adds a hardening layer — watchdog + auto-restart, ingress protection, and rate limiting — on top of that guarantee.

## Adoption

NexusPool is listed among the **production pools on the official Stratum V2 site, [stratumprotocol.org](https://stratumprotocol.org/#sv2-adoption)** — its native V1 & V2 support is interoperable with spec-compliant Stratum V2 clients.

## Status

Live on **Bitcoin mainnet** at [nexuspool.io](https://nexuspool.io), currently **v0.6.26.37** — pre-1.0 (v0.6.x), in active hardening. This repository is being opened up progressively — starting with this README.

## License

**MIT.** NexusPool is original, independent work by SweetHash.

---

<div align="center"><sub>Built by <b>SweetHash</b> · <a href="https://nexuspool.io">nexuspool.io</a></sub></div>
