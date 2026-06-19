---
title: Storage Repair
type: concept
tags: [storage-repair, erasure-coding, storj, data-durability, audits]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3, storj-v2]
---
# Storage Repair

**Definition:** The Satellite-orchestrated maintenance process by which Storj detects erasure-coded segments whose surviving pieces have decayed (via node churn, failure, or disqualification) below a repair threshold, reconstructs the segment, regenerates the missing pieces, and redistributes them to new nodes — restoring durability without any peer-to-peer sync.

## How it works
Each remote segment is split into stripes and Reed-Solomon encoded into `n` pieces stored on distinct nodes; the **pointer** (segment metadata held by the Satellite) records which nodes hold which pieces, the per-piece SHA-256 hashes, the erasure parameters, and the repair threshold. Durability is *monitored* by the audit service: it samples a random stripe, asks the node for that stripe's erasure share, and runs Berlekamp-Welch error correction across the gathered shares to verify correctness. Audit outcomes feed the reputation system; offline nodes enter *containment mode* (audit retried until success/fail). Nodes that fail too many audits, return data too slowly, or stay offline too long are **disqualified** and their pieces are treated as lost.

The repair service (the "data repair checker") scans pointers, counts healthy/available pieces, and queues any segment at or below `m`.

```text
repair_loop(satellite):
  for pointer in metadata_db.segments():
      healthy = count_reachable_undisqualified_pieces(pointer)
      if healthy <= pointer.repair_threshold:          # m
          shares = download_pieces(pointer.healthy_nodes, need=k)  # ingress = k/n of segment
          for stripe in segment:
              data = reed_solomon_decode_BW(shares[stripe])        # Berlekamp-Welch
              verify(data, pointer.piece_hashes)                   # RS alone can't localize corruption
          new_pieces = reed_solomon_encode(data, k, n)             # regenerate up to o
          missing = select(new_pieces, count = o - healthy)
          new_nodes = node_selection(exclude=pointer.nodes,
                                     by=preferential_reputation)
          upload(missing, new_nodes)                               # egress = only the missing pieces
          pointer.update(nodes, piece_hashes); metadata_db.put(pointer)
```

## State & parameters
Four ordered Reed-Solomon numbers `k ≤ m ≤ o ≤ n`:
- **k** — minimum pieces to reconstruct.
- **m** — *minimum safe* / **repair threshold** (`r₀`): repair triggers immediately when available pieces drop ≤ `m`. `m − k` is the safety buffer covering the gap between detection and repair completion.
- **o** — *optimal*: enough pieces for the target durability; uploads/repairs cancel remaining transfers once `o` complete (long-tail mitigation). `o − m` = nodes that can go offline without triggering repair.
- **n** — total pieces generated; `n − o` = long-tail/slow-node tolerance.

**Durability model:** with monthly churn `p`, expected lost pieces λ = `pn` (Poisson). Durability = CDF up to `n − k`: `P(D) = e^(−λ) · Σ_{i=0}^{n−k} λⁱ/i!`. Spreading risk over more nodes raises durability at constant expansion factor — `(20,40)` beats `(10,20)` though both are 2×.

## Why it matters / trade-offs
Repair is the durability engine but is **costly and asymmetric**: it must download `k/n` of the segment (ingress) yet only re-uploads the few missing pieces (egress), so it is bandwidth-, memory-, and CPU-heavy and should be minimized. Storage nodes are paid for repair egress; the Satellite funds repair from withheld escrow and per-segment audit/repair charges. Repair is **centralized** — it only runs while the Satellite runs; if it stops, segments decay with churn. Contrast: Swarm uses peer **pull-sync** anti-entropy (neighbors continuously re-replicate chunks toward their address, no central coordinator); Filecoin has **no automatic repair** — the client must build in redundancy and re-deal expired/faulted sectors itself.

## Related
[[erasure-coding]], [[reed-solomon]], [[audits]], [[data-durability]], [[reputation-system]], [[satellite]], [[storage-node]], [[repair-bandwidth]], [[storj]], [[pull-sync]], [[decentralized-storage]]

## Sources
- [[storj-v3]] — k/m/o/n parameters and thresholds, Poisson durability CDF, audit + Berlekamp-Welch verification, repair pipeline, pointer metadata, reputation/disqualification, repair bandwidth asymmetry and payment.
- [[storj-v2]] — earlier proofs-of-retrievability / Merkle-tree audit approach and per-piece hash validation that v3 simplifies.
