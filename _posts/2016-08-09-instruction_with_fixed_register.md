---
layout: post
title: Instruction with Fixed Register
---
In X86 ISA, the mulplication is a instruction with fixed register, such as AL.
How did LLVM handle this?

In X86TargetLowering::LowerOperation() of X86IselLowering.cpp, the ISD::UMULO is mapped to LowerXALUO().  
Near the end of handling of ISD::UMULO, there is a getNode() for X86ISD::UMUL.  

In X86DAGToDAGISel::Select() of X86ISelDAGToDAG.cpp, there is a handling to add a getCopyToReg() in X86ISD::UMUL  
and the destinaiton register is AL, AX, EAX or RAX upon on the operand size.   
Finally, the node is replaced with a new node of a opcode, X86::MUL8r, MUL16r, MUL32r or MUL64r.   
The lowering of this operation is done and not entering the SelectCode() to do instruction pattern match defined in the td files.


