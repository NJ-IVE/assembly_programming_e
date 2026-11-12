# 03.asm - call and ret

## What happens

`mov rax, 10` puts 10 in RAX.

`call calculate` pushes the return address and jumps to `calculate`. RSP goes down 8. The address it pushes is 0x40100a, which is the `after_call` label (checked with `nm` below).

`add rax, 5` makes RAX 15. This doesn't touch the stack.

`ret` pops 0x40100a into RIP, so execution continues at `after_call`, and RSP goes back up 8.

The return address is the only thing this program puts on the stack, so it only ever uses 8 bytes. RAX = 15 at `after_call`, which matches the comment in the code. Then the exit syscall overwrites RAX with 60 and exits with status 0.

| Point | RAX | RSP |
|---|---|---|
| start | - | 0x7fffffff9f80 |
| inside calculate | 10 | 0x7fffffff9f78 |
| after add | 15 | 0x7fffffff9f78 |
| after_call | 15 | 0x7fffffff9f80 |

## Build and run

```bash
cd 07_stack
nasm -f elf64 -g -F dwarf 03.asm -o 03.o
ld -o 03 03.o
./03
echo $?
```

```text
0
```

The program exits with status 0 and prints nothing.

```bash
nm 03
```

```text
0000000000402000 T __bss_start
0000000000402000 T _edata
0000000000402000 T _end
0000000000401000 T _start
000000000040100a t after_call
0000000000401015 t calculate
```

## GDB

```text
(gdb) break _start
Breakpoint 1 at 0x401000: file 03.asm, line 5.
(gdb) run
Breakpoint 1, _start () at 03.asm:5
5	    mov rax, 10
(gdb) p $rsp
$1 = (void *) 0x7fffffff9f80
(gdb) stepi 2
calculate () at 03.asm:17
17	    add rax, 5
(gdb) p $rax
$2 = 10
(gdb) p $rsp
$3 = (void *) 0x7fffffff9f78
(gdb) x/gx $rsp
0x7fffffff9f78:	0x000000000040100a
(gdb) stepi
18	    ret
(gdb) p $rax
$4 = 15
(gdb) stepi
after_call () at 03.asm:10
10	    nop
(gdb) p $rsp
$5 = (void *) 0x7fffffff9f80
(gdb) p $rax
$6 = 15
(gdb) continue
[Inferior 1 (process 188) exited normally]
```

## Notes

- `x/gx $rsp` inside `calculate` shows 0x40100a, and `nm` shows that's `after_call`. So `call` really just pushes the address of the next instruction.
- `calculate` passes its result back in RAX. It doesn't use the stack for that.
