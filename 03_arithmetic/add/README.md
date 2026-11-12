# ADD

## add1.asm

`120 + 10` using 8-bit registers. AL ends up as `0x82` (130).

GDB: `[ PF AF SF IF OF ]`

- **OF = 1**: 120 and 10 are both positive, but the result has bit 7 set, so as a signed byte it reads as -126. 130 is too big for a signed byte (max is 127), so it overflowed.
- **SF = 1**: bit 7 of the result is 1.
- **CF = 0**: 130 is still under 255, so there's no unsigned carry.
- **AF = 1**: the low nibbles 8 + A = 0x12, so there's a carry from bit 3 into bit 4.
- **PF = 1**: 0x82 is `10000010`, which has two 1s (even).
- **ZF = 0**: the result isn't zero.

So the answer is fine if you treat it as unsigned (CF=0) but wrong if you treat it as signed (OF=1).

## add2.asm

`32000 + 500` using 16-bit registers. AX = `0x7EF4` (32500).

GDB: `[ IF ]`, so none of the arithmetic flags are set.

- **CF = 0**: 32500 fits in 16 bits.
- **OF = 0**: 32500 is below 32767, so it's still a valid positive signed number.
- **SF = 0**: bit 15 is 0.
- **ZF = 0**: not zero.
- **AF = 0**: 0 + 4 in the low nibble, no carry.
- **PF = 0**: low byte 0xF4 = `11110100` has five 1s (odd).

## add3.asm

`0xFFFF + 1`, then `adc ax, 0`.

After `add`: AX = `0x0000`. GDB: `[ CF PF AF ZF IF ]`

- **CF = 1**: 65535 + 1 = 65536 needs 17 bits, so the extra bit goes into CF.
- **ZF = 1**: what's left in AX is 0.
- **AF = 1**: F + 1 carries out of the low nibble.
- **PF = 1**: low byte is 0x00, which has zero 1s (even).
- **OF = 0**: as signed numbers this is -1 + 1 = 0, which is correct.
- **SF = 0**: bit 15 is 0.

After `adc ax, 0`: ADC adds CF too, so 0 + 0 + 1 = 1. GDB: `[ IF ]`

All the flags clear because the result is just 1. 