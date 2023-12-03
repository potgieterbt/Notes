Registers:
* Controller:
* Mask:
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