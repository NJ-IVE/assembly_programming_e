# 02.asm - stack frame with local variables

## What happens

`call work` pushes the return address (8 bytes) and jumps to `work`. The return address is 0x401005, which is the `mov rax, 60` right after the call.

In `work`:

- `push rbp` saves the old RBP (it was 0) and RSP goes down another 8.
- `mov rbp, rsp` makes RBP point at the saved RBP. This is the base of the frame.
- `sub rsp, 16` makes room for two 8-byte locals.

At this point RSP is 32 bytes below where it was at `_start`: 8 for the return address, 8 for RBP, 16 for the locals.

`mov qword [rbp - 8], 100` and `mov qword [rbp - 16], 200` write the locals. Then they get loaded into RAX and RBX. These use RBP as the reference, so RSP doesn't move.

Cleanup goes in reverse:

- `mov rsp, rbp` throws away the locals (RSP up 16).
- `pop rbp` gets the old RBP back (0) and RSP goes up 8.
- `ret` pops the return address into RIP and RSP goes up 8, back to the start.

Then `_start` does the exit syscall with status 0.

| After | RSP | RBP |
|---|---|---|
| start | 0x7fffffff9f80 | 0x0 |
| call work | 0x7fffffff9f78 | 0x0 |
| push rbp | 0x7fffffff9f70 | 0x0 |
| mov rbp, rsp | 0x7fffffff9f70 | 0x7fffffff9f70 |
| sub rsp, 16 | 0x7fffffff9f60 | 0x7fffffff9f70 |
| mov rsp, rbp | 0x7fffffff9f70 | 0x7fffffff9f70 |
| pop rbp | 0x7fffffff9f78 | 0x0 |
| ret | 0x7fffffff9f80 | 0x0 |

## Build and run

```bash
cd 07_stack
nasm -f elf64 -g -F dwarf 02.asm -o 02.o
ld -o 02 02.o
./02
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x401000: file 02.asm, line 5.
(gdb) run
Breakpoint 1, _start () at 02.asm:5
5	    call work
(gdb) p $rsp
$1 = (void *) 0x7fffffff9f80
(gdb) p $rbp
$2 = (void *) 0x0
(gdb) stepi
work () at 02.asm:11
11	    push rbp
(gdb) p $rsp
$3 = (void *) 0x7fffffff9f78
(gdb) x/gx $rsp
0x7fffffff9f78:	0x0000000000401005
(gdb) stepi
12	    mov rbp, rsp
(gdb) p $rsp
$4 = (void *) 0x7fffffff9f70
(gdb) stepi
13	    sub rsp, 16
(gdb) p $rbp
$5 = (void *) 0x7fffffff9f70
(gdb) stepi
15	    mov qword [rbp - 8], 100
(gdb) p $rsp
$6 = (void *) 0x7fffffff9f60
(gdb) stepi 4
20	    nop
(gdb) x/2gd $rbp-16
0x7fffffff9f60:	200	100
(gdb) p $rax
$7 = 100
(gdb) p $rbx
$8 = 200
(gdb) stepi 2
23	    pop rbp
(gdb) p $rsp
$9 = (void *) 0x7fffffff9f70
(gdb) stepi
24	    ret
(gdb) p $rsp
$10 = (void *) 0x7fffffff9f78
(gdb) p $rbp
$11 = (void *) 0x0
(gdb) stepi
_start () at 02.asm:6
6	    mov rax, 60
(gdb) p $rsp
$12 = (void *) 0x7fffffff9f80
(gdb) continue
[Inferior 1 (process 175) exited normally]
```

## Notes

- `x/2gd $rbp-16` shows 200 first because [rbp-16] is the lower address. The locals are below RBP.
- The value on the stack right after `call` is 0x401005. That's the instruction after the call, so that's where `ret` goes back to.
- RBP is 0 at `_start` and it's 0 again after `pop rbp`, so the function put it back.
