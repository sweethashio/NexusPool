<div align="center">

# NexusPool

**Keep the whole block. See your payout before you win.**

[![Version](https://img.shields.io/github/v/tag/sweethashio/NexusPool?label=version&color=01b2aa)](https://nexuspool.io/changelog)
![Stratum V1 + V2](https://img.shields.io/badge/Stratum-V1%20%2B%20V2-01b2aa)
![Fee 0%](https://img.shields.io/badge/fee-0%25-33e6a4)
![Non-custodial](https://img.shields.io/badge/custody-non--custodial-01b2aa)
![Chains: BTC BCH LTC DOGE](https://img.shields.io/badge/chains-BTC%20%C2%B7%20BCH%20%C2%B7%20LTC%20%2B%20DOGE-33e6a4)

**[nexuspool.io](https://nexuspool.io)** · [Connect a rig](#connect-a-rig) · [Payout Preflight](https://nexuspool.io/payout-preflight) · [Changelog](https://nexuspool.io/changelog)

</div>

NexusPool is a free, non-custodial Bitcoin solo mining pool run by SweetHash. If your rig finds a block, the whole reward pays your address inside the block, and [Payout Preflight](https://nexuspool.io/payout-preflight) shows you that transaction before you win. The pool fee is 0%. You connect with a Bitcoin address instead of an account, over Stratum V1 or native encrypted Stratum V2 on the same port, from one rig or a whole fleet.

The same engine runs two more pools, Bitcoin Cash and Litecoin merged with Dogecoin, each chain in its own process.

## Connect a rig

Your username is your payout address, plus an optional worker name. Each host below serves Stratum V1 and V2 on one port, and the pool tells them apart from the first bytes your rig sends.

### Bitcoin

`solo.nexuspool.io` sends each rig to the closest healthy region: Los Angeles, Chicago, Frankfurt or Singapore.

```text
URL:       stratum+tcp://solo.nexuspool.io:3350
Username:  <your-bitcoin-address>.<worker>     e.g. bc1q…z3.bitaxe1
Password:  x     (or d=NNNN for a fixed difficulty)

Stratum V2 (the string carries the pool's authority key, so your client can pin it):
stratum2+tcp://solo.nexuspool.io:3350/9amd6GUzTaGXASESCa75c9Rx3vWYihRyLUAE3Vrmqwgm3T9jtxN
```

### Bitcoin Cash

```text
URL:       stratum+tcp://bch.nexuspool.io:3351
Username:  <bch-address>.<worker>     CashAddr or legacy
Password:  x     (or d=NNNN for a fixed difficulty)

Stratum V2:
stratum2+tcp://bch.nexuspool.io:3351/9b54ZkLxmK7h38ss9avHDhMkBs8TkLghkakjB5MFrCAzqNby6ra
```

### Litecoin + Dogecoin (merged)

```text
URL:       stratum+tcp://lite.nexuspool.io:3354
Username:  <ltc-address>.<doge-address>.<worker>     both addresses required
Password:  x     (Stratum V1: doge=<doge-address> if the username field is too short)

Stratum V2:
stratum2+tcp://lite.nexuspool.io:3354/9b9yiWg7S5sB4aS7wQ3SLpcbFCmvuFuW6CJyubTdcyoihBcT7Bg
```

While the Dogecoin leg is live, the pool checks each Scrypt share against both chains, so the same work can win a Litecoin block, a Dogecoin block or both. The [Litecoin page](https://nexuspool.io/ltc) shows the leg's current state. Login fails without a Dogecoin address: the pool would have nowhere to pay a Dogecoin block.

On AxeOS, set the host, the port and your address as the user. For a fleet, or a rig rented on Mining Rig Rentals, NiceHash or Braiins Hashpower, [pool settings](https://nexuspool.io/configure) lists what to copy for each.

## See your payout before you win

Enter your Bitcoin address in [Payout Preflight](https://nexuspool.io/payout-preflight). It builds the coinbase the pool would use on the live block, byte for byte, and shows the whole reward paying your address. A real block pays its finder the same way, so the pool holds no balance of yours to lose.

## From hash to block on Bitcoin

The steps below cover the Bitcoin pool as of v0.10.26.5, from a new block on the network to a block of yours reaching it. The [changelog](https://nexuspool.io/changelog) describes each change.

- **New block.** We send your rig work on the new block before our node finishes checking it. That first job carries no transactions, so a block found on it pays the subsidy without fees, and that block waits for the node to accept its parent. As soon as the block lands, the pool rebuilds a job with fees from its previous template, and a full node checks each one.
- **Version bits.** Firmware writes rolled version bits in three ways. The pool reads all three and lets proof of work decide, so the reading that forms a block wins. We checked the three readings against firmware source and tested them on a rig: a Bitaxe Gamma (BM1370, AxeOS 2.14.1) on a private test chain whose job version sets a bit inside the rolling mask. All 143 of its shares resolved, and each block reached Bitcoin Core with the header the rig hashed. For other firmware families we have the source only.
- **Late shares.** A winning share that arrives after the pool replaced its job still gets its block test, over Stratum V1 and V2, and goes to the network if it is a block.
- **Share limits.** The per-connection limit counts only shares that prove no work, so an honest rig can no longer lose a block to its own share rate.
- **Found block.** The pool writes the block to its journal on disk first. A block found through Los Angeles, Chicago or Frankfurt then goes to Bitcoin Core nodes in all three of those cities, and Singapore submits to its own node. When no node answers, the pool keeps retrying until one does, and it submits the block again after a restart. A rig that loses its connection after finding a block can deliver it when it reconnects.

## Glass Ledger: signed receipts for the work we counted

Each hour, for each rig, NexusPool signs a receipt for the shares it counted under that rig's address. The signature is BIP340 Schnorr. The [custody manifest](https://nexuspool.io/.well-known/nexuspool-custody.json) documents the signed bytes and each key the pool has signed with, retired regional keys included, so any BIP340 library can check a receipt. In the Los Angeles region, the pool anchors its new receipts in Bitcoin each hour through OpenTimestamps.

A receipt shows what the pool counted. It can't show a share the pool never recorded, so compare it with your miner's own counter. Read yours on the [Glass Ledger](https://nexuspool.io/glass-ledger).

## Solo mining odds

Each hash your rig computes has the same chance of finding a block, set by network difficulty, and no pool can raise it, NexusPool included. That makes solo mining a lottery, and the expected time to a block is an average: a rig can find one in its first hour or go years without. Bitcoin Cash's lower difficulty shortens the expected wait for the same hashrate without making the work worth more. Merged mining gives Scrypt work a second chain to win on and leaves the odds on each chain unchanged.

## Regions and latency

The Bitcoin pool runs in four regions, each a complete pool with its own Bitcoin node: Los Angeles, Chicago, Frankfurt and Singapore. A rig far from all four sees more round-trip latency. Distance adds to the time a rig keeps hashing on the old block after each new one. It doesn't change the odds of each hash, which network difficulty sets. The [latency check](https://nexuspool.io/latency) times the trip from your own network, and the [status page](https://nexuspool.io/status) shows current availability.

## How it's built

- **Core**, in C: the Stratum V1 and V2 server, the link to each chain's node over RPC and ZMQ, block assembly and the block test.
- **Control**, in TypeScript: miner statistics and the public API. It runs as a separate process from the core.
- **Web**, in Next.js: the dashboards and tools at [nexuspool.io](https://nexuspool.io).

NexusPool appears in the adoption section of [stratumprotocol.org](https://stratumprotocol.org/#sv2-adoption). Its Stratum V2 runs over the Noise handshake, and refused shares carry the standard error names V2 mining software expects.

## Status

The current release of the Bitcoin pool is v0.10.26.5. The [changelog](https://nexuspool.io/changelog) describes each release in plain language, including the bugs it fixes. NexusPool is pre-1.0 and live on mainnet.

## Has NexusPool found a block?

On Bitcoin's test network, yes: [testnet4 block 154008](https://mempool.space/testnet4/block/0000000000000001c3a8678774d2d803df6f1c1d23f2e2e481acd422cfee6a5d), accepted by the network and readable on any explorer. Test coins have no market value. On mainnet, solo mining is a lottery, so no pool can tell you when your rig will hit. We can show you where the reward lands before it does. Check your address in [Payout Preflight](https://nexuspool.io/payout-preflight).

## License and terms

The LICENSE file is MIT. This repository holds no code today: the core's source isn't published, so the repository carries this README, the license and the version number. The pool is free to use and non-custodial. SweetHash never holds your funds or keys, and solo mining carries no guarantee of finding a block. Read the full [terms of use](https://nexuspool.io/terms).

Trust nothing. Verify the coinbase before you mine.

---

<div align="center"><sub>Built by <b>SweetHash</b> · <a href="https://nexuspool.io">nexuspool.io</a></sub></div>
