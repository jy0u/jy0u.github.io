
``` C++
def INC8xm : II8<0x34, (outs), (ins xmem:$Imm),                                                                                                                                
        "inc\t{($Imm)}", [(store (add (i8 (load xaddr:$Imm)), 1), xaddr:$Imm)]>;
```

``` C++
def xaddr : ComplexPattern<iPTR, 2, "SelectXAddr", [], []>;  
```

``` C++
def xmem : Operand<i16> {                                                                                                                                                            
  let PrintMethod   = "printXMemOperand";
  let EncoderMethod = "getXMemOpValue";
  let MIOperandInfo = (ops IR16, i8imm);
}
```

``` C++
unsigned Z80MCCodeEmitter::getXMemOpValue(const MCInst &MI, unsigned OpNo,
  SmallVectorImpl<MCFixup> &Fixups) const
{ 
  const MCOperand &MOReg = MI.getOperand(OpNo);
  const MCOperand &MOImm = MI.getOperand(OpNo+1);
  if (MOReg.isReg() && MOImm.isImm())
      return static_cast<unsigned>(MOImm.getImm());
  return 0;                                                                                                                                                                          
}
```


``` C++
bool Z80DAGToDAGISel::SelectXAddr(SDValue N, SDValue &Base, SDValue &Disp)
{
  switch (N->getOpcode())
  {
  case ISD::FrameIndex:
    if (FrameIndexSDNode *FIN = dyn_cast<FrameIndexSDNode>(N))
    {   
      Base = CurDAG->getTargetFrameIndex(FIN->getIndex(), MVT::i16);
      Disp = CurDAG->getTargetConstant(0, MVT::i8);
      return true;
    }   
    break;
  case ISD::CopyFromReg:
    if (RegisterSDNode *RN = dyn_cast<RegisterSDNode>(N.getOperand(1)))
    {   
      unsigned Reg = RN->getReg();
      if (Reg == Z80::IX || Reg == Z80::IY)
      {   
        Base = N;
        Disp = CurDAG->getTargetConstant(0, MVT::i8);
        return true;
      }   
    }   
    break;
  case ISD::ADD:
    if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(N.getOperand(1)))
    {   
      SDValue Op0 = N.getOperand(0);
      if (Op0.getOpcode() == ISD::CopyFromReg)                                                                                                                                       
      {   
        RegisterSDNode *RN = dyn_cast<RegisterSDNode>(Op0.getOperand(1));
        unsigned Reg = RN->getReg();
        if (Reg == Z80::IX || Reg == Z80::IY)
        {   
          Base = N.getOperand(0);
          Disp = CurDAG->getTargetConstant(CN->getZExtValue(), MVT::i8);
          return true;
        }   
      }   
      else if (FrameIndexSDNode *FIN = dyn_cast<FrameIndexSDNode>(Op0))
      {   
        Base = CurDAG->getTargetFrameIndex(FIN->getIndex(), MVT::i16);
        Disp = CurDAG->getTargetConstant(CN->getZExtValue(), MVT::i8);
        return true;
      }   
    }   
    break;
  }
  return false;
}
```
