Actual data from memory is only ever uint_8
Addresses are always uint_16, except for Zero page
// Need to figure out how the stack and stack pointer works.

I need to figure out how to initialise each element of the system:
* figure out what each element belongs to. (i.e. cpu, ppu, emulator as a whole)
* if there is a limited amount of the element that can exist.
* what the starting point is. (the cpu, the bus, or the emulator -> need to create a emu class

SimpleNES:
* There is a emulator class and the other elements belong to.
* Emu class is derived from cpu and ppu.
* CPU takes a bus as an argument to constructor.
* Mapper takes a cartridge as an argument.

MedNES:
*  I'll probably do this way.
* ROM gets defined, then mapper is defined, then the PPU and then initialise the CPU by passing all the previous as arguments to the constructor. 

For all but immediate addressing:
1. Read address from pc and pc+1
2. Read address from the address (Optional)
3. Read data from the address

PPU:
* Screen (256x240) is split into 8x8 pixel bitmap.
* 8 palette tables, each has 3 indexable colours.
* Attribute tables are the screen split into 2x2.

TODO:
* Implement all CPU instructions. (done i think)
	* Need to figure out cycles for instructions.
	* Implement Reset function. (https://bugzmanov.github.io/nes_ebook/chapter_3_2.html)
* Implement status byte functionality. (done i think)
* Figure out the bus. (maybe done)
* Implement cartridge functionality. (half done)
* Figure out some Mappers to be able to play games (ROM, Mapper 0).
* Figure out the PPU
* Scrolling.
* Figure out the APU
* More mappers.
* Other features