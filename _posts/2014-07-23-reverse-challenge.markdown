---
layout: post
title: "Coursera Reverse Challenge"
---

Last month the Coursera course 'Malicious Software and its Underground Economy: Two Sides to Every Story' finished. 

The course was run over 6 weeks and included a bunch of video lectures each week followed by additional reading and a quiz. You can find an overview of the course [here](https://www.coursera.org/course/malsoftware). It looks like this was the second time the course has been run -  which is probably about right, I'm normally late to the party. 

There wasn't as much hands on reversing engineering as I had hoped for - there was a video walk through of reversing a binary and then a 'challenge' as part of the final quiz. Since the course has now finished it's probably safe to post my write up of the final reversing challenge.

<!-- more -->

In a nutshell a download link was provided to a file and the challenge was to determine the correct password. You can find a locally cached copy of the challenge file [here](/downloads/reverse-challenge)


Lets start by running the command **file** on the executable to confirm what type of file it is ( and if they were being kind and had left any debugging symbols :-)



		rich@ubuntu:~$ file reverse-challenge
		reverse-challenge: ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=43cb53541a366b39ab4b73f31a4fac8961ad959c, stripped


And then run strings to look for anything interesting...



		rich@ubuntu:~$ strings reverse-challenge
		...
		/tmp
		/b2.
		XXXXf
		/tmp
		/b1.
		XXXXf
		[^_]
		@EHJ~@DZEL
		woot!
		134.219.148.8
		Dude, no debugging ;-)
		Are you feeling lucky today? 
		[+] WooT! THE KEY IS: %s
		[+] If you want, now open a terminal and type:
		echo "%s" | nc %s %d
		[~] Maybe.
		[-] Nope.



The string **@EHJ~@DZEL** looks very much like an XOR'ed string ( we'll come back to this shortly ) lets now run the executable via IDA Pro's build in debugger and see what happens...

![My helpful screenshot](/public/img/reverse-challenge1.png)

Whoa thats not cool bro! It seems that the executable bombs out if it detects the use of a debugger! Obviously some anti debugging protection going on here...lets strace this file to see whats going on ( or we could just read the source in IDA Pro, but strace is cool right? )



		rich@ubuntu:~$ strace ./reverse-challenge 
		...
		ptrace(PTRACE_TRACEME, 3639303790, 0xbff9fd34, 0) = -1 EPERM (Operation not permitted)
		fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 11), ...}) = 0
		mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb772f000
		write(1, "Dude, no debugging ;-)\n", 23Dude, no debugging ;-)
		...


This a common anti-analysis technique that works on the basis that the executable checks to see if it can debug itself. If the executable is already running in a debugger then the call to ptrace() fails since there can be only be one debugger active at a time - the executable then returns an error, displays the "Dude no debugging" message and quits. Lets look at the code in IDA Pro and figure out a way to bypass or remove this anti debugging...

![My helpful screenshot](/public/img/reverse-challenge2.png)



There are numerous ways to defeat this...we could zero ( nop ) out the:



		mov al, 1Ah



or write our own ptrace() function that always returns 0 and load it via LD_PRELOAD or since the jump at the end is conditional (jump if not zero) we could edit the code to change this to an unconditional jump. I chose the last option, but all are just as good and simple to implement. IDA Pro has a patch function so we can easily change the byte code to alter the **jnz** and change it to **jmp**. Lets change:


![My helpful screenshot](/public/img/reverse-challenge3.png)

to

![My helpful screenshot](/public/img/reverse-challenge4.png)


The exit function should never be reached now as the unconditional jump should completely ignore it and we should be able to run the program in a debugger without any issues


![My helpful screenshot](/public/img/reverse-challenge5.png)


Now we take take a look at what the executable is doing. A good place to start is the "Are you feeling lucky today?" string, since it's a fair assumption that the executable will be looking for input at this point which will determine the output of the program.  We see a bunch of **cmp** and **jz**'s that look interesting...


![My helpful screenshot](/public/img/reverse-challenge6.png)

Lets also have a look at the **@EHJ~@DZEL** string that looks XOR like-ish


![My helpful screenshot](/public/img/reverse-challenge7.png)


This last section is the password checking function which, with review, does the following:

+ Takes a character of our user entered password at **08048779** into **eax** and then at **0804877C** adds one to it ( in hex ) 
+ At **0804878C 0xFA** is assigned to edx ( see **0804876C** for the initial memory assignment ) and is then bitwise ADD'ed with 0x2b 
+ The result of which is then xor'ed with the vaule in eax (a character from our password). This process loops around until each byte of our password has been xor'ed and is them compared with the **@EHJ~@DZEL** string.

Lets reverse the process to work out what our password needs to be. First we need to bitwise ADD 0x2b and 0xfa to work out what we need to XOR our string with - in python we can do:


		>>> print 0x2b & 0xfa
		42


 We can then XOR the string @EHJ~@DZEL with 42:


		>>> print "".join([chr(ord(c) ^ 42) for c in '@EHJ~@DZEL'])
		job`Tjnpof


Finally we now need to account for that +1 at **0804877C** ( so we minus one here, since we are reversing the process )


		>>> print "".join([chr(ord(c) -1 ) for c in 'job`Tjnpof'])
		ina_Simone



At this point we have our password decryption routine, but have not yet worked out how to trigger it. Lets go back to the code after **"Are you feeling lucky today"** and those compare and jump statements:

![My helpful screenshot](/public/img/reverse-challenge8.png)


We know that:

+ 0x43 in ASCII is C ( the compare at 08048AF4 )
+ 0x4E in ASCII is N ( the compare at 08048AF9 )
+ 0x41 in ASCII is A ( the compare at 08048AFE )

This function takes our user entered string, take the first charters and depending on if that character is a C, N, A or another jumps to different areas of code. This pretty much tells us what our password is. Of course if it wasn't obvious we could try all three and step through each code execution branch in a debugger, but at this point it's pretty logical to assume the password is **Nina_Simone**...lets try it:

![My helpful screenshot](/public/img/reverse-challenge9.png)

Rock and roll.... There a bunch of code left we can reverse, like the routine that writes a file to /tmp but I'll leave that for the interested reader to do themselves

