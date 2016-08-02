---
layout: post
title: LLVM TLCS870 Todo List
---

1. builtin function
  * clang/lib/Basic/Targets.cpp
  * clang/include/clang/Basic/TargetBuiltins.h
2. optimize type promotion for switch statement (CLANG follows C99 standard)
  * eliminate unnecessary zext IR (from 8 bit to 16 bit)
  * truncate the integer type from 16 bit to 8 bit
