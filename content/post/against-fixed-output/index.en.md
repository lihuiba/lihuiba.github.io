---
title: "The Myth of Fixed-Output Compression"
date: 2026-08-17
slug: against-fixed-output
tags: ["storage", "compression", "fixed-output"]
description: "Fixed-output compression — variable-length input stuffed into fixed-size output blocks — was credited with eliminating read amplification, aligning I/O, improving compression ratio, and saving memory. Term-by-term attribution finds otherwise: these benefits belong to granularity choices, to the index and the decompressor, hold only inside a restricted framing, or cannot be found in measurement at all — while the costs are format-level and permanent. The conclusion is polite but total: as a format design decision, fixed-output is not worth adopting."
---

In the design of randomly readable compressed storage, the data layout
must first answer one question: where to draw the compression boundary.
Fixed-input splits the input into fixed-size units, compresses each, and
packs the variable-length outputs back to back; fixed-output goes the
opposite way — variable-length input is "stuffed" into fixed-size output
blocks, so that compression units align naturally with storage blocks.

Fixed-output was proposed in [1], and it arrived with a string of fine
stories: eliminating read amplification, aligning I/O, improving
compression ratio, saving memory. This essay checks the fate of each:
**some of these stories credit the wrong owner, some are staged only
inside a restricted framing, and some cannot be found in measurement at
all — while the costs stay in effect forever. As a format design
decision, fixed-output has no reason to exist.**
This essay rejects the technique's value as a general design choice; it
says nothing about any particular system built on it. A system's success
is decided by implementation quality, operations, and ecosystem together,
and an unsound mechanism does not necessarily prevent a successful system.

## 1. Read Amplification and Alignment: Two Fairy Tales

The mechanism of read amplification is simple: decompression must start
from the head of a compressed stream (algorithms like LZ4 cannot seek
into the middle of a stream), so the larger the unit, the more I/O and
decompression a small request costs. It depends only on the unit size,
not on whether the input or the output is fixed-length.

Start with [1]'s own measurements (16MB read, actual I/O issued; the
stride read touches only the first 4KB of every 128KB):

| Layout | Unit size | Random read | Stride read |
|---|---|---|---|
| fixed-input | 128KB | 165.27 MB | 203.91 MB |
| fixed-input | 4KB | 26.19 MB | 26.23 MB |
| fixed-output | 4KB (output) | 26.12 MB | 25.93 MB |

At 4KB granularity the two layouts issue nearly identical I/O; the
six-to-eightfold difference in amplification comes entirely from unit
size. **Here the first fairy tale takes shape: the elimination of read
amplification is granularity's credit, not the layout's — and the
comparison that proves it was given by the very paper that proposed the
technique.**

The same table hides a second fairy tale. Fixed-output's blocks align
naturally with storage blocks, while fixed-input's variable-length blocks
pack tightly and inevitably straddle block boundaries, so a read must
fetch two partially-used physical blocks at the ends — a theoretical
"rubble" loss. The theory holds; the dividend was never paid: the two
layouts issue identical total I/O — the rubble bytes belong to
neighboring units and are equally useful to later reads, and the cache
irons this theoretical loss flat.

The key at the mechanism level is that the cache picks up data fetched
along the way but not yet used, and the two layouts cache different
things. Fixed-input only needs to cache compressed data, while
fixed-output must cache decompressed data, or the decompression work is
wasted. The cache layer not only fails to favor fixed-output — it sides
with fixed-input.

## 2. Compression Ratio: A Victory in a Restricted Framing

Against fixed-input of the same unit size (4KB vs 4KB), fixed-output does
win about 10% in image size [1] — a great victory, and the mechanism is
clear: "stuffing" makes each compressed stream cover a longer input, and
longer streams compress better.

But this victory avoids the realistic opponent. Fixed-output with 4KB
output produces images 11% to 29% larger than large-window fixed-input
(128KB units) [1] (two corpora measured: 0.52GB vs 0.47GB; 100.9MB vs
78.0MB). To recover that gap, fixed-output can only enlarge its output
blocks — that is, turn the granularity knob back up and invite read
amplification back in. **The technique welds compression ratio and
random-read performance onto a single knob, and turning it either way is
a concession.**

There is also a small bill on the storage side: fixed-input packs back to
back without wasting a byte; every fixed-output block leaves its tail
unfilled — internal fragmentation built in.

## 3. Apples to Apples: The Tale of Twin Brothers

The first two sections hid a framing trap: fixed-input's label measures
the **input**, fixed-output's label measures the **output** — both tagged
"4KB", yet the streams are not the same length. A true like-for-like
comparison gives every compressed stream the same logical coverage. At a
2:1 ratio, for example, fixed-input's 256KB unit and fixed-output's 128KB
output block are two cuttings of the same stream. Recount line by line
(serving one 4KB random read):

| | fixed-input, 256KB unit | fixed-output, 128KB output |
|---|---|---|
| One stream | 256KB input → ~128KB compressed | ~256KB input → 128KB compressed |
| Read I/O | whole stream, ~128KB | whole stream, 128KB |
| Decompression | from head to target output, half a stream on average | from head to target output, half a stream on average |
| Read amplification | ~32× | ~32× |
| Compression ratio | identical | identical |

Everything zeroes out: between twin brothers, there is no question of who
stands higher. One corollary follows — Section 2's 10% "advantage" is
itself a framing illusion: under the "4KB vs 4KB" label, fixed-output's
streams are actually twice as long. The source of the advantage is the
stream-length difference, not the layout; level the stream length and the
ratio gap zeroes out. **A 128KB fixed-output block ≡ a 256KB fixed-input
unit plus alignment — and the alignment dividend was never paid (Section
1).**

What remains under equal framing is entirely structural: fixed-input has
a sparse index, arithmetic-free logical locating, and zero storage waste;
fixed-output has block alignment and a priori known I/O sizes (Section
6). **In a fair comparison, the performance and compression ledgers hold
no surplus that belongs to the layout.**

## 4. The Arithmetic of Locating a Read: Flowers in the Mirror

Locating a read takes two mappings: logical offset → compression unit,
and compression unit → physical address. Each layout gets exactly one of
the two steps for free, in opposite directions:

```
fixed-input:  offset ──[÷C, arithmetic]──→ unit ──[lookup]──→ physical
fixed-output: offset ──[÷4K]──→ logical block ──[?]──→ unit ──[×4K]──→ physical
```

The point: fixed-output's free multiply step sits at the end of the
chain, while queries always start from the logical offset — this
arithmetic convenience is flowers in a mirror, visible but forever out of
the read path's reach. The missing "logical → unit" mapping is
data-dependent and cannot be computed arithmetically; two choices remain:
a dense index (one 8-byte entry per logical block [1], an order of
magnitude denser than fixed-input's per-unit index for large windows), or
a sparse index plus binary search (O(log n)).

Fixed-input has no such problem: subtracting two adjacent entries of the
offset table yields the compressed unit's length, so position and length
arrive in a single lookup. On the locating axis, fixed-output's ledger
records only outlays.

## 5. Addressability: A Legend

Beyond [1], a certain expectation of fixed-size alignment still
circulates in discussions of distribution formats: on-demand
distribution, lazy loading, block-level deduplication all seem to
presuppose fixed-size blocks — and this is exactly where the legend
begins to distort. Fixed-input plus an offset table makes variable-length
blocks equally addressable — the address is the (offset, length) pair in
the index entry, which combined with HTTP range requests forms a complete
on-demand distribution. Production-grade fixed-input distribution schemes
have long existed (OverlayBD's ZFile [2]; stargz's TOC [3]), at enormous
scale.

**The legend's true source is now located: addressability comes from the
existence of an index, not from fixed-length output.**

## 6. Memory Savings: A Medal for the Wrong Recipient

[1] carries one more claim, of memory efficiency: fixed-output makes
in-place decompression possible, each decompression reads at most two
compressed blocks whose sizes are known in advance, so memory use is
bounded. This claim likewise fails the attribution test — every key
memory-saving technique is credited to the decompressor.

First, partial decompression: decode from the stream head and stop at the
target output, leaving the bytes beyond untouched. This is a direct
corollary of sequential decoding and holds identically for both layouts.
Second, rolling decompression: with the algorithm's sliding window (e.g.,
LZ4's 64KB window), only a window's worth of history pages must be kept
to sustain decoding — the window is a property of the algorithm, not the
layout. Third, filling target pages directly: decompression output is
written straight into the destination page cache, eliminating a temporary
output buffer; fixed-input's read path can do exactly the same. As for
"sizes known in advance", Section 4 already supplied it — the compressed
unit's length is the difference of two adjacent offset-table entries,
hardly a monopoly of fixed-length output.

The reverse ledger is worth recording too: fixed-output pays more
complexity for this path. It requires the compression algorithm to offer
a fixed-output interface (destSize) — a format-level constraint; locating
needs a dense per-logical-block index; and [1]'s in-place decompression
is itself not free, requiring the image builder to simulate decompression
and rule on feasibility block by block. **This medal belongs to the
decompressor, not the layout; the complexity fixed-output carries for it
is not one whit less.**

## 7. The Costs: The Format's Tattoo

The first six sections audited the stories on the benefit side; the cost
side's ledger is kept in this one.

First, engineering complexity, whose final form is code volume. At both
ends fixed-input is a short loop: on the build side — split, compress,
append, record the offset; on the read side — one table lookup, one
decompression call. Fixed-output has no such straight line: every
structural difference in the preceding sections ultimately lands as
code — a state machine on the build side, a trail of branches on the
read side, and the code volume multiplies severalfold. This overhead is
not billed per use but per implementation: kernel driver, userspace
tools, language bindings — every adopter writes it all over again.

Second, build speed. Mainstream compression libraries shape their trunk
interface for fixed-input: fixed-size input, variable-size output, one
call — and all their optimization effort concentrates there. The
fixed-output (destSize) interface exists only as a side-branch variant in
some algorithms; to avoid it, one must trial-compress with the generic
interface — feed some input, watch whether the output crosses the block
boundary, roll back and recompress when it does. Both roads are slow; and
every library upgrade lands its dividend on the trunk first — fixed-output
watches from across the shore.

Third, the evolution burden. Fixed-input keeps its freedom outside the
format: the unit size is a parameter each image chooses for itself, and
indexing and decompression strategies are implementation details,
adjustable at will. Fixed-output writes the key decisions into the
format: fixed-size output, block alignment, per-block in-place flags —
all clauses of the contract. Every future format extension, every
algorithm swap, every read-path optimization must carry this contract
along; it cannot be revised.

A story's flaw lives in the telling — change the framing or run one
measurement and it shows itself; the costs' flaw lives in the
definition — as long as the format is fixed-output, every image and every
implementation pays. Implementations can iterate; a published format can
only be complied with: internal fragmentation will not vanish for a
cleverer builder, the locating lookup will not be saved by a more
diligent cache, and the build-time interface mismatch will not be healed
by library upgrades. These costs compound with every use of the
format — this is what "format-level and permanent" means.

And so the whole account closes: every claimed benefit, the opponent can
obtain for free by implementation means — granularity, index,
decompressor, none of which needs fixed-length output; while the costs
are all carved into the format itself. Stories fade; tattoos do not.

## Conclusion

With attribution done, the myths bow out one by one: read amplification
goes to granularity, memory efficiency to the decompressor, the alignment
dividend cannot be found anywhere, the compression-ratio victory is
staged only in a framing that avoids the realistic opponent; the
addressability expectation beyond [1] likewise goes to the index — every
claimed advantage either vanishes or changes owner. The costs all stay on
fixed-output's own ledger: locating loses its arithmetic, output blocks
carry internal fragmentation, compression algorithms are constrained to a
destSize interface, engineering complexity on both the build and read
sides, slower builds, and an evolution contract that cannot be
revised — format-level, permanent.

Our conclusion is therefore polite but total: **this technique is not
worth adopting.** In fact, since its proposal in 2019, no second system
has adopted it — its only carrier remains the filesystem that proposed
it. When designing a read-only compressed layout, the right move is to
keep the freedom on the granularity axis (choose unit sizes per
workload), build the index well, and use fixed-input.

Two methodology notes to close; they will outlast this essay's specific
conclusions:

- **The attribution test.** When you see a claimed advantage, first ask
  "can an equally well-implemented opponent obtain it?" Being able to
  answer this question makes most technical marketing show its true form
  on the spot.
- **The time test.** The theses of most papers and techniques fade with
  time; that is the norm. The survival of an artifact is not the survival
  of its argument — a system may be thriving while its central claim has
  long been quietly replaced by its own evolution.

## References

[1] Xiang Gao, Mingkai Dong, Xie Miao, Wei Du, Chao Yu, Haibo Chen.
"EROFS: A Compression-friendly Read-only File System for
Resource-scarce Devices." *USENIX Annual Technical Conference
(ATC '19)*, 2019. <https://www.usenix.org/conference/atc19/presentation/gao>

[2] Huiba Li, Yifan Yuan, Rui Du, Kai Ma, Lanzheng Liu, Windsor Hsu.
"DADI: Block-Level Image Service for Agile and Elastic Application
Deployment." *USENIX Annual Technical Conference (ATC '20)*, 2020.
<https://www.usenix.org/conference/atc20/presentation/li-huiba>

[3] stargz Snapshotter: eStargz (seekable tar.gz) lazy-pulling image
support. <https://github.com/containerd/stargz-snapshotter>
