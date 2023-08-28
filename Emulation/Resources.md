## Wikipedia:
- [Emulator](http://en.wikipedia.org/wiki/Emulator)
- [Dynamic recompilation](http://en.wikipedia.org/wiki/Dynamic_recompilation)

## General Emulation:
- [Zophar](http://www.zophar.net/) -- This is where I got my start with emulation, first downloading emulators and eventually plundering their immense archives of documentation. This is the absolute best resource you can possibly have.
- [NGEmu](http://ngemu.com/) -- Not many direct resources, but their forums are unbeatable.
- [RomHacking.net](http://www.romhacking.net/) -- The documents section contains resources regarding machine architecture for popular consoles

## Emulation Projects
- [IronBabel](http://sourceforge.net/projects/ironbabel) -- This is an emulation platform for .NET, written in Nemerle and recompiles code to C# on the fly. Disclaimer: This is my project, so pardon the shameless plug.
- [BSnes](http://byuu.org/bsnes/) -- An awesome SNES emulator with the goal of cycle-perfect accuracy.
- [MAME](http://www.mamedev.org/) -- **The** arcade emulator. Great reference.
- [6502asm.com](http://6502asm.com/) -- This is a JavaScript 6502 emulator with a cool little forum.
- [dynarec'd 6502asm](http://ironbabel.googlepages.com/6502.html) -- This is a little hack I did over a day or two. I took the existing emulator from 6502asm.com and changed it to dynamically recompile the code to JavaScript for massive speed increases.

## Processor recompilation references:

- The research into static recompilation done by Michael Steil (referenced above) culminated in [this paper](http://www.weihenstephan.org/%7Emichaste/down/steil-recompilation.pdf) and you can find source and such [here](http://www.pagetable.com/?p=35).

Perhaps the most exciting thing in emulation right now is [libcpu](http://www.libcpu.org/wiki/Main_Page), started by the aforementioned Michael Steil. It's a library intended to support a large number of CPU cores, which use LLVM for recompilation (static and dynamic!). It's got huge potential, and I think it'll do great things for emulation.

[emu-docs](http://emudocs.org/) has also been brought to my attention, which houses a great repository of system documentation, which is very useful for emulation purposes. I haven't spent much time there, but it looks like they have a lot of great resources.

