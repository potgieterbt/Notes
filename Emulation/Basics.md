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
- Iterate over code like interpreter but instead of just executing opcodes, build up a list of operations to machine code and execute it 

### Static recompilation:
- 
