# DIV

## div1.asm

`100 / 7`. AX = `0x020E`, so AL = 14 (quotient) and AH = 2 (remainder).

Flags: `[ IF ]` before, `[ AF IF ]` after. The AF doesn't mean anything here, since DIV's flags are undefined.

## div2.asm

`50000 / 300`. AX = 166 (quotient) and DX = 200 (remainder).

Flags: `[ IF ]` before, `[ AF IF ]` after.

DX has to be set to 0 first, because the dividend is DX:AX. If DX had garbage in it, the quotient could be too big for AX and the program would crash.

## div3.asm

`300,000,000 / 1000`. EAX = 300,000 and EDX = 0.

Flags: `[ IF ]` before, `[ AF IF ]` after.

The remainder here is 0, but ZF still isn't set. That shows the flags after DIV don't describe the result.
