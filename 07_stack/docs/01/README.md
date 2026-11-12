# 01.asm - push and pop

## What happens

The program puts 100 in RAX and 200 in RBX. Neither mov touches the stack.

`push rax` moves RSP down 8 bytes and writes 100 there. `push rbx` moves RSP down another 8 and writes 200. So 200 is on top and 100 is under it.

`pop rcx` takes the top value (200) into RCX and moves RSP up 8. `pop rdx` takes the next one (100) into RDX and moves RSP up 8 again. RSP is back where it started.

So the values come out in the opposite order they went in. RAX went in first and came out last, into RDX.

The nops don't change anything. At the end `mov rax, 60` and `xor rdi, rdi` set up the exit syscall with status 0.

| After | RSP | Top of stack |
|---|---|---|
| start | 0x7fffffff9f80 | - |
| push rax | 0x7fffffff9f78 | 100 |
| push rbx | 0x7fffffff9f70 | 200, then 100 |
| pop rcx | 0x7fffffff9f78 | 100 |
| pop rdx | 0x7fffffff9f80 | - |

## Build and run

```bash
cd 07_stack
nasm -f elf64 -g -F dwarf 01.asm -o 01.o
ld -o 01 01.o
./01
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x401000: file 01.asm, line 5.
(gdb) run
Breakpoint 1, _start () at 01.asm:5
5	    mov rax, 100
(gdb) p $rsp
$1 = (void *) 0x7fffffff9f80
(gdb) stepi 3
_start () at 01.asm:9
9	    push rbx
(gdb) p $rsp
$2 = (void *) 0x7fffffff9f78
(gdb) x/gd $rsp
0x7fffffff9f78:	100
(gdb) stepi
_start () at 01.asm:10
10	    nop
(gdb) p $rsp
$3 = (void *) 0x7fffffff9f70
(gdb) x/2gd $rsp
0x7fffffff9f70:	200	100
(gdb) stepi 2
_start () at 01.asm:13
13	    pop rdx
(gdb) p $rcx
$4 = 200
(gdb) p $rsp
$5 = (void *) 0x7fffffff9f78
(gdb) stepi
_start () at 01.asm:14
14	    nop
(gdb) p $rdx
$6 = 100
(gdb) p $rsp
$7 = (void *) 0x7fffffff9f80
(gdb) x/2gd $rsp-16
0x7fffffff9f70:	200	100
(gdb) continue
[Inferior 1 (process 162) exited normally]
```

## Notes

- The last `x/2gd $rsp-16` still shows 200 and 100 after both pops. Popping only moves RSP, it doesn't clear the memory.
- The stack uses 16 bytes at most.
