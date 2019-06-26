---
layout: post
title: "BSides London 2014: Challenge 1"
---
This year I entered my first BSidesLondon challenge. Each year BSidesLondon post challenges in order to win a ticket to the event. The first challenge was to analyze an executable and shutdown the botnet - and in doing so, get the girl and save the world! (OK I made that last bit up, but it's nice to have a story around it)

This post was originally my submission but I didn't win...probably because there are far better people than me submitting reports and in a small way it probably didn't help that I was lazy and didn't bother to decode the password function and just took the password off the stack and shut the botnet down. The good news was that I had a BSides ticket anyway - Yah!.

Reports -> blog posts don't really sit well together which is why this post has sort of has warped into something stuck in between a report and a post, and pasting data from a PDF into markdown sucks a little so the formatting below isn't great - be warned! In addition since the challenge has now ended the website and IRC channels are unavailable, although I have included suggestions on how to replicate yourself locally were appropriate.

You can find a copy of the challenge brief [here](/public/downloads/Mission1.pdf) and the malware file [here](/public/downloads/bot.zip) (the zipfile password is 'malware' )


### 1. Executable Details

<table>
	<tr>
		<th colspan='2'>Hash</th>
	</tr>
<tbody>
	<tr>
		<td>MD5</td>
		<td>ad2e8210ca7c2b4b433b3fba65e87b94</td>
	</tr>
	<tr>
		<td>SHA1</td>
		<td>ed5b7416c6e5a8fcfa9a21fdd59d552bdf69bba7</td>
	</tr>
	<tr>
		<td>SHA256</td>
		<td>a73a6aa23916aff3659394a8f2c39c0bb151549e24e3e2cf10f5ad85556a947c</td>
	</tr>
	<tr>
		<td>SSHDEEP</td>
		<td>768:nFMyDg36D9Gn8bd5HYZeexlkKmWkOf4QmcEMRtVhNX5e:n2yDgqDII49RFwQmctfNk</td>
	</tr>
</tbody>
</table>



<table>
<tr>
<th colspan='2'>File Information</th>
</tr>
<tbody>
<tr>
<td>Size</td>
<td>32.5k (36,864 bytes on disk)</td>
</tr>
<tr>
<td>Type</td>
<td>PE32</td>
</tr>
<tr>
<td rowspan="12">Detections</td>
<td>Generic10_c.AICI</td>
</tr>
<tr>
<td>Gen:Trohan.Heur.RP.cmGfa4xPvndi</td>
</tr>
<tr>
<td>Trojan/Win32.SGeneric</td>
</tr>
<tr>
<td>W32/IRCBot-based!Maximus</td>
</tr>
<tr>
<td>Trojan.Win32.Spy</td>
</tr>
<tr>
<td>RDN/BackDoor-CMQ!e</td>
</tr>
<tr>
<td>Horst.gen32</td>
</tr>
<tr>
<td>Suspicious.Mystic</td>
</tr>
<tr>
<td>Possible_Worm32</td>
</tr>
<tr>
<td>TROJ_GEN>P0CBC0EC114</td>
</tr>
<tr>
<td>Trojan.Downloader.gen.h</td>
</tr>
<tr>
<td>Trojan.Win32.Generic!BT</td>
</tr>
</tbody>
</table>


<table>
	<tr>
		<th colspan='4'>Behaviour Traits</th>
	</tr>
<tbody>
	<tr>
		<td>Alerts Windows Firewall</td>
		<td> </td>
		<td>Hooks Keyboard</td>
		<td> </td>
	</tr>
	<tr>
		<td>Checks for Debugger</td>
		<td><center>X</center></td>
		<td>Injected Code</td>
		<td></td>
</tr>
<tr>
		<td>Copies to Windows</td>
		<td></td>
		<td>Makes Network Connection</td>
		<td><center>X</center></td>
</tr>
<tr>
		<td>Creates DLL</td>
		<td></td>
		<td>Modifies File in System</td>
		<td></td>
</tr>
<tr>
		<td>Creates EXE</td>
		<td></td>
		<td>Modifies Local DNS</td>
		<td></td>
</tr>
<tr>
		<td>Creates Hidden File</td>
		<td><center>X</center></td>
		<td>More than 5 Processes</td>
		<td></td>
</tr>
<tr>
		<td>Creates Mutex</td>
		<td><center>X</center></td>
		<td>Opens Physical Memory</td>
		<td></td>
</tr>
<tr>
		<td>Creates a Service</td>
		<td></td>
		<td>Starts EXE in My Documents</td>
		<td></td>
</tr>
<tr>
		<td>Deletes a File in System</td>
		<td></td>
		<td>Starts EXE in Recycle Bin</td>
		<td></td>
</tr>
<tr>
		<td>Deletes Original Sample</td>
		<td></td>
		<td>Starts EXE in System</td>
		<td></td>
</tr>
</tbody>
</table>

<table>
	<tr>
		<th colspan='4'>Behavioural Traits</th>
	</tr>
<tbody>
	<tr>
		<th><center>Domain</center></th>
		<th><center>IP Address</center></th>
		<th><center>Port</center></th>
		<th><center>Commands</center></th>
	</tr>
	<tr>
		<td>ghowen.me</td>
		<td>109.109.239.243</td>
		<td>80</td>
		<td>GET /malfor/cnc.php?id=xxx&uid=xxx</td>
	</tr>
	<tr>
		<td>irc.efnet.fr</td>
		<td>194.126.217.2</td>
		<td>6667</td>
		<td> USER uop25861 0 * :misery mystery<br/>NICK uop25861</td>
	</tr>
	<tr>
		<td>irc.swepipe.se</td>
		<td>88.80.5.41</td>
		<td>6667</td>
		<td>USER uop35861 0 * :misery mystery<br/>NICK uop25861<br/>JOIN #malford-kansas bubblegum<br/> MODE #malford-kansas +k bubblegum<br/>TOPIC #malfor-kansas :Resistance is futile</td>
	</tr>

</tbody>
</table>



### 2. Summary of Functionality

On startup the executable issues a HTTP GET request to a website to obtain a specific IRC chatroom, and then attempts to connect to one of two IRC servers and join the specific chat room and waits for commands. The executable seems to support the following commands issued in a private message to it:


		HELLO
		IDENT
		SHUTDOWN


No additional files were downloaded, nor does the executable attempt to install or hide itself within the host operating system and can be easily killed via the task manager.


### 3. Anti-Analysis and Packer Stages

Malware authors employ various strategies in an attempt to thwart analysis. The executable was reviewed for the presence of any anti-analysis techniques and the following sections detail what was discovered.


### 3.1 PE Header review

A common approach often used to prevent ‘static’ analysis is to ‘pack’ the executable. This is where the executable is compressed so that all that is actually viewable is the compressed format which hides the true code of the executable. When the executable is run, a small wrapper program embedded at the start of the executable runs to decompress the rest of the packed file.

The PE file format stores interesting information within the headers of the executable that can help identify if the executable has been ‘packed’. PEiD, PE Explorer and PEView are applications that allow review of these headers. Of most interest is the IMAGE_FILE_HEADER and the SECTION headers. Using these tools it is possible to review and determine the names and size of the file headers.

The IMAGE_FILE_HEADER provides a clue to the date and time the executive was created as it contains a ‘time date stamp’ item. This date and time is only suggestive as compilers can set this time to be anything they wish (however it can be useful when reviewing multiple files to determine if there is a correlation - a very close compile time might suggest that the files were created at the same time)


		Bot.exe has a date time stamp of: 2014/02/13 Thu 14:54:24 UTC


The following table shows the PE sections size as shown by PEView. Note that the size of raw data and the virtual size are not aligned which strongly suggests that some ‘packing’ has taken place, and that at runtime the UPX sections will be ‘unpacked’. The section naming also provided evidence that a packer has been used. Normal PE Header sections are .text .data .rdata, and .rsrc


<table>
<tbody>
	<tr>
		<th>Section</th>
		<th>Virtual Size</th>
		<th>Size on Raw Disk</th>
	</tr>
	<tr>
		<td><center>UPX0</center></td>
		<td><center>0000E000</center></td>
		<td><center>00000000</center></td>
	</tr>
	<tr>
		<td><center>UPX1</center></td>
		<td><center>00007000</center></td>
		<td><center>00006E00</center></td>
	</tr>
	<tr>
		<td><center>.rsrc</center></td>
		<td><center>00001000</center></td>
		<td><center>00001000</center></td>
	</tr>

</tbody>
</table>


### 3.2 UPX Packed

UPX is available from http://upx.sourceforge.net and according to the website clams to be the “Ultimate Packer for eXecutables”. The UPX program offers the ability to pack as well as unpack any previously packed executables so unpacking is simply a case of running the executable through UPX with the -d flag to decompress using the following command:

{: .center-image }
![My helpful screenshot](/public/img/malware1.png){: .center-image }


Having run this command the original PE header section are visible and the file can be decompiled and investigated further:

![My helpful screenshot](/public/img/malware2.png){: .center-image }

Having 'unpacked' the executable we are now able to examine the uncompressed code.



The Linux strings command returns each string of printable characters in a given file. It is used to determine the contents of and to extract text from binary files. (Additional information on the strings commands can be found at [http://www.linfo.org/strings.html](http://www.linfo.org/strings.html). Running strings against the executable returned:

```
		http://ghowen.me/malfor/cnc.php?id=
		&uid=
		6667
		malfor
		uop
		%s%d
		misery mystery
		USER %s 0 * :%s
		NICK %s
		Anti-debug
		No malware here, honest guv!
		PING
		001
		Resistance is futile
		bubblegum
		MODE %s +i
		JOIN %s %s
		MODE %s +k %s
		TOPIC %s :%s
		PRIVMSG %s :Hi %s, I'm a bot that's part of the BSides London Challenge.
		IDENT
		PRIVMSG %s :%s / %s
		SHUTDOWN
		PRIVMSG %s :bingo - botnet shutting down
		QUIT :Botnet shutdown
		Botnet shutdown
		Botnet has been shutdown - restart bot?
		PRIVMSG %s :so close yet so far... wrong pw!
		PRIVMSG %s :wrong pw
		#malfor
		irc.efnet.fr
		irc.swepipe.se
		irc.efnet.net
		HCNFJT@
		&Help
		h&About ...
		About malfor-cw
		malfor-cw, Version 1.0
```


These strings can provide a clue as to what the executable does at runtime

### 3.4 Anti Debugger - IsDebuggerPresent

When attempting to run the executable in a debugger the application quits with an error message of ‘No Malware here, honest gov!’ as seen in the following screen shot.

![My helpful screenshot](/public/img/malware3.png){: .center-image }

The Windows API has several functions that can be used by malicious software to determine if a debugger is being used for analysis. The IsDebuggerPresent API function searches the Process Environment Block (PEB) for the field “IsDebugged”, which will return zero if the executable is not running in the context of a debugger or a nonzero value if a debugger is attached.

Decompiling the executable it is possible to locate the following function that calls IsDebuggerPresent which then jumps to a function that sets up a dialog box and then exits if a non zero value is returned

![My helpful screenshot](/public/img/malware4.png){: .center-image }

Typically the easiest way to overcome a call to an anti-debugger API function is to manually modify the executable during execution to not call the function, or modify the flag post call to ensure that a negative response is returned.  Below you can see the call to the function and the return with a nonzero value.

![My helpful screenshot](/public/img/malware12.png){: .center-image }

![My helpful screenshot](/public/img/malware13.png){: .center-image }

We could dynamically patch the return so that it returns zero, or we could patch the binary however Immunity Debugger provides a simple method of doing this with the *!Hidebug IsDebuggerPresent* pyCommand which patches kernel32.dll to return a false flag. This allowed investigations into the executable to continue.


### 4. Core Runtime

Having completed static analysis and bypassed the anti-analysis that is built into bot.exe the executable was then run while attached to a debugger and a network packet sniffer was started. This allowed for both the capture of all network traffic as well as review of the application and memory stack during operation.

The following section details the execution flow of the application and looks at the reverse engineering some of the more important sections

The analysis was done live. Although the executable was run in a virtual machine, DNS, Web, or IRC were not re-directed and ran against the original intended locations. An alternative approach could have been to use a tool such as [apateDNS](http://www.mandiant.com/resources/download/research-tool-mandiant-apatedns) to redirect queries to a server under our control. The simplest method would have been to re-direct traffic to a Kali linux instance running a simple python script and an IRC server - maybe [NgIrcD](http://ngircd.barton.de/documentation.php.en) or you could just:


		sudo apt-get install ircd-irc2


and you should be good to go.

### 5. Windows Registary

Before and after running the bot a registry snapshot was taken with [Regshot](http://sourceforge.net/projects/regshot/). No significant changes were made to the registry by the executable



### 6. HTTP GET Request

On execution bot.exe first issues a HTTP GET request for the following URL


		http://ghowen.me/malfor/cnc.php?id=xx%uid=yy


Where xx is the computer name and yy is the name of the account the application is running under. The servers responds with an IRC chat room The screen shot below shows this request captured via WireShark:

![My helpful screenshot](/public/img/malware5.png){: .center-image }

**Note:** Now the challenge is over the link is no longer valid and the server will return a 404 when queried. In order to replicate this off-line. ApateDNS can be used to re-direct the DNS for ghowen.me to a server under your control. For a simple python interface to return a similar response try something like this


		from twisted.web import server, resource
		from twisted.internet import reactor
		class HelloResource(resource.Resource):
		    isLeaf = True
		    numberRequests = 0
		        def render_GET(self, request):
		        self.numberRequests += 1
		        request.setHeader("content-type", "text/plain")
		        return "#malfor-china" + "\n"
		reactor.listenTCP(80, server.Site(HelloResource()))
		reactor.run()

#### Code stolen from [TwistedWeb](https://twistedmatrix.com)


Alternatively you could let the executable run as is - you'll see a bunch of html error on the stack, but the executable will fall back to attempting to connect to #malfor when all else fails - or you can simply '/msg' the executable direct without joining an IRC chatroom.

When analyzing the software, repeated requests were made to the ghowen.me server over the course of two weeks to try and identify the full list and pattern of group names returned. The following tables lists the dates and the response received when making a request to the website.


<table>
	<tr>
		<th>Date</th>
		<th>Returned Chat Room</th>
	</tr>
<tbody>
	<tr>
		<td><center>Monday 10th March</center></td>
		<td><center>#malfor-france</center></td>
	</tr>
	<tr>
		<td><center>Tuesday 11th March</center></td>
		<td><center>#malfor-peru</center></td>
	</tr>
	<tr>
		<td><center>Wednesday 13th March</center></td>
		<td><center>#malfor-kansas</center></td>
	</tr>
	<tr>
		<td><center>Thursday 14th March</center></td>
		<td><center>#malfor-oz</center></td>
	</tr>
	<tr>
		<td><center>Friday 15th March</center></td>
		<td><center>#malfor-japan</center></td>
	</tr>
	<tr>
		<td><center>Saturday 16th March</center></td>
		<td><center>#malfor-russia</center></td>
	</tr>
	<tr>
		<td><center>Sunday 17th March</center></td>
		<td><center>#malfor-egypt</center></td>
	</tr>
	<tr>
		<td><center>Monday 18th March</center></td>
		<td><center>#malfor-france</center></td>
	</tr>
	<tr>
		<td><center>Tuesday 19th March</center></td>
		<td><center>#malfor-peru</center></td>
	</tr>
	<tr>
		<td><center>Wednesday 20th March</center></td>
		<td><center>#malfor-kansas</center></td>
	</tr>
	<tr>
		<td><center>Thursday 21st March</center></td>
		<td><center>#malfor-oz</center></td>
	</tr>
	<tr>
		<td><center>Friday 22nd March</center></td>
		<td><center>#malfor-japan</center></td>
	</tr>
	<tr>
		<td><center>Saturday 23rd March</center></td>
		<td><center>#malfor-russia</center></td>
	</tr>
	<tr>
		<td><center>Sunday 24th March</center></td>
		<td><center>#malfor-egypt</center></td>
	</tr>
</tbody>
</table>



Based on this two week review it is reasonable to conclude that the website returns a specific chat-room based on the day of the week.

Browsing the website at ghowen.me the domain appears to be owned by Gareth Owen a university lecturer at Portsmouth University in the UK and hosted at a virtual server at [Tagabab](http://www.tagadab.com). This was confirmed by the whois information. While the domain was registered under the company name of ‘XYZ Circuits Ltd’ and and an individual of ‘Larry Bevois', the email address used ties back to Gareth Owen’s gmail email account, used elsewhere. In addition the address used in the registration is also part of the University of Portsmouth campus. The website appears to be 3  - 4 month old according to on-line web archives such at Netcraft.


### 7. IRC Communication

Having gathered the irc chat room information from the ghowen.me website the executable attempts to join the IRC server irc.efnet.fr. Each time the executable was run during analyses this connection was dropped from the server with the error message ‘You are not authorized to use this server’. This is probably because the analysis is being performed in the UK.

![My helpful screenshot](/public/img/malware6.png){: .center-image }

Having failed to connect to irc.efnet.fr the executable then attempts to connect to irc.swepipe.net.

When this connection is successful the executable then sets a username, and joins the password protected chat room that was returned in the website GET request previously. The executable contains the password for the chat room, which is ‘bubblegum’ and the password is the same for each chatroom tested over the 2 week period. The executable then sets a room topic of ‘Resistance is futile’ and waits for further commands.

Reviewing the output from the strings run earlier we are able to determine that bot.exe seems to accept the following commands issued in a private message:


		HELLO
		IDENT
		SHUTDOWN


Testing these commands with the executable in the live chatroom the following was determined:

**Hello** -  Echo’s the following to the main chat room “Hello %s I’m a bot thats part of the BSides London Challenge”  where %s is the user who send the private message

**IDENT** - Echo’s back the computer name and the user account that the bot is running on to the main chat room

**SHUTDOWN** - when supplied with the correct password, the following is echo’ed to the user in a private message “bingo - botnet shutting down…” and then the executable quits IRC with the message of “Botnet shutdown” and display a pop up window on the machine its running on asking if the Bot should exit or restart. When an incorrect password is supplied to the SHUTDOWN message the executable echo’s the following to the main chat room “wrong pw” - one discrepancy was discovered with this - when using the password “iamborg” the private message “so close yet so far…wrong pw!” was returned.Interestingly the string "iamborg" was not found while running strings on the executable, but can be seen on the stack during runtime. This suggests that it is build dynamically at runtime. Often malware authors do this deliberately to make it harder for analysis to occur.

![My helpful screenshot](/public/img/malware14.png){: .center-image }

Loading the executable into IDA Pro and searching for the ASCII string ‘bingo - botnet shutting down’ leads to a section IDA Pro labeled as loc_410143C. Reviewing functions that call this sequence of commands at loc_410143C the application flow can start to be determined. Prior to the section that shutdown the bot is the following:

![My helpful screenshot](/public/img/malware7.png){: .center-image }

We can't see the string 'iamborg yet' but the ASCII string **HCNFJT@** seems related. If we search IDAPro for occurrences of this string we're lead to the following function:

![My helpful screenshot](/public/img/malware16.png){: .center-image }

Here the application is taking the ASCII string **HCNFJT@** and byte by byte XOR'ing it with the hex code 0x21.... One each loop around 1 is added to ebx, which in turn increments the XOR'ed string, so 0x21 becomes 0x22, then 0x23 etc until the XOR is complete which results in the string **"iamborg"**. As the testing above showed this string isn't the password needed to shutdown the botnet so further analysis was needed

Looking at the function prior to the shutdown function, we can see that the **HCNFJT@** is pushed on to the stack and then we jump to a function labeled as sub_4014BE. This function takes a bunch or arguments and variables:

![My helpful screenshot](/public/img/malware17.png){: .center-image }

Using Immunity Debugger we are able to discover what these are:

![My helpful screenshot](/public/img/malware18.png){: .center-image }

Here we can see that arg_0 maps to the IRC username of the person sending the private message, arg_04 maps to the string 'iamborg'

If we follow this function further we get to the following code:

![My helpful screenshot](/public/img/malware19.png){: .center-image }

Also of note here that is 1Ah is pushed and then popped off the stack, and that 41h is being added to dl.

Here's whats happening... The two strings (arg_0 and arg_04) are added together - then the modulo of that number and 26 is taken and then 65 is added. The function then loops using ebx and edi as counters. EDI contains the username length, and on each loop EBX is incremented until it reaches the same number as EDI and then the function returns.

The condensed code is:


		push  1Ah 		; push 26 on to stack
		...
		...
		pop ecx 		; pop 26 of the stack into ecx
		add eax, esi 	; bitwise add a single character of each string together
		...
		IDIV ecx 		; take modulo
		add dl, 41 		; add 65
		...


As the function loops around you can see in a debugger the password developing, however sometimes it is easier doing the maths outside of a debugger in order to understand what's happening. The following is a python representation of a single loop around the above block:


		print (chr((ord("j") + ord("i")) % 0x1a + 0x41))



Once this loop has finished looking at the Registers (top right section of the above screen shot) ECX contains the password that needs to be supplied with the SHUTDOWN command in order to ensure the jump to the shutdown function occurs - in this case it is **DPFANJB**. Suppling this with the SHUTDOWN command results in the executable shutting down with the behavior described previously:

![My helpful screenshot](/public/img/malware9.png){: .center-image }

Once supplied the executable pop's up a window on the client machine and terminates the IRC session

![My helpful screenshot](/public/img/malware10.png){: .center-image }


### 8. Conclusion

Relax - we saved the world!


### 9. Tools used

**Static Analysis:** PEiD, PEView, PEProfessional, Strings, IDAPro

**Dynamic Analysis:** ApateDNS, Immunity Debugger, Wireshark, RegShot, Dependency Walker, Resource Hacker, Process Explorer
