# SUB

## sub1.asm

`50 - 80` using 8-bit registers. AL = `0xE2`, which is -30 in two's complement.

GDB: `[ CF PF SF IF ]`

- **CF = 1**: 50 is less than 80, so it had to borrow.
- **SF = 1**: the result is negative (bit 7 is 1).
- **OF = 0**: both numbers are positive, and subtracting two numbers with the same sign can't overflow. -30 fits in a byte anyway.
- **ZF = 0**: not zero.
- **AF = 0**: low nibbles 2 - 0, no borrow.
- **PF = 1**: 0xE2 = `11100010` has four 1s (even).

## sub2.asm

`1000 - 2000` using 16-bit registers. AX = `0xFC18` (-1000).

GDB: `[ CF PF SF IF ]`

This gives the same flags as sub1, for the same reasons. 1000 < 2000 so CF=1, the result is negative so SF=1, and there's no signed overflow so OF=0. PF only looks at the low byte, 0x18 = `00011000`, which has two 1s, so PF=1.

## sub3.asm

`0 - 1`, then `sbb ax, 0`.

After `sub`: AX = `0xFFFF` (-1). GDB: `[ CF PF AF SF IF ]`

- **CF = 1**: 0 - 1 needs a borrow.
- **AF = 1**: the low nibble 0 - 1 also needs a borrow.
- **SF = 1**: the result is negative.
- **PF = 1**: 0xFF has eight 1s.
- **OF = 0** and **ZF = 0**: -1 is a valid result and isn't zero.

After `sbb ax, 0`: SBB subtracts CF as well, so 0xFFFF - 0 - 1 = `0xFFFE` (-2). GDB: `[ SF IF ]`

- **CF = 0**: no new borrow this time.
- **SF = 1**: still negative.
- **PF = 0**: 0xFE has seven 1s (odd).
- Everything else is 0.

