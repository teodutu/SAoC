# Teodor Dutu - Resume
GitHub account: https://github.com/teodutu

## About Myself
My passion for programming started in high school when I started taking part in
school olympiades in both informatics and mathematics, where I obtained national
bronze and silver medals. As a freshman at the Politehnica University of
Bucharest I developed a passion for low-level programming and cybersecurity.
This is when I decided that, after graduating, I wanted to pursue a Master's
Degree in Advanced Cybersecurity, which I will be starting in October 2021.

But even before starting my master's degree, I pursued my interest in security
by attending and then contributing to the
[Security Summer School](https://github.com/security-summer-school). Continuing
to deepen my knowledge of low-level programming, I aced the Operating Systems
and Compilers classes at my university, for which I successfully implemented the
projects below:

### COOL Compiler - Java
- Used the `ANTLRv4` Java library to parse the COOL code in the front end
- The output from the compiler's back end is a MIPS assembly file that contains
all the runtime necessary to run it

### PITIX File System - C
- Created a kernel module implementing a file system similar to MINIX, called
PITIX
- The filesystem supports all usual operations, such as inode creation,
deletion, read, write, truncate, except for links (soft or hard)
- PITIX stores  the IMAP and DMAP in two separate blocks, as well as
separating the inodes (IZONE) from the data (DZONE)

### RAID 1 Driver - C
- Designed a kernel module implementing a software RAID 1 scheme
- The two physical drives are kept identical by storing and comparing a CRC for
each of their sectors

### Asynchronous HTTP Server - C
- File transmission server implemented for both `POSIX` and `WinAPI`
- Used nonblocking socket operations and asynchronous reads or zero-copy
mechanisms in order to serve tens of thousands of connections at once

Wanting to learn even more, I took interest in the D community and its
ecosystem. I completed the [D Summer School](https://ocw.cs.pub.ro/courses/dss),
during which I worked on a few PRs:
- https://github.com/dlang/phobos/pull/8179
- https://github.com/dlang/druntime/pull/3526
- https://github.com/dlang/dmd/pull/12937

## Why I Chose this Project
Since I attended the D Summer School, I have been eager to be a part of the
D community and use my low-level knowledge and skills to improve its ecosystem
as much as I can. I have also taken note of the previous failed attempts at
replacing druntime hooks with template functions For this reason, I chose to
complete this major improvement to DMD and druntime, as part of my master's
dissertation project. This project aims to run D applications on
microcontrollers with 4KB of RAM or less, for which executables need to be as
small as possible. The first step of my masters' dissertation is represented by
this project: to convert the DMD runtime hooks into templates, in order to
decrease the size of the resulting binaries.

## Work Schedule
I plan to dedicate my master's degree's research to completing this project and
to further developing the D environment after this project is finished. Most MSc
lectures and classes at my master's take place early in the morning (until 12
am) or late in the afternoon/evening (after 4 pm), which is why I will be doing
most of the work required for this project in the time between 12 am to 4 pm.

In total, courses and lectures take up 16 hours per week during the first
semester. Being a teaching assistant will also add about 4 hours to my weekly
schedule, bringing my workload to about 20 hours each week. This leaves me
plenty of time to work on and finish this project. I will also be working on
weekends in order to make sure I complete all milestones on time.
