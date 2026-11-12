# 04.asm - 32-bit stack frame

This is the same idea as 02.asm, but in 32-bit. The registers are ESP/EBP and every stack slot is 4 bytes instead of 8.

## What happens

`call work` pushes a 4-byte return address (0x08049005, the `mov eax, 1` after the call). ESP goes down 4.

In `work`:

- `push ebp` saves the old EBP (0). ESP goes down 4.
- `mov ebp, esp` sets up the frame.
- `sub esp, 8` makes room for two 4-byte locals.

ESP is now 16 bytes below the start: 4 return address + 4 EBP + 8 locals.

The two `mov dword` lines write 100 to [ebp-4] and 200 to [ebp-8]. Then they're loaded into EAX and EBX. ESP doesn't move here.

Cleanup:

- `mov esp, ebp` drops the locals (ESP up 8).
- `pop ebp` restores EBP to 0 (ESP up 4).
- `ret` pops the return address (ESP up 4), back to the start.

Back in `_start`, `mov eax, 1` and `xor ebx, ebx` set up the 32-bit exit call, and `int 0x80` exits with status 0.

| After | ESP | EBP |
|---|---|---|
| start | 0xffff93a0 | 0x0 |
| call work | 0xffff939c | 0x0 |
| push ebp | 0xffff9398 | 0x0 |
| mov ebp, esp | 0xffff9398 | 0xffff9398 |
| sub esp, 8 | 0xffff9390 | 0xffff9398 |
| mov esp, ebp | 0xffff9398 | 0xffff9398 |
| pop ebp | 0xffff939c | 0x0 |
| ret | 0xffff93a0 | 0x0 |

## Build and run

```bash
cd 07_stack
nasm -f elf32 -g -F dwarf 04.asm -o 04.o
ld -m elf_i386 -o 04 04.o
./04
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x8049000: file 04.asm, line 6.
(gdb) run
Breakpoint 1, _start () at 04.asm:6
6	    call work
(gdb) p $esp
$1 = (void *) 0xffff93a0
(gdb) p $ebp
$2 = (void *) 0x0
(gdb) stepi
13	    push ebp
(gdb) p $esp
$3 = (void *) 0xffff939c
(gdb) x/wx $esp
0xffff939c:	0x08049005
(gdb) stepi
14	    mov ebp, esp
(gdb) p $esp
$4 = (void *) 0xffff9398
(gdb) stepi
work () at 04.asm:15
15	    sub esp, 8
(gdb) p $ebp
$5 = (void *) 0xffff9398
(gdb) stepi
17	    mov dword [ebp - 4], 100
(gdb) p $esp
$6 = (void *) 0xffff9390
(gdb) stepi 4
22	    nop
(gdb) x/2wd $ebp-8
0xffff9390:	200	100
(gdb) p $eax
$7 = 100
(gdb) p $ebx
$8 = 200
(gdb) stepi 2
25	    pop ebp
(gdb) p $esp
$9 = (void *) 0xffff9398
(gdb) stepi
26	    ret
(gdb) p $esp
$10 = (void *) 0xffff939c
(gdb) p $ebp
$11 = (void *) 0x0
(gdb) stepi
_start () at 04.asm:8
8	    mov eax, 1
(gdb) p $esp
$12 = (void *) 0xffff93a0
(gdb) continue
[Inferior 1 (process 201) exited normally]
```

## Notes

- Every step is 4 bytes here, where 02.asm moved by 8. The whole frame is 16 bytes, half of 02.asm.
- The stack addresses start with 0xffff... because it's a 32-bit program.
- `x/2wd` is needed instead of `x/2gd` because the slots are 4 bytes (w = word).
