# 05.asm - full 64-bit values on the stack

## What happens

RAX gets 0x1122334455667788 and RBX gets 0x1234.

`push rax` moves RSP down 8 and stores the big value. `push rbx` moves RSP down another 8 and stores 0x1234, which is now on top.

`pop rbx` takes the top value (0x1234) back into RBX, and `pop rax` takes 0x1122334455667788 back into RAX. Each pop moves RSP up 8, so RSP ends up back at the start.

This time the pops go in reverse order of the pushes (push rax, push rbx, then pop rbx, pop rax). Because of that, each register gets its own value back. In 01.asm the pops were in a different order and the values ended up in RCX and RDX instead.

Then the exit syscall runs with status 0.

| After | RSP | Top of stack |
|---|---|---|
| start | 0x7fffffff9f80 | - |
| push rax | 0x7fffffff9f78 | 0x1122334455667788 |
| push rbx | 0x7fffffff9f70 | 0x1234 |
| pop rbx | 0x7fffffff9f78 | 0x1122334455667788 |
| pop rax | 0x7fffffff9f80 | - |

## Build and run

```bash
cd 07_stack
nasm -f elf64 -g -F dwarf 05.asm -o 05.o
ld -o 05 05.o
./05
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x401000: file 05.asm, line 5.
(gdb) run
Breakpoint 1, _start () at 05.asm:5
5	    mov rax, 0x1122334455667788
(gdb) p $rsp
$1 = (void *) 0x7fffffff9f80
(gdb) stepi 3
_start () at 05.asm:9
9	    push rbx
(gdb) p $rsp
$2 = (void *) 0x7fffffff9f78
(gdb) stepi
_start () at 05.asm:10
10	    nop
(gdb) p $rsp
$3 = (void *) 0x7fffffff9f70
(gdb) x/2gx $rsp
0x7fffffff9f70:	0x0000000000001234	0x1122334455667788
(gdb) x/16bx $rsp
0x7fffffff9f70:	0x34	0x12	0x00	0x00	0x00	0x00	0x00	0x00
0x7fffffff9f78:	0x88	0x77	0x66	0x55	0x44	0x33	0x22	0x11
(gdb) stepi 2
_start () at 05.asm:13
13	    pop rax
(gdb) p/x $rbx
$4 = 0x1234
(gdb) p $rsp
$5 = (void *) 0x7fffffff9f78
(gdb) stepi
_start () at 05.asm:15
15	    mov rax, 60
(gdb) p/x $rax
$6 = 0x1122334455667788
(gdb) p $rsp
$7 = (void *) 0x7fffffff9f80
(gdb) continue
[Inferior 1 (process 214) exited normally]
```

## Notes

- 0x1234 is small but it still takes up a full 8-byte slot (34 12 and then six 00 bytes). A 64-bit push always pushes 8 bytes.
- The byte dump is backwards from how the numbers are written: 0x1122334455667788 is stored as 88 77 66 55 44 33 22 11. That's little-endian, the lowest byte goes at the lowest address.
