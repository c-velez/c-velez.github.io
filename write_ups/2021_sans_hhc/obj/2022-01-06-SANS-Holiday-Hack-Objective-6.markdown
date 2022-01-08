---
layout: post
# title: "Shellcode Primer"
date: 2022-01-06 21:47 -500
categories: jekyll update
---

# Shellcode Primer

**Objective**: Learn about assembly, shellcode, and have fun. Oh... and we need to read the contents of /var/nothpolesecrets.txt using shellcode  
**Location**: Jack Frost Tower - Jack's Office  
**Hint Terminal**: [Chimney Scissorsticks](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Holiday-Hero)  
**Hints**:
1. Shellcode Primer Primer: If you run into any shellcode primers at the North Pole, be sure to read the directions and the comments in the shellcode source.
2. Debugging Shellcode: Also, troubleshooting shellcode can be difficult. Use the debugger step-by-step feature to watch values.
3. Register Stomping: Lastly, be careful not to overwrite any register values you need to reference later on in your shellcode.

## Write Up

Ruby Cyster seems to think that someone is trying to learn how to hack the North Pole. Shellcoding is a fundamental skill for any exploit developer, so she may be on to something... Let's load the terminal and see what we have to work with.

When we first open the terminal, we are greeted with some introductory instructions.

![Welcome](/assets/img/2021_sans_hhc/obj/obj06/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Welcome Training Interface</strong></p>

The primer walks you through Tasks 1 to 10, so we won't spend time on those. Task 11 is a capstone that rolls up everyting that should have been learned in the previous tasks, with a little bit of extra, so we'll skip right to task 11.

The best way to approach a problem is to tackle it block by block. In order to print the contents of /var/northpolesecrets.txt to stdout, there are 4 discrete steps we need to take: 1) we need to open the file, 2) read the file, 3) write the file to stdout, and 4) exit cleanly

So let's approach this one block at a time starting with `sys_open`

```
01 | ; sys_open
02 |	
03 | mov  rax, 2   ; First we set RAX to the correct syscall number using the linux syscall table 
		     that was provided
04 | call get_addr ; Then we want to get a reference to the file we want. We use the techniques 
                     that were taught in the previous sections
05 | db   '/var/northpolesecrets.txt', 0 		
06 |	
07 | get_addr:	; Here is the start of our function call from line 04
08 |  pop rdi	; This will pop the address of the reference for line 5 in to RDI (filename)
09 |	
10 | mov rsi, 0 ; We don't need flags for the call so we can set it to 0
11 | mov rdx, 0 ; We don't need any special modes for this call, so we can set this to zero as 
                  well
12 |	
13 | syscall 	; Open the file!
```

According to the syscall table, `sys_read` needs RAX  set to 0, RDI pointing to our file descriptor, RSI pointing a buffer, and RDX which will hold the expected size of the contents of the file.

sys_open will return the file descriptor to rax, so we'll need to do something with that when we read.

```
14 | ; sys_read
15 |
16 | push rax             ; Push the file descriptor on to the stack so we don't overwrite it
17 | sub  rsp, 16         ; Build space in the stack for our buffer
18 |	
19 | mov  rax, 0          ; Set RAX to the correct syscall number, in this case it will be 0
20 | mov  rdi, [rsp + 16] ; Set RDI to the location of the file descriptor. In line 15, we pushed 
                            the fd on to the stack, and in line 16, we moved the stack pointer to
                            make space for the buffer. This is why we have to offset the address 
                            by 16, hence [rsp + 16]
21 | mov rsi, rsp         ; We want to point RSI to the top of the stack, so that it can read in 
                            to the space
22 | mov rdx, 256         ; Hopefully this is enough bytes to read the whole file
23 |
24 | syscall              ; Make the call!
```

Now all that is left is writing to stdout and closing out cleanly. We'll accomplish both of these in this last block

```
25 | ;sys_write
26 |	
27 | mov rax, 1   ; Set RAX to the correct syscall number, for `sys_write` we'll need to set 
                    it to 1
28 | mov rdi, 1   ; Set RDI to the file descriptor to write to, in this case, 1 is std out
29 | mov rdx, 256 ; Set RDX to the expected size of the contents of the file
30 |              ; When we make the call, the contents of the stack should be read to 
                    standard out
31 | syscall      ; Make the call
32 |
33 | ; sys_exit
34 |
35 | mov rax, 60  ; Set RAX to `sys_exit`
36 | mov rdi, 99  ; Set error code to 99
37 | 	
38 | syscall      ; Send it!
```

At this point, we'll learn the secret to KringleCon success!


![Santa's Message](/assets/img/2021_sans_hhc/obj/obj06/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Santa's Message</strong></p>

Answer: Secret to KringleCon success: all of our speakers and organizers, providing the gift of CYBER SECURITY KNOWLEDGE, free to the community.

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-5">< Objective 5</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-7">Objective 7 ></a></p>