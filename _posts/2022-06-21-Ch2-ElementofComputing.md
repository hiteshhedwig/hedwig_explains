---
layout: post
title: "[Ch-2] The Element of Computing System - Solution"
subtitle: "Solving and practicing"
date: 2022-06-21
author: "Hitesh Kumar"
header-img: "img/post-bg-universe.jpg"
tags: [hdl, arithmetic, basics, logic gates]
---

# Chapter 2 - Boolean Arithmetic
``` 
      Counting is the religion of this generation, its hope and salvation.
                                   ~ Gertrude Stein
```
## Implementations :

### Half Adder :

```
CHIP HalfAdder {
        IN a,b ;
        OUT sum, carry;
        PARTS :
           Or(a=a,b=b, out=sum);
           And(a=a,b=b, out=carry);
}
```

### Full Adder
```
CHIP FullAdder {
        IN a,b,c ;
        OUT sum,
            carry;
        PARTS:
            HalfAdder(a=a,b=b, sum=w1, carry=c1);
            HalfAdder(a=w1, b=c, sum=sum, carry= c2);
            Or(a=w1,b=sum, out=carry);
}
```

### Adder :
```
CHIP Adder16 {
        IN a[16], b[16] ;
        OUT  out[16];

        PARTS :
            HalfAdder(a=a[0], b=b[0], sum=w1, carry=c1);
            Adder(a=a[1], b=b[1], c=w1, sum=w2, carry=c2);
            Adder(a=a[2], b=b[2], c=w2, sum=w3, carry=c3);
            Adder(a=a[3], b=b[3], c=w3, sum=w4, carry=c4);
            Adder(a=a[4], b=b[4], c=w4, sum=w5, carry=c5);
            Adder(a=a[5], b=b[5], c=w5, sum=w6, carry=c6);
            Adder(a=a[6], b=b[6], c=w6, sum=w7, carry=c7);
            Adder(a=a[7], b=b[7], c=w7, sum=w8, carry=c8);
            Adder(a=a[8], b=b[8], c=w8, sum=w9, carry=c9);
            Adder(a=a[9], b=b[9], c=w9, sum=w10, carry=c10);
            Adder(a=a[10], b=b[10], c=w10, sum=w11, carry=c11);
            Adder(a=a[11], b=b[11], c=w11, sum=w12, carry=c12);
            Adder(a=a[12], b=b[12], c=w12, sum=w13, carry=c13);
            Adder(a=a[13], b=b[13], c=w13, sum=w14, carry=c14);
            Adder(a=a[14], b=b[14], c=w14, sum=w15, carry=c15);
            Adder(a=a[15], b=b[15], c=w15, sum=sum, carry=carry);
    }
```
    
### Incrementer :
```
CHIP Inc16 {
    IN in[16];
    OUT out[16];
    PARTS:
        Add16(a=in,b[0]=true,out=out);
}
```
    
    
    
    
    
    

