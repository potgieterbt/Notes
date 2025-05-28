### FUN_ffffffffce0d8540:
* return (uint)((ulong)\*(undefined8 \*)(param_1 + 0x60) >> 0x3e) & 1;
* 0x60 = 01100000
* 0x3e = 00111110 = 62
* param1 + 0x60 = xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx + 01100000
* (undefined8 \*)(param_1 + 0x60) dereferences a pointer (param_1 + 0x60)
* (ulong)\* dereferences a the the value at pointer above
* >> 0x3e bit shiftes right by 62 giving top 2 bits
* & 1 isolates lowest bit 
* 