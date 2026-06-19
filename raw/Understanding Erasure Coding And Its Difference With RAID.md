---
title: Understanding Erasure Coding
author: danielbui12
created: 2026-06-14
description: A technical walkthrough of erasure coding — what it is, the math underneath, the repair problem, how real storage systems tune it, and how it differs from RAID.
tags:
  - object-storage
  - erasure-coding
  - distributed-storage
---

## Overview

This document explains erasure coding from the ground up: starting with what it is and why it matters, moving through the mathematics that make it work, and ending with the hard production problems — repair cost, parameter selection, and failure modes — where most of the active research lives. The later sections also contrast erasure coding with RAID, since the two are often conflated.

If you already know the basics, jump to [Part 3](#part-3--the-hard-problems), where erasure coding gets genuinely interesting.

![Erasure Coding](https://cdn-cnekb.nitrocdn.com/tcxNzLphnaITHzvqTMBrqGOHJtbzDjle/assets/images/optimized/rev-a0baba0/stonefly.com/wp-content/uploads/2018/05/erasure.png)

---

# Part 1 — The Fundamentals

## What erasure coding is

Erasure coding is a way of trading a little extra storage for the ability to lose chunks of your data and still recover everything.

The mental model is best understood against replication. Replication says: *"I don't trust this disk, so I'll keep three whole copies."* Erasure coding says something cleverer: *"I'll chop the data into pieces, compute a few extra pieces that are mathematically derived from the originals, and spread all of them around. As long as I can later collect* enough *pieces — not all, just enough — I can rebuild the original exactly."*

The word *erasure* is the key, and it comes from coding theory. An **erasure** is a symbol you know is missing — a dead disk, an offline node — as opposed to an **error**, which is a symbol that is silently wrong but whose location you don't know. Erasure coding is built for the first case, which happens to be exactly what storage failures look like: when a drive dies, the system knows it's gone. That assumption — *we know which pieces are missing* — is what makes the math dramatically cheaper than full error correction.

## Why it matters

Triple replication is brutally expensive at scale, and erasure coding buys the same durability for far less.

Replicating three times means a 3x storage cost: store 1 PB, pay for 3 PB. In exchange it tolerates two failures. Compare that to a typical erasure code such as a 10-of-14 scheme — ten data pieces plus four parity pieces. That tolerates *four* simultaneous failures and costs only 1.4x. Fault tolerance has doubled while the storage bill has been cut by more than half. At exabyte scale, that difference is the difference between a viable business and an absurd one.

There is always a catch. Replication is dead simple and cheap to *operate*, whereas erasure coding spends CPU on every write and — more painfully — spends network and disk I/O every time it has to repair something. That tension between storage savings and repair cost is the central theme of this document.

## Notation: k, m, and n

Schemes are written as `RS(10,4)` or `k=10, m=4`. This notation is the entire contract of the code:

- **k** — the number of *data* fragments the original object is split into.
- **m** — the number of *parity* (coding) fragments computed on top of them.
- **n = k + m** — the total number of fragments stored, usually one per failure domain (a disk, host, or rack — whatever can fail independently). This full set of `n` fragments for one object is called a **stripe**.

The promise: you can reconstruct the original object from *any* `k` of the `n` fragments, which means you can lose up to **m** fragments and still be fine. In a 10-of-14 scheme you store 14 pieces, any 10 will do, and you survive any 4 failures.

Two derived quantities fall straight out of this:

- **Storage overhead** = `n/k`. For 10+4 that is `14/10 = 1.4x`.
- **Fault tolerance** = `m`. The number of pieces you can lose.

Tune those two knobs and you have described almost any erasure-coded system on earth.

## The mechanism, by example

The simplest erasure code in existence is a single XOR parity — a `(k, 1)` code, and the same trick RAID-5 uses.

Take three data bytes A, B, C. Compute one parity byte `P = A ⊕ B ⊕ C` (bitwise XOR) and store all four on four different disks. Now lose any one of them — say B dies. Because XOR is self-inverting:

```
B = A ⊕ C ⊕ P
```

B is recovered from the three survivors. One parity, one failure tolerated. The limitation is obvious: XOR alone protects against only a *single* loss. The moment two disks die, plain XOR cannot disentangle the two unknowns.

To survive multiple failures you need more independent equations. Borrowing the classic intro example: imagine your secret value is 95, split into `x = 9` and `y = 5`. Generate several equations that all involve x and y:

```
x + y  = 14
x − y  = 4
2x + y = 23
```

Any *two* of those three equations recover x and y — pick the first two and you get `x = 9, y = 5`; pick the last two and you get the same answer. You have stored three "fragments" (the three equations) but need only two to recover. That is a `(2, 1)` code in disguise, and it captures the essential idea:

> **Encode the data into more equations than you strictly need, so that any sufficient subset is enough to solve the system.**

Real codes don't use ordinary arithmetic like this — for reasons covered in Part 2 — but the intuition is exactly right.

## Systematic codes

A code is **systematic** if the original, unmodified data fragments appear verbatim among the `n` stored fragments, with the `m` parity fragments tacked on. A **non-systematic** code scrambles everything, so no stored fragment is plain readable data.

The reason to care is performance on the happy path. In a systematic scheme, when nothing has failed — which is almost always — a read just hands back the data fragments directly: no decoding, no math, no CPU. The decode cost is paid only during the rare repair. Nearly every storage-oriented erasure code (Reed–Solomon as deployed in HDFS, Ceph, and others) is systematic for exactly this reason. Non-systematic codes show up more in streaming and networking contexts, where there is no stable "original" to read back anyway.

## MDS codes and the Singleton bound

**MDS** stands for *Maximum Distance Separable*, and it is the gold standard for storage efficiency.

An MDS code achieves the theoretical best: it can recover from *any* `k` fragments, full stop, with no wasted space. No scheme can tolerate `m` failures with fewer than `m` parity fragments — that limit is the **Singleton bound**, and MDS codes sit exactly on it. Reed–Solomon is the canonical MDS code, which is a large part of why it dominates.

MDS matters in practice because not every erasure code is MDS, and the ones that aren't are making a deliberate trade. Local Reconstruction Codes (covered later) sacrifice strict MDS optimality to win something more valuable in a data center: cheaper repairs. So when a code is described as "near-MDS" or "non-MDS," that usually means it gave up a sliver of storage efficiency to buy back repair bandwidth.

---

# Part 2 — Under the Hood

## Why finite-field arithmetic

Normal integer arithmetic overflows and loses precision, which breaks the guarantee that *any* `k` fragments can recover the data. You need an arithmetic where every operation stays inside a fixed-size box and nothing ever rounds or overflows. That box is a **finite field**, also called a **Galois field**, written `GF(2^w)`.

The problem with ordinary numbers: your data is bytes — values from 0 to 255. Start adding and multiplying them with the equations above and the results blow past 255 immediately. You would need ever-larger integers, yet the whole point of erasure coding is that each fragment is the *same size* as a data piece. Finite-field arithmetic fixes this. In `GF(2^8)`, every element is an 8-bit value, and addition, subtraction, multiplication, and division are all defined so that the result of any operation on two bytes is *still a byte*. The system is closed — nothing escapes.

The concrete details:

- In `GF(2^w)`, addition and subtraction are both just **bitwise XOR** (which is why XOR keeps showing up).
- Multiplication and division are defined modulo an **irreducible polynomial**. In implementations they are usually done with lookup tables (log/antilog tables) or, on modern hardware, with vector instructions.
- Storage systems almost always pick **GF(2^8)**, because a symbol is exactly one byte and the lookup tables are tiny (256 entries).
- The constraint `n ≤ 2^w − 1` means `GF(2^8)` caps you at 255 total fragments — the `−1` is because classic Reed–Solomon needs distinct *nonzero* evaluation points, so the zero element is excluded. That is plenty for storage, where `k+m` rarely exceeds a few dozen. Codes that need more fragments move up to `GF(2^16)`.

## Encoding

Encoding is one matrix multiplication. Once you're inside the field, that is genuinely all it is.

You build a **generator matrix** of size `n × k`. The standard construction uses a **Vandermonde** matrix, or a **Cauchy** matrix — a numerically nicer variant that makes the field multiplications cheaper. The key property is that *every* `k × k` submatrix is invertible. That is the algebraic statement of "any `k` fragments are enough."

Take your `k` data symbols as a column vector and multiply:

```
   [ generator matrix ]     [ d1 ]     [ f1  ]
   [    n × k          ]  ×  [ d2 ]  =  [ ... ]   ← n encoded fragments
   [                   ]     [ .. ]     [ fn  ]
                            [ dk ]
```

If you arrange the generator matrix so its top `k` rows form the identity matrix, the first `k` outputs come back as your original data untouched — and you have built a *systematic* code. The bottom `m` rows do the actual mixing and produce the parity fragments. Every fragment `fi` is just a weighted XOR of all the data symbols, with weights drawn from the field. Repeat this across the whole object, symbol position by symbol position.

## Recovery

Recovery exploits the "any `k × k` submatrix is invertible" property, turning reconstruction into solving a linear system.

Some fragments died, but you have collected `k` surviving ones. Each survivor corresponds to one row of the original generator matrix — and you know exactly which rows, because you know which fragments you hold. The procedure:

1. Stack the `k` rows belonging to your surviving fragments into a `k × k` submatrix.
2. Invert it (over the Galois field). It is guaranteed invertible by construction.
3. Multiply that inverse by the vector of surviving fragment values.
4. Out come the original `k` data symbols.

This is the same "two equations, two unknowns" move from the `95` example, done with field arithmetic and a matrix instead of by hand. The cost is dominated by the inversion plus the matrix-vector multiply, which is why repair is more expensive than a plain read.

## Performance and where it hurts

Erasure coding costs you in three places: encode CPU, decode/repair CPU, and — the big one — repair I/O. The first two are mostly solved; the third is the open wound.

**CPU.** Finite-field multiplication used to be the bottleneck, but modern libraries lean on SIMD vector instructions (Intel SSSE3/AVX2, ARM NEON) to do field multiplies on whole vectors of bytes at once. A well-tuned Reed–Solomon library on current hardware encodes at several GB/s per core, so for archival and object-storage workloads the encode cost is basically noise. Jerasure, Intel's ISA-L, and Backblaze's open-source Java library are the usual reference implementations, and the gap between a naïve table-lookup version and a SIMD-accelerated one can be an order of magnitude.

**Repair I/O.** What you cannot optimize away is the I/O profile of repair, and that is why erasure coding has a reputation for being unsuitable for hot, latency-sensitive primary workloads. A **degraded read** — a read of an object whose data fragment is currently missing — can't just hand back the bytes; it must pull `k` fragments and decode on the fly, which adds latency, and repair traffic (next section) can saturate the network. The honest summary: cheap on space, cheap on CPU now, but it makes failures more expensive to recover from than replication does — and that single drawback shapes everything in Part 3.

It therefore shines for the opposite of hot workloads: archival storage, backups, object stores, and cold tiers — anywhere data is written once, read rarely, and kept for a long time.

---

# Part 3 — The Hard Problems

*(and where the research lives)*

## The repair problem

Here is the single most important practical fact about classical erasure coding:

> **To rebuild one lost fragment, you have to read `k` fragments.**

Consider what that means. In a 10+4 scheme, a single disk dies — one fragment, maybe a few hundred GB. To regenerate that one fragment, the system must read **ten** other fragments, pull them across the network to one node, decode, and write the replacement. Repairing one unit of lost data moves *ten* units of data over the network. With replication, repairing a lost copy means reading exactly one other copy — a 1:1 ratio. Erasure coding turns that into 10:1.

At data-center scale this is a genuine operational problem. Disk failures are not rare when you run hundreds of thousands of drives; they are a constant background hum. If every failure triggers a `k`-fold read amplification, repair traffic competes with user traffic for network bandwidth, and rebuild times stretch out — which in turn lengthens the window of vulnerability to a second failure. This is *the* reason the field didn't stop at Reed–Solomon. Two big ideas grew out of trying to fix it: **locality** and **regeneration**.

## Local Reconstruction Codes (LRC)

LRCs add a layer of **local** parity so that the common case — a single failure — can be repaired by reading only a handful of nearby fragments instead of all `k`.

The construction: on top of the usual global parities, partition the data fragments into small groups and compute one local parity per group. When a single fragment dies — the overwhelming majority of real failures — you don't touch the whole stripe. You read just the other members of its local group plus that group's local parity, and reconstruct from those. Microsoft's deployed LRC in Azure, for example, is often described as a `(12, 2, 2)` layout: 12 data fragments split into two local groups of six (each with a local parity) plus two global parities — 16 fragments total at a 1.33x overhead. A single-fragment repair reads roughly 6 fragments instead of 12, and the code still tolerates **any 3 simultaneous failures** (and most 4-failure patterns) — comparable durability to a 12+4 Reed–Solomon code, but with half the common-case repair traffic.

The trade is explicit. LRC is *not* strictly MDS — you spend a little extra parity space (those local parities) that a pure Reed–Solomon code wouldn't need for the same worst-case fault tolerance. In return you slash the repair read amplification for the common case, which is what actually dominates operational cost. Microsoft Research introduced this around 2012; it now runs in Azure Storage and influenced erasure coding in HDFS and Ceph. The idea proved important enough that "how much repair locality can you buy for how little storage" became its own research subfield (**Locally Repairable Codes**).

## Regenerating codes

Where LRC reduces *how many* fragments you read, regenerating codes reduce *how much you download from each*. They approach the repair problem through network coding theory.

The insight, due to Dimakis and collaborators around 2010, is that a repairing node does not need the *full content* of `k` helper fragments — only enough information to reconstruct the one missing fragment. So instead of downloading whole fragments from `k` helpers, the new node contacts `d` helper nodes (where `d` can be larger than `k`) and downloads a small *function* — β symbols — from each. The helpers do a little local computation and send a compressed contribution. The total bytes pulled across the network for a repair can be dramatically lower than the full-object download classical codes demand.

There is a fundamental tradeoff curve here, and regenerating codes are usually described by which end of it they sit on:

- **MSR — Minimum Storage Regenerating.** Keeps per-node storage as small as MDS (storage overhead stays optimal, like Reed–Solomon) and minimizes repair bandwidth *subject to* that storage constraint.
- **MBR — Minimum Bandwidth Regenerating.** Pushes repair bandwidth to the absolute theoretical floor, accepting slightly more storage per node in exchange.

You cannot minimize both storage and repair bandwidth at once — that is the entire point of the tradeoff curve. A system picks an operating point based on whether storage cost or network cost hurts more for its workload. Regenerating codes were long considered more of a research technology than an everywhere-default, but MSR constructions have since crossed into production: **Clay codes** (an MSR design by Vajha et al.) ship in Ceph as a usable erasure-code plugin from the Nautilus release (2019) onward, delivering MDS-optimal storage with substantially reduced repair bandwidth. They have deeply influenced how people think about repair, and they share intellectual DNA with LRC's locality goal.

## Choosing k and m

Picking parameters means balancing four things: durability, storage cost, repair burden, and how many failure domains you even have. There is no universal answer, but the reasoning is concrete:

- **`m` sets fault tolerance.** It is literally the number of fragments — and therefore failure domains — you can lose at once. Want to survive a whole rack going down? Then `m` and your fragment placement must account for that. More `m`, more safety.
- **`n/k` sets the storage bill.** A 4+2 profile costs 1.5x. A 10+4 costs 1.4x. A 17+3 gets you down near 1.18x. Bigger `k` relative to `m` means better space efficiency.
- **Bigger `k` means more expensive repairs.** Recall the `k`-fold read amplification — a 17+3 scheme must read 17 fragments to fix one. Aggressive space efficiency directly worsens repair I/O. This is the tension you manage.
- **You need at least `k+m` independent failure domains** — realistically more — for placement to make sense. A 10+4 code needs 14 separate fault domains before it is even legal.

In practice this produces a couple of well-worn defaults:

- **Small clusters / resilience-first:** something like **4+2** — only six failure domains needed, survives two losses, 1.5x overhead, cheap repairs. 4+2 is a common resilience-first starting point in Ceph (its built-in default profile is actually the smaller 2+2, which most deployments override).
- **Large cost-sensitive deployments:** push toward **10+4** or wider, accepting heavier repairs for the storage savings.

The meta-point: pick `m` for the failure scenarios you genuinely need to survive, then make `k` as large as your repair-bandwidth budget and failure-domain count will tolerate.

## What erasure coding does *not* protect against

This is the part that gets glossed over, and where real incidents come from. Erasure coding protects against *known loss* of fragments. It is not a substitute for several things people assume it covers:

- **It's not backup.** Erasure coding protects against hardware failure within a system. It does nothing about a fat-fingered `rm`, a buggy application overwriting good data, ransomware, or a bad deploy that corrupts records. Those operations are dutifully encoded and spread across all your fragments. You need real backups and versioning for that, full stop.
- **It doesn't fix silent corruption by itself.** Classical erasure coding handles *erasures* (known-missing fragments), not *errors* (silently-wrong fragments). If a fragment goes bad without the system noticing — bit rot, a lying drive — and you naively decode using it, you get garbage out. Production systems pair erasure coding with **checksums and scrubbing** so corruption is *detected* and the bad fragment converted into a known erasure, which the code can then handle. Without that detection layer, the code can be fooled.
- **It widens the failure-correlation footprint.** Because the original data no longer lives intact on any single device, a read of a degraded object touches many nodes. Correlated failures — a whole rack losing power, a top-of-rack switch dying — can take out multiple fragments at once, which is why fragment *placement* across independent failure domains matters as much as the code parameters.
- **It adds latency and CPU to the degraded read path.** Worth repeating as a "don't get burned": erasure coding is a poor fit for low-latency primary/hot workloads. Its home is archival, object, and cold storage where the space savings dominate and reads tolerate the occasional decode.

If you remember one sentence: **erasure coding is a *durability* mechanism against component failure, not a *recovery* mechanism against human error or corruption — and it leans on checksums to even handle corruption at all.**

---

# Part 4 — Erasure Coding vs. RAID

RAID and erasure coding are frequently conflated, and for a good reason: classic RAID parity *is* a form of erasure coding. RAID-5 is a single-XOR-parity `(k, 1)` code, and RAID-6 is a two-parity `(k, 2)` code, typically built on Reed–Solomon math. The difference is less about the algorithm and more about *scope, placement, and operational model*.

| Dimension | Traditional RAID | Distributed erasure coding |
|---|---|---|
| **Scope** | A set of disks inside one array/controller | Fragments spread across many nodes, racks, or datacenters |
| **Failure domain** | Individual disks | Independent domains (disk, host, rack, region) |
| **Parity scheme** | Fixed levels: RAID-5 (1 parity), RAID-6 (2 parity) | Arbitrary, tunable `k+m` profiles |
| **Fault tolerance** | 1 (RAID-5) or 2 (RAID-6) disks | Any `m`, configurable per pool/object |
| **Rebuild scope** | Rebuild the whole disk onto a hot spare | Regenerate only the lost fragments, anywhere in the cluster |
| **Correlated failure** | Vulnerable to controller / enclosure loss | Designed to survive whole-rack or whole-site loss |
| **Typical home** | Single server, NAS, storage appliance | Object stores, distributed file systems, cloud storage |

The key conceptual differences:

- **RAID is fixed; erasure coding is parameterized.** RAID-6 always gives you exactly two parity disks. An erasure-coded pool lets you dial `k` and `m` independently to hit a chosen durability and storage-overhead target — `4+2`, `10+4`, `17+3`, and so on.
- **RAID protects a box; erasure coding protects a cluster.** RAID's failure domain is the disk inside one chassis, so a dead controller or a fried enclosure can still take everything down. Distributed erasure coding places each fragment in an *independent* failure domain, which is what lets it survive an entire rack or datacenter outage — something no single-array RAID level can do.
- **Rebuild behavior differs.** When a RAID disk dies, the array rebuilds the *entire* replacement disk onto a hot spare, and the array typically runs degraded (and slower) until that finishes. A distributed erasure-coded system regenerates only the specific lost fragments and can parallelize the work across many surviving nodes — though it pays the `k`-fold repair read amplification discussed in Part 3.
- **RAID write hole vs. distributed consistency.** RAID has the classic "write hole" (a power loss between writing data and its parity can corrupt the stripe), historically patched with battery-backed caches or journaling. Distributed systems instead solve consistency at the cluster level with transactions, versioning, and quorum protocols.

The shortest way to put it: **RAID is erasure coding confined to one box with a fixed parity recipe; modern distributed erasure coding generalizes the same math to arbitrary `k+m` across independent failure domains.**

---

## Quick reference — the codes by name

For the algorithm-shopping version, here is the landscape the sections above touched on:

- **Reed–Solomon** — the workhorse. MDS, recovers from multiple losses; used in storage, RAID, CDs/DVDs, QR codes, and deep-space links. When someone says "erasure coding" with no qualifier, this is usually what they mean.
- **Cauchy Reed–Solomon** — an RS variant built on a Cauchy matrix instead of Vandermonde, restructured so the field multiplications are cheaper. Same guarantees, faster in software.
- **XOR / simple parity** — the `(k, 1)` building block; survives a single loss; what RAID-5 is underneath.
- **Local Reconstruction Codes (LRC)** — RS plus local parities for cheap single-failure repair. Trades strict MDS optimality for low repair I/O. Microsoft Research (~2012); in Azure, HDFS, Ceph.
- **Regenerating Codes** — minimize *repair bandwidth* via network coding. Two operating points: **MSR** (minimal storage) and **MBR** (minimal bandwidth). Dimakis et al., 2010. The MSR family reached production as **Clay codes** in Ceph (Nautilus, 2019).
- **Low-Density Parity-Check (LDPC)** — sparse parity-check matrices; huge in communications, used in some storage.
- **Luby Transform (LT)** and **Raptor** — fountain codes that emit an unbounded stream of symbols; built for lossy networks where you reconstruct from "enough, whichever arrived." Raptor adds a pre-code for lower overhead and faster decoding.

## Where erasure coding runs in the wild

Pretty much everywhere data is stored at scale, even when it's invisible to you:

- **Object and cloud storage** — the flagship use case. Azure Storage (running LRC, publicly documented), Google's storage layer, and on-prem object stores all use erasure coding under the hood to hit their durability numbers cheaply. AWS S3 is widely believed to do the same, though Amazon has never published the internal scheme.
- **Distributed file systems** — HDFS added native erasure coding (Reed–Solomon, plus support for the LRC family) specifically to cut the 3x replication cost on cold data. Ceph exposes erasure-coded pools as a first-class option with tunable `k/m` profiles.
- **Archival and backup systems**, tape replacements, and cold tiers — write-once-read-rarely data where storage cost dominates. The sweet spot for the technology.
- **Beyond storage entirely** — the same Reed–Solomon math protects QR codes, CDs/DVDs/Blu-ray, deep-space communication (the Voyager probes used RS as an outer code concatenated with a convolutional inner code), and DSL/data transmission. The fountain-code branch (LT and Raptor) was built for *unreliable networks*, where you generate a limitless stream of encoded symbols and the receiver reconstructs as soon as it has collected enough, no matter which ones got dropped.

The common thread across all of it is the one idea we started with: **spend a controlled amount of redundancy so that *enough* surviving pieces always beats needing *every* piece.**
