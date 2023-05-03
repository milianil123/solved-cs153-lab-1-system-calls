Download Link: https://assignmentchef.com/product/solved-cs153-lab-1-system-calls
<br>
<h1>1. Introduction</h1>

In this assignment, you will design a new system call for xv6, and use it in some fork synchronization problems. The goals of this assignment are:

Understand the system call interface

Understand how user programs send parameters to the kernel, and receive values back

See how the event handling in the Operating System works

Understand the process structure and modify it

The <strong>deliverables</strong> for the assignment are

<ol>

 <li>The code that includes the required changes (the new system call, and modi cations to exit) and</li>

 <li>The user test program(s).</li>

</ol>

You will be graded by interview and expected to demonstrate that you understand what you did and why you did it that way.

<h1>2. Getting Started</h1>

Follow the <a href="https://www.cs.ucr.edu/~csong/cs153/21f/xv6.html">instructions</a> to set up xv6 either on sledge or on your own machine using the vagrant option. Repeating from the optional <a href="https://www.cs.ucr.edu/~csong/cs153/21f/lab0.html">Lab0</a><a href="https://www.cs.ucr.edu/~csong/cs153/21f/lab0.html">:</a>

Open two terminal windows. In one, enter make qemu-gdb (or make qemu-nox-gdb ). This starts up QEMU, but QEMU stops just before the processor executes the rst instruction and waits for a debugging connection from GDB. In the second terminal, from the same directory you ran <sub>make </sub>, run <sub>gdb </sub>. If you have problems, make sure that you have followed the instructions for installing and running xv6. Note that the order of how you start these matters. You should see something like this,

<table width="675">

 <tbody>

  <tr>

   <td width="675">sledge% gdbGNU gdb (GDB) 6.8-debianCopyright (C) 2008 Free Software Foundation, Inc.License GPLv3+: GNU GPL version 3 or later &lt;http://gnu.org/licenses/gpl.html&gt; This is free software: you are free to change and redistribute it. There is NO WARRANTY, to the extent permitted by law.  Type “show copying” and “show warranty” for details. This GDB was configured as “i486-linux-gnu”.+ target remote localhost:26000The target architecture is assumed to be i8086 [f000:fff0] 0xffff0:ljmp   $0xf000,$0xe05b0x0000fff0 in ?? ()+ symbol-file obj/kern/kernel(gdb)</td>

  </tr>

 </tbody>

</table>

Now set a breakpoint on exec() by typing break exec in the gdb window then continue . You should see something like:

<table width="675">

 <tbody>

  <tr>

   <td width="675">gdb) cont Continuing<strong>.</strong>[New Thread 2][Switching to Thread 2]The target architecture <strong>is</strong> assumed to be i386<strong>=&gt;</strong> 0x80100af8 :push   <strong>%</strong>ebp Breakpoint 1, exec (path<strong>=</strong>0x1c “/init”, argv<strong>=</strong>0x8dfffe98) at exec<strong>.</strong>c:1212{(gdb)</td>

  </tr>

 </tbody>

</table>

Here we stop execution after the OS is initialized at the stage where it is starting the rst process ( init ). If you type continue again, you will break again as follows:

<table width="675">

 <tbody>

  <tr>

   <td width="675">gdb) cont Continuing<strong>.</strong>[Switching to Thread 1]<strong>=&gt;</strong> 0x80100af8 <strong>&lt;</strong>exec<strong>&gt;</strong>:push   <strong>%</strong>ebp Breakpoint 1, exec (path<strong>=</strong>0x8c3 “sh”, argv<strong>=</strong>0x8dffee98) at exec<strong>.</strong>c:12 12{</td>

  </tr>

 </tbody>

</table>

As you can see, at this stage, <sub>init</sub> started a shell process which is the xv6 shell we get when the OS boots. If you continue again, gdb will not return since it is waiting for a command to be started in the shell. Switch to the other window and try typing a command (for example, cat README ) at which time you will get another break as the shell forks then execs the cat program.

Feel free to look around at the program when it breaks to see how we reach the system call which should give you ideas about how to add one.

<h1>3. Assignment</h1>

Extend the current xv6 process implementation to maintain an exit status. To get this done, add a eld to the process structure (see proc.h ) in order to save an exit status of the terminated process. We will need this for implementing wait. Next, you have to change all system calls affected by this change (e.g., exit , wait , etc.).

<ol>

 <li>Change the exit system call signature to void exit(int status) . The exit system call must act as previously de ned (i.e., terminate the current process) but it must also store the exit status of the terminated process in the corresponding structure. In order to make the changes in exit system call you must update the following les: user.h, defs.h, sysproc.c, proc.c and all the user space programs that uses exit system call. Note, from now on, all user space programs must supply an exit status when terminated.</li>

</ol>

<strong>Hassle</strong>: one hassle that this change (and the one in part b below) introduces is that all existing places that used exit() , including ones that are in test programs have now to be changed to use the new prototype. You can either do that yourself (e.g., use <sub>grep</sub> to nd all locations of this call and change them), or create a new exit call to match the new prototype.

Goals of this part of the assignment:

Get familiar with system call arguments and how arguments are passed given the presence of two stacks (user mode and kernel mode).

Understand the backward compatibility hassles that come from modifying the system call prototype.

Carry out a gentle modi cation to an existing system call and to the Process Control Block (PCB), which will be needed by the next part of the Lab.

<ol start="2">

 <li>Update the wait system call signature to int wait(int *status) . The wait system call must prevent the current process from execution until any of its child processes is terminated (if any exists) and return the terminated child exit status through the status argument. The system call must return the process id of the child that was terminated or -1 if no child exists (or unexpected error occurred). <strong>Note</strong>: that the wait system call can receive “NULL as an argument. In this case the child’s exit status must be discarded.</li>

</ol>

Goal of this part of the assignment

Continue to get familiar with system call arguments, in this case with how to return a value.

<ol start="3">

 <li>Add a waitpid system call: int waitpid(int pid, int *status, int options) . This system call must act like wait system call with the following additional properties: The system call must wait for a process (not necessary a child process) with a pid that equals to one provided by the pid argument. The return value must be the process id of the process that was terminated or -1 if this process does not exist or if an unexpected error occurred. We are required only to implement a blocking waitpid where the kernel prevents the current process from execution until a process with the given pid terminates.</li>

 <li>Write an example program to illustrate that your waitpid works. You have to modify the make le to add your example program so that it can be executed from inside the shell once xv6 boots.</li>

 <li>(2% credit + Bonus 5%; you can get 98% the lab credit without implementing this part): Check out the waitpid option WNOHANG , for example as speci ed in this <a href="https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.1.0/com.ibm.zos.v2r1.bpxbd00/rtwaip.htm">link</a><a href="https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.1.0/com.ibm.zos.v2r1.bpxbd00/rtwaip.htm">.</a> Implement <em>WNOHANG </em>and create a version of CELEBW02 example on the same page that checks of a child process is still running (it has to be simpli ed to work with xv6, for example, avoiding the use of time). You can also make assumptions on what is returned in status and implement only an exited status (i.e., enough to run something like the CELEBW02 example).</li>

</ol>

Following are two walk through questions to help you:

<ol>

 <li>Explain how you would add your own hello world program so that you are able to execute it from the xv6 shell.</li>

 <li>Track the operation of the wait system call system call using the debugger. Explain what happens when the process has no children, and when it has a child process. You should be clear, showing all major steps in implementing the operation of the system call.</li>

</ol>

<h1>4. Help, Hints, Questions to Explore</h1>

We will be extending this section based on how I see progress (for example, to help you understand the code base, or provide ideas to make you more effective in working with the code base).

<strong>   Investing in learning gdb will really make your life easier this quarter!</strong> There are only a few commands that you need to learn to get started.

break places a break point which causes the program to stop when it reaches that point. There are many ways to set a break point; I like to use break function to cause a stop at the function that I want to look at.

continue&lt; causes gdb to continue execution until the next break point. If you want to

execute only one line of the code use next or step . The difference between the two is if the next line contains a function call, next will execute the full call before stopping while step steps into the function and stops at the rst line there.

print allows you to examine a variable. You can use some of the C semantics here, for

example to derference a pointer and see the memory location its point to (e.g., print *p ), to print a speci c format such as string or hex (e.g., print %x p ). display is

like print but it repeats the print every time gdb stops (e.g., after a break point or after a step or next).

list shows the code around the area you are.

You can nd more gdb help in the <a href="https://www.cs.ucr.edu/~csong/cs153/21f/ref.html">resources page</a>.

Another really useful tool is <sub>grep </sub>. You can grep for a keyword (or if you are adventurous, a regular expression) in a le or group of les to nd if/where it exists. So, lets say I am interested in a speci c function or variable and want to see where it is used. In the xv6 directory, you can type: grep wait *.c to search for the string in all les ending with .c .

There are a couple of other useful options for grep. The -i option (e.g., grep -i wait *.c will make grep ignore case. The <sub>-v</sub> option excludes a pattern. So, lets say you want to search for wait, but not waitpid. One way you can do it is grep wait *.c | grep -v waitpid .

Deeper understanding: When any event in xv6 occurs, the hardware switches mode to the operating system, and jumps to the code in trapasm.S (starting at label alltraps). This code builds the trap frame ( rst segment), and then sets up the memory segments for the kernel (second segment–unimportant for now).

The trap frame is a data structure built on the stack that is used to pass some important arguments to the trap handler including pointers to the user stack to enable getting arguments to system calls. Note that the user stack and the kernel stack for a process are separate. We discussed why in class. So, when we push arguments for a system call on the user stack, we need to give a pointer to the kernel to be able to access them.

You can see the trapframe structure in x86.h – its ok to abstract it away since we will not be working with it in detail.

After lling parts of the trapframe, it calls trap() with the trap frame as argument, which takes us to trap in trap.c .

Here you nd a big switch statment based on the trapno that caused the trap (this is stored in the trap frame). Each case represents an event and implements its trap handler. Only a few events are currently supported such as system calls and the timer interrupt which are necessary for the barebones xv6 to run.

If you look under the system call case (the rst case), you see some sanity checks (OS code is typically paranoid to avoid kernel panics), you see that we eventually call syscall() which is the top level handler for system calls. Let’s follow syscall which is located at the bottom of syscall.c .

We get the system call number from the <sub>EAX</sub> register in the user code (saved on the trapframe) and use it to index into the system call table and pick up the appropriate handler to call. The return value from the system call handler is stored in the register <sub>EAX</sub> in the trapframe (which is used by convention to store return values in Linux/x86) to provide the return value back to the user.

The handlers are all of the same function type (they are all called sys_xyz where xyz is the system call they handle) which enables us to use this trap handler table and call any one of them as appropriate.

Lets look at the implementation of one of these, let’s say sys_kill() which passes a signal to a process, often to kill it.

sys_kill() and several other handlers are implemented in sysproc.c because they have to do

with processes. Other handlers are implemented in the le system code, or memory code as appropriate to their operation. Use <sub>grep</sub> to nd them if you can’t gure out where they are.

You will note that sys_kill eventually calls kill where the real implementation of the system call is. This enables us to pass different parameters to each system call to get around the fact that the sys_handler are all the same type.

Note also that sys_kill had to get those parameters fro the user stack. To help with this low level read, there are a number of accessory functions ( argint , argstr , etc…) de ned to get access to the arguments based by the user by using information in the trapframe.