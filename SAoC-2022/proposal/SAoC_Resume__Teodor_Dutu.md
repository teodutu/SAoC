# Teodor Dutu - Resume

GitHub account: https://github.com/teodutu

## About Myself

I became interested in maths and computer science in high school when I took part in school olympiades in both of these subjects and won national-level bronze and silver medals.
As a freshman at University POLITEHNICA of Bucharest, I was drawn to low-level programming and cybersecurity.
Therefore I aced the Assembly, Operating Systems and Compilers courses at my university.
I am now also a teaching assistant for the Assembly, Operating Systems courses.

Then I decided that, after graduating, I wanted to pursue a Master's Degree in Advanced Cybersecurity, which I started in October 2021.
But even before starting my master's degree, I pursued my interest in security by attending and then contributing to the [Security Summer School](https://github.com/security-summer-school), first to the [Binary Security track](https://security-summer-school.github.io/binary/) and then by creating a new track: [Security Essentials](https://security-summer-school.github.io/essentials/).

I first learned about D from attending the [D Summer School](https://dlang-upb.github.io/D-Summer-School/), hosted at my university by my current mentors and dissertation thesis advisors Razvan Nitu and Edi Staniloiu.
This year I also helped improve the D Summer School by developing the [Advanced Meta-Programming session](https://dlang-upb.github.io/D-Summer-School/advanced-meta/advanced-meta.html).

I am currently a Software Engineering Intern at Google and I am working on their JavaScript engine: [V8](https://v8.dev/docs).
My project aims to increase the size of the compresed V8 heap from 4GB to 8GB.
My internship ends prior to the start of SAoC 2022.

While being a student, I worked on the following university projects:

### COOL Compiler - Java

- Used the `ANTLRv4` Java library to parse the [COOL](https://theory.stanford.edu/~aiken/software/cool/cool.html#:~:text=Cool%3A%20The%20Classroom%20Object%2DOriented,management%2C%20and%20strong%20static%20typing.) code in the front end
- The output from the compiler's backend is a MIPS assembly file that contains all the runtime necessary to run it

### PITIX File System - C

- Created a kernel module implementing a file system similar to MINIX, called PITIX
- The filesystem supports all usual operations, such as inode creation, deletion, read, write, truncate, except for links (soft or hard)
- PITIX stores  the IMAP and DMAP in two separate blocks, as well as separating the inodes (IZONE) from the data (DZONE)

### RAID 1 Driver - C

- Designed a kernel module implementing a software RAID 1 scheme
- The two physical drives are kept identical by storing and comparing a CRC for each of their sectors

### Asynchronous HTTP Server - C

- File transmission server implemented for both `POSIX` and `WinAPI`
- Used nonblocking socket operations and asynchronous reads or zero-copy mechanisms in order to serve tens of thousands of connections at once

## Why I Chose This Project

Since I attended the D Summer School, I have been eager to be a part of the D community and use my low-level knowledge and skills to improve its ecosystem as much as I can.
I have also taken note of the previous failed attempts at replacing druntime hooks with template functions.
For this reason, I chose to complete this major improvement to DMD and druntime, as part of my master's dissertation project.
This project aims to run D applications on microcontrollers with 16KB of RAM or less, for which executables need to be as small as possible.
One major step of my masters' dissertation is represented by this project: to convert the DMD runtime hooks into templates, in order to decrease the size of the resulting binaries.

I also participated in and won last year's SAoC with the same project.
Up to now I have converted 9 runtime hooks to templates, more than double the amount converted by my forerunners: 4.
Now I want to move further with this project and convert even more hooks to templates using the experience gained while working on the other hooks.

## Work Schedule

I have dedicated my master's degree's research to completing this project and to further developing the D environment after this project is finished.
Most MSc lectures and classes at my master's take place early in the morning (until 12:00) or late in the afternoon/evening (after 16:00), which is why I will be doing most of the work required for this project in the time between 12:00 to 16:00.

In total, courses and lectures take up 16 hours per week during the first semester.
Being a teaching assistant will also add about 4 hours to my weekly schedule, bringing my workload to about 20 hours each week.
This leaves me plenty of time to work on and finish this project.
