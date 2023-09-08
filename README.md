"From the Transistor to the Web Browser"

Wrote this a few years ago, wanted to put it online. Hiring is hard, a lot of modern CS education is really bad, and it's so hard to find people who understand the modern computer stack from first principles.

Maybe if I ever get 12 free weeks again I'll offer this as a play at home course. I want to play too.

## Section 1: Cheating our way past the transistor -- 0.5 weeks
- So about those transistors -- Course overview. Describe how FPGAs are buildable using transistors, and that ICs are just collections of transistors in a nice reliable package. Understand the LUTs and stuff. Talk briefly about the theory of transistors, but all projects must build on each other so we can’t build one. On the first day the board and a kit is handed out.
- Building an FPGA board -- Board design, FPGA BGA reflow, FPGA flash, a 50mhz clock, a USB JTAG port and flasher(no special hardware, a little cypress usb mcu to do jtag), a few leds, a reset button, a serial port(USB-FTDI) also powering via USB, an sd card, expansion connector(ide cable?), and an ethernet port. Optional, expansion board, host USB port, NTSC TV out, an ISA port, and PS/2 connector on the board to taunt you. We provide a toaster oven and a multimeter thermometer to do reflow.

## Section 2: What language is hardware coded in? -- 0.5 weeks
- Talking to an FPGA(C, 200) -- A little code for the USB MCU to bitbang JTAG. A good warmup for the intensity of what’s to come.
- Blinking an LED(Verilog, 10) -- Getting all the Xilinx crap installed and your first bit file downloaded. Getting the simulator working. Learning Verilog.
- Building a UART(Verilog, 100) -- An intro chapter to Verilog, copy a real UART, introducing the concept of MMIO, though the serial port may be semihosting. Serial test echo program and led control. On the real board.

## Section 3: What is a processor anyway? -- 3 weeks
- Coding an assembler(Python, 500) -- Straightforward and boring, write in python. Happens in parallel with the CPU building. Teaches you ARM assembly. Initially outputs just binary files, but changed when you write a linker.
- Building a ARM7 CPU(Verilog, 1500) -- Break this into subchapters. A simple pipeline to start, decode, fetch, execute. How much BRAM do we have? We need at least 1MB, DDR would be hard I think, maybe an SRAM. Simulatable and synthesizable.
- Coding a bootrom(Assembler, 40) -- This allows code download into RAM over the serial port, and is baked into the FPGA image. Cute test programs run on this.

## Section 4: A “high” level language -- 3 weeks
- Building a C compiler(Haskell, 2000) -- A bit more interesting, cover the basics of compiler design. Write in haskell. Write a parser. Break this into subchapters. Outputs ARM assembly.
- Building a linker(Python, 300) -- If you are clever, this should take a day. Output elf files. Use for testing with QEMU, semihosting.
- libc + malloc(C, 500) -- The gateway to more complicated programs. libc is only half here, things like memcpy and memset and printf, but no syscall wrappers.
- Building an ethernet controller(Verilog, 200) -- Talk to a real PHY, consider carefully MMIO design.
- Writing a bootloader(C, 300) -- Write ethernet program to boot kernel over UDP. First thing written in C. Maybe don’t redownload over serial each time and embed in FPGA image.

## Section 5: Software we take for granted -- 4 weeks
- Building an MMU(Verilog, 1000) -- ARM9ish, explain TLBs and other fun things. Maybe also a memory controller, depending on how the FPGA is, then add the init code to your bootloader.
- Building an operating system(C, 2500) -- UNIXish, only user space threads. (open, read, write, close), (fork, execve, wait, sleep, exit), (mmap, munmap, mprotect). Consider the debug interface you are using, ranging from printf to perhaps a gdbremote stub into kernel. Break into subchapters.
- Talking to an SD card(Verilog, 150) -- The last hardware you have to do. And a driver
- FAT(C, 300) -- A real filesystem, I think fat is the simplest
- init, shell, download, cat, ls, rm(C, 250) -- Your first user space programs.

## Section 6: Coming online -- 1 week if you did everything else well
- Building a TCP stack(C, 500) -- Probably coded in the kernel, integrate the ethernet driver into the kernel. Add support for networking syscalls to kernel. (send, recv, bind, connect)
- telnetd, the power of being multiprocess(C, 50) --  Written in C, user can connect multiple times with telnet. Really just a bind shell.
- Space saving dynamic linking(C, 300) -- Because we can, explain how dynamic linker is just a user space program. Changes to linker required.
- So about that web(C, 500+) -- A “nice” text based web browser, using ANSI and terminal niceness. Dynamically linked and nice, nice as you want.

=================================================================================================================================================================================================================================================================================================================

METAMETA: This project is about fun and learning. It is about the journey, NOT THE FUCKING DESTINATION. Enjoy yourself. If you aren't, stop and go weave baskets.

META: Perhaps this is project is what you've been thinking about. Moving past academic exercises, this is computing and why does it suck so bad?

Let's go slow and make the right decisions. Can we code in languages that are less likely to have bugs? i.e. avoid Python and C, think OCaml, Haskell, and Rust. Things worth learning, not trash you'll code fast than throw yourself at for days debugging. We are writing the whole stack, we have choices.

Am I just Urbit but a couple years late? TODO: Explore the Urbit stack.


What's changed is there's tons of compute in the world for compilation. Self hosting is an explicit non-goal. Compilers should do more "search" and "model", and less "translate".


Section 1:

Do the board in EAGLE, it's free for hobbyists
https://www.autodesk.com/products/eagle/free-download

FPGA Choice, Artix-7? Spartan-7? Ugh, all BGA
Spartan-6

Hmm, https://wiki.debian.org/FPGA/ says Lattice has an open toolchain

How many LUTs will we need, I think 10k+

I like ($34):
https://www.digikey.com/product-detail/en/xilinx-inc/XC7A35T-1FTG256C/122-1910-ND/5039074
33208 LUTs
1843200 bits of RAM = 225kb, should be enough. can swap on SD :)

It's BGA, but big pitch, should be doable on a two layer board, and overcome BGA fear.
Pin conpatible with up to 100k LUT.

For programming and serial ($12):
https://www.digikey.com/products/en?mpart=CY7C68013A-56PVXC&v=428
USB mini B

2x tricolor LED (one for each microcontroller)
SD card connector, need special hardware?

Ethernet PHY, MII Interface ($2.38)
https://www.digikey.com/product-detail/en/microchip-technology/KSZ8041TL/576-1619-ND/1534022

Jack with built in magnetics ($4.30)
https://www.digikey.com/product-detail/en/stewart-connector/SI-50170-F/380-1103-ND/1033369

Power over USB
Reset the FPGA over USB

Update: Can we find an FPGA dev board I can just buy? This should also all work in simulation, so we can start soon. Perhaps I can find something good in Shenzhen. Arty A7?

Update: Bought a http://em.avnet.com/s6microboard

Section 2:

We'll have to use Linux, there's no ISE for Mac.
Actually it looks like it's "Vivado" now.
Nope, ISE for Spartan 6, but Vivado for Artix 7

Update: Can we find something that works on Mac? Can we hack something? I really hate the bloated synth tools anyway. Or avoid Xilinx?


Section 3:

What's the minimal subset of ARM needed to be a decent processor? Based off Thumb? I don't think subleq is O(1) to real assembly languages. Turing machines aren't.

It's fine to implement instructions in "software" aka "microcode"

Also consider an LLVM machine. Is the LLVM spec nice?


Section 4:

Does that stuff make sense or is it legacy? The CompCert docs have an interesting take on what C is.



Section 5:

TODO: Read docs on microkernels.
