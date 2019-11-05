---
layout: single
title:  What I'm doing in Cambridge
date:   2019-11-05 01:14:12 +0100
categories: misc 
comments: true
---

The main reason for this post is to keep track of what I'm doing in Cambridge.

Since I'm here, I spend most of my time reading papers, writing essays, and attending talks of the
astronomical society. Since I don't have I good memory, it's probably better to write down a few
high level things and interesting concepts.

Another reason why I'm writing this is because I think writing papers doesn't allow all the people
out there to understand the ideas, for the simple reason that the number of people with the time to read
\>10 pages are not so many. 


## Advanced topics in Computer Architecture

I'll write here the papers I've read, or at least the most relevant, from the latest I read, to the oldest:

### Specification, verification and test
- Genesys-Pro: Innovations in Test Program Generation for Functional Processor Verification, IBM Research, IEEE Design and Test 2004
\[[IEEExplore](http://dx.doi.org/10.1109/MDT.2004.1277900)\]

    This paper shows how random tests for processors were generated. Pretty outdated, but I found
    interesting how constraint solver were used for it.

### Memory system design
- Linearizing Irregular Memory Accesses for Improved Correlated Prefetching Jain and Lin, MICRO 2013 \[[ACM Digital Library](https://dl.acm.org/citation.cfm?id=2540730)\]

    Prefetching means to carry data in cache before they are needed, to save time of possible cache
    misses. If you access an array, then everything is easy: just prefetch the next elements.
    What if you are accessing random elements in memory? This paper address this problem.
    I also have a [presentation](https://docs.google.com/presentation/d/16YriXKEbeZ2ruTnAyFClQRxQBfEH5MLABoha2QpB07g/edit?usp=sharing) on it!

### Processor design
- The Celerity Open-Source 511-Core RISC-V Tiered Accelerator Fabric: Fast Architectures and Design Methodologies for Fast Chips Davidson et al. IEEE Micro, 38(2), March-April, 2019 \[[IEEE Xplore](https://ieeexplore.ieee.org/document/8344478)\]
  
    Probably only relevant for research purposes, so far. The implementation of several specific chip for
    (1) general computing, (2) neural networks and (3) massive parallelism is very interesting.
    There is a pointer to "store programming": let's imagine to have multiple cores (like 500). You
    can store data to each core you want, and put a core in an idle state waiting for data.

- Inside 6th-Generation Intel Core: New Microarchitecture Code-Named Skylake Doweck et al, IEEE Micro, vol. 37, 2017 \[[IEEE Xplore](https://ieeexplore.ieee.org/document/7924286)\]

    This paper is about the CPU I have on my laptop. An interesting point is on the algorithm to
    chose the correct frequency for the best performance/power comsumption ratio, called 
    [EARtH](https://ieeexplore.ieee.org/document/6327195). Although this should work, I had to
    disable it on my laptop to make things faster