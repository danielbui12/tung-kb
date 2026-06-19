Efficient generation and dispersion of entropy during data preparation
for Arweave.

Digital History Association (DHA) team

Weavemail: vLRHFqCw1uHu75xqB4fCDW-QxpkpJxBtFD9g4QYUbfw

Draft-5
December 2024

Abstract

This paper introduces a major improvement to Arweave’s
data preparation algorithm, reducing upfront mining
computation costs by significant margins for honest par-
ticipants while preserving the network’s existing security
guarantees. We analyze the security framework of Ar-
weave’s existing data preparation scheme, where the op-
portunity for optimization arises, and outline the design
of this new approach. By optimizing entropy generation
and dispersion in the data preparation process, the pro-
posed algorithm incentivizes wider-participation and the
development of a more efficient storage market in the
network.

1 Background

Arweave is a decentralized and permissionless permanent
data storage network, backed by an endowment. Addi-
tional context on Arweave, its purpose, and the architec-
ture of its core protocol can be found in its paper [18].

Arweave 2.8.x and prior versions of the network utilize
a data ‘packing’ scheme that requires miners to perform
work in order to incentivize them to make and store many
replicas of the dataset, rather than one ‘unpacked’ copy
mined across many identities. In order for this system to
function correctly, the following invariant must hold:

ms(dsz, t) < mp(dsz)

where,

ms(n, t) = Cost to a miner m to store n bytes for t.

mp(n) = Cost to a miner m to pack n bytes.

t = Mean time between access of given bytes.

dsz = The number of bytes in a unit of data.

If this invariant holds, it is always preferable for miners
to store the data in many packed copies, rather than to
mine while packing ‘on-demand’ (o). However, satisfy-
ing this invariant places a compute burden on miners in

the network that are executing the ‘honest’ strategy (h).
Where possible, the design goal of Arweave’s proof sys-
tem is to minimize the requirements (such as these) on
miners beyond the simple provisioning of storage capac-
ity, as codified in the immutable principles of the network
[12].

While preparing their data d for storage, h must per-
form mp work upon each individual piece of data that
they would like to store using the RandomX CPU-bound
work algorithm [14] developed by the Monero commu-
nity. This unit of work produces an equivalently sized
series of bytes de that are sufficiently entropic [13, 15]
that they cannot be effectively compressed. Assuming
the on-demand packing invariant, after production of de
the rational strategy for a miner is to mask it against
d to produce the necessary dp that is utilized in proof
generation, as

mxms(dp) < (ms(du) + mxms(de))

where,

mx = Number of replications of the data to mine.
du = Unpacked form of d.
dp = du packed; masked against du with e(de, du).

At any point in time during mining, each packed ’parti-
tion’ (a 3.6 TB section of the network’s data) has a ’read-
head’. The position of this read-head is pseudo-randomly
determined by the VDF [6, 3] of the network, mixed with
entropy from the miner’s packing address and the parti-
tion number. Once the read-head (and subsequently, the
miner’s underlying storage medium) has ’seeked’ to the
new position in a partition, a number (r) of dp are read
from the dataset. Because the position inside a partition
is chosen in a way that miners cannot predict, o would
either have to recalculate rdp in real-time as their data
is sampled, or store a cache of their packed data chunks.
The caching strategy is only effective proportionate to
the quantity of the stored chunks, resolving to simply
storing dp as h does in extremis.

1

One seek of the read-head for a miner results in the
release of r ‘challenges’ that may satisfy the network’s
global difficulty nd (granting the right to generate a
block, as per Bitcoin [8] and similar Proof-of-Work net-
works). Each r requires the reading of (and consequently,
access to) one unit of data. This imparts a consistent
work requirement of rmp upon o in order to execute
their strategy. Arweave 2.8 introduced the ability for a
miner to optionally perform additional work during pack-
ing in order to lower their data read-speed requirement
(r). This mechanism enforces a custom difficulty (md)
associated with their packed data, granting them a mod-
ified nd that can be modeled in the formula. For the sake
of comprehensibility, this equation omits modifiers to nd
that relate to mechanisms outside the scope of this paper.

P(V alid(r, md)) =

nd
md

where,

V alid(r, md) : One of m’s proofs from r satisfies nd.

The network’s global challenge rate nr is reduced for

m in accordance with md:

mr =

nr
md

Subsequently, m is able to read from their disk at rate
mr, while their likelihood of finding a valid work proof
remains unchanged:

P(V alid(r, md=1)) = P(V alid(r, md=x)) ∀x ∈ [2..ndm]

where ndm is the maximum admissible packing diffi-
culty in the network (32, as of Arweave 2.8). This limit
is necessary as md=n mining a block requires all network
verifiers to perform pd=n at nmp cost, bared by the val-
idator not block producer.

as de in e(de, du), yielding dp. Notably, this mechanism
discards almost all of the sufficiently entropic data that
it produces, approximately ≈ 99.6%). This observation
gives rise to the opportunity for generating significant
efficiencies.

A simple but flawed solution to the inefficient use of
this entropy would be to allow miners to use the full
scratchpad output as de to mask against their unpacked
data in e consecutively. However, this mechanism would
mean that o would be required to compute only 1
256 rmp,
as the de for r1 would be reusable for all r2..256 of the
subsequent challenges. As a consequence, the on-demand
packing invariant would break and o would become the
dominant strategy for mining, rather than h.

Additionally, we note that in Arweave 2.8 h must per-
form mxmdmp(dsz) work in order to pack their replicas,
while o performs only trmxmp calculations (where t is
the duration of their mining period). Over time, the to-
tal units of mp for o is likely to exceed that of h, however
o’s requirement to perform mp in real-time gives rise to
the potential to differentiate and discourage the strategy
in ways that do not affect h.

3 The data activation algorithm

Building on the insights above, we propose a data prepa-
ration scheme that captures the described efficiencies, for
deployment in the next proposed version of the proto-
col (Arweave 2.9). We call this mechanism ‘activation’
rather than packing, as while it serves the same security
purposes for the network as traditional packing, it re-
moves virtually all noticeable computation requirements
for honest miners (approximately 99.3% less than Ar-
weave 2.8’s pd=1 and 99.95% less than pd=16, depending
upon parameters). The algorithm introduces the follow-
ing parameters:

2 Potential efficiencies

In the ideal case, the Arweave data preparation scheme
would minimize honest miner compute costs (hp), while
maximizing the costs for a miner executing the on-
demand strategy op. While the on-demand mining in-
variant referenced above and in the Arweave paper re-
mains a valid constraint, this specification describes an
algorithm that increases op while maintaining existing hp
levels. The effect of this algorithm is a significant reduc-
tion in the overhead (non-ms) costs for honest miners,
while also allowing the network to purchase storage from
mh’s at a more efficient price.

In Arweave 2.8, pd=n is performed by running Ran-
domX using the standard 2 MiB scratchpad n times se-
quentially, mixing the scratchpad entropy with an AES-
256 Feistel cipher [7] n times during execution. The first
8 KiB of the resulting scratchpad is then taken and used

al = Parallel ‘lanes’ of computation.
am = Number of entropy ‘mixes’ between lanes.
az = Number of zones in a single partition.
ad = Effective difficulty, implying mr.
agm(x) = Generator function for mixing data x.

Proposals for these parameters, as well as constants

utilized by RandomX itself, are described in section 4.

At a high level, the data preparation algorithm exe-

cutes as follows:

1. The network’s data partitions are sliced into az ap-

proximately equally sized zones.

2. For z = 1..az:

(a) Miners perform al operations of pd=1, using
the offset of the data they are attempting to
prepare (as well as their packing key, etc) as

2

rather than sequentially, we additionally avoid the prob-
lem outlined by the na¨ıve algorithm described in sec-
In this scheme, o would be forced to perform
tion 2:
the work required to generate the entropy used by h in
many places, only to discard it (wasting the associated
mp work), or store it – just like h. This improvement
|Rpad|
8KiB decrease in the total num-
alone yields a pd
ber of individual RandomX instructions that h must ex-
ecute in order to pack a replica of the network’s dataset.
In concrete terms, using equivalent parameters from Ar-
weave 2.8 for a md=1 miner this mechanism makes the
process of packing 256 times more efficient for h, without
affecting the hp : op ratio. For a pd=32, this algorithm
(with the parameters proposed in section 4) 8,192 times
more efficient.

dsz
8KiB

Depending upon parameters, this algorithm also grants
significant benefits to honest miners in the form of highly
reduced read speed requirements. In Arweave 2.7.x, min-
ers needed to read from their drives at r of 200 MiB/s
(approximately 800 chunks per second). This was due to
the need to for potential o’s to be forced to do enough
work such that the hp : op ratio stayed within safe mar-
gins. In practice, this placed a very significant strain on
miners with many replicas of the network’s dataset due
to the volume of data that needed to be transferred be-
tween disks and main memory in order for proofs to be
computed. Arweave 2.8 improved upon this by allowing
miners to trade-off additional upfront packing work in ex-
change for reduced r (and effective challenge difficulty)
during mining.

The proposed algorithm in this paper takes the im-
provements from Arweave 2.8 a significant step further:
By forcing miners attempting the o strategy to perform
the work required for h to pack an extremely large num-
ber of chunks, r can safely be reduced globally. For ex-
ample, using parameters al = 4 and ad = 3 would safely
reduce r globally to approximately 5 MiB/s – 20 chunks
of data, while simultaneously keeping the hp costs for o
the same. These parameters would grant h an extremely
significant reduction in practical mining costs without
the overhead of packing their data to higher difficulties,
as Arweave 2.8 made possible.

3.2 RandomXSquared

Arweave 2.9 utilizes a new algorithm (RandomXSquared,
shown in 1) for its entropy generation scheme, con-
structed on-top of RandomX.

RandomXSquared is designed to meet the following ob-

jectives:

1. Efficient generation of entropy: The algorithm
must maximize the amount of sufficiently random
data that it produces for a given quantity of work.
Notably, we do not see cryptographically secure en-
tropy generation as a necessary objective. Data re-
sulting from the algorithm must be random enough

Figure 1: RandomXSquared uses a ‘square’ of RandomX
invocations in order to generate and mix entropy across
a scratchpad of increased size.

their seed entropy. Each operation (’lane’) uses
a unique nonce, generating a unique 2 MiB
scratchpad with the same non-compressibility
properties of Arweave 2.8.

(b) am times during the execution of RandomX
upon each scratchpad computation is paused
and entropy is mixed across all al using agm
across a concatenation of the scratchpad states
as[1..al] as input.

(c) After the final RandomX executions have fin-
ished,
the mix generator agm is executed
upon the concatenated scratchpad an addi-
tional time.

(d) The resulting combined result of all lanes is

treated as a single, large de of size al ∗ 2MiB.
(e) Rather than using only the first 8 KiB (as in
al∗2MiB chunks of

Arweave 2.8), de is split into 8KiB
entropy de[1..ac].

(f) dp is calculated for the appropriate subchunk

in each zone as e(de[n], du).

This algorithm is composed of two core elements: the
data packing loop which merges network data and gener-
ated entropy, as well as the algorithm for generating the
entropy itself. We discuss each in turn in the following
sections.

3.1 Efficient

distribution

of

compressible
dataset

entropy across

non-
the

As shown in listing 1, the Arweave 2.9 data prepara-
tion scheme uses the full entropy generated by the Ran-
domXSquared algorithm, rather than just the first 8 KiB
of the final scratchpad in the execution chain. By dis-
tributing the created entropy across an entire partition,

3

Algorithm 1 Data enciphering and entropy distribution
across partitions, using RandomXSquared.
1: RX2pad ← 2 MiB
2: RX2progs ← 2048
am
3: zsz ← |RX2pad|×al
csz
4: az ← ceil( Psz
zsz
5:
6: for z ← 0 to az − 1 do
7:

)

s ← SHA256(maddr ∥ Pn ∥ z)
ze ← RandomXSquared(s)
for c ← 0 to zsz − 1 do
ci ← pnpsz + czsz + z
if du ← Read(ci) is not null then

dp ← e(ze[c], du)
Write(ci, dp)

8:
9:
10:
11:
12:
13:

end if

14:
15:
16: end for

end for

the a would be o cannot predict any significant
strings of the algorithms outputs without first per-
forming the intended work.

2. Random-access resistance: The resulting en-
tropy produced by the scheme must be created in a
single, non-divisible unit of work, such that a would-
be o miner cannot truncate any significant part of the
work in order to access only a subset of the resulting
bytes.

3. Parameterizable CPU-bound workload: In or-
der to make the Arweave network’s on-demand
safety margin (the ratio of hs : op) secure, the al-
gorithm must enforce a workload with predictable
execution speeds with low volatility over time. A
workload tailored to execution upon general-purpose
CPUs appears to be the best route to this object, as
general-purpose computation is the form that has al-
ready experienced the highest incentive to maximize
efficiency. Subsequently, work of this kind is less
likely to experience extreme declines in its execution
time than less well-optimized forms workloads.

Before describing RandomXSquared in-depth, we will
first explore the design and properties of RandomX itself,
as well as its suitability as a primitive for CPU-bound
entropy generation.

3.2.1 RandomX

RandomX is a hashing algorithm designed for use in
proof-of-work contexts, rather than in the generation of
cryptographically secure commitments [14]. Its core de-
sign goal is the construction of a work mechanism that
could not be feasibly improved with the use of custom-
built ASICs, such that the Monero network’s miners

4

could operate competitively using only commodity, non-
specialized hardware. A RandomX hash is generated by
first creating a ‘scratchpad’ of approximately L3 CPU-
cache size and initial register states, seeded with entropy
created by a BLAKE-2b [2] hash of its inputs. This
BLAKE-2b hash is extended using an AES1R-based gen-
erator [1]. Additionally, a large dataset that serves as
a ‘ROM’, optimized for random access, is created from
an intermediate Argon2 [4] derived cache. While Ar-
gon2 itself does meet some of our design requirements,
as noted by Boneh et al in [5], it does not enforce suffi-
ciently strong guarantees against work-truncation attacks
to make it random-access resistant. Additionally, it is op-
timized for memory-bound work in a similar fashion to
Cuckoo Cycle [17], which experienced significant execu-
tion speed volatility when GPU-based implementations
were created.

After initialization, RandomX runs pseudo-randomly
generated programs based on its input entropy. While the
output of a RandomX execution is always deterministic,
the algorithm is designed such that it is not to be pre-
dictable without expending a specific (tunable) amount of
CPU work. In order to achieve this, each RandomX pro-
gram execution involves parts of its VM’s register states
being periodically written into the scratchpad in a man-
ner that cannot can only be calculated via execution of
the programs themselves. Subsequently, over many itera-
tions a RandomX scratchpad becomes increasingly mixed
and cannot be reproduced without re-execution of the
CPU instructions, or caching of memory writes (ISTORE
instructions). Finally, after many iterations (2048 by de-
fault) of a number of unique RandomX programs, a fin-
gerprint of the scratchpad is generated and combined in
another BLAKE-2b hash with the register states. This
hash represents the result that is given to the caller.

3.2.2 ISTORE caching attacks

While ISTORE caching attack vectors are not a relevant
concern for RandomX’s original use case (Monero proof-
of-work mining), without additional measures they could
pose a significant risk to alternative uses of RandomX’s
work-bound entropy generation. An example attack flow
is as follows:

1. Eva initializes a bitmap of size |RXpad|
RXword

bytes. Us-
ing the standard RandomX parameters, the result-
ing bitmap would be 32.7kb for a 2 MiB scratchpad.

2. Eva augments the RandomX virtual machine’s im-
plementation of the ISTORE instruction to addition-
ally set the corresponding bit for the memory cell in
the bitmap to ‘1’.

3. The full RandomX hash is executed.

4. A memorized representation of the scratchpad en-
tropy is constructed by storing the bitmap and a

concatenation of each of the final values for the data
at the associated memory locations.

5. When Eva needs to access the entropy again, she
simply executes the fast BLAKE2b and AES1R gen-
erator calls as normal to initialize the scratchpad,
but instead of executing the programs she copies and
replaces the data at the appropriate slots using her
bitmap and cache.

The RandomX parameter tuning documentation [16]
encourages users not to choose parameters that break
the following invariant:

(128 + |RXprog|RXistores)(RXprogsRXits) ≥ |RXpad|

With the default RXistores and RXprog sz parame-
ters, this equation simplifies to RXtotal istores ≥ |RXpad|.
That is; there should be at least as many ISTORE oper-
ations across the totality of instructions executed in the
generation of a hash as there are cells in the scratchpad.
The design documentation [15] also notes that this leads
to 66% of the scratchpad being overwritten. In the con-
text of our hypothetical attack, Eva would be able to
compress an average scratchpad by 32.5% as her bitmap
is only 1.5% of the size of the entropy.

Due to the birthday paradox and associated results
[11], attempting to rectify this issue by simply increas-
ing the total number of RandomX instructions that are
executed (or, as the algorithm allows for, increasing the
frequency of ISTORE operations relative to others in the
instruction set) has only limited impact. Multiplying the
total amount of work performed in the entropy generation
process would still leave Eva with the ability to discard
12.5% of the entropy.

As noted, this attack vector is not of significant con-
sequence to Monero, or to Arweave 2.8. Arweave 2.8
utilizes only the first 8 KiB of the scratchpad which re-
ceives significantly higher numbers of writes due to Ran-
domX’s L1-L3 cache utilization parameters. This attack
would, however, make the proposed Arweave 2.9 data
preparation scheme vulnerable to ‘packing compression’
(a mixture of on-demand mining behavior and RandomX
ISTORE cachine behaviors) if left unaddressed.

3.3

ISTORE caching resilience with Ran-
domXSquared

In order to alleviate the issues with using only a
‘stock’ deployment of RandomX as an entropy genera-
tion source for Arweave data preparation, we propose
RandomXSquared: A simple construction on top of Ran-
domX that intends to maximize the amount of sufficiently
random entropy that can be generated in an atomic unit
of CPU-bound work. Additionally, RandomXSquared is
structured such that only minimal quantities of work can

Algorithm 2 RandomXSquared: A grid of parallel Ran-
domX channels, yielding a variable quantity of data that
cannot be randomly accessed without full computation.
This listing describes a sequential implementation of the
algorithm for brevity, although implementations paral-
lelized by lanes is also possible.
1: function RandomXSquared(s)
b ← Array[al × |RXpad|]
2:
for l ← 0 to al − 1 do
3:
RXinit(bl, s ∥ l)
4:

5:
6:
7:
8:
9:
10:

end for
for m ← 0 to am − 1 do
for l ← 0 to al − 1 do
RXexec(bl, RXprogs)

end for
b ← agm(b)

end for
return b

11:
12:
13: end function

be truncated by an attacker that wishes to access only a
small subset of the generated entropy.

RandomXSquared operates by constructing a square of
‘lanes’ and ‘rows’ of individual RandomX contexts. Each
lane in RandomXSquared has a RandomX scratchpad
with a unique seed as its starting entropy. The initial
scratchpads in each lane are created using RandomX’s
standard AesGenerator1R generator. Each lane can
be executed in parallel on separate CPU threads, with
the sequential (non-parallelizable) mix steps momentar-
ily merging execution and ensuring that entropy is shared
across all lanes. The result is a non-compressible chunk
of data with a width proportionate to al|Rpad|.

3.4 Work-Truncation attack resilience

By mixing entropy regularly across every byte of every
scratchpad on multiple occasions during execution, IS-
TORE caching attacks of the previously described form
become impossible. If executed exactly as described pre-
viously, Eva would end up storing the full scratchpad
size, plus her additionally generated bitmap. A modi-
fication of the ISTORE caching approach in the Ran-
domXSquared context would be to generate caches at
each of the mix phases of the execution. This approach,
too, would lead to the attacker storing more than |RXpad|
bytes, as every additional mix phase (assuming the same
amount of work at each layer) would generate a cache of
equivalent size to a base RandomX hash.

When utilizing RandomXSquared, random access to
any individual subchunk of data should not be possible
without having first computed all of the associated lanes
of RandomX execution, as the entropy of one is mixed
with the entropy of all others at multiple stages during
execution. An exception is in the final scratchpad en-
tropy mix, in which hypothetically o could ignore com-

5

Work Mult. RandomX RandomXSquared

1x
2x
3x

63.3%
86.6%
95.3%

63.3%
126.6%
189.9%

Table 1: ISTORE cache attack storage requirements for
RandomX alone and RandomXSquared. While Ran-
domX tends towards all memory cells being written to
during execution (yielding an ISTORE cache of |RXpad|
for the attacker, the cost to attack RandomXSquared
scales linearly with work, exceeding |RXpad| rapidly.

pute after the final write to a subchunk that they want
to access. This issue is alleviated by ensuring that the
choice of ags has a very low proportionate execution cost
relative to the totality of work in the RandomXSquared
run. Specifically, the work truncation possible on an av-
erage unit of compute can be calculated as 1
2

c(ags)
c(RX2) .

4 Parameters

As described in section 3, utilizing RandomXSquared in-
troduces a number of new parameters to the network.
Given that these parameters are global (affecting all min-
ers – and consequently, users – of the network), they must
be chosen with care. In this section we will outline a set
of proposed parameters, their implications and potential
trade-offs.

The parameters that we propose for adoption in Ar-

weave 2.9 are as follows:

al = 4
am = 4
ad = 3

agm(x) = CRC32 → Shuffle [block=6 bytes]

For agm, we propose a combination of a local (applied
to each individual RandomX scratchpad) CRC32, fol-
lowed by a global application of a deterministic shuffling
algorithm across the full concatenation of scratchpads.
The CRC32 algorithm was original described in 1961 by
Peterson and Brown et al [9]. Since that time it has
been widely deployed and is implemented in hardware
accelerated form in commodity Intel, AMD, ARM, and
RISC-V processors. This proposal employs the CRC-32C
Castagnoli variant. Additionally, the proposed shuffling
algorithmic is static and ensures next lane will use some
portion of data from all of other lanes. Utilizing a 6 byte
block size ensures that each RandomX scratchpad word
(8 bytes) is shifted and mixed. This mechanism guar-
antees entanglement of each write in one lane will affect
every execution of RandomX in other lanes during later
mix steps. As noted in section 3.2, RandomXSquared
does not require that agm creates outputs that satisfy
the typical requirements of cryptographic security, but
instead that they create a sufficiently random stream of

6

bytes that o cannot reliably infer the result without ex-
ecuting the surrounding RandomX computations. Addi-
tionally, as noted in section 3.4 it is imperative that the
execution cost of agm remains low relative to an RX2
invocation, such that work-truncation attacks are not vi-
able. For this reason, the CRC32 → shuffle construction
is proposed, as opposed to computationally expensive al-
ternatives (SHA-2, BLAKE-3, etc).

4.1 Parameter implications

The implications of the suggested parameters are as fol-
lows:

• Reduced compute requirements

for data
preparation. An al value of 4 would yield a 99.6%
reduction in the amount of compute required by hon-
est miners, when compared against Arweave 2.7 and
Arweave 2.8 with a packing difficulty of 1. Arweave
2.8 does not allow for packing greater than a diffi-
culty of 32, rendering a real-world comparison with
the proposed Arweave 2.9 global difficulty of ad = 3.
However, taking the highest available packing diffi-
culty of pd=32 renders an improvement of 99.987%
(one mp in Arweave 2.9 for every 8,192 mp in Ar-
weave 2.8 at pd=32). In our benchmarks, excluding
bookkeeping overhead this parameter should allow
low-cost hardware (for
even extremely low-power,
example, an ARM Raspberry Pi 5) to prepare a
replica of the network’s data in an acceptable time-
frame.

• Reduced read-rate requirements during min-
ing. Utilizing a global effective difficulty (ed) setting
of 50 would yield a necessary read rate (r) for honest
miners of 1 MB per second, rather than the stan-
dard rate of 50 MB/s for data packed with pd=1 in
Arweave 2.8, and 200 MB/s for all data in Arweave
2.7.x.

• Increased on-demand mining safety margin.
An additional effect of the parameters al = 4 and
ed = 50 is that the on-demand mining safety margin
of the network – the ratio c(hp) : c(op) of profitability
for honest (h) miners against on-demand (o) miners
– would increase approximately 52.87%.

• Increased proof validation times. One draw-
back of this approach is an increase in the proof
validation time for nodes in the network, although
still within acceptable ranges (≈200ms on our ref-
erence hardware). While this work does represent
an increase from Arweave 2.8, the practical impact
of this effect is mediated by the introduction of par-
allelizability of proof validation. With al = 4 and
the proposed fast choice of agm, miners would be
able to parallelize large parts of validation on up to
4 threads.

[7] Alfred J. Menezes, Paul C. van Oorschot, and
Scott A. Vanstone. Handbook of Applied Cryp-
isbn:
tography. 5th. CRC Press, 2001, p. 251.
978-0849385230. url: https : / / archive . org /
details/handbookofapplie0000mene/page/250/
mode/2up.

[8] Satoshi Nakamoto. “Bitcoin: A Peer-to-Peer Elec-
tronic Cash System”. In: Cryptography Mailing list
at https://metzdowd.com (Mar. 2009).

[9] William Wesley Peterson and Daniel T Brown.
“Cyclic codes for error detection”. In: Proceedings
of the IRE 49.1 (1961), pp. 228–235.

[10] Lev Berman Sam Williams James Piechota
and Sergii Glushkovskyi. Arweave 2.9 Param-
eter Tuning Model. url: https : / / docs .
google . com / spreadsheets / d / 1C0t8Yuz1SSOm -
CVTtyoqgieC5wTagWacf7pV8zRRbk8 / edit ? usp =
sharing.

[11] Kazuhiro Suzuki et al. “Birthday paradox for
multi-collisions”.
Information Security and
Cryptology–ICISC 2006: 9th International Con-
ference, Busan, Korea, November 30-December 1,
2006. Proceedings 9. Springer. 2006, pp. 29–40.

In:

[12] Arweave Core Team. Principles of

the Ar-
weave network. url: https : / / arweave . net /
1rL73ctmqTVv7qkqAsD4jIz5tU6WxgOAqABYuMHg5mQ.
tevador. url: https : / / github . com / tevador /
RandomX/blob/master/src/tests/scratchpad-
entropy.cpp.

tevador. RandomX. https : / / github . com /
tevador/RandomX. Accessed: December, 2024. Dec.
2022.

tevador. RandomX design. https://github.com/
tevador/RandomX/blob/master/doc/design.md.
Accessed: December, 2024. Dec. 2022.

tevador. RandomX design. https : / / github .
com / tevador / RandomX / blob / master / doc /
configuration . md. Accessed: December, 2024.
Dec. 2022.

[17] John Tromp. “Cuckoo cycle: A memory bound
graph-theoretic proof-of-work”. In: International
Conference on Financial Cryptography and Data
Security. Springer. 2015, pp. 49–62.

[18] Sam Williams et al. “Arweave: The Permanent
Information Storage Protocol”. In: (2023). url:
https://draft-17.arweave.net/.

[13]

[14]

[15]

[16]

A full model of the of the effects of these proposed pa-
rameters, as well as a simulation of the effects on entropy
quality of additional mix phases (am) has been published
to accompany this paper [10].

5 Conclusion

In this paper, we have proposed a new data prepa-
ration mechanism for Arweave which significantly en-
hances the efficiency of entropy generation and disper-
sion during mining operations. The introduction of Ran-
domXSquared as a core component achieves several key
objectives:
reducing computational overhead for hon-
est miners by significant margins, lowering the necessary
data access rate for miners by 90%, while maintaining
the existing security properties of the network.

References

[1] Ako Muhamad Abdullah et al. “Advanced encryp-
tion standard (AES) algorithm to encrypt and de-
crypt data”. In: Cryptography and Network Secu-
rity 16.1 (2017), p. 11.

[2] Jean-Philippe Aumasson et al. “BLAKE2: simpler,
smaller, fast as MD5”. In: Applied Cryptography
and Network Security: 11th International Confer-
ence, ACNS 2013, Banff, AB, Canada, June 25-28,
2013. Proceedings 11. Springer. 2013, pp. 119–135.
[3] Lev Berman and Sergii Glushkovskyi. url: https:
/ / github . com / ArweaveTeam / arweave / blob /
master/apps/arweave/src/ar_vdf.erl.

[4] Alex Biryukov, Daniel Dinu, and Dmitry Khovra-
tovich. “Argon2: new generation of memory-hard
functions for password hashing and other appli-
cations”. In: 2016 IEEE European Symposium on
Security and Privacy (EuroS&P). IEEE. 2016,
pp. 292–302.

[5] Dan Boneh, Henry Corrigan-Gibbs, and Stuart
Schechter. “Balloon hashing: A memory-hard func-
tion providing provable protection against se-
quential attacks”. In: Advances in Cryptology–
ASIACRYPT 2016: 22nd International Conference
on the Theory and Application of Cryptology and
Information Security, Hanoi, Vietnam, December
4-8, 2016, Proceedings, Part I 22. Springer. 2016,
pp. 220–248.

[6] Dan Boneh et al. Verifiable Delay Functions. Cryp-
tology ePrint Archive, Paper 2018/601. https://
eprint.iacr.org/2018/601. 2018. url: https:
//eprint.iacr.org/2018/601.

7

