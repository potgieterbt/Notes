on the NES, addresses are 16 bits but registers (and thus read/write instructions) are 8 bits, and address space is precious and scarce.

therefore, to write a 16 bit address, you need to write two bytes, 16 bits, to PPUADDR.

basically, there's a latch that can be either set to HIGH or LOW, for the high or low byte.

When PPUADDR is written, if this internal latch is set to HIGH, it will mask out the upper 8 bits of an internal temporary address register and set them to the value written to PPUADDR. The latch is then flipped to LOW. This value is &'d with 0x3F so only the lowest 6 bits of it are actually used.

This is because the PPU can only access 14-bit addresses, from $0000 to $3FFF.

If the latch is set to LOW, the lower 8 bits  of the temporary address register are set to the value written to PPUADDR. the latch is then flipped to HIGH, and the temporary address register's value is copied to the actual internal address register that reads/writes from/to PPUDATA use.

Additionally, when PPUSTATUS is read, the latch is set to HIGH. Games will (usually?) read from PPUSTATUS first to reset the latch before writing a new address to PPUADDR. 

[[6502]]
[[PPU_old]]
