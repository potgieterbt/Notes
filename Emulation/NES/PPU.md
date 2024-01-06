Registers:
* Controller:
	* Bit 0-1: Base nametable address: (0 = $2000; 1 = $2400; 2 = $2800; 3 = $2C00)
		* Are the most significant bits of the scrolling coordinates, a.k.a. when reached the end of a nametable, must switch to the next one → changing nametable address.
	* Bit 2: VRAM address increment per CPU read/write of PPUDATA: (0: add 1, going across; 1: add 32, going down)
	* Bit 3: Sprite pattern table for 8x8 sprites: (0: $0000; 1: $1000; ignored in 8x16 mode)
	* Bit 4: Background pattern table address: (0: $0000; 1: $1000)
	* Bit 5: Sprite size: (0: 8x8 pixels; 1: 8x16 pixels)
	* Bit 6: PPU master/slave select: (0: read backdrop from EXT pins; 1: output color on EXT pins)
	* Bit 7: Generate NMI at start of V-Blank interval: (0: off; 1: on)

	* If PPU is in V-Blank and PPUSTATUS($2002) vblank flag is set, changing NMI flag from 0 to 1 will immediately generate a NMI, can result in graphical errors, to avoid can read $2002 immediately to clear vblank flag before writing $2000 to enable NMI.
	* Master/Slave mode & EXT pins:
		* When bit 6 is clear(usual), PPU gets palette index for background color from EXT pins. Stock NES grounds these pins making palette index 0 the background color as expected. Secondary picture generator connected to the EXT pins would be able to replace the background with different image using colors from background palette, could be used e.g. implement parallax scrolling.
		* Setting bit 6 causes PPU to output the lower 4 bits of the palette memory index on the EXT pins for each pixel, since only 4 bits are output background and sprite pixels can't normally be distinguished this way. As EXT pins are grounded on the unmodded NES, setting bit 6 is discouraged as it could damage chip whenever it outputs a non-zero pixel value.
	* Bit 0 race condition:
		* Be careful when writing to register when outside of V-Blank if using vertical mirroring or 4-screen VRAM, when write starts on dot 257 will cause only the next scanline to be drawn from left nametable 
* Mask:
	* 
* Status:
* OAM address:
* OAM data:
* Scroll:
* Address:
* Data:
* OAM DMA:

Drawing:
*  

Read/Write of PPU:
*  

Read/Write done by CPU:
*  

Pattern Tables:
* Defines shapes of tiles that make up background and sprites.
* Each tile in pattern table is 16 bytes, made of 2 planes.
* Each bit in first plane controls bit 0 of a pixel's colour, corresponding bit in second plane controls bit 1. e.g. if only first plane bit is set to 1 then colour index is 1, if only second plane bit is set to 1 the colour index is 2. If both are set to 1 then index is 3, if neither are set then index is 0.
* Pattern table is divided into 2 256-tile sections, 0x0000-0x0FFF, a.k.a. "left", and 0x1000-0x1FFF, a.k.a. "right". In debugging they were displayed side-by-side, 128x128 pixel sections, representing 16x16 tiles from each plane.
* Addresses within pattern tables:
	* 0xD bit - 0: pattern table is at 0x000-0x1FFF
	* 0xC bit - H: Half pattern table, 0 = "left", 1 = "right".
	* 0xB-0x4 bits - N: Tile number from name table.
	* 0x3 - P: Bit plane, 0: less significant bit, 1: more significant bit.
	* 0x2-0x0 - y: Fine Y offset, row number within tile.
* Value written to PPUCTRL controls whether background and sprites use left half or right half. PPUCTRL bit 4 applies to backgrounds, bit 3 applies to 8x8 sprites and bit 0 of each OAM entry's tile number applies to 8x16 sprites.

Nametables:
* 1024 byte area of memory lays out background.
* Each byte in nametable controls an 8x8 pixel character cell.
* 30 rows and 32 tiles, rest used by attribute table.
* 4 logical nametables.
* NES only has enough VRAM for 2 nametables, therefore mirroring is used:
	* Vertical:
	* Horizontal:
	* One-screen:
	* Four-screen:
	* Other:

Attribute Tables:
* 64-byte array, 8x8 array, at end of each nametable, controls which palette is assigned to each part of the background.
* Each byte controls the palette of a 32x32 pixel or 4x4 part of the nametable, each area covers 16x16 pixel or 2x2 tiles. Given palette numbers, each in range 0 to 3.
	* Byte value:
		* bits 7 & 6: bottem right quadrant.
		* bits 5 & 4: bottem left quadrant.
		* bits 3 & 2: top right quadrant.
		* bits 1 & 0: top left quadrant.

OAM:
* Internal memory inside PPU, contains a display list of up to 64 sprites, each sprite occupies 4 bytes.
* Byte 0:
	* Y position of sprite.
	* Sprite data is delayed by one scanline, so must subtract 1 from sprites Y coordinate before writing here.
	* Hide sprite by moving offscreen, any value between 0x##EF-0x##FF
	* Sprites are never displayed on first life of picture, impossible to place sprite partially off top of screen.
* Byte 1:
	* Tile index number.
	* For 8x8 sprites: tile number of sprite within the pattern table selected in bit 3 of PPUCTRL.
	* For 8x16 sprites: PPU ignores pattern table selection and selects pattern table from bit 0 of this number.
		* 0x7-0x1 bits: Tile number of top of sprite, 0 to 254, bottom half gets next tile.
		* 0x0 bit: Bank (0x0000 or 0x1000) of tiles.
* Byte 2:
	* Attributes:
		* 0x7 bit: Flip sprite vertically
		* 0x6 bit: Flip sprite horizontally
		* 0x5 bit: Priority (0: in front of bg; 1: behind bg)
		* 0x4-0x2 bits: Unimplemented (read 0)
		* 0x1-0x0 bits: Palette (4 to 7) of sprite
	* 
* Byte 3:
	* X position of left side of sprite.
	* X-scroll values of 0xF9-0xFF results in parts of sprite past right edge of screen.
	* Not possible to have sprite partially visible on left side of screen, rather left-clipping through PPUMASK can be used to simulate this.
* DMA:
* Sprite 0 hits:
* Sprite overlapping:
* Internal operation:
* Dynamic RAM decay:

Palettes:


Memory Map:
* 0x0000 - 0x0FFF: Pattern Table 0.
* 0x1000 - 0x1FFF: Pattern Table 1.
* 0x2000 - 0x23FF: Nametable 0.
* 0x2400 - 0x27FF: Nametable 1.
* 0x2800 - 0x2BFF: Nametable 2.
* 0x2C00 - 0x2FFF: Nametable 3.
* 0x3000 - 0x3EFF: Mirrors of 0x2000 - 0x2EFF.
* 0x3F00 - 0x3F1F: Palette.
* 0x3F20 - 0x3FFF: Mirrors of 0x3F00 - 0x3F1F.