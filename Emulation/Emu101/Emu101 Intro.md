#emulation #emulation101
## Numbers:
- The base-10 number system (decimal) is what we use normally and each digit multiplies by a power of 10 and there are 10 possible values for each digit.
- Base-2 (Binary) each digit multiplies by a power of 2 and only has 2 possible values for each digit. Each digit is called a bit and 8 bits makes a byte.
- Base-16 (Hexadecimal) multiplies each digit by a power of 16 and has 16 possible values for each digit. 2 digits makes a byte and 1 digit is a nibble.

## CPU Intro (8080)
- Has several 8-bit registers: A, B, C, D and E. Can be represented in C as: unsigned char A, B, C, D, E;
- Also has a Program Counter (PC), can be thought of as a pointer: unsigned char* pc;
- A program is a collection of assembly instructions and in this example corresponds to 1-3 bytes.
- An instruction takes a certain amount of time to execute and is measured in cycles, older processors the timing was constant and was provided my the manufacturer. Timing is useful for writing efficient code, avoid instructions that take many cycles to complete.

## Logical Operations:
- AND operation results in 1 only if both numbers are 1. In C is the &.
- OR operation results in 1 if at least one of the numbers are 1. In C is the |.
- A lot of older processors often want to work with only one bit of a number.
```
/* Example 1: read the control panel */    
   char *buttons_ptr = (char *)0x2043;    
   char buttons = *buttons_ptr;    
   if (buttons & 0x4)    
    HandleLeftButton();    

   /* Example 2: turn on one LED on the control panel */    
   char * LED_pointer = (char *) 0x2089;    
   char led = *LED_pointer;    
   led = led | 0x40; //set LED controlled by bit 6    
   *LED_pointer = led;    

   /* Example 3: turn off one LED */    
   char * LED_pointer = (char *) 0x2089;    
   char led = *LED_pointer;    
   led = led & 0xBF; //mask out bit 6    
   *LED_pointer = led;
```
- Above code reads a button press, turns on an LED and turns off an LED (using masking)

## Assembly:
- A low level language, the exact steps taken to accomplish a task. Each line translates to one processor instruction.
- E.g. int a = b + 100:
	1. Load address of variable B into register #1
	2. Load contents of that memory address into register #2
	3. Add the immediate value 0x64 to register #2
	4. Load address of variable A into register #1
	5. Write contents of register #2 to address stored in register #1
	- Looks like below:
```asm
	lea    a1, #$1000        ; the address of variable a    
	lea    a2, #$1008        ; the address of variable b    
	move.l d0,(a2)    
	add.l  d0, #$64    
	mov    (a1),d0>)
```
- Have to decide what every location in memory is going to be used for.
- Parenthesis indicate memory at this address.
- Most assembly, # is an indicator a literal number (immediate value).
- Each processor has a different assembly language, difficult to move code from one processor to another.

## Stacks:
- CPUs by convention have a construct called a stack. Put number on stack (push), take number off (pop).
```
0x1000  move.l (sp), d0       ; put d0 on the stack    
0x1004  add.l  sp, #4         ; increment the stack pointer    
0x1008  move.l (sp), d1       ; put d1 on the stack    
0x1010  add.l  sp, #4         ; etc.    
0x1014  move.l (sp), a0    
0x1018  add.l  sp, #4    
0x101C  move.l (sp), a1    
0x1020  add.l  sp, #4    
0x1024  move.l (sp), #0x1030  ; return address    
0x1028  add.l  sp, #4    
0x102C  jmp #0x2040           ; the subroutine's address is 0x2040    
0x1030  move.l a1, (sp)       ; restore the values of the registers    
0x1034  sub.l  sp, #4         ; in reverse order    
0x1038  move.l a0, (sp)       ; restore the values of the registers    
0x103c  sub.l  sp, #4    
etc
```
- Code above, pushes values of d0, d1, a0, a1 on the stack, most processors use a stack pointer.
- May be a normal register used by convention as a stack pointer, or may be a special register with features for specific instructions.
## High level languages
- All register saving and restoring gets done every time a function is called, compiler does it. Inline functions avoid saving and restoring registers by including a subroutine's code into the caller.

## Calling Conventions:
- The processor dictates how subroutines talk to each other, e.g. how a caller is returned.
- The 8080  has a CALL instruction that puts return address on the stack.
- Another example, is the saving the registers the responsibility of the caller or the subroutine. A way to do this is to allow the subroutine to use, say, registers r10-r32 without saving their content, but cannot destroy r1-r9. Caller knows:
	- When functions returns contents of r1-r9 are exactly what they were.
	- Can't depend on contents of r10-r32.
	- If need value in r10-r32 after subroutine call, need to save it somewhere before calling it.
- Subroutine knows:
	- Can destroy r10-r32.
	- If want to use r1-r9, need to save contents and restore them prior to returning to caller.

## ABI:
- Application Binary Interface (ABI), made by engineers, using this, compiler writes know how to compile a code that can call code compiled by other compilers. Knowing ABI can aid debugging code when don't have the source.
- If you want to figure out what a program is doing, a good way is by labeling the addresses that are targets of CALL instructions.