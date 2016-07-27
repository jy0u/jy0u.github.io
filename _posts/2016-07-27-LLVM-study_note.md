---
layout: post
title: LLVM Study Notes
---

Keep some useful information for LLVM study.

* How TableGen's DAGISel Backend Works [link](https://github.com/draperlaboratory/fracture/wiki/How-TableGen's-DAGISel-Backend-Works)

* A Beginner's Guide to Fracture [link](https://github.com/draperlaboratory/fracture/wiki/A-Beginner%27s-Guide-to-Fracture)

* A deeper look into the LLVM code generator, Part 1 [link](http://eli.thegreenplace.net/2013/02/25/a-deeper-look-into-the-llvm-code-generator-part-1)

* What are glue and chain dependencies in an LLVM DAG? [link](http://stackoverflow.com/questions/33005061/what-are-glue-and-chain-dependencies-in-an-llvm-dag)
> Black arrows mean data flow dependency  
> Red arrows mean glue dependency  
> Blue dashed arrows mean chain dependency  
> Chain dependencies prevent nodes with side effects (including memory operations and explicit register operations)   
> from being scheduled out of order relative to each other.  
> Glue prevents the two nodes from being broken up during scheduling.   
> It's actually more subtle than that [1], but most of the time you don't need to worry about it.

* Writing an LLVM Backend [link](http://llvm.org/docs/WritingAnLLVMBackend.html)

* x86 Instruction Set Reference [link](http://x86.renejeschke.de/)

* X86 Assembly/Machine Language Conversion [link](https://en.wikibooks.org/wiki/X86_Assembly/Machine_Language_Conversion)

* lldb [link](http://stackoverflow.com/questions/26705506/how-to-set-the-discover-path-for-lldb-in-xcode)

* Tutorial: Creating an LLVM Backend for the Cpu0 Architecture [link](http://jonathan2251.github.io/lbd/)

* Tutorial: Creating an LLVM Toolchain for the Cpu0 Architecture [link](http://jonathan2251.github.io/lbt/)
