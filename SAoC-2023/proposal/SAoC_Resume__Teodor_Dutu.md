# Teodor Dutu - Resume

GitHub account: https://github.com/teodutu

## About Myself

I became interested in maths and computer science in high school when I took part in school olympiads in both of these subjects and won national-level bronze and silver medals.
As a freshman at University POLITEHNICA of Bucharest, I was drawn to low-level programming and cybersecurity.
Therefore I aced the Assembly, Operating Systems and Compilers courses at my university.
I am now also a teaching assistant for the Assembly, Operating Systems courses.

Then I decided that, after graduating, I wanted to pursue a Master's Degree in Advanced Cybersecurity, which I started in October 2021.
But even before starting my master's degree, I pursued my interest in security by attending and then contributing to the [Security Summer School](https://github.com/security-summer-school), first to the [Binary Security track](https://security-summer-school.github.io/binary/) and then by creating a new track: [Security Essentials](https://security-summer-school.github.io/essentials/).

I first learned about D from attending the [D Summer School](https://dlang-upb.github.io/D-Summer-School/), hosted at my university by my current mentors and dissertation thesis advisors Razvan Nitu and Edi Staniloiu.
Then I also helped improve the D Summer School by developing the [Advanced Meta-Programming session](https://dlang-upb.github.io/D-Summer-School/advanced-meta/advanced-meta.html) and then teaching D labs as part of an optional course at my university

### Experience

#### Compiler for the COOL Language

I had my first contact with compilers during a university course on compilers.
While attending it, I built a compiler for the [COOL language](https://theory.stanford.edu/~aiken/software/cool/cool.html#:~:text=Cool%3A%20The%20Classroom%20Object%2DOriented,management%2C%20and%20strong%20static%20typing.).
It used the `ANTLRv4` Java library to parse the code in the front end and its output was a MIPS assembly file.
I used the `qtspim` simulator to run this assembly code and verify its correctness.

#### Supporting Larger Compressed Heaps in V8 - Google Internship

During the summer of 2022 I worked as a Software Engineering Intern at Google Munich.
My project added support for an increased compressed heap in [V8](https://v8.dev/docs)from 4GB to 8GB and 16GB.
The resulting memory overhead only reached to 5 and 15% on real-world user stories.
I also proposed a new pointer compression scheme for larger V8 heaps and reduced its memory overhead to only 25-35% on real-world user stories.

#### Research Intern - National University of Singapore

I am currently a Research Intern at the National University of Singapore.
My internship ends at the end of August, so prior to the start of SAoC.
During it I have developed a compiler framework for a CGRA tightly coupled with a CPU.
The compiler offloads annotated loops from regular applications to the CGRA to speed up their execution.

## Why I Chose This Project

Since I attended the D Summer School, I have been eager to be a part of the D community and use my low-level knowledge and skills to improve its ecosystem as much as I can.
I have also taken note of the previous failed attempts at replacing DRuntime hooks with template functions.
For this reason, I chose to work on this major improvement to DMD and DRuntime, as part of my master's dissertation project.
Now after getting my master's degree, I plan to continue working on this project as part of my PhD thesis.
Together with Răzvan Nițu we plan to publish a paper on this work once it is complete.
Our claim is that only converting the functions exposed by a library to templates is enough to obtain most of the performance benefits that come from using a template-only library while reducing the compilation time overhead.

I also participated in and won last 2 years' SAoC with the same project.
Up to now I have converted 15 runtime hooks to templates, more than triple the amount converted by my forerunners: 4.
Now I want to move further with this project and convert even more hooks to templates using the experience gained while working on the other hooks.

## Work Schedule

After dedicating my master's degree's research to this project, I want to continue working on it during my PhD.
I plan to dedicate at least 20 hours per week to this project, which shall fit nicely into my schedule even after accounting for other projects and teaching. 
