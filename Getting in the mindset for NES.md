Actual data from memory is only ever uint_8
Addresses are always uint_16, except for Zero page
// Need to figure out how the stack and stack pointer works.

For all but immediate addressing:
1. Read address from pc and pc+1
2. Read address from the address (Optional)
3. Read data from the address

TODO:
* Implement all CPU instructions.
* Implement status byte functionality.
* Figure out the bus.
* Figure out some Mappers to be able to play games (ROM, Mapper 0).
* Figure out the PPU
* Scrolling.
* Figure out the APU
* More mappers.
* Other features