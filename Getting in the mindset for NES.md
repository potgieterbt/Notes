Actual data from memory is only ever uint_8
Addresses are always uint_16, except for Zero page
// Need to figure out how the stack and stack pointer works.


For all but immediate addressing:
1. Read address from pc and pc+1
2. Read address from the address (Optional)
3. Read data from the address

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