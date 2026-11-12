# 06.asm - 32-bit values on the stack

This is the 32-bit version of 05.asm.

## What happens

EAX gets 0x11223344 and EBX gets 0x1234.

`push eax` moves ESP down 4 and `push ebx` moves it down another 4. 0x1234 is on top.

`pop ebx` gets 0x1234 back and `pop eax` gets 0x11223344 back. Each pop moves ESP up 4, so it ends where it started. Like in 05.asm, the pop order is the reverse of the push order, so each register gets its own value back.

`mov eax, 1` and `xor ebx, ebx` set up the exit call and `int 0x80` exits with status 0.

| After | ESP | Top of stack |
|---|---|---|
| start | 0xffff93a0 | - |
| push eax | 0xffff939c | 0x11223344 |
| push ebx | 0xffff9398 | 0x1234 |
| pop ebx | 0xffff939c | 0x11223344 |
| pop eax | 0xffff93a0 | - |

## Build and run

```bash
cd 07_stack
nasm -f elf32 -g -F dwarf 06.asm -o 06.o
ld -m elf_i386 -o 06 06.o
./06
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x8049000: file 06.asm, line 6.
(gdb) run
Breakpoint 1, _start () at 06.asm:6
6	    mov eax, 0x11223344
(gdb) p $esp
$1 = (void *) 0xffff93a0
(gdb) stepi 3
10	    push ebx
(gdb) p $esp
$2 = (void *) 0xffff939c
(gdb) stepi
11	    nop
(gdb) p $esp
$3 = (void *) 0xffff9398
(gdb) x/2wx $esp
0xffff9398:	0x00001234	0x11223344
(gdb) x/8bx $esp
0xffff9398:	0x34	0x12	0x00	0x00	0x44	0x33	0x22	0x11
(gdb) stepi 2
14	    pop eax
(gdb) p/x $ebx
$4 = 0x1234
(gdb) p $esp
$5 = (void *) 0xffff939c
(gdb) stepi
16	    mov eax, 1
(gdb) p/x $eax
$6 = 0x11223344
(gdb) p $esp
$7 = (void *) 0xffff93a0
(gdb) continue
[Inferior 1 (process 227) exited normally]
```

## Notes

- The two pushes only use 8 bytes total, half of 05.asm, because each slot is 4 bytes.
- The bytes are little-endian here too: 0x1234 is 34 12 00 00 and 0x11223344 is 44 33 22 11.
