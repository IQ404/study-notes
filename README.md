# Basic Terminologies

- <ins>Script</ins> is a list of commands, interpreted by a scripting language, used to automate processes.

- Operating system (OS): a software that interacts with hardware to manage resources and perform tasks.

- Unix: a family of OS.

  E.g. MacOS

- BSD (Berkeley Software Distribution): An add-on to Unix providing additional software capabilities.

  MacOS was derived from BSD.

- When using the word Linux, people often refer to some specific distribution of Linux OS, a free, open-source family of Unix-like OS.

- Servers serving the modern webs run Linux.

- Ubuntu is a modern Linux OS.

- Linux is scure.

  Linux supports multiple users accessing the OS simultaneously.

  Linux supports multitasking.

  Linux is portable (can be run on many different hardwares).

- Linus created a free, open-source version of the unix kernel called the Linux kernel.

  A kernel here is a component of OS that communicate with the hardware through device drivers. Kernel acts as an API for other components via system calls. Kernel provides higher-level abstractions for the device drivers. Programs and other components don’t poke hardware directly, they call into the kernel.

- GNU is a free/open-source Unix-like OS (a re-implementation of Unix).

  Linux OS is the unification of GNU and the Linux kernel.

# Different Linux Distributions (i.e., Distros)

- All Linux distros use Linux kernel.

- <ins>Shell</ins>: a window for entering and receiving output from commands.

- Each Linux distro is prepackaged with a unique set of shell commands, applications, GUI etc.

  Each Linux distro provides differing level of support.

  E.g. Community-backed project vs. Commerical-enterprise-maintained.

  It can be a LTS version (Long-term Support) or a rolling release.

- Some Common Linux Distributions:

  - Debian (A core Linux distro, meaning it is not built on top of other distro)
  
  - Ubantu (built on top of Debian)
 
    3 offical editions:

    Ubantu Desktop (for PC)

    Ubantu Server

    Ubantu Core (for Internet of Things)
    
  - Red Hat Linux (another core Linux distro)

    It is shipped as RHEL (Red Hat Enterrise Linux) and is focus on enterprise customers.

  - Fedora
 
  - SLE (SUSE Linux Enterprise)
 
    Available in 2 editions: SLES (for Server), SLED (for Desktop).
 
  - Arch Linux
 
    Allows users to customize every part of the system.

# Five Layers of Linux Architecture

### UI Layer (User Interface)

The interfaces that allow users to interact with applications using devices like keyboard.

Deskotp version of Linux may contain a GUI layer, which is similar to the interface of Windows.

### Application Layer

Applications are just softwares, softwares are just programs.

Applications include:

- System tools (e.g., `top`, a program that shows live information about running processes and system resources (CPU, memory, load).)
- Programming languagues implementations
- Shells (special applications which themselves are often part of the OS)
- User apps (e.g., browers, text editors, games)

### OS Layer (Operating System)

OS controls programs (applications) and keeps them running, so the whole machine stays stable. For examples:

It boots and watches key daemons (networking, logging, ssh, cron, etc.). If one crashes, it can auto-restart it (i.e., It Detects errors and implements measures to prevent complete system failures.)

It assigns software to users (i.e., controlling who can use which programs).

It performs file management tasks.

### Kernel Layer

In a Linux system, OS is built on top of a kernel.

A Linux kernel is the lowest-level software.

It has the complete control of the OS.

The kernel starts as soon as the machine boots, and remains in RAM as long as the OS is running.

4 key jobs of a kernel:

- Memory Management
- Process Management (It decides which processes run and when, so no single task starves everything else. i.e. scheduling)
- Being the "device driver manager"
- Assuring the security of the OS.

### Hardware Layer

The physical/electronic devices of the machine (e.g., CPU, RAM, storage, screen, usb devices).

# Linux Filesystem

A tree-like structure collecting directories and files on the machine.

The filesystem assigns the access rights to the directories and files on the machine.

The top level of the filesystem is the root directory: `/`

## Some Essential Directories:

### `/bin`

A directory, exists directly below the root directory, that contains user binary files (the executables for running programs and core commands).

### `/usr`

A directory that contains user programs.

### `/home`

A directory for storing user's files.

### `/boot`

A directory that contains instructions for system startup.

### `/media`

A directory that contains files related to temporary media (e.g., CD, USB drive).
