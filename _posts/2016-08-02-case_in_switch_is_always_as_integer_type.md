---
layout: post
title: Case in Switch is always as Integer type
---

It is well known that the case of switch statement should be a number or an expression to a number.  
However, what type of number is also defined in C standard. It sould be integer type!!  
It is straightforward before you deeply think it.  
[C Standard](http://c0x.coding-guidelines.com/6.8.4.2.html)
> 1748 The controlling expression of a switch statement shall have integer type.
> 1750 The expression of each case label shall be an integer constant expression and no two of the case constant expressions in the same switch statement shall have the same value after conversion.  
> 1755 The integer promotions are performed on the controlling expression.  

There is an example which showing some flaws such that the rule is not so good.
The below are the example, the source code snippet and emmited llvm IRs from a modified CLANG for TLCS870.  
The native supported integer size of TLCS870 should be 8 bit and 16 bit and the int type is defined as 16 bit.
(Actually, I am not sure, but lots of instructions have both 8bit and 16bit operation versions.)  

``` C
typedef enum {
  ADD,
  SUB,
  MUL,
  DIV,
  MOD,
} OP_E;

int compute(OP_E op, int a, int b) {
  int res;
  
  switch (op) {
    case (ADD):
      res = a + b;
      break;
    case (SUB):
      res = a - b;
      break;
    case (MUL):
      res = a * b;
      break;
    case (DIV):
      res = a / b;
      break;
    case (MOD):
      res = a % b;
      break;
    default:
      res = 0;
      break;
  }
  return res;
}
```

IR emmited from CLANG with -fshort-enum:
```
; ModuleID = '101.c'                                                                                                                                                                 
source_filename = "101.c"
target datalayout = "e-m:e-p:16:8:8-i1:8:8-i8:8:8-i16:8:8-i32:8:8-n8:16-S8"
target triple = "tlcs870"

; Function Attrs: nounwind
define i16 @compute(i8 zeroext %op, i16 %a, i16 %b) #0 {
entry:
  %op.addr = alloca i8, align 1
  %a.addr = alloca i16, align 2
  %b.addr = alloca i16, align 2
  %res = alloca i16, align 2
  store i8 %op, i8* %op.addr, align 1
  store i16 %a, i16* %a.addr, align 2
  store i16 %b, i16* %b.addr, align 2
  %0 = load i8, i8* %op.addr, align 1
  %conv = zext i8 %0 to i16
  switch i16 %conv, label %sw.default [
    i16 0, label %sw.bb
    i16 1, label %sw.bb1
    i16 2, label %sw.bb2
    i16 3, label %sw.bb3
    i16 4, label %sw.bb4
  ]
```
The original purpose of option -fshor-enum is to optimize the size of enum to 8 bit.   
However, the case of switch statement should be integer type, so the zext instruction is inserted for type promotion.  
It seems be implemented in Sema::ActOnStartOfSwitchStmt() of clang/lib/Sema/SemaStmt.cpp .  
``` C
// C99 6.8.4.2p5 - Integer promotions are performed on the controlling expr.
    CondResult = UsualUnaryConversions(Cond);
    if (CondResult.isInvalid()) return StmtError();
    Cond = CondResult.get();
```

Because of this, the optimization is necessary when llvm does the compilation.  
It seems a little redundant effort, but it is hard to escape in CLANG/LLVM architecture...

There is [another example](http://stackoverflow.com/questions/31054609/cast-to-larger-type-in-switch-case) 
in stackoverflow and the author of the right answer is an C expert!!
