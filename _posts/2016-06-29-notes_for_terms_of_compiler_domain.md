---
layout: post
title: Notes for terms of compiler domain
---
Just notes for me to reference.

* Rematerializable:

> It usually means that the result of the operation can be  
> computed at compile time and need not be assigned to a  
> register, for example  
>
> itemp1 = & Some Global variable  
> itemp2 = itemp1 + 20;
> 
> Then itemp2 can be computed at compile time and the  
> addition need not be done during execution.  
> @[orginal page](https://sourceforge.net/p/sdcc/mailman/message/5474241/)
>  
> Rematerialization is a strategy for register allocator to determine if it is cheaper to recompute the value than to use memory load/store.


* PHI instruction

> an easier explanation,  
> http://www.cnblogs.com/ilocker/p/4897325.html
> http://www.cnblogs.com/ilocker/p/4892439.html
