# Dev:
* Watching the Mesen debugger I think that the PPU vram address starts at 0 and does not increment because rendering is disabled at the start, once rendering is enabled and the vram address has been set by the CPU the PPU then starts fetching tiles and rendering and incrementing vram address. I think that even when rendering is disabled, when reached scanline 241 VBlank is set and an NMI is generated. This is because the SMB PRG is checking if VBlank is set to continue(LDA 0x2002 → BPL 0xE).
* ##### NROM:
	* I think that the Nametables are written to vram by the CPU 1 byte at a time.
	* Pattern tables seem to be Read-Only so are read from the CHR-ROM
	* Attribute tables are attached to the end of the nametables.
	* Palettes are internal to the PPU, read from palette ram.
* ### Assumptions:
	* When console starts up there is no display for a while because the cpu needs to write the nametables to ppu vram.
	* Once the vram is populated the ppu can start rendering, so the cpu the writes to the ppu registers to enable bg rendering and sprite rendering.
	* 
# Understanding:
* The PPU does the same thing every cycle. In the beginning there is a nametable and patterntable selected(I don’t know how, ?but has to be from the CPU as it is the only part that can be directly controlled)
* There are visible scanline and non-visible scanlines as well as visible and non-visible dots. The visible scanlines and dots are where the picture is displayed and the non-visible ones are where memory is accessed for the next scanlines and next frames.
## Visuals:
* ### [Pattern tables](https://www.nesdev.org/wiki/PPU_pattern_tables):
	* A pattern table consists tiles which consist of 16 bytes made of 2 planes. Each pixel colour is represented by 2 bits, bit 0 is controlled by the first plane, bit 1 is controlled by the second plane.
	* A pattern table is divided into 2 256-tile (16x16) sections, first at 0x0000-0x0FFF, second at 0x1000-0x1FFF. Often called left and right respectively due to how they affect the tile.
	* Addressing - PPUCTRL bit 4(Background pattern table location 0x0000 or 0x1000) and bit 3(Sprite pattern table location 0x0000 or 0x1000)
		DCBA98 76543210
		--------------–
		0HNNNN NNNNPyyy
		|||||| |||||+++- T: Fine Y offset, the row number within a tile
		|||||| ||||+---- P: Bit plane (0: less significant bit; 1: more significant bit)
		||++++-++++----- N: Tile number from name table
		|+-------------- H: Half of pattern table (0: "left"; 1: "right")
		+--------------- 0: Pattern table is at $0000-$1FFF

* ### [Name tables](https://www.nesdev.org/wiki/PPU_nametables):
	* 1024 byte area used for background layout, each byte controls one 8x8 char cell, name table is made up of 30 rows of 32 tiles each, 960 bytes with the remaining 64 bytes for the name table attribute table. 8x8 for each byte means that the total is 256x240 pixels or the entire screen.
	* ##### Mirroring:
		* 4 logical name tables, arranged in 2x2 pattern. occupies 1 KiB of memory, starting at 0x2000(top-left), 0x2400(top-right), 0x2800(bottom-left), 0x2C00(bottom-right)
		* NES only has 2 KiB of memory, can store 2 name tables
		* Types of mirroring:
			* Vertical - 0x2000 equals 0x2800 and 0x2400 equals 0x2C00(Super Mario Bros.).
			* Horizontal - 0x2000 equals 0x2400 and 0x2800 equals 0x2C00(Kid Icarus).
			* One-screen - All name tables refer to the same memory at a given time, mapper directly manipulates the CIRAM address bit 10.
			* Four-screen - Cartridge contains extra RAM used for all name tables.
	* ##### Background evaluation:
		* Conceptually, PPU does this 33 times for each scanline.
			1. Fetch name table entry from 0x2000-0x2FFF.
			2. Fetch corresponding attribute table entry from 0x23C0-0x2FFF, increment VRAM address within same row.
			3. Fetch low-order byte of an 8x1 pixel sliver of pattern table from 0x0000-0x0FF7 or 0x1000-0x1FF7.
			4. Fetch high-order byte of sliver from address 8 bytes higher.
			5. Turn attribute data and pattern table data into palette indices, and combine with data from sprite data using priority.
			* Also does a fetch of a 34th(nametable, attribute, pattern) tuple that is never used, but some mappers require for timing.

* ### [Attribute tables](https://www.nesdev.org/wiki/PPU_attribute_tables):
	* 64-byte array at the end of a nametable that controls which palette is assigned to each part of the background.
	* Each attribute table, starting at 0x23C0, 0x27C0, 0x2BC0, or 0x2FC0 is arranged as an 8x8 array.
	* Each byte controls the palette of a 32x32 pixel, or 4x4 tile part of the nametable and is divided into four 2-bit areas.
		Each covers 16x16 pixels or 2x2 tiles, size of the ? block in Super Mario Bros. Given palette numbers topleft, topright, bottomleft, bottomright each in the range 0-3 the value of the byte is: value = (bottomright << 6) | (bottomleft << 4) | (topright << 2) | (topleft << 0)
		or
		7654 3210
		|||| ||++- Color bits 3-2 for top left quadrant of this byte
		|||| ++--- Color bits 3-2 for top right quadrant of this byte
		||++------ Color bits 3-2 for bottom left quadrant of this byte
		++-------- Color bits 3-2 for bottom right quadrant of this byte

* ### [Palette](https://www.nesdev.org/wiki/PPU_palettes)
	* A 6-bit value in palette memory corresponds to 1 of 64 colours.
	* The emphasis bits in the PPUMASK(0x2001) provide an additional colour modifier.
	* 

## [Rendering](https://www.nesdev.org/wiki/PPU_rendering):
* Renders 262 scanlines per frame, each scanline lasts 341 PPU cycles with each cycle producing 1 pixel.
* ### Pre-render scanline(-1 or 261):
	* This is a dummy scanline, sole purpose is to fill shift registers with data for first 2 tiles of next scanline. No rendering by ppu still makes the same memory accesses it usually would. Sprite fetches, whatever data is currently in secondary OAM.
	* Scanline varies in length, depending whether an even or odd frame is being rendered.
		* For odd frames, the cycle at the end of the scanline is skipped(done internally by jumping directly from 339,261 to 0,0 replacing idle tick at the beginning of the first visible scanline with the last tick of the last dummy nametable fetch).
		* For even frames, the last cycle occurs normally, done to compensate for shortcoming with the way the ppu physically outputs its video, end result being a crisper image when screen isn’t scrolling. However, behaviour can be bypassed by kiiping rendering disabled until after this scanline has passed, results in image with dot crawl, effects similar(not the same) as interlaced video.
	* During pixels 280 through 304 vertical scroll bits are reloaded if rendering is enabled.
* ### Visible scanlines (0-239):
	* Visible scanlines, contain the graphics to be displayed on the screen. Includes rendering of both the BG and the sprites. During scanlines, the PPU is busy fetching data, so the program should not access PPU memory duing this time, unless [rendering is off](https://www.nesdev.org/wiki/PPU_registers#Mask_.28.242001.29_.3E_write).
	* #### Cycle 0:
		* This is an idle cycle, value on the ppu address bus during this cycle appears to be the same CHR address that is later used to fetch low BG tile byte starting at dot 5 (possibly calculated during the 2 unused NT fetches at end of previous scanline)
	* #### Cycles 1-256:
		* The data for each tile is fetched during this phase, each memory access takes 2 PPU cycles to complete, and 4 must be performed per tile:
			1. Nametable byte
			2. Attribute table byte
			3. Pattern table tile low
			4. Pattern table tile high (+8 bytes from pattern table tile low)
		* Data fetched from these accesses is placed into internal latches, and fed to appropriate shift registers when it’s time to do so(every 8 cycles). Because PPU can only fetch an attribute every 8 cycles, each sequential string of 8 pixels is force to have the same palette attribute.
		* Sprite 0 hit acts as if the image starts at cycle 2(which is the same cycle that the shifters shift for the first time), so sprite 0 flag will be raised at this point at the earliest. Actual pixel output is delayed further due to internal render pipelining, and first pixel is output during cycle 4.
		* Shifters are reloaded during ticks 9, 17, 25, …, 257.
		* At the beginning of each scanline, the data for the first 2 tiles is already loaded into shift regisers(ready to be rendered), so first tile the gets fetched is Tile 3.
		* While all of this is going on, sprite evaluation for the next scanline is taking place as a seperate process, independent to what’s happening here.
	* #### Cycles 257-320:
		* Tile data for the sprites on the next scanline are fetched here. Again, memory access takes 2 cycles & 4 are performed for each of the 8 sprites:
			1. Garbage nametable byte
			2. Garbage nametable byte
			3. Pattern table tile low
			4. pattern table tile high(+8 bytes from pattern table tile low)
		* The garbage fetches occur so the same circuitry that performs BG tile fetches can be used for sprite tile fetches.
		* If there are less than 8 sprites on the next scanline, dummy fetches to tile 0xFF occur for the left-over sprites, because of dummy sprite data in the secondary OAM. This data is discarded, and sprites are loaded with a transparent set of values instead.
		* In addition to this, the X positions and attributes for each sprite are loaded from teh secondary OAM ito the respective counters/latches. This happens during the second garbage nametable fetch, with attribute byle loaded during the first tick and the X coordinate during the second.
	* #### Cycles 321-336:
		* Where the first 2 tiles for the next scanline are fetched and loaded into shift registers:
			1. Nametable byte
			2. Attribute table byte
			3. Pattern table tile low
			4. Pattern table tile high (+8 bytes from pattern table tile low)
	* #### Cycles 337-340:
		* Two bytes are fetched but the purpose is unknown:
			1. Nametable byte
			2. Nametable byte
		* Both of the bytes are the same nametable byte that will be fetched at the beginning of the next scanline (tile 3). At least 1 mapper, MMC5, is known to use this string of three consecutive nametable fetches to clock a scanline counter.
* ### Post-render scanline (240):
	* PPU idles during this scanline. Even though accessingn PPU memory from the program would be safe, VBlank flag isn’t set until after this scanline.
* ### V-Blanking lines (241-260):
	* VBlank flag of PPU is set at tick 1(second tick) of scanline 241, where VBlank NMI also occurs. PPU makes no memory accesses during these scanlines, so PPU memory can be freely accessed by program.

## [Scrolling](https://www.nesdev.org/wiki/PPU_scrolling)
* 