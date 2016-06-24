---
layout: post
title: x86 register in tbgen
---
If you are not familiar with the tbgen of LLVM, the best way to writing one for your target is to trace one which's register arch. is similar to your target.

I'm trying TLCS870C, which is a Toshiba 8bit mcu.
It is CISC and some register design is similar to x86, so it is quite easy to get a clear idea after tracing x86's.

Followings are what I got from X86RegisterInfo.td.  
It is better that you read with Target.td with followings. 

### one 16bit regiser is from two 8bit register
Some parts of code are not important for this, so they are removed.  
Basic parts:

```
class X86Reg<string n, bits<16> Enc, list<Register> subregs = []> : Register<n> {
  let Namespace = "X86";
  let HWEncoding = Enc;
  let SubRegs = subregs;
}

// Subregister indices.
let Namespace = "X86" in {
  def sub_8bit    : SubRegIndex<8>;
  def sub_8bit_hi : SubRegIndex<8, 8>;
}

// 8-bit registers
def AL : X86Reg<"al", 0>;
def AH : X86Reg<"ah", 4>;
```

Register AX is from AH and AL. The lower 8 bit from AL and higher 8 bit from AH.  
SubRegIndices is the description list of register bits.  
sub_8bit is the first element in the list and described the bit field which from AL.
It is similar for the sub_8bit_hi and AH.
Besides, the whole register bits are covered by the both 8 bit registers, so CoveredBySubRes is set to 1.

```
// 16-bit registers
let SubRegIndices = [sub_8bit, sub_8bit_hi], CoveredBySubRegs = 1 in {
def AX : X86Reg<"ax", 0, [AL,AH]>;
}
```

EAX is extented from AX and the extented part is not from any existed registers.  
From this and above one, It should be clear that how to set SubRegIndices and CoveredBySubRegs.

```
// 32-bit registers
let SubRegIndices = [sub_16bit] in {
def EAX : X86Reg<"eax", 0, [AX]>, DwarfRegNum<[-2, 0, 0]>;
}
```

### define top-level register classes 
All the 8 bit registers are listed as general 8bit register.  
The order of elements in added register list is the register allocation order.  
The AltOrders is a special allocation list and order for certain subtarget which has some limitation.  
The AltOrderSelect is a function list. Each function does the decision to use AltOrders or the default one.  
The comments in the codes are quite clear to describe the reason why x86 to do this.  

```
def GR8 : RegisterClass<"X86", [i8],  8,
                        (add AL, CL, DL, AH, CH, DH, BL, BH, SIL, DIL, BPL, SPL,
                             R8B, R9B, R10B, R11B, R14B, R15B, R12B, R13B)> {
  let AltOrders = [(sub GR8, AH, BH, CH, DH)];
  let AltOrderSelect = [{
    return MF.getSubtarget<X86Subtarget>().is64Bit();
  }];
}
```

### more constrains on register allocatoins
It is possible that there are constrains on register allocations in certain instrction or situation.  
The solution in x86 tbgen design is to set different top level register class for register allocation under the constrains.

```
// GR8_NOREX - GR8 registers which do not require a REX prefix.
def GR8_NOREX : RegisterClass<"X86", [i8], 8,
                              (add AL, CL, DL, AH, CH, DH, BL, BH)> {
  let AltOrders = [(sub GR8_NOREX, AH, BH, CH, DH)];
  let AltOrderSelect = [{
    return MF.getSubtarget<X86Subtarget>().is64Bit();
  }];
}
```

### don't allocate special registers
Just set the CopyCost to -1 and isAllocatable to 0.  
There are clear descriptions in Target.td. 

```
// Status flags registers.
def CCR : RegisterClass<"X86", [i32], 32, (add EFLAGS)> {
  let CopyCost = -1;  // Don't allow copying of status registers.
  let isAllocatable = 0;
}
```
