#Stackoverflow 
## Ways of handling processor emulation
- Interpretation
- Dynamic recompilation
- Static recompilation

Emulation has the goal of executing a piece of code to modify processor state and interact with hardware.
Processor state is a conglomeration of processor registers, interrupt handlers, etc for a given processor. E.g. 6502, has several 8-bit integers representing registers: A, X, Y, P, S and a 16-bit PC register.

### Interpretation:
- Start at IP (instruction pointer or PC - program counter) read the instruction from memory. Code parses instruction and uses this info to alter processor state as specified by your processor.
- Interpretation is very slow, each instruction needs to be decoded and perform requisite operation.

### Dynamic recompilation:
- Iterate over code like interpreter but instead of just executing opcodes, build up a list of operations once you reach a branch instruction, compile list of operations into machine code for host platform, cache this compiled code and execute it. When you hit a given instruction group again, can execute from cache. Most people compile machine code on the fly rather than make a list and then compile. 

### Static recompilation:
- Same as dynamic recompilation but follow branches. End up building a chunk of code that represents all the code in the program, can be executed with no further interference.
- Would be great except for the following problems:
	- Code that isn't in the program won't be recompiled and won't run.
	- It's been proven that finding all the code in a given binary is equivalent to the halting problem.
- Makes static recompilation impossible in 99% of cases.

### Processor Timing:
 - Certain platform require your emulator to have strict timing to be completely compatible.
 - NES, have PPU (pixels processing unit) which requires that the CPU put pixels into its memory at precise moments. If interpretation. can easily count cycles and emulate proper timing; with recompilation, a lot more complex.

### Interrupt Handling:
- Interrupts are primary way CPU communicates with hardware, generally hardware tell CPU what interrupts it cares about. When code throws a given interrupt, look at interrupt handler table and call proper callback.

## Hardware emulation:
Two sides of emulation:
- Emulating functionality of device
- Emulating actual device interfaces

HDD - Functionality is emulated by creating the backing storage, read/write/format routines, etc. This is generally straightforward.

Actual interface of the device is more complex, generally some combination of memory mapped registers and interrupts. For HDD, may have a memory mapped area where place read commands, writes, etc, then read this data back.