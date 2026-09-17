## Software Engineering Experience

#### **Describe a skill or knowledge you acquired recently that has been impactful for you. Why did you make this investment? What has the outcome been?**

Recently I have been putting notable effort into learning Rust. I have been working in the low-level software/embedded space for about 3 years now and have gotten quite proficient at C and C++. As talked about in the next question, I have also been reading a lot about security. With the knowledge I have gathered, I can tell what problems C and C++ have and what Rust is trying to do to fix them. I believe later systems will definitely be preferred to be written in Rust and I feel like it is a worthwhile technical skill to invest in.

As for what I am doing to learn this skill, making projects. Currently I am working on creating malloc in rust. This will help me understand heap usage in detail while technically testing my rust abilities to the limit and showing me the limits of low level systems programming in Rust.

The outcome of this has been a shift in mentality while programming and how I should be approaching memory safety and system design.

#### **What new skill would you like to learn? Why do you think this is important or timely or interesting? Why do you think you will be good at it?**

A major area I have been wanting to build skills on recently has been security in low-level systems. My process of gaining knowledge has been a lot of reading. I have been reading about vulnerabilities and exploits that I would describe as Spectre-Meltdown corresponding like Cache Side-Channel attacks, KASLR bypass, Memory Tagging exploits, CFI violation with ROP and JOP, C vulnerabilities like Use-After-Free and Stack Buffer Overflows and other similar topics. I have realized that I have gained immense amount of knowledge but I would not say I know how to use exploits in reality. To gain the necessary skills, my plan is to look up CVEs that are Remote Code Execution or Privilege Escalation vulnerabilities in the linux kernel, pulling and building the relevant vulnerable kernel version and trying to recreate the exploit to get some real world exploitation experience. Currently I am in the process of building a lab-like system so that later I have the least resistance to recreate a vulnerability. Additionally, I am trying to get into malware reverse engineering. I am implementing required containerization and safety so that I dont have to worry about getting my machine compromised in the mentioned lab.

I find cybersecurity to be an extremely pivotal field that provides a better perspective on a lot of good coding practices and general decisions made while building a low-level important software like kernels or hypervisors. Several questions like what are the issues with C/C++, why is Rust a preference nowadays and in general why do systems have these features and what they are trying to mitigate has been made clearer to me. I feel like if I am able to grasp the nature of security, that would in-general make me a better software engineer and a better programmer. This investment also opens gates to new opportunities in cybersecurity, which I presume will be in high demand in the world of AI generated codebases. I have a lot of theoretical knowledge and interest in this field and from personal experience, if I am interested in something, I can be exceptional in that field with time and hard work.

#### **What kinds of software projects have you worked on before? Which operating systems, development environments, languages, databases?**

Projects is the strongest asset on my resume. I have worked on a lot of projects that can be found on my github. My interest mainly lie in the general area of software and hardware interactions. Any topic where I require the specifications or understanding of how the hardware works to write software, I am interested. This includes low-level fields like systems programming, embedded systems and VLSI. Additionally, as mentioned above, security in these fields is also right up my alley. Here is a list with brief description of each of my project:

- eSC-V: Machine-Mode CFI-Hardened RISC-V SoC in VHDL: This is my magnum opus (atleast for now). I will talk more about this project in the _Favourite or Best Personal Project_ Section.

- Blight: This is a bias-lighting software for linux wayland. I used Pipewire and Gstreamer to screen capture and process the edges of the screen. The data then is sent to and ESP32 over Serial or WiFi where the ESP32 is connected to neopixel LEDs that create the bias lighting.

- SPEC: This is a layered micro-ROS alternative I built at my internship. The layered approach allows interchanging of the HAL without having to touch the other codebase. The goal was to create efficient low-level to high-level protocol shift and choice between ROS2 and Zenoh. The layers go as follows:
  1. The protocol layer using COBS, Protobuf and CRC16
  2. The HAL layer where Serial implementations for various platforms like linux, ESP32, Arduino and STM32.
  3. Session Creation and Management layer
  4. Linux system layer that manages conversion of protocol to ROS2 or Zenoh based on requirement
- 6502-Simulation: This is a 6502 microprocessor cycle accurate simulation written in C++.

- carve-xb: This is an XFS carving tool using CUDA. The tool parses XFS structures and efficiently only carves the unallocated area with CUDA.

- RK-G68 linux utility: My first mechanical keyboard is the Royal Kludge G68 keyboard. This unfortunately does not have a linux util, so I cannot control modes, colours and animations on linux. What I did was open a windows VM in linux and run Wireshark to sniff packets in the linux machine while giving USB passthrough to windows over qemu. Then I reverse engineered the protocol the keyboard was using to communicate with proprietary utility software and then created this utility using libusb.

- JBD_BMS: This is a Python + Arduino Library to communicate with Jiabaida Battery Management Systems. This has a documented protocol but just created it into a library for easier usage.

- Redis-From-Scratch: This is a simple tutorial based redis built from scratch in C++.

- Phishing Detection Toolkit: This was a 2 part project for a hackathon.
  1. Extension + Server: A firefox extension actively monitors what URLs the browser was reaching and sending that to the Server. The server has an ML model running that checks over 30 different features to check if the website is a phishing website with a 94% accuracy and alerts the user with the extension. I had to create the data from scratch as most phishing websites are pulled down immediately.
  2. App: This is a python Deep Learning script with VGG16 for similarity analysis. It checks a list of websites and if they are trying to imitate the original target website like a banking website. It also checks URL similarity with Levenshtein Distance Algorithm and also Favicon Similarity with Hash comparison.
- NFC-Ticket-Printer: This was a freelance project for a company that required ticketing services with their server and printing tickets aswell. An NFC card reader takes user details, ESP32 over WiFI checks with the server for authentication and validation, the prints the receipt over ESC-POS printer. A nice OLED display to show all stages.
- Simple baremetal bootloader for STM32 with libopencm3.

This list still does not contain all projects I have made but are the notable ones. I have also worked on Malloc in Rust, HTTP-Server in C and similar mini projects.

I have been using linux for the past 5 years now as my main operating system. I switched distros from Fedora -> Arch -> Ubuntu -> NixOS (Current). I use Neovim as my text editor and most (if not all) of my toolchain configurations and project configurations are managed with Nix Flakes. Most of my work (and interests) lie in low level software/hardware. Hence I have become quite proficient at languages like C, C++, Assembly (RISC-V and x86_64), Rust (learning it atm), Python, Nix and Bash. Due to my focus on low-level systems and embedded, I dont have much knowledge about Databases except one of my university classes teaching SQL.

#### **Would you describe yourself as a high quality coder? Why? When you are looking at code, how do you decide if it is high quality?**

Yes, I would say I am a high quality coder. Especially if you put the fact that I am about to graduate University into perspective, I would say I am at the very least top 5% compared to my peers. It comes down to experience and knowledge. With a 1 year long internship experience where I was responsible for complete embedded low-level systems and the numerous numbers of massive projects I have made, I feel like I have quite a good amount of programming experience. For knowledge, as mentioned above, I understand security allowing me to use safe coding practices.

There are several things that I personally look for in a high quality codebase:

1. Simplicity: While certain low level operations may require complexity, most dont. Unnecessary complexity and use of language features is bad.
2. Formatting: I can live without LLMs, I cannot without my autoformatter. Badly formatted code is very difficult to read.
3. Good commenting: Explain where explainations are required. This is probably where I have been lazy but only for my personal projects. I had the most well documented codebase at my internship.
4. Modularity: Having the codebase modular makes it so simple to test and work with it. Although I would say, some people sometimes go to the extreme. Moderate Modularity is what I would say is good.
5. Verbose Naming conventions: I dont know if this is the norm, but i truly appreciate long function and variable names. Make it easier to know exactly what they are supposed to do without going to their source where the comment might document the usage.

To boil it down to 1 sentence:

> Code that is written to be read is good code ~ Candidate 2026

#### **Outline your thoughts on open source software development. What is important to get right when you are working on open source projects? What open source projects have you worked on? Have you been an open source maintainer, and can you point to those projects?**

I am a big fan and advocate of open source. I have been using open source software since my engineering journey began and I prefer it over proprietary software. For my biggest project "eSC-V", I have gone out of my way to build the SoC end-to-end using open source tooling even when FPGA adjacent development is arguably one of the core systems that lacks much open source support. But I have still maintained my stance for my projects.

Open source is the most collaborative approach to building a project. There could be thousands of contributers and maintainers all around the globe working on something. From a technical standpoint, when working on open source projects it is extremely important to get understand the project itself but also the goals the project is trying to achieve. A good understanding of both, the subsystem codebase you might be working on and also the result that the system is trying to achieve is important. A non-technical but extremely important factor is communication. Getting your point across when communicating in a pragmatic way is what I think is one of the most important skill required to open source projects. Another important factor is simply good vibes. Everyone is trying and working to build an incredible project. There is never a need for negativity.

My most notable open source contribution is to Newlib. The optimized hand coded assembly files in RISC-V Newlib repository did not have compliance with the CFI protection RISC-V extension called Zicfilp. I implemented the required "Landing Pads" in all these assembly files and got my patch in Newlib. A general contribution (not a patch) was to RISC-V GNU Toolchain GCC where I had back and forth discussion on the behaviour of the compiler for the aforementioned extension Zicfilp. Certain compiler optimizations did not add the required "Landing Pads" at the start of functions. I sent a patch to fix this but upon having more conversation with Kito Cheng, a GCC maintainer, we came to the conclusion that this inherently provided better protection by reducing the attack surface. The linker did the hardwork in this case. These two contributions allowed me to learn the mailing lists usage. On github I have had some contributions in minor projects and issues, but nothing that notable.

My goal with most of my personal projects have been to learn some core functionality adhering to what Richard Feynman said “What I cannot create, I do not understand.”. So I have created a lot of from scratch projects to learn rather than create something valuable. I was the core maintainer for the embedded subsystem at my internship, but that is not open source. Some of my projects have stars and have been contributed to by others, but nothing worth mentioning.

#### **Our primary languages are Python, Golang, C/C++, Rust, JavaScript / TypeScript, and Bash. Please write about your familiarity with any of these, prioritizing the ones you know best. How extensive is your experience of them? How do you test applications in these languages?**

The most comfortable language for me is C and C++. I have worked with C and C++ for the past 3 years now. My experience goes from multiple embedded/low level projects to 1 year internship. I would say confidently that I am at the very least intermediate level if compared to talented engineers.

I am also familiar with Python, I used to use it before C/C++ and have done a bunch of ML projects in it. Additionally, I generally use python for writing scripts and exploits for the security CTFs.

I am at this very moment learning Rust and doing a memory allocator project in it to grasp the edge of Rust functionality in low level space. I have used basic Bash for some scripts but I dont have much fluency in that language. Similarly, I have used Nix a lot for configuration and flake creation/management but I would not say I am proficient at the language. I am also proficient at VHDL since I have built a full SoC with it.

I do not have experience with Javascript as I have focused on low-level systems. I would like to learn Golang sometime in the future but at the moment I have 0 experience in it.

Most of my projects have been personal learning endeavours so I have not put much effort into writing testcases. During my internship, we had a complete python setup to test robot behaviour and I have had contributions to it. I have extensively tested my RISC-V SoC writing testcases personally for each component and also done verification with RISCOF to get guarantee of architectural compliance. Essentially, I have worked on tests for major projects that contain loads of subsystems to test, but I lack here and will take this question as a cue to learn some testing frameworks used in the low level space like GoogleTest.

#### **How comprehensive would you say your knowledge of networking is? How have you gained this knowledge?**

I would say that I generally have a good understanding of networking. Various parts of my engineering journey have contributed to this.

1. University Classes: My degree is Electronics and Telecommunication Engineering. The telecommunication part gave us multiple classes for Computer Networks, Network Security and even further for non software engineering adjacent subjects like Cellular. I am hence familiar with most of the communication protocols and networking concepts.
2. Projects: I have built HTTP Server from scratch in C and a Redis from Scratch in C++. Hence, I have good knowledge about the linux networking stack. Coming to some embedded projects, I have worked heavily with protocols like MQTT, ROS2, Zenoh. I have also a custom built middleware for embedded devices that allows these microcontrollers to talk with the aforementioned protocols. Additionally, I have worked with WiFi and USB stack of ESP32 IDF making me more informative about these aswell.
3. Tinkering: I am heavily invested in the home lab ecosystem. From Pi-Hole to custom DNS server, from IP forwarding to Tailscale, VLANS and custom immich server for storage, I have done lot of personal networking. I also have a decent experience with tooling. I have used Wireshark to sniff USB packets to build a custom keyboard utility for linux for my unsupported keyboard and sniffed my smart bulbs packets to use them programmatically with custom scripts.

#### **How have you used relational data, such as SQL databases, in your work? How much data model design work have you done? What has been unexpected for you in working with structured data models and design?**

This is one aspect I lack heavily. While we had classes on Database Management and learnt some SQL, my focus on low level has not allowed me to learn more about databases. So, I lack in this area. I would definitely love to learn more about this area in more detail, especially its working at a low level.

**Which NoSQL or unstructured data stores have you used? Are there any you would not use again, and why? How do you reason about choosing the right data store for a project?**

As mentioned in the above, I am not very knowledgeable about this side of software engineering. I did make redis from scratch, but my goal with that project was learning the networking and polling concepts in linux, not really the data storage. Hence, I had simple hash table for storage.

#### **How comprehensive would you say your knowledge of a Linux distribution is, from the kernel up? How familiar are you with low-level system architecture, runtimes and Linux distro packaging? How have you gained this knowledge?**

I have been using linux for the past 5 years and tried out various distributions (Fedora, Arch Linux, Ubuntu, NixOS). I have tinkered a lot with my systems and am even a part of the linux-ricing community. Additionally, I am a gamer and understand fundamental concepts that the operating system is doing for me.

I have made a small program that boots up in Qemu. I would not say I made a kernel as there is literally nothing in there just a printed line saying "Hello, World". This was made to understand the UEFI booting process and what the headers looks like, what the magic numbers is and similar details. I have built the linux kernel using `make defconfig` and booted into it with Qemu and BusyBox. Again, the goal was to get familiar with the kernel build process for the aforementioned lab and also to learn more about the operating system. I understand fundamental features and their usage like bootloaders like grub and init system like systemd. I have worked with syscalls, created system daemons for my Bias Lighting program and Keyboard utility, understand dynamic linking of programs and also understand various security systems like ASLR. I understand processes and the overall infrastructure like scheduler and File descriptors of the linux kernel and the distributions associated. While not specifically worked with runtimes and languages requiring one, I understand the basic `$PATH` and environment variable structures. I have packaged a few programs with nix-pkgs aswell.

Most of the knowledge in this field I have gained to daily usage, tinkering and projects that I have built.

#### **Describe any experience you have with low-level embedded systems engineering, on Linux or other embedded operating systems and code bases.**

This is where I have had the most experience. As you mightve seen in all these previous answers, my interests lie in this domain where I write software that interacts directly or close with hardware. The low-level computation space like kernels, hypervisors and embedded systems. Essentially, I am the most excited about writing code about a system where I require the knowledge of the hardware and even more in the security in this domain.

I have made multiple embedded systems projects. Most of them are usually bare-metal and dont really require an operating system. I have also interned at a Robotics Startup as an Embedded Software Engineer - Intern. My goal at my internship was to write firmware for all the robots there. The existing firmware was extremely inefficient, monolithic and bad performing with PID refresh rate at 200 ms i.e. 5 Hz. Imagine the PID kicking in 200 ms later for correction of velocity. The previous firmware used single core, no RTOS and was virtually unmaintainable due to bad coding practices and as a result, I was hired. I do not blame the programmer, startup culture was just too intense to ship features in 1 day and eventually the codebase was 30k lines long, non-modular and inefficient. I wrote the firmware from scratch that used both ESP32 core and RTOS to boost performance by 7-10x with PID rates at 30 ms (could go lower without any issues but the encoder resolution was not that good). I wrote and maintained their core robot models firmware and also wrote a highly optimised variant for their brand new social robots that also required multi-ESP32 setup providing some distributed embedded knowledge. The internship lasted 1 year with several similar projects and I gained a massive amount of experience. I have also made 2 freelance projects where I used way more constrained microcontrollers that can be normally used for mass production. This has provided me an immense knowledge, using various MCUs like ESP32, Arduino and STM8/STM32 (worked on 4 different variants).

I have used FreeRTOS for most of my projects. I have been meaning to try out Zephyr but I have not found a useful project for it. I have the basic RTOS primitives completely mastered so I do feel learning a new RTOS would not be a major issue. As for embedded linux, presumably my usage of linux and embedded systems will have 0 issues to try it. My only usage of embedded linux was creating my Pi-Hole with raspberry pi and simple learning projects like HTTP servers.

---

## Favourite or Best Personal Project

#### **Briefly describe this personal project. What were you hoping to achieve?**

The best personal project I have made is eSC-V: Machine-Mode CFI-Hardened RISC-V SoC in VHDL from absolute scratch. This SoC features a 5 stage pipelined RISC-V RV32I Zicsr Zicfilp Smcfiss Smpmpind CPU, UART for I/O and BRAM dual channel memory. I have designed it with Hardware Enforced CFI with the ratified Zicfilp and draft Smcfiss extensions to protect from forward-edge (JOP) and backward-edge (ROP) attacks respectively. This is the first implementation that provides CFI protection for Machine-Mode Cores. Machine Mode Microcontroller-Class CPUs are nowadays used for various IoT, Edge Computing and Robotics usecases where an arbitrary read from user may lead to Stack Buffer Overflows and CFI violation. This is why I chose this topic to study and implement this CPU. The CPU has a dedicated Physical Memory Protection Unit with Smpmpind for indirect CSR configuration, Hazard detection unit and forwarding unit for a safe and stable CPU.

I have made it with end-to-end open source toolchain maintained by a Nix flake for infinite reproducibility and has been synthesized to the Tang Primer 20k FPGA working at 60 MHz. I have also verified the CPU with RISCOF Architectural Framework against the golden models Spike and Sail to ensure 100% coverage and compliance of the required specifications.

This project was massive in scale. It was not only about creating a 5 stage pipelined CPU which in itself is a massive project. It required me to create a complete SoC so I can interact with it like a normal CPU over UART. I had to bootstrap C for this CPU by creating a startup script that set the C Runtime (memory constraints, bss zeroing, stack pointer setup, heap allocation) that GCC requires. Talking about GCC reminds me, I had to build the complete RISC-V GNU Toolchain to specifically work with my unique variants of RISC-V extensions. I also packaged my built toolchain with Nix to further support reproducibility. The next step was setting up Newlib for this CPU implementing the necessary syscalls to get libc functions running on my CPU. With Newlib working I ported a few games lie tetris and pong that I could physically play over UART.

Overall this project allowed me to learn about bare-metal systems, even more than my internship and all other projects combined. It gave me complete insight on how CPUs work and I continued reading more about more advanced features that modern CPUs use like OoO (Out of Order Execution), Branch Prediction, Speculative execution etc. This project also landed me my first major contribution in open source due to the bugs I found in Newlib. I am extremely proud of this project and the hardwork I put in and I could write a novel about this project. I am also in the procedure to publish research regarding this project and the results in IEEE-xplore.

The reason why I attempted this project was that I had already been wondering and diving deep into CPUs with a previous project, 6502 Simulator in C++. The goal with this project was to absolutely understand the workings of the then-felt black box called a Computer. I gained immense knowledge from this projects starting from hardware design, how CPUs work and how code actually runs on it. My goals were met with flying colours.

#### **What inspired you to write this piece of software?**

While I would not strictly consider VHDL and CPU design as software, overall I learnt a lot about software engineering through this project. The reason why I saw this as a good project was quite simple, I wanted to go deeper into core of how Computer Systems work. Before this project, I was already aware about what assembly and C programming was and how in general it worked (6502 Simulator). But I was very curious on how CPUs work. Like how does the CPU know what instruction `x` is and how does the execution work for `y` instruction? How does CPU interact with memory? What issues and hazards does the CPU have to deal with? What kind of threats does the CPU have to worry about and what are the security measures built into the hardware itself? I built this project to answer all these questions. As a result, I now know and understand how software interacts with hardware at the lowest level.

#### **What language did you choose to write it in, and why?**

The main CPU design is written in VHDL. The reason for it is simply because during the duration of working on this project, I also had VLSI classes at my University. I was learning VHDL in class already and both the project and the classes complemented each other. The result was good grades at my class and faster implementation of the CPU.

Low level software implementation like crt0 (C runtime) was (obviously) written in RISC-V Assembly. UART drivers, Newlib Syscalls and software like CFI test scripts, Pong and Tetris were implemented in C.

I used python for scripting CFI exploits for the tests and a small backend that would be the intermediate system between UART and a Website. The website was built in React and the reason for building it was to host some fast documentation and a more modern looking CPU Demonstration (Most normal peoples eyes rollback when I display CPU stuff on a terminal. The website was built for a cooler more interesting demonstration). Also, the website was vibe-coded since I am not as familiar with Javascript.

**Which part of the exercise did you find most difficult? What did you learn?**

This project was extremely complicated. I was learning new things daily. I can recall 2 stages where I was genuinely frustrated and the process was getting a bit too complicated:

1. RISCOF Setup and verification: RISCOF is the RISC-V Compatibility Framework and is used in verification for RISC-V Architectural compliance. The project is in active development and hence, setting it up is a bit frustrating. Setting up golden models, setting up tests for my CPU, getting signatures from my CPU memory and comparing it with golden models signatures for verification can be a bit of hassle. Setting up the framework took more time than I would like to admit and was annoying. The next difficulty was the actual verification. As a newbie in CPU development, my pipeline was riddled with errors. The debugging was quite complicated where I had to read the location of signature mismatch, read the relevant objdump for the given test, look through my CPU waveforms and compare the pipeline usage to golden models log outputs. The process for multiple issues was quite intense and I could feel brainfog after debugging a few issues. This took quite a bit of time but I learnt the required process and got things working. It also provided me insight on what issues are faced during CPU development, how the hazards propagate, how BRAM 1 cycle delay must be dealt with and overall, this issue had a very positive effect on me.
2. Toolchain Bugs: Zicfilp and Smcfiss are bleeding-edge RISC-V extensions and their implementation in core utilities like GCC and Newlib are actively being developed. Naturally, there exist a few bugs. For someone who was newly getting into such low-level development, if something went wrong, I would think that there was a CPU issue. After very long debug session, I noticed that Newlib functions being called in Pong and Tetris were misbehaving. To fix the issue, I had to check what function in Newlib was actually causing the bug, then hunt down the source, make changes to the source and fix the bug and finally rebuild the toolchain(which took >30 mins). This process for each and every missing `lpad` was quite frustrating. Eventually, this was the reason I got the patch in Newlib and I am quite proud of the backstory as well. From debugging the CPU for hours to realizing it was not a CPU issue to then going though the source in `riscv-gnu-toolchain`, fixing the issue and then getting a Assembly + Security patch in Newlib was an amazing experience. It made me feel that I am part of a bigger project and also gave me some comfort, that my CPU was right and I could even way bigger projects are also written by humans.

#### **How did your code perform?**

I successfully synthesized my SoC onto an FPGA with open source toolchain like yosys, ghdl, nextpnr and apycula to get my SoC working at 60 MHz on the Tang Primer 20K FPGA. The CFI features added to the CPU provided a minimal <1 MHz overhead to my CPU. Additionally, my CPU is a 5 stage pipelined CPU so the overall throughput is still quite good. I also have 100% verified compliance with all the RISC-V extensions used. UART module written by me sits comfortable at 921600 Baud Rate. I have yet to setup a benchmark like CoreMark for my CPU and that is something I would love to do in the future. For now I am extremely happy with the results.

**Was there any particular part of this system that you felt a need to get absolutely, completely perfect? Why? How did you go about that?**

Well, the whole CPU should be perfect. Any mistake would immediately result in wrong execution and break the program running on it. Additionally, one of the most important aspects of this SoC being secure requires me to focus and never leave any issues in the CPU.

Originally, the CPU still got 100% coverage of RISCOF with a massive flaw that I noticed later on. The CSR Unit sits in the EX stage and Read and Write to CSRs would be atomic and immediately executed. This was a massive flaw because if a invalid load was detected in the memory stage, the pipeline would be flushed. However, the CSR instruction that was next in the pipeline would execute and would create remnants in the CPU i.e. Something that should never had been executed just got executed. This was a massive issue that threatened the security of the CPU and I had to change the execution to writeback the CSRs after the instruction retires in the writeback stage. This meant additional hardware for Forwarding aswell. This issue was not detected by RISCOF and general usage would also not cause this issue. But the bar was set high for a Secure and Stable CPU and meant that I had to get everything in the hardware design absolutely correct. This meant writing custom tests and talking with experts to get the CPU absolutely right. RISCOF verification was just the start.

#### **Is there anything you would do very differently if you were tackling this problem again now?**

Philosophically speaking, I genuinely believe, even though I made loads of mistakes, I would not do anything differently. Each mistake I made was such a massive learning experience that I gained more knowledge and fell deeper in love with low-level computing and design.

If I was tasked with building a similar project right now after all the knowledge gained, I would invest more heavily in verification frameworks and get formal verification with tools like SymbiYosys. The CSR bug I caught was lucky and even though my personal project is not going to affect anybody, for the future the lesson learnt was to test more. If working on an actual CPU and I did not get as lucky, yeah that would be horrifying to figure out such a bug after tapeout.

---

## Education

#### **How did you rank in your final year of high school in mathematics? Were you a top student? On what basis would you say that?**

I had 96/100 in mathematics which was highest in my school. So yes, I would say for mathematics I was a top student. I dont have a factual metric but it is quite rare (I have not met a single person) with 96+ marks at math.

**How did you rank in your final year of high school, in your home language? Were you a top student? On what basis would you say that?**

We did not have ranks, we get percentages. But based on my marks, I would say I was a decent student. I had 96/100 in mathematics and 95/100 in science, highest in my school. In these 2 subjects I was a top student. I struggled in history, political science, geography and languages which is why I would consider myself decent overall.

#### **Please state your high school graduation results or university entrance results, and explain the grading system used. For example, in the US, you might give your SAT or ACT scores. In Germany, you might give your scores out of a grading system of 1-5, with 1 being the best.**

I had 84% in 12th grade. I did not attempt competitive exams like CET/JEE and instead got direct admission in my college. This was mainly due to health issues (I got pneumonia during that time unfortunately).

#### **Can you make a case that you are in the top 5% in your academic year, or top 1%, or even higher? If so please outline that case. Make reference where possible to standardized testing results at regional or national level, or university entrance results. Please explain any specific grading system used.**

I would consider myself atleast top 10%. If specifically asked about math + science I would consider myself top 1% simply based on the marks I got for boards. Unfortunately, the reference is only actual marks since I could not go for competitive exams and get an actual rank.

For my university results, I would consider myself being top 1%. I come to this conclusion based on the following aspects:

1. Decent Grades: I have 8.05 CGPA currently while university considers a good grade to be 7.5
2. Strong co-curricular activities: Won 3 hackathons and participated in multiple. Did a 1 year long internship. Getting publication in IEEE-xplore, strong projects;

My academic grades place me around top 10%. My overall profile including projects, publication and internship places me in top 1% when compared to my graduate peers.

#### **What sort of high school student were you? Outside of class, what were your interests and hobbies? What would your high school peers remember you for?**

I was a very active student. I used to partake in sports like football, rollball and swimming competitively. I was the captain for the football team in my school. Additionally, I used to play a lot of video games, read a lot of books and was generally interested in math, physics and computer science. My school would definitely remember me for being active in all areas and generally excelling in most like sports and studies.

#### **Which university and degree did you choose? What other universities did you consider, and why did you select that one?**

I had options in a few universities by direct admission through their own entrance exams. I applied for MIT(Pune), AISSMS and DY Patil. At the end, I got feedback from a senior to consider going for SPPU (Savitribai Phule Pune University) affiliated universities as they have a wider program and are generally preferred for masters in my country and abroad. So that is the reason why I went for Ajeenkya DY Patil School of Engineering which has a massive campus and usually put effort about extra and co-curricular activites like sports and hackathons.

I went for Electronics and Telecommunication Engineering Degree. My mentality behind this was, I could learn Computer Science Concepts online through projects which I would argue is a better way to learn rather than classes. Also I was and still am quite interested in electronics. I also preferred the curriculum being closer to physics than CS degrees.

#### **Overall, what was your degree result and how did that reflect on your ability? Please help us understand the grading system for your results.**

At the moment of writing this assessment, I am about to complete my degree in 1 month. I am quite happy I took electronics and telecommunication since I gained a lot more breadth of knowledge that I genuinely find interesting. All of this while making tons of projects and going for hackathons and internships that allowed me to increase depth in low level computing.

Currently my CGPA is 8.05 out of 10. SGPA is a per semester result while CGPA is the overall current result. This is calculated based on marks with 70% given based on written exams and 30% based on Oral/Practicals/Viva, Attendance, Assignments and Termwork. I lack a bit in the attendance category and rightly have lower marks in it, hence 8.05 CGPA. I would say it was worth it for the 1 year internship, 5 different hackathons and the huge repository of projects I made.

#### **During all of your education years, from high school to university, can you describe any achievements that were truly exceptional?**

I was a top performer in sports and have multiple medals in it. I have won multiple hackathons and project competitions. I have decent grades. I cannot really pinpoint 1 occasion that is more exceptional than the others.

In my school days, I was proud of when we went State-Level in football and I was a captain.
More recently I am proud of getting my eSC-V project related research paper in IEEE-xplore.
In the future I may be the proudest for working at Canonical (Hopefully).

#### **What leadership roles did you take on during your education? Did you conceive of, and drive to completion, any initiatives outside of your required classwork?**

As mentioned earlier, I was the school football captain. We won 2 trophies under my leadership. Then generally speaking, I was the class monitor a few times in school and also a part of the student council in 10th grade.

More recently in University, I was the team leader for a few different hackathons and project competitions. We won some and lost some but we learnt a lot. This was not specifically me saying I want to be a leader but recently I have noticed people generally prefer me taking responsibilities. Similar thing happened at my internship where I was solely responsible for a large chunk of the project i.e. the embedded systems department. As an Intern, I had meetings with clients, sat in executive rooms to discuss projects and definitely did not feel like I was an intern. After some self analysis, I noticed that my competency and reliability was often rewarded with respect and as a result I got put in such leadership roles.

Other than that, I was also part of the Student Association Council in 1st year where we had various different activities setup. This was not leadership but active participation for cultural fests, tech fests and extra curricular visits. As for initiatives, I could point to the self-initiated projects that I have spoken throughout this assessment like eSC-V.

#### **Did you participate in any social or community initiatives during your education? Were you recognized in any way for this?**

I did not participate in social or community initiatives. I was generally more focused on building and learning new things and most of it was a solo venture. In hindsight, I would have enjoyed such activities.

---

## Context

#### **Outline your thoughts on the mission of Canonical. What is it about the company's purpose and goals which is most appealing to you? What do you see as risky or unappealing?**

Canonicals mission is to deliver open source software to the world. I truly respect this and a commercially successful company working on open source software is where Id point if someone asked where I would love to work.

The appealing part is working on open source, monumental projects like the linux kernel. For the past 5 years, my entire world has revolved around open source, low level projects that I find interesting. Working on projects like the linux kernel and Ubuntu Core, low level security like AppArmor and LSM and other core products Canonical contributes to is the most appealing to me.

As an advocate of the open-source ecosystem, Canonicals occasional tendency to build walled gardens within open-source is a point of friction and would be considered unappealing. However, I do understand the commercial and security aspects and reasoning behind it.

#### **How should Canonical set about winning, commercially?**

I believe Canonical is already in a very decent path (Hence would love to build a Career here). I feel like "Security as a Premium" model for open source software is an amazing area, specifically due to the recent developments in AI and LLMs that are a threat to security in open source. Canonical setting up as a secure first open source solution for all your commercial needs could be something that sets Canonical to win.

Or the other option is to change name to _Canonical AI_ to get all the VC funding (Joke).

#### **How should Canonical amplify its impact in open source?**

Canonical can amplify its impact by increasing upstream collaboration and investing more heavily in next-gen technologies. For instance, I believe RISC-V is the future of open hardware. By officially dedicating more engineering support to kernel, GNU toolchain and other crucial software in RISC-V, Canonical can cement itself at the forefront of truly open-source and next generation of computing.

#### **Why do you most want to work for Canonical?**

Canonical is genuinely one of the few companies where my exact skillset and interests map directly onto real production work. I have spent the last 5 years on Linux, contributed to Newlib and had back and forth with GCC maintainers on RISC-V toolchain behaviour, built a CFI-hardened RISC-V SoC and read CVEs for fun. The overlap with what Canonical actually works on is not forced.

Specifically, the security work appeals to me the most. AppArmor, LSM integration and Ubuntu Pros security compliance features are exactly the kind of kernel-adjacent security work I want to be doing. My entire eSC-V project was motivated by understanding and implementing hardware enforced CFI and I want to continue going deeper into this space at the software level. I am also genuinely excited about Canonicals position in the RISC-V ecosystem. Ubuntus RISC-V support is growing and given my hands-on experience building a RISC-V SoC, bootstrapping a GNU toolchain and contributing patches to core RISC-V infrastructure, I feel like I can actually contribute meaningfully here rather than spend 6 months getting up to speed.

The remote-first culture is also something I actively want. I work best when I have autonomy and am judged purely on output. I dont need a manager standing over me to be productive.

#### **Which Canonical products would you most like to work on?**

Ubuntu Core and the kernel security adjacent work is where I would love to be. This is something that I want to learn and contribute to.

I am also very interested in Canonicals work around RISC-V. Ubuntus RISC-V support is an area where I feel I can contribute meaningfully from day 1 rather than spending months getting context. I have built the toolchain, I have contributed to Newlib, I have had conversations with GCC maintainers about RISC-V extension behaviour. That context is not easy to build from scratch and I already have it.

AppArmor and LSM work would also be extremely exciting for me given my current deep dive into linux kernel security and CVE recreation lab I am building.

#### **What gives you the most satisfaction at work?**

I get the most satisfaction when I understand something at its absolute lowest level. When I bootstrapped C for my custom RISC-V CPU, setting up the C runtime from scratch, zeroing BSS, setting the stack pointer, that moment when a simple C program ran on hardware I built from scratch in VHDL was probably the most satisfied I have ever felt at a desk.

Complexities excite me and getting to the end goal of figuring out and solving a complex problem is the most satisfying feeling at work.

#### **What would you most want to change about Canonical?**

I would want Canonical to invest more heavily upstream in RISC-V infrastructure. The work is being done but it feels like it could be accelerated. Given where embedded linux and edge computing is heading, being the definitive linux distribution for RISC-V devices is an enormous opportunity and I feel like Canonical is not moving as aggressively as it could in that direction.

#### **What gets you most excited about this role?**

As stated earlier, I love complex problems. If you go through all the projects I have made, you would see that I like complicated things. The first Rust project I am attempting is a memory allocator. Canonical deals with a lot of such complex projects and problems. Working on such problems in the low-level space, may it be kernel, embedded or security is what excites me the most.

I am also excited to work with engineers who are genuinely excellent at this. My internship was great but I was largely the most experienced person in the embedded domain there which meant I was not learning from peers as much as I would have liked. Canonicals engineering team is the kind of team where I expect to be the least experienced person in the room and I think that is exactly what I need at this stage.

The RISC-V angle aswell. If there is any company where my very specific combination of RISC-V toolchain knowledge, kernel security interest and open source contributions is actually relevant to daily work, it is Canonical.
