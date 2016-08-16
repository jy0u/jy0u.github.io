---
layout: post
title: Define DAG Node for Target Specific Instruction
---
I didn't find any tutorial or document in the internet for adding a DAG Node for a target specific instruciton.  
So here it comes.  

Actually it is not hard if you spend time to read an existed backend codes of a target.  
However, I still write down it for someone who want to understand it quickly.  

Before we dive into the details, let's have some concepts about this.
The DAG represents IRs and semantic, relationships between the IR instrucitons.
Most handling process in LLVM are based on DAG and try to manipulate the nodes such as insert, delete or replace.
When we do the legalization or lowering, it is possible that some representation should be created for the target specific instruction.  
As you know, a DAG node is represented by the a operation, certain number of operands and some relationship to other nodes.  
Hence, the key point is to learn   

  * how to define a operation
  * how to define operands
  * how to make relationships  

Then, you can define your own DAG nodes.   

First, it is about the operands.
It lets LLVM to know which operand have certain attributes, then it will be easy to handle based on the characteristics.
SDTypeConstraint is the base class to represent a operand with certain constraints (attirbutes) and 
there are several predifined constraints for standard DAG nodes.
The codes and comments are very easy to understand. 
For example, opnum can let you to identify the operand and give it constraint.
isSameAS means the two operands which you identify are the same type.  

```C++
class SDTypeConstraint<int opnum> {
  int OperandNum = opnum;
}

// SDTCisVT - The specified operand has exactly this VT.
class SDTCisVT<int OpNum, ValueType vt> : SDTypeConstraint<OpNum> {
  ValueType VT = vt;
}

class SDTCisPtrTy<int OpNum> : SDTypeConstraint<OpNum>;

// SDTCisInt - The specified operand has integer type.
class SDTCisInt<int OpNum> : SDTypeConstraint<OpNum>;

// SDTCisFP - The specified operand has floating-point type.
class SDTCisFP<int OpNum> : SDTypeConstraint<OpNum>;

// SDTCisVec - The specified operand has a vector type.
class SDTCisVec<int OpNum> : SDTypeConstraint<OpNum>;

// SDTCisSameAs - The two specified operands have identical types.                                                                                                                   
class SDTCisSameAs<int OpNum, int OtherOp> : SDTypeConstraint<OpNum> {
  int OtherOperandNum = OtherOp;
}
```

Then, the above constraints could be used to represent the operands of a kind of DAG node.
It is called SDTypeProfile. Later, you will find that a type profile is necessary to define each kind of DAG node.
It is easy to understand the fields of SDTypeProfile by the codes, such as number of results, number of operands and the contraints of these operands. 
There are lots of predefined SDTypeProfiles. Following is an example to be interpreted.   
The constraints of operands in type profile, SDTIntBinOp,  

  * a typical profiles for operands of binary operation,
  * two operands and one result
  * both two operands have the same type and their type is integer.  

```C++
//===----------------------------------------------------------------------===//
// Selection DAG Type Profile definitions.
//
// These use the constraints defined above to describe the type requirements of
// the various nodes.  These are not hard coded into tblgen, allowing targets to
// add their own if needed.
//

// SDTypeProfile - This profile describes the type requirements of a Selection
// DAG node.
class SDTypeProfile<int numresults, int numoperands,
                    list<SDTypeConstraint> constraints> {
  int NumResults = numresults;
  int NumOperands = numoperands;
  list<SDTypeConstraint> Constraints = constraints;
}

// Builtin profiles.
def SDTIntLeaf: SDTypeProfile<1, 0, [SDTCisInt<0>]>;         // for 'imm'.
def SDTFPLeaf : SDTypeProfile<1, 0, [SDTCisFP<0>]>;          // for 'fpimm'.
def SDTPtrLeaf: SDTypeProfile<1, 0, [SDTCisPtrTy<0>]>;       // for '&g'.
def SDTOther  : SDTypeProfile<1, 0, [SDTCisVT<0, OtherVT>]>; // for 'vt'.
def SDTUNDEF  : SDTypeProfile<1, 0, []>;                     // for 'undef'.                                                                                                         
def SDTUnaryOp  : SDTypeProfile<1, 1, []>;                   // for bitconvert.

def SDTIntBinOp : SDTypeProfile<1, 2, [     // add, and, or, xor, udiv, etc.
  SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisInt<0>
]>;
```

Now, the only lost part is properties of a operation of a node.
Following are the predefined properties.   

```C++
//===----------------------------------------------------------------------===//
// Selection DAG Node Properties.
//
// Note: These are hard coded into tblgen.
//
class SDNodeProperty;
def SDNPCommutative : SDNodeProperty;   // X op Y == Y op X
def SDNPAssociative : SDNodeProperty;   // (X op Y) op Z == X op (Y op Z)
def SDNPHasChain    : SDNodeProperty;   // R/W chain operand and result
def SDNPOutGlue     : SDNodeProperty;   // Write a flag result
def SDNPInGlue      : SDNodeProperty;   // Read a flag operand
def SDNPOptInGlue   : SDNodeProperty;   // Optionally read a flag operand
def SDNPMayStore    : SDNodeProperty;   // May write to memory, sets 'mayStore'.
def SDNPMayLoad     : SDNodeProperty;   // May read memory, sets 'mayLoad'.
def SDNPSideEffect  : SDNodeProperty;   // Sets 'HasUnmodelledSideEffects'.
def SDNPMemOperand  : SDNodeProperty;   // Touches memory, has assoc MemOperand
def SDNPVariadic    : SDNodeProperty;   // Node has variable arguments.
def SDNPWantRoot    : SDNodeProperty;   // ComplexPattern gets the root of match
def SDNPWantParent  : SDNodeProperty;   // ComplexPattern gets the parent

```

Finally, the way to define a DAG node. The parts introduced above could be used to defined the DAG node
such as type profile, node property. Please read the following codes with above parts carefully.   

```C++
//===----------------------------------------------------------------------===//
// Selection DAG Pattern Operations
class SDPatternOperator {
  list<SDNodeProperty> Properties = [];
}

//===----------------------------------------------------------------------===//
// Selection DAG Node definitions.
//
class SDNode<string opcode, SDTypeProfile typeprof,
             list<SDNodeProperty> props = [], string sdclass = "SDNode">
             : SDPatternOperator {
  string Opcode  = opcode;
  string SDClass = sdclass;
  let Properties = props;
  SDTypeProfile TypeProfile = typeprof;
}
```

Now, it should be easy to understand the meaning of the following DAG node.  

```C++
def add : SDNode<"ISD::ADD", SDTIntBinOp, [SDNPCommutative, SDNPAssociative]>;
def sub : SDNode<"ISD::SUB", SDTIntBinOp>;
def mul : SDNode<"ISD::MUL", SDTIntBinOp, [SDNPCommutative, SDNPAssociative]>;

def SDT_XCoreBR_JT : SDTypeProfile<0, 2, [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
def XCoreBR_JT : SDNode<"XCoreISD::BR_JT", SDT_XCoreBR_JT, [SDNPHasChain]>;
def XCoreBR_JT32 : SDNode<"XCoreISD::BR_JT32", SDT_XCoreBR_JT, [SDNPHasChain]>;
```
