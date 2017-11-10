---
title: Matters Computational
---

```cpp
!(a&&b)     // a == 0 || b == 0
!(a|b)      // a == 0 && b == 0
(!a)^(!b)   // a == 0 && b != 0 || a != 0 && b == 0

(x&y)+((x^y)>>1)    // (x + y) / 2
(x|y)-((x^y)>>1)    // (x + y) / 2 if x >= y

t = a ^ b; x ^= t   // toggle x between a, b
t = a + b; x = t - x

(x|1)+1     // next even
(x-1)&~1    // prev even
(x+1)|1     // next odd
(x&~1)-1    // prev odd
(x+1)&~1    // next even if odd else self
x&~1        // prev even if odd else self
x|1         // next odd if even else self
(x-1)|1     // prev odd if even else self

#define DOUBLE2INT(i,d) { double t = ((d) + 6755399441055744.0); i = *((int *)(&t)); }
// 6755399441055744 = 2^52 + 2^51

a&(1<<i)        // test bit
(a&(1<<i)) != 0 // if we need 0/1
a|(1<<i)        // set bit
a&~(1<<i)       // clear bit
a^(1<<i)        // toggle bit

a ^= (((a>>isrc)^(a>>idst))&1)<<idst    // copy bit from isrc to idst
x = mdst; if (msrc & a) x = 0; x^= mdst; a &= ~mdst; a |= x
// copy bits with mask from msrc to mdst (-> CMOV)

a^((1<<k1)^(1<<k2)) // swap bits at k1, k2

x & -x          // x..x10..0 -> 0..010..0
x = ~x; x & -x  // x..x01..1 -> 0..010..0
x & (x-1)       // x..x10..0 -> x..x00..0
x | (x+1)       // x..x01..1 -> x..x11..1

asm("bsfq eax, esi")    // bit scan forward(<-), get the last 1's index

l = x & -x; y = x + 1; x ^= y; x & (x>>1) // x..x01..10..0 -> 0..01..10..0

x | -x          // x..x010..0 -> 1..10..0
x ? x^(x-1) : 0 // x..x010..0 -> 0..01..1
```
