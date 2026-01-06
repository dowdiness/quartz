---
title: Virtual machine
publish: false
tags: [compiler]
aliases: []
created: 2025-07-24T21:56:46+09:00
modified: 2025-07-24T22:03:28+09:00
---

# Virtual machine

[Reddit source](https://www.reddit.com/r/ProgrammingLanguages/comments/1m2ecz3/three_papers_to_read_if_you_are_implementing_a/)

- [_"A Portable VM-based Implementation Platform for non-restrict Functional Programming Languages"_ ](https://annas-archive.org/scidb/10.1145/3064899.3064903/)by Jan Martin Jensen & John van Gronigan. This paper discusses implementation of `asm.js` which was widely used to run C code (such as DOOM) in browser pre-WASM. Discusses architecture of the VM which you can use to implement your own.
    
- [_"Optimizing code-copying JIT compilers for virtual stack machines"_](https://annas-archive.org/scidb/10.1002/cpe.1016/) by David Gregg and ~~Antol~~ Anton Ertl. This paper discusses how you can use C code to create JIT. Basically, instead of using an Assembly framework like libkeystone to just-in-time compile your JIT code, you can use C code instead, hence "Code-copying". Ertl is one of GForth's authors by the way, and creator of VMGen. So he knows something about language VMs.
    
- [_"The Essence of Meta-Tracing JIT Compilers"_](https://soft.vub.ac.be/~mvdcamme/The%20Essence%20of%20Meta-Tracing%20JIT%20Compilers.pdf), a thesis by Maarten Vandercammen. This thesis explains whatever there is to know about Meta-tracing. PyPy is, for example, a meta-tracing Python interpreter. In a simple Tracing-JIT interpreter, you 'trace' busy parts of the code (mostly loops) and you generate machine code for them, and optimize it as you go. In a 'Meta-tracing' JIT, you hand it off to another interpreter to trace it for ya. PyPy uses a subset of Python to do that.