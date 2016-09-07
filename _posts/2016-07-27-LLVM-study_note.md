---
layout: post
title: LLVM Study Notes
---

Keep some useful information for LLVM study.

* Writing an LLVM Backend [link](http://llvm.org/docs/WritingAnLLVMBackend.html)

* Cormack-BuildingAnLLVMBackend.pdf [link](http://llvm.org/devmtg/2014-10/Slides/Cormack-BuildingAnLLVMBackend.pdf)

* LLVM programmer's manual [link](http://kito.wikidot.com/llvm-programmer-s-manual)

* Howto: Implementing LLVM Integrated Assembler - A Simple Guide [link](http://www.embecosm.com/appnotes/ean10/ean10-howto-llvmas-1.0.html)

* a back-end maintainer has to provide a few "guarantees" to 
continue supporting in tree [link](https://groups.google.com/d/msg/llvm-dev/0D9KO7QiZuE/gSGIJAa9GckJ)

* x86 Instruction Set Reference [link](http://x86.renejeschke.de/)

* X86 Assembly/Machine Language Conversion [link](https://en.wikibooks.org/wiki/X86_Assembly/Machine_Language_Conversion)

* lldb [link](http://stackoverflow.com/questions/26705506/how-to-set-the-discover-path-for-lldb-in-xcode)

* Tutorial: Creating an LLVM Backend for the Cpu0 Architecture [link](http://jonathan2251.github.io/lbd/)

* Tutorial: Creating an LLVM Toolchain for the Cpu0 Architecture [link](http://jonathan2251.github.io/lbt/)

* How TableGen's DAGISel Backend Works [link](https://github.com/draperlaboratory/fracture/wiki/How-TableGen's-DAGISel-Backend-Works)

* A deeper look into the LLVM code generator, Part 1 [link](http://eli.thegreenplace.net/2013/02/25/a-deeper-look-into-the-llvm-code-generator-part-1)

* ISD::NodeType [link](http://llvm.org/docs/doxygen/html/ISDOpcodes_8h_source.html) 

* SelectionDAG [link](https://github.com/draperlaboratory/fracture/wiki/A-Beginner%27s-Guide-to-Fracture)   

> * Instruction nodes have a list of operands on the top, and result values on the bottom.   
> The name of the instruction is in the middle.   
> * CopyFromReg (CFR) nodes have two operands: the chain and a register. 
>    They have two results: a value and the chain. They denote that the value of the given register is being copied and  
>    placed into the result.
> * CopyToReg (C2R) nodes have three operands: the chain, a register, and a value.  
> They have one result: the chain. They denote that the value of the given register is being replaced  
> with the value operand. Register nodes are only used by the CFR nodes and the C2R nodes.   
> They indicate which register the operation is using. There is one register node for each individual   
> register used in the SelectionDAG. If multiple CFRs and C2Rs use the same register, they will all point to the same node.  
> * The EntryToken node is the first node in the graph and represents the start of the basic block.  
> There is only one EntryToken per SelectionDAG.
> * The GraphRoot node is the last node in the graph and represents the end of the basic block.  
> There is only one GraphRoot per SelectrionDAG.
> * The chain represents the flow of control.  
> Starting from the EntryToken, the chain can be followed all the way down to the GraphRoot.   
> This is the order in which the instructions must execute for the program to work correctly.  
> The instructions that do not involve the chain can be executed in any order, as long as all of  
> their operands have executed in the right order.

* What are glue and chain dependencies in an LLVM DAG? [link](http://stackoverflow.com/questions/33005061/what-are-glue-and-chain-dependencies-in-an-llvm-dag)  

> * Black arrows mean data flow dependency  
> * Red arrows mean glue dependency  
> * Blue dashed arrows mean chain dependency  
> * Chain dependencies prevent nodes with side effects (including memory operations and explicit register operations)   
> from being scheduled out of order relative to each other.  
> * Glue prevents the two nodes from being broken up during scheduling.   
> It's actually more subtle than that [1], but most of the time you don't need to worry about it.

* The meaning of SDNPHasChain [link](http://lists.llvm.org/pipermail/llvm-dev/2006-October/006905.html)  

> to model relative ordering of memory operations.   
> SDNPHasChain is defined in TargetSelectionDAG.td as a node property.  
> It tells tblegen that specific node read / write chains so tblgen can emit the correct    
> selection code for patterns that use these SDNode's.  

* The meaning & example of MIOperandInfo [link](http://lists.llvm.org/pipermail/llvm-dev/2015-October/091860.html)  

> an address can be formed from a 16-bit register plus a zero-extended 8-bit register.  
> a ComplexPattern to match the address expression and MIOperandInfo to specify the classes of the registers  

``` C++
def memR16R8 : Operand<i16> {
		let MIOperandInfo = (ops Reg16Class, Reg8Class);
		...
	}
```
