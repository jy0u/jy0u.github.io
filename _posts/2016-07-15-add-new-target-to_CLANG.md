---
layout: post
title: Add new target to CLANG
---

Although CLANG generates LLVM IRs, not the target instrucitons, 
the differences of ABI of each targets still make the generated IRs are not portable between different targets. 

If you want to support new target in LLVM, you have probably to add new target support in CLANG.

Just few things should be added into CLANG, and following is an example. 

### Add your TargetInfo class
Add XXXTargetInfo, which is derived from TargetInfo class in lib\Basic\Targets.cpp .
* Setting the size of basic types (int, short, char, ...), pointer size, endian, ... in the constructor.
* implement following virtual functions.

```
    virtual void getTargetDefines(const LangOptions &Opts,
                                  MacroBuilder &Builder) const ;
    virtual void getTargetBuiltins(const Builtin::Info *&Records,
                                   unsigned &NumRecords) const ;
    virtual bool hasFeature(StringRef Feature) const ;
    virtual void getGCCRegNames(const char* const *&Names,
                                unsigned &NumNames) const;
    virtual void getGCCRegAliases(const GCCRegAlias *&Aliases,
                                  unsigned &NumAliases) const ;
    virtual bool validateAsmConstraint(const char *&Name,
                                       TargetInfo::ConstraintInfo &info) const ;
    virtual const char* getClobbers() const ;
    virtual BuiltinVaListKind getBuiltinVaListKind() const;

```

### Add your TargetInfo to target selection function
Add your XXXTargetInfo for the architecture switch in the following function in lib\Basic\Targets.cpp .  
It is quite easy to do when refering other targets' implementation in the fucntion.

```
static TargetInfo *AllocateTarget(const llvm::Triple &Triple) {
  llvm::Triple::OSType os = Triple.getOS();
   
  switch (Triple.getArch()) {

```

### Add your ABIInfo class
Add XXXABIInfo, which is derived from ABIInfo class in lib\CodeGen\TargetInfo.cpp .  
Implement following virtual functions. 

```
  ABIArgInfo classifyReturnType(QualType RetTy) const;
  ABIArgInfo classifyArgumentType(QualType Ty) const;
  virtual void computeInfo(CGFunctionInfo &FI) const;
  virtual llvm::Value *EmitVAArg(llvm::Value *VAListAddr, QualType Ty,
                                    CodeGenFunction &CGF) const;
```

### Add your TargetCodeGenInfo class
Add XXXTargetCodeGenInfo, which is derived from TargetCodeGenInfo class in lib\CodeGen\TargetInfo.cpp .  
After reading other targets' implementations, you will find it is simple.

### Add your XXXTargetCodeGenInfo to target selection function
Add your case for the architecture switch in the following function in lib\CodeGen\TargetInfo.cpp .  
It is quite easy to do when refering other targets in the fucntion.

```
const TargetCodeGenInfo &CodeGenModule::getTargetCodeGenInfo() {
  if (TheTargetCodeGenInfo)
    return *TheTargetCodeGenInfo;

  const llvm::Triple &Triple = getTarget().getTriple();
  switch (Triple.getArch()) {
  
```
