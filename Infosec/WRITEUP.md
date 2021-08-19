# **INFOSEC Task**
Firstly, let's start the bash-terminal and get to the folder that contains the _rev_ file. Using the `file rev` command tells us the details about the file.
```
franticalien@Ryuk:~/Downloads$ file rev
rev: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=fe9ddc13d0659e1badb3fd04934d02b4aa60893a, not stripped

```
We can notice that the file is executable. We can then use the `chmod +x rev` so we can execute it from bash. [NOTE: You may have to install relevant 32-bit architectures in case you use a 64-bit system.] Now executing it as `./rev` gives the following custom message.

```
franticalien@Ryuk:~/Downloads$ chmod +x rev
franticalien@Ryuk:~/Downloads$ ./rev
It's not that easy as you think so
franticalien@Ryuk:~/Downloads$ 
```
Notice that it doesn't even ask for a prompt. This means that there must be some code that gets bypassed in the main or maybe some uncalled functions. So we use gdb to start debugging. We use the `gdb rev` to start the debugging process. We then use the `info functions` command to get to know about the functions in this file. This gives the following output.
```
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x08048318  _init
0x08048350  printf@plt
0x08048360  __stack_chk_fail@plt
0x08048370  puts@plt
0x08048380  __gmon_start__@plt
0x08048390  __libc_start_main@plt
0x080483a0  _start
0x080483d0  __x86.get_pc_thunk.bx
0x080483e0  deregister_tm_clones
0x08048410  register_tm_clones
0x08048450  __do_global_dtors_aux
0x08048470  frame_dummy
0x0804849c  print_flag
0x08048571  main
0x08048590  __libc_csu_init
0x08048600  __libc_csu_fini
0x08048604  _fini
(gdb) 

```
Among these functions, the `print_flag` seems like the one we should be after. And there is probably no mention of it in the main. So we'll start by trying to access it.
For this, we'll first have to introduce a breakpoint inside the `main` function before it returns. The custom message that we saw must've been due to the `printf` or `puts`.
Lets start by putting a breakpoint at the address for `puts`. 
```
(gdb) b*0x08048370
Breakpoint 1 at 0x8048370
(gdb) run 
Starting program: /home/franticalien/Downloads/rev 
It's not that easy as you think so[Inferior 1 (process 2801) exited normally]
(gdb) 
```
Well, the program exits normally. So next we set a breakpoint at the `printf` address and try to run again.
```
(gdb) b*0x08048350
Breakpoint 2 at 0x8048350
(gdb) run
Starting program: /home/franticalien/Downloads/rev 

Breakpoint 2, 0x08048350 in printf@plt ()
(gdb) 
```
Success! Now we will jump to the `print_flag` function and proceed. Doing this we get,
```
(gdb) jump print_flag
Continuing at 0x80484a2.

Breakpoint 2, 0x08048370 in puts@plt ()
(gdb) 
```
We now end up at our previously set beakpoint. And this is probably just before it tries to print the flag. We use `continue` to proceed further.
```
(gdb) continue
Continuing.
841f980abd04b26fe804ca0c207a574bef504cb6a3c3599a449e845ca993d2cf

Program received signal SIGSEGV, Segmentation fault.
0x00000000 in ?? ()
(gdb) 
```
Yay! This gives a long string which definently looks lika an sha-256 encoding. We can further check that it indeed has 64-digits. 

WARNING: Huge P.S. ahead !

P.S. Well, I'm not sure why I got that Segmentation fault though. The address is null and the function is also unknown. Searching on the internet I see that segmentation fault may appear if there is an overflow, etc. So just to make sure that I'm not missing anything, I installed the _Ghidra_ software to analyze further. From it, I found out that the address of the `return` statement inside the `print_flag` function is `0x08048570`.

![screenshot](./assets/ghidra.png)

With the addition of another breaking point here, I repeat the above steps again to get the following result.
```
(gdb) kill
Kill the program being debugged? (y or n) y
[Inferior 1 (process 2912) killed]
(gdb) run
Starting program: /home/franticalien/Downloads/rev 

Breakpoint 1, 0x08048350 in printf@plt ()
(gdb) jump print_flag
Continuing at 0x80484a2.

Breakpoint 2, 0x08048370 in puts@plt ()
(gdb) b*0x08048570  (here, I put another breakpoint at the return statement of print_falg function.)
Breakpoint 3 at 0x8048570
(gdb) continue
Continuing.
841f980abd04b26fe804ca0c207a574bef504cb6a3c3599a449e845ca993d2cf

Breakpoint 3, 0x08048570 in print_flag ()
(gdb)
```
Now we can see that the segmentation fault wasn't from anything inside of `print_flag`. So finally, the flag is : `0x841f980abd04b26fe804ca0c207a574bef504cb6a3c3599a449e845ca993d2cf`
