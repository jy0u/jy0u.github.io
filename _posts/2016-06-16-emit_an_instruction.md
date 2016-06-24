---
layout: post
title: Emit An Instruction in LLVM
---

這[一篇文章](http://eli.thegreenplace.net/2012/11/24/life-of-an-instruction-in-llvm)解釋一個instruction是如何從LLVM IR轉變成machine code的。

最讓人覺得有用的是作者不是是拿一個很簡單的範例，而是拿特殊限制的target instruction來解釋，這把instruciton selection需要做特殊處理和register allocation的部分也帶出。
簡單來說，跟某些register綁定的instruction是可以在instruction selection時，把需要對映操作的register (virtual registers)直接換成所綁定的registers，剩下無綁定限制的再讓register allocator去處理。

這可對LLVM鮮明的phase/pass設計中所保留的彈性又有更清楚的理解。
