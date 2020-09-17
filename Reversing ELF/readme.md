# Reversing ELF

My first reverse engineering box (but not my first time reverse engineering). Lets see what we can do!

## Crackme1

Using the ```file``` command I got the following:

```bash
➜  Downloads file crackme1
crackme1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=672f525a7ad3c33f190c060c09b11e9ffd007f34, not stripped
```

So we can see that we are working with a 64 bit ELF binary. An ELF binary is the default Unix Binary, synonymous to a .exe for windows. ELF stands for Executable and Linkable Format. When I tried to run this program on my Mac, I got a ```exec format error``` which pointed to it not being able to run on a Mac, but specifically a Linux specific machine. Time to boot up my Ubuntu 19.04. 

Initially I wasn't able to run the program because I didn't have permissions. Running sudo didn't work either. I had to end up giving the program ```chmod +x``` permissions to run it. 

```bash
osboxes@osboxes.org: ~ Downloads$ chmod +x crackme1
osboxes@osboxes.org: ~ Downloads$ ./crackme1
flag{REDACTED}

## Crackme2
THe 2nd level involved us finding a "super secret password" which I had 0 clue how to obtain. Through some googling, I found a command ```strings``` which will find text strings embedded in executables. I decided to flex distro's and  boot up Centos 7 box:

```bash
strings crackme2
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
puts
printf
memset
strcmp
__libc_start_main
/usr/local/lib:$ORIGIN
__gmon_start__
GLIBC_2.0
PTRh
j3jA
[^_]
UWVS
t$,U
[^_]
Usage: %s password
super_secret_password
```

Well well well, look what we have here. This user needs to pick more secure passwords to use :P


```bash
[osboxes@osboxes.org: ~ Downloads]$ ./crackme2 super_secret_password
Access Granted
flag{REDACTED}
```

## Crackme3

### Strings
Now that I have a better understanding of some of the processes regarding reverse engineering, I can use them! Lets start with ```strings```:

```bash
➜  Downloads strings crackme3 
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
puts
strlen
malloc
stderr
fwrite
fprintf
strcmp
__libc_start_main
GLIBC_2.0
PTRh
D$
  )
iD$$
D$,;D$ 
UWVS
[^_]
Usage: %s PASSWORD
malloc failed
ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==
Correct password!
Come on, even my aunt Mildred got this one!
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
;*2$"8
GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.ctors
.dtors
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
```

We have a couple clues here to process. Trying to input "ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==" and "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/" as passwords give me a ```Permission Denied```. Trying those passwords without the 2 symbols at the end didn't work either. 

### Dynamic Analysis

Since that didn't get us anywhere (I think) it was time to utilize ltrace and strace to see what we can find:

