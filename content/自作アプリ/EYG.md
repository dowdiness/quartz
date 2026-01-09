---
created: 2025-04-11T13:56:07+09:00
modified: 2026-01-09T18:40:20+09:00
---

# EYG

[CID](https://github.com/multiformats/cid)
[IPLD](https://ipld.io/)
[DAG-JSON](https://ipld.io/docs/codecs/known/dag-json/)
[Spec](https://github.com/CrowdHailer/eyg-lang/tree/main/spec)

[Interpreter](https://github.com/CrowdHailer/eyg-lang/blob/main/packages/javascript_interpreter/src/interpreter.mjs)

## Videos

[Peter Saxton - EYG a predictable, and useful, programming language](https://www.youtube.com/watch?v=bzUXK5VBbXc)

[EYG a predictable, and useful, programming language by Peter Saxton](https://www.youtube.com/watch?v=dh3sdHWQ2Ms)
[The EYG Language with Peter Saxton](https://youtu.be/w7mHY7CW51o?si=urhoNl8BwSBj8zS-)


To enable this programs need to be completely predictable in their behavior and EYG has several features to support this predictability.

- Managed effects, all program semantics are independent of the environment the program runs in. As well it is possible to statically analyze any effects a given program will rely on.
- Hash references of AST fragments. Programs always fully described there dependencies via these hashes.
- Closure serialization generation of program fragments which can be sent to other location. This allows static analysis with the type system to extend over multiple execution locations, for example a build machine and web server.
- A minimal AST, it should be easy to re-implement interpreters or compilers in the future to run EYG programs

コードのシリアライズによって扱いやすく色々な環境で動かせれるようにする
