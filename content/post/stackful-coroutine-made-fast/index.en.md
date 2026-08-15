---
title: "Stackful Coroutine Made Fast"
date: 2024-10-14
slug: stackful-coroutine-made-fast
tags: ["coroutine", "photon", "c++", "performance"]
description: "Stackful coroutine is not intrinsically slow — current implementations are. With context-aware context switching (CACS), saving only the minimal registers per caller/callee context, stackful outperforms stackless in most cases. Implemented in Photon."
---

## Abstract

Stackful coroutine, also known as user-space cooperative thread, offers
the promise of more intuitive and accessible concurrent programming. With
the growing demand for highly concurrent programs, stackful coroutine has
gained increasing interest in recent years. It has, however, been much
maligned for poor performance compared to stackless coroutine because of
its heavy reliance on context switching. In this paper we perform in-depth
measurement and analysis of several advanced implementations of stackful
and stackless coroutine. Our analysis indicates that although current
implementations of stackful coroutines are indeed significantly slower
than their stackless counterparts, stackful coroutine is not intrinsically
slow. Rather, stackful coroutine performs poorly mainly because the
current implementations do not fully leverage the fact that control flow
is cooperatively passed among stackful coroutines. Based on this
observation, we propose context-aware context switching (CACS) among
stackful coroutines. Instead of a full set of registers, CACS saves
(restores) only the minimum necessary set of registers according to the
caller (callee) context. It also enables the branch to be inlined at the
caller site so that branch prediction is more accurate. We have
implemented CACS in Photon, a highly optimized libOS based on coroutine.
Performance measurements show that with the optimizations proposed in this
paper, stackful coroutine out-performs stackless coroutine in most cases,
and ties in the generator paradigm which is an especially challenging
scenario for stackful coroutine. We also suggest a few supporting changes
in computer architecture, programming language, compiler and OS that can
further improve the performance of stackful coroutine. Our work is
open-sourced on GitHub.

## Introduction

Modern servers may have network connectivity with bandwidth that is on the
same order as that of its memory or CPU interconnect, and host dozens of
SSDs each offering more than 10GB/s of throughput. With such advances in
the hardware I/O capability, software stacks must become both highly
concurrent and efficient to unleash the growing performance potential of
the modern server. Recent software improvements to this effect include the
development of high-performance I/O frameworks, such as DPDK and SPDK,
that budget to process a single packet or request in terms of CPU cycles.

Coroutine has also been gaining increasing attention in recent years to
handle concurrent programming efficiently and effectively. Programming
languages that have or are embracing coroutine include C++20, Rust 1.39 in
2019, C# 5.0 in 2012, Python 3.5 in 2015, JavaScript ES2017, Swift 5.5 in
2021, Java 21 in 2023, Dragonwell 8 in 2019, etc. Golang has provided
coroutine (goroutine) as a first-class construct since its initial design.

There are two types of coroutine — stackless and stackful. The former
shares a default stack among all the coroutines while the latter assigns a
separate stack to each coroutine. With stackless coroutine, the code is
transformed into event handlers at compile time, and driven by an event
engine at run time, i.e. the scheduler of stackless coroutine.
Transferring control of CPU to a stackless coroutine is merely a function
call with an argument pointing to its context. Conversely, transferring
CPU control to a stackful coroutine requires a context switch. This
context switch is widely regarded as a heavy-weight operation when
compared to the function call. In reality, this context switch is much
more efficient than a kernel task switch because it does not incur the
overhead of a round-trip transition from user-space to kernel-space, and
it is also possible to perform optimizations by making use of the
cooperative nature of the coroutines within a single program.

Nevertheless, due primarily to the perceived performance concern (among
other issues), more and more systems are abandoning stackful coroutine for
stackless coroutine, especially systems that emphasize performance such as
C++20, Rust, C#, Swift, etc. There was once a heated debate in the C++
standards committee over stackless or stackful coroutine for C++20.
Although stackful coroutine is generally easier to use, more compatible
with existing codebases and more efficient in many scenarios, the
proponents of stackless coroutine constructed a microbenchmark to show
that "fibers (stackful coroutines) have 20 times larger context switch
overhead" than stackless coroutine. The defendants of stackful coroutine
did not give a direct response to the challenge, and ultimately the
committee adopted stackless coroutine for C++20. We believe that the
demonstrated outsized difference in overhead played an important role in
this decision, as well as similar decisions for other systems.

In this paper we argue that stackful coroutine is not intrinsically slow.
It just has not been implemented to fully exploit the cooperative nature
of stackful coroutine, and there are opportunities to make context
switching more efficient. We perform in-depth measurement and analysis of
several current coroutine implementations. Based on the analysis, we
propose context-aware context switching (CACS) to improve the efficiency
of register saving, accuracy of branch prediction, and hit rate of CPU
cache. CACS leverages the fact that context switches among stackful
coroutines occur only at specific caller sites. With CACS, each context
switching saves only the registers that will be needed after it switches
back, and these registers are determined by the compiler at the caller
site. CACS also enables the branching to be inlined at the caller site so
that branch prediction is more accurate. We apply CACS to optimize
stackful asymmetric coroutine by designing in-stack generator. With CACS,
it performs as efficiently as the corresponding stackless coroutine
implementation. We also introduce a new function calling convention named
`preserve_none` as an expansion of CACS for all switching functions to
further improve performance. With our work the cost of yield operation
(scheduling and switching to the next coroutine) is greatly reduced. CACS
is implemented in Photon, a sophisticated libOS based on coroutine. The
proposed calling convention is implemented in Clang.

On the other hand, we demonstrate that stackless coroutine is inefficient
in handling multi-level invocation, an inevitable pattern in real-world
programs. It even incurs an overhead proportional to the length of the
call chain when dealing with recursion. Despite there are optimizations
that can reduce this overhead to a constant in some scenarios, it is still
much higher than the corresponding overhead with stackful coroutine. Our
results suggest that stackful coroutine is the better choice for efficient
concurrency.

The contributions of this paper are as follows:

1. We conduct an in-depth performance characterization of
   state-of-the-art implementations of both stackless and stackful
   coroutines, analyzing the root causes for various measured differences.
2. We observe that there are untapped opportunities to improve the
   performance of stackful coroutine by exploiting the fact that
   coroutines are cooperatively scheduled user-space threads.
3. We propose and implement CACS to optimize the performance of stackful
   coroutine, and demonstrate that it can effectively eliminate the
   performance concern of context switching, thereby raising the
   performance upper bound of stackful coroutine. CACS makes stackful
   coroutine even feasible for challenging scenarios such as the generator
   paradigm. The overall result is promising: a single Xeon CPU core can
   perform a yield operation in ~1.52 ns or ~3.34 cycles, comparable to
   the cost of a function call, and out-performing state-of-the-art result
   by several times.
4. While we demonstrate promising results with CACS, the current
   implementation is still constrained by existing architecture,
   programming language, compiler and OS. We suggest several supporting
   changes in these areas that can further improve the performance of
   stackful coroutine.

---

Read the full paper: <https://photonlibos.github.io/blog/stackful-coroutine-made-fast>
