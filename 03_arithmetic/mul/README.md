# MUL


## mul1.asm

`25 × 10` = 250. AX = `0x00FA`, so AH = 0.

GDB: `[ IF ]`

- **CF = 0, OF = 0**: 250 fits in one byte, so AH is 0 and nothing spilled into the top half.

## mul2.asm

`3000 × 200` = 600,000. That's too big for 16 bits, so DX = `0x0009` and AX = `0x27C0`.

GDB: `[ CF IF OF ]`

- **CF = 1, OF = 1**: DX isn't 0, so the answer needed both registers. If you only looked at AX you'd get 10,176, which is incorrect. That's why the program saves both AX and DX.

## mul3.asm

`100000 × 300000` = 30,000,000,000. EDX = `0x6` and EAX = `0xFC23AC00`.

GDB: `[ CF IF OF ]`

- **CF = 1, OF = 1**: 30 billion is way more than 32 bits can hold, so EDX holds the extra part.
