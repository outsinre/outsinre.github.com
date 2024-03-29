---
layout: post
title: Terminal Shell Sed
---

I was aimed to be an ICT expertise. However, still today, I don't quite get the basics of ICT origin. This post will help clarify the relations between Terminal, Shell and Sed.

This is long story. Let's begin with Terminal. At the very beginning, the only interface with the giant (as big as a house) computer was _punched tape_ along with an human being operator. Afterwards, _terminal_ was created allowing individuals to interact with computer. Terminal is a _physical device_ like the following one:

![vt100](/assets/vt100.jpg)

Usually a computer allows multiple terminals online. Each terminal is connected to input devices (e.g. keyboard) and output devices (e.g. monitor screen). Hence, a terminal is more like a serial interface/port between the computer and its peripheral devices. Sometimes, we call the physical terminal as *console*.

As time flies, standalone terminals were outdated and _virtual terminal_ emerged with smaller computer - Personal Compuer (PC). PC has only one screen and one keyboard embedded but allows switching (`Ctrl-Alt-Fn`) between multiple (6 by default, vt1 - vt6) virtual terminals. Each virtual terminal has a kernel device (/dev/tty1-6) that provides a means of input and output. It is a software terminal instead of a physical one. Most often, a virtual terminal has a _login manager_ in front before an account can interact with his default _login shell_ (i.e. POSIX Bash).

Apart from virtual terminal, we have _terminal emulator_ when X window presents. Terminal emulator is similar to a virtual terminal but managed by an X server instead of directly by the kernel.

We also have [pseudo terminal](https://unix.stackexchange.com/q/21147) (pts/xy) that is created and owned by a applications like *sshd*, terminal emulator (e.g. *xterm*) etc. A pseudo terminal is a device (e.g. file */dev/pts/04*), actually consisting of the master side and the slave side, though they appear to a single device name.

When we *ssh* to a remote server, we establish a secure TCP connection to the remote *sshd* daemon. We run a remote application through SSH, by default, which is a Shell like */bin/bash* if we does not explicitly provide a command. The remote application needs a terminal for I/O, like STDIN/STDOUT/STDERR. So *sshd* creates a pseudo terminal for the application (e.g. Bash). *sshd* is associated with the master, while the remote application is associated with the slave. Here is simple data flow:

```
|        local     |           Internet          |             remote            |
|                  |                             |                               |
| terminal --- ssh | --------------------------- | sshd - master -- slave - bash |
|                  |                             |                               |
|                  |                             |                               |
```

After that, ssh/sshd transfer data between local terminal and remote pseudo terminal. Without the slave, how would the Bash process get its STDIN/STDOUT/STDERR? Similarly, terminal emulator (e.g. xterm) also creates pseudo terminal upon startup. The emulator connects to the master side while user processes connect to the slave side. The local terminal above probably is a pseudo terminal of terminal emulator nowadays.

Sometimes you may hear of "attach STDIN/STDOUT" (like *docker attach*). It actually means creating a pipe between local terminal (e.g. xterm) and the remote terminal (e.g. docker container).

Now let's move on to Shell. Terminal is where input and output happen (like typing program names), but Shell is a _job_ manager (desktop manager does the same thing). Nowadays, OS (Linux, Unix etc.) schedules multiple processes concurrently, namely _mutli-tasking_ support. Shell is the multi-tasking interface with end users, capable of starting, stopping, suspending, resuming etc. jobs. Upon login, the default Shell is ready for interaction.

With Shell, we can _suspend_ the current _job_ to background with `Ctrl-Z`. On the contratry, `fg` put the first background job _foreground_ running again. Alternatively, `bg` send the job running background instead of suspending it. To start a program running in background, we can append `&` to program like `program-name &`. Different Shells may have different syntax to manage jobs.

Job is distinct from _process_, but both of them relate to _program_. A program is a static executable binary while a process is a running program. Job is only meaningful to Shell. Jobs refer to a _pipeline_ of processes that run interactively. For example, `ls | head` launches two processes but the pipeline defines a single job. Therefore, a job of Shell corresponds to a _process group_ of OS. Shell assign each job a job ID apart from process PID assigned by the OS. To list existing jobs, just run `jobs -l` like:

```bash
user@tux ~ $ sleep 300 &
[1] 21810
user@tux ~ $ sleep 301 &
[2] 21811
user@tux ~ $ sleep 302 &
[3] 21812
user@tux ~/workspace/outsinre.github.com $ jobs -l
[1]  21810 Running                 sleep 300 &
[2]- 21811 Running                 sleep 301 &
[3]+ 21812 Running                 sleep 302 &
```

Numbers in brackets are job IDs while numbers that follow job IDs are PID. More about job control, please refer to [Manual 7.1 Job Control Basics ](http://www.faqs.org/docs/bashman/bashref_78.html).

BTW, a _daemon_ does not belong to job as it _detach_es from Shell and gets out of Shell management. Process suspended or running in background belong to job. Putting a job background allows starting another job as the Shell is _release_d.

Finally, we go to `sed`. It is just a sample of the many programs managed by Shell. Nothing more required to clarify.

Refer to [the TTY demystified](http://www.linusakesson.net/programming/tty/index.php) and [Using pseudo-terminals (pty) to control interactive programs](http://www.rkoucha.fr/tech_corner/pty_pdip.html).
