Download Link: https://assignmentchef.com/product/solved-cs2011-lab-3-attack-lab
<br>
<h1></h1>

This assignment involves generating a total of five attacks on two programs having different security vulnerabilities. Outcomes you will gain from this lab include:

<ul>

 <li>You will learn different ways that attackers can exploit security vulnerabilities when programs do not safeguard themselves well enough against buffer overflows.</li>

 <li>Through this, you will get a better understanding of how to write programs that are more secure, as well as some of the features provided by compilers and operating systems to make programs less vulnerable.</li>

 <li>You will gain a deeper understanding of the stack and parameter-passing mechanisms of x8664 machine code.</li>

 <li>You will gain a deeper understanding of how x86-64 instructions are encoded.</li>

 <li>You will gain more experience with debugging tools such as <strong>gdb</strong> and <strong>objdump</strong>.</li>

</ul>

<strong>Note:</strong> In this lab, you will gain firsthand experience with methods used to exploit security weaknesses in operating systems and network servers. Our purpose is to help you learn about the runtime operation of programs and to understand the nature of these security weaknesses so that you can avoid them when you write system code. We do not condone the use of any other form of attack to gain unauthorized access to any system resources.

You should study Sections 3.10.3 and 3.10.4 of the CS:APP3e book as reference material for this lab.

<h1>Logistics</h1>

To obtain your <em>target</em>, visit the <em>AttackLab</em> server at <a href="https://cs2011.cs.wpi.edu/">https://cs2011.cs.wpi.edu/</a> and follow the link.

This will display an Attack Lab Target Request form for you to fill in. Enter your WPI user name and email address and click the Submit button. The server will build a new target for you and return it to your browser as a <strong>tar</strong> file with the name <strong>target<em>k</em>.tar</strong> (where <strong><em>k</em></strong> is a unique number).

<strong>Note:</strong> Experience in previous years is that the server does not have enough power to generate new targets on demand for an entire Lab/Recitation session. Please download your target as soon as possible.

Regardless of how you obtained your target, save the <strong>target<em>k</em>.tar</strong> file to a separate directory in your virtual machine where you plan to do your work. Then extract it using the command:

<strong>tar</strong> <strong>-xvf</strong> <strong>target</strong><strong><em>k</em></strong><strong>.tar</strong>.

This will create a directory called <strong>./target</strong><strong><em>k</em></strong> containing the following files:

<ul>

 <li><strong>txt</strong>: Describes the contents of the target directory.</li>

 <li><strong>ctarget</strong>: Linux binary with a code-injection vulnerability. To be used for phases 1-3 of the assignment.</li>

 <li><strong>rtarget</strong>: Linux binary with a return-oriented programming vulnerability. To be used for phases 4-5 of the assignment.</li>

 <li><strong>txt</strong>: A text file containing a unique 4-byte signature for you, required for this lab instance.</li>

 <li><strong>c</strong>: Source code for <em>gadget farm</em> present in this instance of <strong>rtarget</strong>. You can compile (use flag <strong>-Og</strong>) and disassemble it to look for gadgets.</li>

 <li><strong>hex2raw</strong>: Utility program to generate byte sequences for passing to targets. See documentation on Page 11.</li>

</ul>

If for some reason you request or obtain multiple targets, this is not a problem. Choose one target to work with and ignore the rest.

<strong>Note:</strong> You <em>must</em> use your WPI login ID when requesting a new target. Each target is associated with a unique student, and the login ID is how the graders identify whose target it is.

<strong>Note:</strong> The URL for downloading targets may not be reachable from off campus due to firewalls.

We did not attempt to build targets for the IA32 architecture to run on CCC systems or for Macintosh or Windows platforms. Therefore, these bombs will only work on generic 64-bit Linux systems, such as the virtual machine made available to this course. In fact, the program checks the name of the computer it is running on; if you will not be using the course VM, let your instructor know.

<strong>Note: </strong>Modern operating systems such as Linux employ various techniques designed to thwart the kinds of exploits you will be using. We have deliberately disabled some of these <u>in some of the target programs</u>. There is one more thing you need to do on your virtual machine to disable one more protection <u>on that machine</u>. This technique is called <em>ASLR</em> (meaning Address Space Layout Randomization. To disable it, download the shell script <strong>no_randomize.sh </strong>from Canvas. You will then need to use the <strong>chmod</strong> command to make it executable by you. Then run it; enter your password if requested. You should only have to run it once; the setting should persist even after you reboot your VM. When you are all done with Attacklab, if you want to continue using your VM, you should execute the script (also available on Canvas) <strong>yes_randomize.sh</strong> to increase security.

<h1>Important Points</h1>

Here is a summary of some important rules regarding valid solutions for this lab. These points will not make much sense when you read this document for the first time. They are presented here as a central reference of rules once you get started.

<ul>

 <li>You must do the assignment on a machine that is similar to the one that generated your targets — i.e., the course virtual machine.</li>

 <li>Your solutions may not use attacks to circumvent the validation code in the programs. Specifically, any address you incorporate into an attack string for use by a <strong>ret</strong> instruction should be to one of the following destinations:–</li>

</ul>

 The addresses for functions <strong>touch1</strong>, <strong>touch2</strong>, or <strong>touch3</strong>.

 The address of your injected code.

 The address of one of your gadgets from the gadget farm.

<ul>

 <li>You may only construct gadgets from file <strong>rtarget</strong> with addresses ranging between those for functions <strong>start_farm</strong> and <strong>end_farm</strong>.</li>

</ul>

<h1>Target Programs</h1>

Both <strong>ctarget</strong> and <strong>rtarget</strong> read strings from standard input. They do so with the function <strong>getbuf</strong> defined below:–

<ul>

 <li><strong>unsigned getbuf() { </strong></li>

 <li><strong>char buf[BUFFER_SIZE]; </strong></li>

 <li><strong>Gets(buf); </strong></li>

 <li><strong>return 1; </strong></li>

</ul>

<h2>5 }</h2>

The function <strong>Gets</strong> is similar to the standard library function <strong>gets</strong>—it reads a string from standard input (terminated by <strong>‘
’</strong> or end-of-file) and stores it (along with a null terminator) at the specified destination. In this code, you can see that the destination is an array <strong>buf</strong>, declared as having <strong>BUFFER_SIZE</strong> bytes. At the time your targets were generated, <strong>BUFFER_SIZE</strong> was a compile-time constant specific to your version of the programs.

Functions <strong>Gets()</strong> and <strong>gets()</strong> have no way to determine whether their destination buffers are large enough to store the string they read. They simply copy sequences of bytes, possibly overrunning the bounds of the storage allocated at the destinations.

If the string typed by the user and read by <strong>getbuf</strong> is sufficiently short, it is clear that <strong>getbuf</strong> will return the value <strong>1</strong>, as shown by the following execution examples:–

<strong>linux&gt; ./ctarget </strong>

<strong>Cookie: 0x1a7dd803 </strong>

<strong>Type string: Keep it short! </strong>

<strong>No exploit. Getbuf returned 0x1 Normal return </strong>

Typically an error occurs if you type a long string:–

<strong>linux&gt; ./ctarget Cookie: 0x1a7dd803 </strong>

<strong>Type string: This is not a very interesting string, but it has the property … </strong>

<strong>Ouch!: You caused a segmentation fault! Better luck next time </strong>

Note that the value of the cookie shown will differ from yours. Program <strong>rtarget</strong> will have the same behavior. As the error message indicates, overrunning the buffer typically causes the program state to be corrupted, leading to a memory access error. Your task is to be more clever with the strings you feed <strong>ctarget</strong> and <strong>rtarget</strong> so that they do more interesting things. These are called <em>exploit</em> strings.

Both <strong>ctarget</strong> and <strong>rtarget</strong> take several different command line arguments:–

<strong>-h:</strong> Print list of possible command line arguments

<strong>-q:</strong> Don‘t send results to the grading server

<strong>-i FILE:</strong> Supply input from a file, rather than from standard input

Your exploit strings will typically contain byte values that do not correspond to the ASCII values for printable characters, so you would not be able to enter them directly by typing on a keyboard. The program <strong>hex2raw</strong> will enable you to generate these <em>raw</em> strings. See Page 11 for more information on how to use <strong>hex2raw</strong>.

<strong>Important points: </strong>

<ul>

 <li>Your exploit string must <em>not</em> contain byte value <strong>0x0a</strong> at any intermediate position, since this is the ASCII code for newline (<strong>‘
’</strong>). When <strong>Gets()</strong> encounters this byte, it assumes you intended to terminate the string.</li>

 <li><strong>hex2raw</strong> expects two-digit hex values separated by one or more white spaces. So if you want to create a byte with a hex value of <em>0</em>, you need to write it as <strong>00</strong>. To create the word <strong>0xdeadbeef</strong> you should pass ―<strong>ef be ad de</strong>‖ to <strong>hex2raw</strong> (note the reversal required for little-endian byte ordering).</li>

</ul>

When you have correctly solved one of the levels, your target program will automatically send a notification to the grading server. Here is an example. It takes a human-readable file which is called <strong>ctarget.12.txt</strong>, feeds it to the program <strong>hex2raw</strong> to convert it to raw (binary) format, then feeds that raw string to the target.

<strong>linux&gt; ./hex2raw &lt; ctarget.l2.txt | ./ctarget </strong>

<strong>Cookie: 0x1a7dd803 </strong>

<strong>Type string:Touch2!: You called touch2(0x1a7dd803) </strong>

<strong>Valid solution for level 2 with target ctarget PASSED: Sent exploit string to server to be validated. </strong>

<strong>NICE JOB!  </strong>

The server will test your exploit string to make sure it really works, and it will update the Attacklab scoreboard page indicating that your userid (listed by your target number for anonymity) has completed this phase.

You can view the scoreboard by pointing your Web browser at

<a href="http://cs2011.cs.wpi.edu:15513/scoreboard">http://cs2011.cs.wpi.edu:15513/scoreboard</a>

Unlike the BombLab, there is no penalty for making mistakes in this lab. Feel free to fire away at <strong>ctarget</strong> and <strong>rtarget</strong> with any strings you like.

<strong>Important Note:</strong> You can work on your solution on any Linux machine, but in order to submit your solution, you will need to be running on an instance of the course virtual machine (or one of the others on the approved list).

Figure 1 below summarizes the five phases of the lab. As can be seen, the first three involve codeinjection (CI) attacks on <strong>ctarget</strong>, while the last two involve return-oriented-programming (ROP) attacks on <strong>rtarget</strong>.

<em> </em>

<table width="460">

 <tbody>

  <tr>

   <td width="95"> Phase</td>

   <td width="102">Program</td>

   <td width="44">Level</td>

   <td width="51">Method</td>

   <td width="117">        Function</td>

   <td width="51">Points</td>

  </tr>

  <tr>

   <td width="95">1</td>

   <td width="102"><strong>ctarget </strong></td>

   <td width="44">1</td>

   <td width="51">CI</td>

   <td width="117"><strong>touch1 </strong></td>

   <td width="51">10</td>

  </tr>

  <tr>

   <td width="95">2</td>

   <td width="102"><strong>ctarget </strong></td>

   <td width="44">2</td>

   <td width="51">CI</td>

   <td width="117"><strong>touch2 </strong></td>

   <td width="51">25</td>

  </tr>

  <tr>

   <td width="95">3</td>

   <td width="102"><strong>ctarget </strong></td>

   <td width="44">3</td>

   <td width="51">CI</td>

   <td width="117"><strong>touch3 </strong></td>

   <td width="51">25</td>

  </tr>

  <tr>

   <td width="95">4</td>

   <td width="102"><strong>rtarget </strong></td>

   <td width="44">2</td>

   <td width="51">ROP</td>

   <td width="117"><strong>touch2 </strong></td>

   <td width="51">35</td>

  </tr>

  <tr>

   <td width="95">5</td>

   <td width="102"><strong>rtarget </strong></td>

   <td width="44">3</td>

   <td width="51">ROP</td>

   <td width="117"><strong>touch3 </strong></td>

   <td width="51">5</td>

  </tr>

 </tbody>

</table>

<em>CI:                  </em>Code injection

<em>ROP:                </em>Return-oriented programming

<em>Figure 1</em><em>: Summary of Attack Lab phases </em>

<h1>Part 1: Code Injection Attacks</h1>

For the first three phases, your exploit strings will attack <strong>ctarget</strong>. This program is set up in a way that the stack positions will be consistent from one run to the next and so that data on the stack can be treated as executable code. These features make the program vulnerable to attacks where the exploit strings contain the byte encodings of executable code.

<h2>Part 1, Level 1</h2>

For Phase 1, you will not inject new code. Instead, your exploit string will redirect the program to execute an existing function.

Function <strong>getbuf</strong> is called within <strong>CTARGET</strong> by a function <strong>test</strong> having the following <em>C</em> code:

<ul>

 <li><strong>void test() { </strong></li>

 <li><strong>int val; </strong></li>

 <li><strong>val = getbuf(); </strong></li>

 <li><strong>printf(“No exploit. Getbuf returned 0x%x
”, val); </strong><strong>5 </strong><strong>} </strong></li>

</ul>

When <strong>getbuf</strong> executes its <strong>return</strong> statement (line 4 of the <strong>getbuf</strong> code near the top of Page 3), the program ordinarily resumes execution within function <strong>test</strong> (at the <strong>printf()</strong> function on line 4 above). We want to change this behavior. Within the file <strong>ctarget</strong>, there is code for a function <strong>touch1</strong> having the following <em>C</em> representation:–

<strong>1 </strong><strong>void touch1() </strong>

<h3>2 {</h3>

<ul>

 <li><strong>vlevel = 1; /* Part of validation protocol */ </strong></li>

 <li><strong>printf(“Touch1!: You called touch1()
”); </strong></li>

 <li><strong>validate(1); </strong></li>

 <li><strong>exit(0); </strong></li>

</ul>

<h3>7 }</h3>

Your task is to get <strong>ctarget</strong> to execute the code for <strong>touch1</strong> when <strong>getbuf</strong> executes its return statement, rather than returning to <strong>test</strong>. Note that your exploit string may also corrupt parts of the stack not directly related to this stage, but this will not cause a problem, since <strong>touch1</strong> causes the program to exit directly.

<h3>Some Advice:–</h3>

<ul>

 <li>All the information you need to devise your exploit string for this level can be determined by examining a disassembled version of <strong>ctarget</strong>. Use <strong>objdump -d</strong> to get this dissembled version.</li>

 <li>The idea is to position a byte representation of the starting address for <strong>touch1</strong> so that the <strong>ret </strong>instruction at the end of the code for <strong>getbuf</strong> will transfer control to <strong>touch1</strong>.</li>

 <li>Be careful about byte ordering.</li>

 <li>You might want to use <strong>gdb</strong> to step the program through the last few instructions of <strong>getbuf</strong> to make sure it is doing the right thing.</li>

 <li>The placement of <strong>buf</strong> within the stack frame for <strong>getbuf</strong> depends on the value of compiletime constant <strong>BUFFER_SIZE</strong>, as well the allocation strategy used by <strong>gcc</strong>. You will need to examine the disassembled code to determine its position.</li>

</ul>

<h2>Part 1, Level 2</h2>

Phase 2 involves injecting a small amount of code as part of your exploit string. Within the file <strong>ctarget</strong> there is code for a function <strong>touch2</strong> having the following <em>C</em> representation:–

<strong>1 </strong><strong>void touch2(unsigned val) </strong>

<h3>2 {</h3>

<ul>

 <li><strong>vlevel = 2; /* Part of validation protocol */ </strong></li>

 <li><strong>if (val == cookie) { </strong></li>

 <li><strong>printf(“Touch2!: You called touch2(0x%.8x)
”, val); </strong></li>

 <li><strong>validate(2); </strong></li>

 <li><strong>} else { </strong></li>

 <li><strong>printf(“Misfire: You called touch2(0x%.8x)
”, val); </strong></li>

 <li><strong>fail(2); </strong></li>

 <li><strong>} </strong></li>

 <li><strong>exit(0); </strong></li>

</ul>

<h3>12 }</h3>

Your task is to get <strong>ctarget</strong> to execute the code for <strong>touch2</strong> rather than returning to <strong>test</strong>. In this case, however, you must make it appear to <strong>touch2</strong> as if you have passed your cookie as its argument.

<h3>Some Advice:–</h3>

<ul>

 <li>You will want to position a byte representation of the address of your injected code in such a way that <strong>ret</strong> instruction at the end of the code for <strong>getbuf</strong> will transfer control to it.</li>

 <li>Recall that the first argument to a function is passed in register <strong>%rdi</strong>.</li>

 <li>Your injected code should set the register to your cookie, and then use a <strong>ret</strong> instruction to transfer control to the first instruction in <strong>touch2</strong>.</li>

 <li>Do not attempt to use <strong>jmp</strong> or <strong>call</strong> instructions in your exploit code. The encodings of destination addresses for these instructions are difficult to formulate. Use <strong>ret</strong> instructions for all transfers of control, even when you are not returning from a call.</li>

 <li>See the discussion in <strong>Appendix B</strong> on Page 11 regarding how to use tools to generate the byte-level representations of instruction sequences.</li>

</ul>

<h2>Part 1, Level 3</h2>

Phase 3 also involves a code injection attack, but passing a string as argument.

Within the file <strong>ctarget</strong> there is code for functions <strong>hexmatch</strong> and <strong>touch3</strong> having the following <em>C</em> representations:–

<ul>

 <li><strong>/* Compare string to hex represention of unsigned value */ </strong></li>

 <li><strong>int hexmatch(unsigned val, char *sval) </strong></li>

</ul>

<h3>3 {</h3>

<ul>

 <li><strong>char cbuf[110]; </strong></li>

 <li><strong>/* Make position of check string unpredictable */ </strong></li>

 <li><strong>char *s = cbuf + random() % 100; </strong></li>

 <li><strong>sprintf(s, “%.8x”, val); </strong></li>

 <li><strong>return strncmp(sval, s, 9) == 0; </strong></li>

 <li><strong>} </strong><strong>10 </strong></li>

</ul>

<strong>11 </strong><strong>void touch3(char *sval) </strong>

<h3>12 {</h3>

<ul>

 <li><strong>vlevel = 3; /* Part of validation protocol */ </strong></li>

 <li><strong>if (hexmatch(cookie, sval)) { </strong></li>

 <li><strong>printf(“Touch3!: You called touch3(”%s”)
”, sval); </strong></li>

 <li><strong>validate(3); </strong></li>

 <li><strong>} else { </strong></li>

 <li><strong>printf(“Misfire: You called touch3(”%s”)
”, sval); </strong></li>

 <li><strong>fail(3); </strong></li>

 <li><strong>} </strong></li>

 <li><strong>exit(0); </strong></li>

</ul>

<h3>22  }</h3>

Your task is to get <strong>ctarget</strong> to execute the code for <strong>touch3</strong> rather than returning to <strong>test</strong>. You must make it appear to <strong>touch3</strong> as if you have passed a string representation of your cookie as its argument. <strong>Some Advice</strong><strong>: </strong>

<ul>

 <li>You will need to include a string representation of your cookie in your exploit string. The string should consist of the eight hexadecimal digits (ordered from most to least significant) without a leading ―<strong>0x</strong>.‖</li>

 <li>Recall that a string is represented in <em>C</em> as a sequence of bytes followed by a byte with value <strong>0</strong>. Type ―<strong>man ascii</strong>‖ on any Linux machine to see the byte representations of the characters you need.</li>

 <li>Your injected code should set register <strong>%rdi</strong> to the address of this string.</li>

 <li>When functions <strong>hexmatch</strong> and <strong>strncmp</strong> are called, they push data onto the stack, overwriting portions of memory that held the buffer used by <strong>getbuf</strong>. As a result, you will need to be careful where you place the string representation of your cookie.</li>

</ul>

<h1>Part II: Return-Oriented Programming</h1>

Performing code-injection attacks on program <strong>rtarget</strong> is much more difficult than it is for <strong>ctarget</strong>, because it uses two techniques to thwart such attacks:

<ul>

 <li>It uses randomization so that the stack positions differ from one run to another. This makes it impossible to determine where your injected code will be located.</li>

 <li>It marks the section of memory holding the stack as nonexecutable, so even if you could set the program counter to the start of your injected code, the program would fail with a segmentation fault.</li>

</ul>

Fortunately, clever people have devised strategies for getting useful things done in a program by executing existing code, rather than injecting new code. The most general form of this is referred to as <em>return-oriented programming</em> (ROP) [1, 2]. The strategy with ROP is to identify byte sequences within an existing program that consist of one or more instructions followed by the instruction <strong>ret</strong>. Such a segment is referred to as a <em>gadget</em>.




<em>Figure 2: Setting up sequence of gadgets for execution. Byte value </em><em>0xc3 </em><em>encodes the </em><em>ret </em><em>instruction. </em>

Figure 2 illustrates how the stack can be set up to execute a sequence of <strong>n</strong> gadgets. In this figure, the stack contains a sequence of gadget addresses. Each gadget consists of a series of instruction bytes, with the final one being <strong>0xc3</strong>, encoding the <strong>ret</strong> instruction. When the program executes a <strong>ret</strong> instruction starting with this configuration, it will initiate a chain of gadget executions, with the <strong>ret</strong> instruction at the end of each gadget causing the program to jump to the beginning of the next.

A gadget can make use of code corresponding to assembly-language statements generated by the compiler, especially ones at the ends of functions. In practice, there may be some useful gadgets of this form, but not enough to implement many important operations. For example, it is highly unlikely that a compiled function would have <strong>popq %rdi</strong> as its last instruction before <strong>ret</strong>. Fortunately, with a byte-oriented instruction set, such as x86-64, a gadget can often be found by extracting patterns from other parts of the instruction byte sequence.

For example, one version of <strong>rtarget</strong> contains code generated for the following <em>C</em> function:

<strong>void setval_210(unsigned *p) </strong>

<strong>{ </strong>

<strong> *p = 3347663060U; </strong>

<strong>} </strong>

The chances of this function being useful for attacking a system seem pretty slim. But, the disassembled machine code for this function shows an interesting byte sequence:

<strong>0000000000400f15 &lt;setval_210&gt;: </strong>

<strong>     400f15: c7 07 d4 48 89 c7 movl $0xc78948d4,(%rdi) </strong>

<strong>     400f1b: c3 retq </strong>

The byte sequence <strong>48 89 c7</strong> encodes the instruction <strong>movq %rax, %rdi</strong>. (See Figure 3A for the encodings of useful <strong>movq</strong> instructions.) This sequence is followed by byte value <strong>c3</strong>, which encodes the <strong>retq</strong> instruction. The function starts at address <strong>0x400f15</strong>, and the sequence starts on the fourth byte of the function. Thus, this code contains a gadget, having a starting address of <strong>0x400f18</strong>, that will copy the 64-bit value in register <strong>%rax</strong> to register <strong>%rdi</strong>. Note that the address might be different in your copy of the program.

Your code for <strong>rtarget</strong> contains a number of functions similar to the <strong>setval_210</strong> function shown above in a region we refer to as the <em>gadget farm</em>. Your job will be to identify useful gadgets in the gadget farm and use these to perform attacks similar to those you did in Phases 2 and 3.

<strong>Important:</strong> The gadget farm is demarcated by functions <strong>start_farm</strong> and <strong>end_farm</strong> in your copy of <strong>rtarget</strong>. Do not attempt to construct gadgets from other portions of the program code. <strong>Part II, Level 2 </strong>

For Phase 4, you will repeat the attack of Phase 2, but do so on program <strong>rtarget</strong> using gadgets from your gadget farm. You can construct your solution using gadgets consisting of the following instruction types, and using only the first eight <strong>x86-64</strong> registers (<strong>%rax–%rdi</strong>).

<strong>movq</strong> : The codes for these are shown in Table 1 below <strong>popq</strong> : The codes for these are shown in Table 2 below. <strong>ret</strong> : This instruction is encoded by the single byte <strong>0xc3</strong>. <strong>nop</strong> : This instruction (pronounced ―no op,‖ which is short for ―no operation‖) is encoded by the single byte <strong>0x90</strong>. Its only effect is to cause the program counter to be incremented by one.

<h2>Some Advice:–</h2>

<ul>

 <li>All the gadgets you need can be found in the region of the code for <strong>rtarget</strong> demarcated by the functions <strong>start_farm</strong> and <strong>mid_farm</strong>.</li>

 <li>You can do this attack with just two gadgets.</li>

 <li>When a gadget uses a <strong>popq</strong> instruction, it will pop data from the stack. As a result, your exploit string will contain a combination of gadget addresses and data.</li>

</ul>

<em>Table 1: Encodings of </em><strong>movq</strong><em> instructions </em>

<strong>movq <em>S</em>, <em>D</em> </strong>

<table width="578">

 <tbody>

  <tr>

   <td width="57"><strong> Source</strong></td>

   <td colspan="8" width="521"><strong>                          Destination <em>D</em> </strong></td>

  </tr>

  <tr>

   <td width="57"><strong> <em>S</em> </strong></td>

   <td width="65"><strong>%rax </strong></td>

   <td width="65"><strong>%rcx </strong></td>

   <td width="65"><strong>%rdx </strong></td>

   <td width="65"><strong>%rbx </strong></td>

   <td width="65"><strong>%rsp </strong></td>

   <td width="65"><strong>%rbp </strong></td>

   <td width="65"><strong>%rsi </strong></td>

   <td width="65"><strong>%rdi </strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rax </strong></td>

   <td width="65"><strong>48 89 c0</strong></td>

   <td width="65"><strong>48 89 c1</strong></td>

   <td width="65"><strong> 48 89 c2</strong></td>

   <td width="65"><strong> 48 89 c3</strong></td>

   <td width="65"><strong> 48 89 c4</strong></td>

   <td width="65"><strong> 48 89 c5</strong></td>

   <td width="65"><strong> 48 89 c6</strong></td>

   <td width="65"><strong>48 89 c7</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rcx </strong></td>

   <td width="65"><strong>48 89 c8</strong></td>

   <td width="65"><strong>48 89 c9</strong></td>

   <td width="65"><strong> 48 89 ca</strong></td>

   <td width="65"><strong> 48 89 cb</strong></td>

   <td width="65"><strong> 48 89 cc</strong></td>

   <td width="65"><strong> 48 89 cd</strong></td>

   <td width="65"><strong> 48 89 ce</strong></td>

   <td width="65"><strong>48 89 cf</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rdx </strong></td>

   <td width="65"><strong>48 89 d0</strong></td>

   <td width="65"><strong>48 89 d1</strong></td>

   <td width="65"><strong> 48 89 d2</strong></td>

   <td width="65"><strong> 48 89 d3</strong></td>

   <td width="65"><strong> 48 89 d4</strong></td>

   <td width="65"><strong> 48 89 d5</strong></td>

   <td width="65"><strong> 48 89 d6</strong></td>

   <td width="65"><strong>48 89 d7</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rbx </strong></td>

   <td width="65"><strong>48 89 d8</strong></td>

   <td width="65"><strong>48 89 d9</strong></td>

   <td width="65"><strong> 48 89 da</strong></td>

   <td width="65"><strong> 48 89 db</strong></td>

   <td width="65"><strong> 48 89 dc</strong></td>

   <td width="65"><strong> 48 89 dd</strong></td>

   <td width="65"><strong> 48 89 de</strong></td>

   <td width="65"><strong>48 89 df</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rsp </strong></td>

   <td width="65"><strong>48 89 e0</strong></td>

   <td width="65"><strong>48 89 e1</strong></td>

   <td width="65"><strong> 48 89 e2</strong></td>

   <td width="65"><strong> 48 89 e3</strong></td>

   <td width="65"><strong> 48 89 e4</strong></td>

   <td width="65"><strong> 48 89 e5</strong></td>

   <td width="65"><strong> 48 89 e6</strong></td>

   <td width="65"><strong>48 89 e7</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rbp </strong></td>

   <td width="65"><strong>48 89 e8</strong></td>

   <td width="65"><strong>48 89 e9</strong></td>

   <td width="65"><strong> 48 89 ea</strong></td>

   <td width="65"><strong> 48 89 eb</strong></td>

   <td width="65"><strong> 48 89 ec</strong></td>

   <td width="65"><strong> 48 89 ed</strong></td>

   <td width="65"><strong> 48 89 ee</strong></td>

   <td width="65"><strong>48 89 ef</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rsi </strong></td>

   <td width="65"><strong>48 89 f0</strong></td>

   <td width="65"><strong>48 89 f1</strong></td>

   <td width="65"><strong> 48 89 f2</strong></td>

   <td width="65"><strong> 48 89 f3</strong></td>

   <td width="65"><strong> 48 89 f4</strong></td>

   <td width="65"><strong> 48 89 f5</strong></td>

   <td width="65"><strong> 48 89 f6</strong></td>

   <td width="65"><strong>48 89 f7</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>%rdi </strong></td>

   <td width="65"><strong>48 89 f8</strong></td>

   <td width="65"><strong>48 89 f9</strong></td>

   <td width="65"><strong> 48 89 fa</strong></td>

   <td width="65"><strong> 48 89 fb</strong></td>

   <td width="65"><strong> 48 89 fc</strong></td>

   <td width="65"><strong> 48 89 fd</strong></td>

   <td width="65"><strong> 48 89 fe</strong></td>

   <td width="65"><strong>48 89 ff</strong></td>

  </tr>

 </tbody>

</table>

<strong> </strong>

<em>Table 2: Encodings of </em><strong>popq</strong><em> instructions </em>

<table width="460">

 <tbody>

  <tr>

   <td width="92"><strong>Operation  </strong></td>

   <td width="46"><strong>%rax </strong></td>

   <td width="46"><strong>%rcx </strong></td>

   <td width="46"><strong>%rdx </strong></td>

   <td width="46"><strong>Register %rbx </strong></td>

   <td width="46"><strong><em>R</em></strong><strong> %rsp </strong></td>

   <td width="46"><strong>%rbp </strong></td>

   <td width="46"><strong>%rsi </strong></td>

   <td width="46"><strong>%rdi </strong></td>

  </tr>

  <tr>

   <td width="92"><strong> popq <em>R</em> </strong></td>

   <td width="46"><strong>58 </strong></td>

   <td width="46"><strong>59 </strong></td>

   <td width="46"><strong>5a </strong></td>

   <td width="46"><strong>5b </strong></td>

   <td width="46"><strong>5c </strong></td>

   <td width="46"><strong>5d </strong></td>

   <td width="46"><strong>5e </strong></td>

   <td width="46"><strong>5f </strong></td>

  </tr>

 </tbody>

</table>




<em>Table 3: Encodings of movl instructions </em>

<strong>movl <em>S</em>, <em>D</em> </strong>

<table width="460">

 <tbody>

  <tr>

   <td width="62"><strong> Source</strong></td>

   <td colspan="8" width="398"><strong>                   Destination <em>D</em> </strong></td>

  </tr>

  <tr>

   <td width="62"><strong> <em>S</em> </strong></td>

   <td width="50"><strong>%eax </strong></td>

   <td width="50"><strong>%ecx </strong></td>

   <td width="50"><strong>%edx </strong></td>

   <td width="50"><strong>%ebx </strong></td>

   <td width="50"><strong>%esp </strong></td>

   <td width="50"><strong>%ebp </strong></td>

   <td width="50"><strong>%esi </strong></td>

   <td width="50"><strong>%edi </strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%eax </strong></td>

   <td width="50"><strong>89 c0 </strong></td>

   <td width="50"><strong>89 c1 </strong></td>

   <td width="50"><strong>89 c2 </strong></td>

   <td width="50"><strong>89 c3 </strong></td>

   <td width="50"><strong>89 c4 </strong></td>

   <td width="50"><strong>89 c5 </strong></td>

   <td width="50"><strong>89 c6 </strong></td>

   <td width="50"><strong>89 c7</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%ecx </strong></td>

   <td width="50"><strong>89 c8 </strong></td>

   <td width="50"><strong>89 c9 </strong></td>

   <td width="50"><strong>89 ca </strong></td>

   <td width="50"><strong>89 cb </strong></td>

   <td width="50"><strong>89 cc </strong></td>

   <td width="50"><strong>89 cd </strong></td>

   <td width="50"><strong>89 ce </strong></td>

   <td width="50"><strong>89 cf</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%edx </strong></td>

   <td width="50"><strong>89 d0 </strong></td>

   <td width="50"><strong>89 d1 </strong></td>

   <td width="50"><strong>89 d2 </strong></td>

   <td width="50"><strong>89 d3 </strong></td>

   <td width="50"><strong>89 d4 </strong></td>

   <td width="50"><strong>89 d5 </strong></td>

   <td width="50"><strong>89 d6 </strong></td>

   <td width="50"><strong>89 d7</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%ebx </strong></td>

   <td width="50"><strong>89 d8 </strong></td>

   <td width="50"><strong>89 d9 </strong></td>

   <td width="50"><strong>89 da </strong></td>

   <td width="50"><strong>89 db </strong></td>

   <td width="50"><strong>89 dc </strong></td>

   <td width="50"><strong>89 dd </strong></td>

   <td width="50"><strong>89 de </strong></td>

   <td width="50"><strong>89 df</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%esp </strong></td>

   <td width="50"><strong>89 e0 </strong></td>

   <td width="50"><strong>89 e1 </strong></td>

   <td width="50"><strong>89 e2 </strong></td>

   <td width="50"><strong>89 e3 </strong></td>

   <td width="50"><strong>89 e4 </strong></td>

   <td width="50"><strong>89 e5 </strong></td>

   <td width="50"><strong>89 e6 </strong></td>

   <td width="50"><strong>89 e7</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%ebp </strong></td>

   <td width="50"><strong>89 e8 </strong></td>

   <td width="50"><strong>89 e9 </strong></td>

   <td width="50"><strong>89 ea </strong></td>

   <td width="50"><strong>89 eb </strong></td>

   <td width="50"><strong>89 ec </strong></td>

   <td width="50"><strong>89 ed </strong></td>

   <td width="50"><strong>89 ee </strong></td>

   <td width="50"><strong>89 ef</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%esi </strong></td>

   <td width="50"><strong>89 f0 </strong></td>

   <td width="50"><strong>89 f1 </strong></td>

   <td width="50"><strong>89 f2 </strong></td>

   <td width="50"><strong>89 f3 </strong></td>

   <td width="50"><strong>89 f4 </strong></td>

   <td width="50"><strong>89 f5 </strong></td>

   <td width="50"><strong>89 f6 </strong></td>

   <td width="50"><strong>89 f7</strong></td>

  </tr>

  <tr>

   <td width="62"><strong>%edi </strong></td>

   <td width="50"><strong>89 f8 </strong></td>

   <td width="50"><strong>89 f9 </strong></td>

   <td width="50"><strong>89 fa </strong></td>

   <td width="50"><strong>89 fb </strong></td>

   <td width="50"><strong>89 fc </strong></td>

   <td width="50"><strong>89 fd </strong></td>

   <td width="50"><strong>89 fe </strong></td>

   <td width="50"><strong>89 ff</strong></td>

  </tr>

 </tbody>

</table>




<em>Table 4: Encodings of 2-byte functional </em><strong>nop</strong><em> insructions </em>

<table width="460">

 <tbody>

  <tr>

   <td rowspan="2" width="199"><strong>Operation </strong><strong>       </strong></td>

   <td width="65"></td>

   <td colspan="2" width="131"><strong>Register <em>R</em> </strong></td>

   <td width="65"></td>

  </tr>

  <tr>

   <td width="65"><strong>%al </strong></td>

   <td width="65"><strong>%cl </strong></td>

   <td width="65"><strong>%dl </strong></td>

   <td width="65"><strong>%bl </strong></td>

  </tr>

  <tr>

   <td width="199"><strong> andb         <em>R</em>, <em>R</em> </strong></td>

   <td width="65"><strong>20 c0 </strong></td>

   <td width="65"><strong>20 c9 </strong></td>

   <td width="65"><strong>20 d2 </strong></td>

   <td width="65"><strong>20 db </strong></td>

  </tr>

  <tr>

   <td width="199"><strong> Orb          <em>R</em>, <em>R</em> </strong></td>

   <td width="65"><strong>08 c0 </strong></td>

   <td width="65"><strong>08 c9 </strong></td>

   <td width="65"><strong>08 d2 </strong></td>

   <td width="65"><strong>08 db </strong></td>

  </tr>

  <tr>

   <td width="199"><strong> cmpb         <em>R</em>, <em>R</em> </strong></td>

   <td width="65"><strong>38 c0 </strong></td>

   <td width="65"><strong>38 c9 </strong></td>

   <td width="65"><strong>38 d2 </strong></td>

   <td width="65"><strong>38 db </strong></td>

  </tr>

  <tr>

   <td width="199"><strong> testb        <em>R</em>, <em>R</em> </strong></td>

   <td width="65"><strong>84 c0 </strong></td>

   <td width="65"><strong>84 c9 </strong></td>

   <td width="65"><strong>84 d2 </strong></td>

   <td width="65"><strong>84 db </strong></td>

  </tr>

 </tbody>

</table>




<h2>Part II, Level 3</h2>

Before you take on the Phase 5, pause to consider what you have accomplished so far. In Phases 2 and 3, you caused a program to execute machine code of your own design. If <strong>ctarget</strong> had been a network server, you could have injected your own code into a distant machine. In Phase 4, you circumvented two of the main devices modern systems use to thwart buffer overflow attacks. Although you did not inject your own code, you were able to inject a type of program that operates by stitching together sequences of existing code.

You have also gotten 95/100 points for this lab. That‘s a good score. If you have other pressing obligations consider stopping right now.

Phase 5 requires you to do an ROP attack on <strong>rtarget</strong> to invoke function <strong>touch3</strong> with a pointer to a string representation of your cookie. That may not seem significantly more difficult than using an ROP attack to invoke <strong>touch2</strong>, except that we have made it so. Moreover, Phase 5 counts for only 5 points, which is not a true measure of the effort it will require. Think of it as more an extra credit problem for those who want to go beyond the normal expectations for the course.

To solve Phase 5, you can use gadgets in the region of the code in <strong>rtarget</strong> demarcated by functions <strong>start_farm</strong> and <strong>end_farm</strong>. In addition to the gadgets used in Phase 4, this expanded farm includes the encodings of different <strong>movl</strong> instructions, as shown in Table 3 above. The byte sequences in this part of the farm also contain 2-byte instructions that serve as <strong>functional nops</strong>, i.e., they do not change any register or memory values. These include instructions, shown in Table 4 above, such as <strong>andb %al,%al</strong>, that operate on the low-order bytes of some of the registers but do not change their values.

<h3>Some Advice:–</h3>

<ul>

 <li>You‘ll want to review the effect a <strong>movl</strong> instruction has on the upper 4 bytes of a register, as is described on page 183 of the text.</li>

 <li>The official solution requires eight gadgets (not all of which are unique).</li>

 <li>Good luck and have fun!</li>

</ul>

<h2>Appendix A: Using hex2raw</h2>

<strong>hex2raw</strong> takes as input a hex-formatted string. In this format, each byte value is represented by two hex digits. For example, the string ―012345‖ could be entered in hex format as ―<strong>30 31 32 33 34 35 00</strong>.‖ (Recall that the ASCII code for decimal digit <strong>x</strong> is<strong> 0x3x</strong>, and that the end of a string is indicated by a null byte.)

The hex characters you pass to <strong>hex2raw</strong> should be separated by whitespace (blanks or newlines). We recommend separating different parts of your exploit string with newlines while you‘re working on it. <strong>hex2raw</strong> supports <em>C</em>-style block comments, so you can mark off sections of your exploit string. For example:

<strong>48 c7 c1 f0 11 40 00 /* mov $0x40011f0,%rcx */ </strong>

Be sure to leave space around both the starting and ending comment strings (<strong>“/*”</strong>,<strong>“*/”</strong>), so that the comments will be properly ignored.

If you generate a hex-formatted exploit string in the file <strong>exploit.txt</strong>, you can apply the raw string to <strong>ctarget</strong> or <strong>rtarget</strong> in several different ways:

<ol>

 <li>You can set up a series of pipes to pass the string through <strong>hex2raw</strong>.</li>

</ol>

<strong>linux&gt;</strong> <strong>cat exploit.txt | ./hex2raw | ./ctarget </strong>

<ol start="2">

 <li>You can store the raw string in a file and use I/O redirection:</li>

</ol>

<strong>linux</strong> <strong>&gt;</strong> <strong>./hex2raw &lt; exploit.txt &gt; exploit-raw.txt linux</strong> <strong>&gt;</strong> <strong>./ctarget &lt; exploit-raw.txt </strong>

This approach can also be used when running from within <strong>gdb</strong>:

<strong>unix&gt;</strong> <strong>gdb ctarget </strong>

<strong>(gdb)</strong> <strong>run &lt; exploit-raw.txt </strong>

<ol start="3">

 <li>You can store the raw string in a file and provide the file name as a command-line argument:</li>

</ol>

<strong>unix&gt; ./hex2raw &lt; exploit.txt &gt; exploit-raw.txt unix&gt; ./ctarget -i exploit-raw.txt </strong>

This approach also can be used when running from within GDB.

<h2>Appendix B:– Generating Byte Codes</h2>

Using <strong>gcc</strong> as an assembler and <strong>objdump</strong> as a disassembler makes it convenient to generate the byte codes for instruction sequences. For example, suppose you write a file <strong>example.s</strong> containing the following assembly code:

<strong># Example of hand-generated assembly code pushq $0xabcdef  # Push value onto stack addq $17,%rax  # Add 17 to %rax movl %eax,%edx  # Copy lower 32 bits to %edx </strong>

The code can contain a mixture of instructions and data. Anything to the right of a ‗#‘ character is a comment.

You can now assemble and disassemble this file:

<strong>unix&gt;</strong> <strong>gcc -c example.s </strong>

<strong>unix&gt;</strong> <strong>objdump -d example.o &gt; example.d </strong>

The generated file <strong>example.d</strong> contains the following:–

<strong>example.o: file format elf64-x86-64 </strong>

<strong>Disassembly of section .text: </strong>

<strong>0000000000000000 &lt;.text&gt;: </strong>

<strong>0: 68 ef cd ab 00 pushq $0xabcdef </strong>

<strong>5: 48 83 c0 11 add $0x11,%rax </strong>

<strong>9: 89 c2 mov %eax,%edx </strong>

The lines at the bottom show the machine code generated from the assembly language instructions. Each line has a hexadecimal number on the left indicating the instruction‘s starting address (starting with 0), while the hex digits after the ‗<strong>:</strong>‘ character indicate the byte codes for the instruction. Thus, we can see that the instruction <strong>push $0xABCDEF</strong> has hex-formatted byte code

<strong>68 ef cd ab 00</strong>.

From this file, you can get the byte sequence for the code:

<strong>68 ef cd ab 00 48 83 c0 11 89 c2 </strong>

This string can then be passed through <strong>hex2raw</strong> to generate an input string for the target programs.. Alternatively, you can edit example.d to omit extraneous values and to contain <em>C</em>-style comments for readability, yielding:

<strong>68 ef cd ab 00 /* pushq $0xabcdef */ </strong>

<strong>48 83 c0 11 /* add $0x11,%rax */ </strong>

<strong>89 c2 /* mov %eax,%edx */ </strong>

This is also a valid input you can pass through HEX2RAW before sending to one of the target programs.

<h2>References</h2>

<ul>

 <li>Roemer, E. Buchanan, H. Shacham, and S. Savage. Return-oriented programming: Systems, languages, and applications. <em>ACM Transactions on Information System Security</em>, 15(1):2:1–2:34, March 2012.</li>

 <li>J. Schwartz, T. Avgerinos, and D. Brumley. Q: Exploit hardening made easy. In <em>USENIX Security Symposium</em>, 2011.</li>

</ul>




<h1>More Logistical Notes</h1>

Turn-in occurs to the grading server whenever you correctly solve a level <em>and</em> use the <strong>-s</strong> option. Upon receiving your solution, the server will validate your string and update the Attack Lab scoreboard Web page, which you can view by pointing your Web browser at <a href="https://cs2011.cs.wpi.edu/">https://cs2011.cs.wpi.edu/</a> and follow the scoreboard link.

You should be sure to check this page after your submission to make sure your string has been validated. (If you really solved the level, your string <em>should</em> be valid.)

Note that each level is graded individually. <em>You do not need to do them in the specified order</em>, but you will get credit only for the levels for which the server receives a valid message. You can check the Buffer Lab scoreboard to see how far you‘ve gotten.

The grading server creates the scoreboard by using the latest results it has for each phase.

Good luck and have fun!

<em>For redundancy</em> (in case something goes wrong with the server), please also submit your exploit strings to <em>Canvas</em>. This is the <em>Attacklab</em> assignment.

Rename the folder that you originally extracted with tar (see Page 2) with the name

<strong>userName-target<em>#</em> </strong>

where <strong>username</strong> is replaced by your WPI username and <strong>target<em>#</em></strong> is the number of the target you are solving. For example, the user <strong>lauer</strong> would submit a folder named <strong>lauer-target1023</strong>.

Next, name each individual exploit string as follows:–

<strong>ctarget.l1, ctarget.l2, ctarget.l3, rtarget.l2, rtarget.l3 </strong>

If needed, the graders will execute a script for each student of the form

<strong>cd userID </strong>

<strong>cat exploit_string | ./hex2raw | ./ctarget </strong>

or

<strong>cat exploit_string | ./hex2raw | ./rtarget </strong>

Insert these strings into the renamed folder above. Finally, <em>zip</em> the folder together and submit to <em>Canvas</em> under the project <em>Attacklab.</em>