---
layout: post
title: Pseudo Instruction
---

From the comments in Instruction structure as below, A pseudo instruction is not a real instruction. 
It is for code-gen and will not emit the binary.  

``` C++
//===----------------------------------------------------------------------===//
// Instruction set description - These classes correspond to the C++ classes in
// the Target/TargetInstrInfo.h file.
//
class Instruction {
.
.
.
  bit isPseudo     = 0;     // Is this instruction a pseudo-instruction?
                            // If so, won't have encoding information for
                            // the [MC]CodeEmitter stuff.
.
.
.
  // Is this instruction a "real" instruction (with a distinct machine
  // encoding), or is it a pseudo instruction used for codegen modeling
  // purposes.
  // FIXME: For now this is distinct from isPseudo, above, as code-gen-only
  // instructions can (and often do) still have encoding information
  // associated with them. Once we've migrated all of them over to true 
  // pseudo-instructions that are lowered to real instructions prior to
  // the printer/emitter, we can remove this attribute and just use isPseudo.
  //
  // The intended use is:
  // isPseudo: Does not have encoding information and should be expanded,
  //   at the latest, during lowering to MCInst.
  //
  // isCodeGenOnly: Does have encoding information and can go through to the
  //   CodeEmitter unchanged, but duplicates a canonical instruction
  //   definition's encoding and should be ignored when constructing the
  //   assembler match tables.
  bit isCodeGenOnly = 0; 
  
```

The implementation of XCore also has pseudo instruction. It is obvious that the size is 0 and isPsuedo is true.  

``` C++
class InstXCore<int sz, dag outs, dag ins, string asmstr, list<dag> pattern>
    : Instruction {
  field bits<32> Inst;

  let Namespace = "XCore";
  dag OutOperandList = outs;
  dag InOperandList = ins;
  let AsmString   = asmstr;
  let Pattern = pattern;
  let Size = sz;
  field bits<32> SoftFail = 0;
}

// XCore pseudo instructions format                                                                                                                                                  
class PseudoInstXCore<dag outs, dag ins, string asmstr, list<dag> pattern>
   : InstXCore<0, outs, ins, asmstr, pattern> {
  let isPseudo = 1;
}
```

One kind of pseudo instruction in XCore is as the above definition, "emit nothing."  

``` C++
def LDWFI : PseudoInstXCore<(outs GRRegs:$dst), (ins MEMii:$addr),
                             "# LDWFI $dst, $addr",
                             [(set GRRegs:$dst, (load ADDRspii:$addr))]>;

def STWFI : PseudoInstXCore<(outs), (ins GRRegs:$src, MEMii:$addr),
                            "# STWFI $src, $addr",
                            [(store GRRegs:$src, ADDRspii:$addr)]>;                                                                                                                  
```

The other one is the instructions with custom inserter (usesCustomInserter is set as true).  
It is expanded after ISel operations. 

``` C++
// SELECT_CC_* - Used to implement the SELECT_CC DAG operation.  Expanded after
// instruction selection into a branch sequence.
let usesCustomInserter = 1 in {
  def SELECT_CC : PseudoInstXCore<(outs GRRegs:$dst),
                              (ins GRRegs:$cond, GRRegs:$T, GRRegs:$F),
                              "# SELECT_CC PSEUDO!",
                              [(set GRRegs:$dst,
                                 (select GRRegs:$cond, GRRegs:$T, GRRegs:$F))]>;
}

```

When usesCustomInserter is set, the function, "EmitInstrWithCustomInserter" should be defined.
It is in XCoreISelLowering.cpp.

``` C++
MachineBasicBlock *
XCoreTargetLowering::EmitInstrWithCustomInserter(MachineInstr *MI, 
                                                 MachineBasicBlock *BB) const {
  const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
  DebugLoc dl = MI->getDebugLoc();
  assert((MI->getOpcode() == XCore::SELECT_CC) &&
         "Unexpected instr type to insert");
                    
.
.
.
```

The place to execute the custom inserter is in runOnMachineFunction() of ExpandISelPseudos.cpp.

``` C++
bool ExpandISelPseudos::runOnMachineFunction(MachineFunction &MF) {
   bool Changed = false;
   const TargetLowering *TLI = MF.getSubtarget().getTargetLowering();
 
   // Iterate through each instruction in the function, looking for pseudos.
   for (MachineFunction::iterator I = MF.begin(), E = MF.end(); I != E; ++I) {
     MachineBasicBlock *MBB = &*I;
     for (MachineBasicBlock::iterator MBBI = MBB->begin(), MBBE = MBB->end();
          MBBI != MBBE; ) {
       MachineInstr *MI = MBBI++;
 
      // If MI is a pseudo, expand it.
       if (MI->usesCustomInsertionHook()) {
         Changed = true;
         MachineBasicBlock *NewMBB =
           TLI->EmitInstrWithCustomInserter(MI, MBB);
         // The expansion may involve new basic blocks.
         if (NewMBB != MBB) {
           MBB = NewMBB;
           I = NewMBB->getIterator();
           MBBI = NewMBB->begin();
           MBBE = NewMBB->end();
         }
       }
     }
   }
 
   return Changed;
 }

```

reference: [discussion of [LLVMdev] pseudo lowering in mail list](https://groups.google.com/forum/#!topic/llvm-dev/r3Djh59uh24)

* possible usage  

> LLVM’s register allocators assume that spills, fills, and copies can be safely inserted anywhere (except after terminator instructions). The register allocators simply can’t work without that assumption.
> 
> As far as I know, your only option is to make sure that CCR is never live anywhere. This means that you need to create pseudo-instructions that bundle cmp+jump so the register allocator will never attempt to insert spill code between them.  
> ref: [[LLVMdev] Rematerialization and spilling](http://lists.llvm.org/pipermail/llvm-dev/2013-June/062630.html)
