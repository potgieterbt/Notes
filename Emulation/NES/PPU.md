Understanding:
* I think it is best to have a variable that contains the entire screen to be rendered for imgui or sdl2
* A nametable is a quarter of the render screen and is used for the background, only 2 nametables can be held in memory at once as there is only 2KiB and each one is 1KiB.
* To be able to render an entire screen with 2 nametables [[#^e76062|mirroring]] is used where the nametable is mirrored to a different part of the screen.
* There is a way for the CPU to access data from the PPU memory, it is done by writing to an address twice and reading the same address?
* PPU runs 3x faster than CPU, 
* I think the best way of rendering is to instantly make the render screen rather than copy the PPU exactly by only drawing 1 scanline per tick. Also avoids any of the during render glitches maybe. There may be instructions that may need to run during rendering, need to decide how to deal with this.
* When doing the sprite evaluation, I think the tile for the background is held in memory and is then compared to the sprite, I think it happens like this because this would ensure that there is only one pass when rendering.
* [Reddit: understanding ppu](https://www.reddit.com/r/EmuDev/comments/evu3u2/what_does_the_nes_ppu_actually_do/)

Registers:
* Controller >:
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
		* Be careful when writing to register when outside of V-Blank if using vertical mirroring or 4-screen VRAM, when write starts on dot 257 will cause only the next scanline to be drawn from left nametable → can cause visible glitch, can also interfere with sprite 0 hit for scanline. Glitch has no effect in horizontal or 1-screen mirroring. Only writes that start on dot 257 and continue through dot 258 can cause glitch; any other horizontal timing is safe. Glitch specifically writes value of open but to reg, which will almost always be the upper byte of the address. Writing to the reg or mirror of reg at $2100 according to desired nametable appears to be a functional workaround.
* Mask >:
	* Bit 0: Greyscale (0: normal, 1: greyscale)
	* Bit 1: 1: Show background in leftmost 8 pixels of screen; 0: Hide.
	* Bit 2: 1: Show sprites in leftmost 8 pixels of screen; 0: Hide.
	* Bit 3: 1: Show background
	* Bit 4: 1: Show sprites
	* Bit 5: Emphasize red (green on PAL/Dendy)
	* Bit 6: Emphasize green (red on PAL/Dendy)
	* Bit 7: Emphasize blue
	* NOTES:
		* Render Control:
			* Bits 3 & 4 enable rendering of background and sprites, respectively.
			* Bits 1 & 2 enable rendering of background and sprites in the leftmost 8 pixel columns. Setting these bits to 0 will mask these columns, useful in horizontal scrolling where partial sprites or tiles scroll in from the left.
			* A value of $1E or %00011110 enable all rendering, with no color effects, value of $00 disables all rendering. usually best practice to write to this reg only in vblank, to prevent partial-frame visual artifacts.
			* If either bits 3 | 4 is enabled at any time outside of vblank interval the PPU will be making continual use of PPU address and data bus to fetch tiles to render, and fetching sprite data from OAM. To make changes to the PPU memory outside vblank (via $2007), must set both bits (3 & 4) to 0 to disable rendering to prevent conflicts.
			* Disabling rendering (clearing 3 & 4) during visible part of the frame can be problematic. Can cause corruption of sprite state, which will display incorrect sprite data on next frame. It is perfectly fine to mask out sprites but leave the background on (set 3, clear 4) at any time in the frame.
			* Sprite 0 hit does not trigger in any area where the background or sprites are hidden.
		* Color Control:
			* Bit 0 controls a greyscale mode, causes palette to use only colors from grey column: $00, $10, $20, $30. Implemented as a bitwise AND with $30 on any value read from PPU $3F00-$3FFF, both on display and through black colours like $0F will be replaced by non-black grey $00.
			* Bits 5, 6 & 7 control color emphasis or tint effect. Emphasis bits are applied independently of bit 0, so will still tint color of grey image.
* Status <:
	* Bit 0-4: PPU open bus; returns stale PPU bus contents.
	* Bit 5: Sprite overflow. Intent was for flag to be set when more than 8 sprites appear on a scanline, but hardware bug caused false positives as well as false negatives; [PPU sprite evaluation](https://www.nesdev.org/wiki/PPU_sprite_evaluation). This flag is set during sprite evaluation and cleared at dot 1 of pre-render line.
	* Bit 6: Sprite 0 Hit. Set when a non-zero pixel of sprite 0 overlaps with a non-zero background pixel; cleared at dot 1 of pre-render line, used for raster timing.
	* Bit 7: Vertical blank has started (0: not in vblank; 1: in vblank). Set at dot 1 of line 241 (line *after* post-render line); cleared after reading $2002 and at dot 1 of pre-render line.
	* NOTES:
		* Reading status reg will clear bit 7 and address latch used by PPUSCROLL and PPUADDR. Does not clear sprite 0 hit or overflow bit.
		* Once sprite 0 hit flag is set, will not clear until end of next vertical blank. If attempting to use flag for raster timing, important to ensure that the sprite 0 hit check happens outside vblank, otherwise CPU will “leak” through and check will fail. Easiest way is to place an earlier check for bit 6 = 0, which will wait for pre-render scanline to begin.
		* If using sprite 0 hit to make a bottom scroll bar below a vertically scrolling or freely scrolling playfield, be careful to ensure that tile in playfield behind sprite 0 is opaque.
		* Sprite 0 hit is not detected at x=255, nor as x=0-7 if the background or sprites are hidden in this area.
		* [PPU Rendering](https://www.nesdev.org/wiki/PPU_rendering) more info on timing and clearing flags.
		* Some [Vs. System](https://www.nesdev.org/wiki/Vs._System) PPUs return a constant value in bits 4–0 that the game checks.
* OAM address >:
	* Write address of OAM to access here. Most games just write $00 and use OAMDMA.
	* Values during rendering:
		* Reg is set to zero for each of ticks 257-320 (sprite tile loading interval) of pre-render and visible scanlines. Means at end of normal complete rendering frame, reg will always return 0.
		* If rendering is enabled mid-scanline, consequences of an OAMADDR that was not 0 before OAM sprite evaluation at tick 65 of visible scanline. Value of reg at this tick determines starting address for sprite evaluation for this scanline, can cause sprite at OAMADDR to be treated as it was sprite 0, both sprite-0-hit and priority. If reg is unaligned and does not point to Y position of OAM entry, then what it points to will be reinterpreted as a Y position and following bytes will be reinterpreted. No more sprites will be found once at end of OAM is reached, hiding any sprites before starting OAMADDR.
	* [OAMADDR precautions](https://www.nesdev.org/wiki/PPU_registers#OAMADDR_precautions)
* OAM data <>:
	* Write OAM data here, writes will increment OAMADDR after write; reads don’t. Reads during vertical or forced blanking return value from OAM at that address.
	* **DO NOT write directly to this register in most cases.** Because changes to OAM should normally be made only during vblank, writing through OAMDATA is only effective for partial updates (too slow), and partial writes cause corruption, most games use DMA through OAMDMA.
	* Reading OAMDATA during rendering will expose internal OAM accesses during sprite evaluation and loading; Micro Machinces does this.
	* Writes to OAMDATA during rendering (on pre-render line and visible lines 0-239, if rendering enabled) do not modify values in OAM, but do perform a glitchy increment of OAMADDR, bumping only the high 6 bits. Extends to DMA transfers via OAMDMA, since uses writes to $2004. **Probably best to completely ignore writes during rendering.**
* Scroll >>:
	* Used to change scroll position, telling PPU which pixel of nametable selected through PPUCTRL should be at top left corner of render screen.
	* PPUSCROLL takes 2 writes: first is X scroll and second is Y scroll. Whether first or second write tracked by w register, shared with PPUADDR.
	* Typically register is written to during vblank to make next frame start rendering from desired location, but can be modified during rendering to split screen. Changes made to vertical scroll during rendering only take effect on next frame.
	* With nametable bits in PPUCTRL, scroll can be thought as 9 bits per component and PPUCTRL must be updated along with PPUSCROLL to fully specify scroll position.
	* After reading PPUSTATUS to clear w (write latch), write horizontal and vertical offsets to PPUSCROLL just before turning on screen.
	* [PPUSCROLL setting scroll offsets](PPUSCROLL)
	* Horizontal offsets range 0-255. Normal vertical offsets range 0-239, values 240-255 cause attributes data at current nametable to be used incorrectly as tile data. PPU normally skips from 239 → 0 of next nametable automatically, so invalid scroll positions only occur if explicitly written.
	* By changing scroll values here across several frames and writing tiles to newly revealed areas of nametables, can achieve effect of camera panning over large background.
* Address >>:
	* Because CPU and PPU are on separate buses, neither has direct access to each other’s memory. CPU writes to VRAM through pair of registers on PPU by first loading and address into PPUADDR and then writing data repeatedly to PPUDATA. The 16-bit address is written to PPUADDR 1 byte at a time, upper byte first, whether first or second tracked by w register, shared by PPUSCROLL.
	* After reading PPUSTATUS to clear w (write latch), write write 16-bit address of VRAM to access, upper byte first. [[PPUADDR]]. Valid addresses are $0000-$3FFF; higher addresses are mirrored down.
	* NOTE:
		* Access to PPUSCROLL and PPUADDR during screen refresh produces interesting raster effects; starting position of each scanline can be set to any pixel position in nametable memory, see [PPU scrolling](https://www.nesdev.org/wiki/PPU_scrolling).
	* [Palette corruption](https://www.nesdev.org/wiki/PPU_registers#Palette_corruption)
	* Bus conflict:
		* During raster effects, if second write to PPUADDR happens at specific times, at most one axis of scrolling will be set to bitwise AND of written value and current value. Only safe time to finish the second write is during blanking, see [PPU scrolling](https://www.nesdev.org/wiki/PPU_scrolling) for specific timings.
* Data <>:
	* VRAM read/write data register. After access, VRAM address will increment by amount determined by bit 2 of $2000.
	* When screen is turned off by disabling rendering flags with PPUMASK or during vblank, you can read or write data from VRAM through this port. Since accessing outside vertical or forced blanking because it will cause graphical glitches and if writing, write to an unpredictable address in VRAM. However, two games are know to [read from PPUDATA during rendering](https://www.nesdev.org/wiki/Reading_2007_during_rendering), See [tricky to emulate games](https://www.nesdev.org/wiki/Tricky-to-emulate_games).
	* VRAM reading and writing shares the same internal address register that rendering uses. After loading data into video memory, program should reload scroll position afterwards with PPUSCROLL and PPUCTRL (bits 1…0) writes in order to avoid wrong scrolling.
	* PPUDATA read buffer (post-fetch):
		* When reading PPUDATA while the VRAM address in the range 0-$3EFF (before palettes), read will return contents of internal read buffer. Read buffer is updated on every PPUDATA read, only after the previous PPUDATA read, but only after previous contents have been returned to CPU. Because PPU bus reads are too slow and cannot be complete in time to service the CPU read. Because of this after the VRAM address has been set through PPUADDR, one should first read PPUDATA to prime read buffer (ignoring result) before then reading desired data from it.
		* Some PPUs support reading palette data from $3F00-$3FFF.
			* Reads work differently, palette RAM is separate memory space internal to PPU that is overlaid onto PPU address space.
			* Referenced 6-bit palette data is returned immediately instead of going to internal read buffer, and hence no priming read is required.
			* Simultaneously the PPU also performs a normal read from PPU memory at specified address, underneath the palette data and result of this read goes into read buffer as normal.
			* Old contents of read buffer are discarded when reading palettes, but changing the address to point outside palette RAM and performing one read, contents of this shadowed memory (usually mirrored nametables) can be accessed.
			* On PPUs that do not support reading palette RAM, this memory range behaves the same as the rest of PPU memory.
		* Internal read buffer is updated only on PPUDATA reads. Not affected by other PPU processes like rendering, maintains its value indefinitely until next read.
	* Read conflict with DPCM samples:
		* If currently playing DPCM samples, chance that an interruption from APU’s sample fetch will cause extra read cycle if it happened at same time as instruction that read $2007. Will cause an extra increment and a byte to be skipped over, corrupting data being read. [APU DMC](https://www.nesdev.org/wiki/APU_DMC#Conflict_with_controller_and_PPU_read).
* OAM DMA >:
	* Port is located on CPU. Writing $XX will upload 256 bytes of data from CPU page $XX00- $XXFF to internal PPU OAM. Page is typically located in internal RAM, commonly $0200-$02FF, but cartridge RAM or ROM can be used as well.
	* CPU is suspended during the transfer, will take 513 or 514 cycles after $4014 write tick. (1 wait state cycle while waiting for writes to complete, +1 if on a put cycle, then 256 alternating get/put cycles. See [DMA](https://www.nesdev.org/wiki/DMA))
	* OAM DMA is the only effective method for initializing all 256 bytes of OAM. Because of the decay of OAM’s dynamic RAM when rendering is disabled, initialization should take place within vblank. Writes through OAMDATA are generally too slow for this task.
	* DMA transfer will begin at current OAM write address. It is common practice to initialize it to 0 with a write to OAMADDR before DMA transfer. Different starting addresses can be used for a simple OAM cycling technique, to alleviate sprite priority conflicts by flickering. If using this technique, after DMA OAMADDR should be set to 0 before the end of vblank to prevent potential OAM corruption. However due to OAMADDR writes also having a corruption effect, this technique is not recommended.

The PPU also has 4 internal registers, described in detail on PPU scrolling:
* v: During rendering, used for the scroll position. Outside of rendering, used as the current VRAM address.
* t: During rendering, specifies the starting coarse-x scroll for the next scanline and the starting y scroll for the screen. Outside of rendering, holds the scroll or VRAM address before transferring it to v.
* x: The fine-x position of the current scroll, used during rendering alongside v.
* w: Toggles on each write to either PPUSCROLL or PPUADDR, indicating whether this is the first or second write. Clears on reads of PPUSTATUS. Sometimes called the 'write latch' or 'write toggle'.

Rendering:
* Timing:
	* Pre-render:
	* Visible:
		* Cycle 0:
		* Cycles 1-256:
		* Cycles 257-320:
		* Cycles 321-336:
		* Cycles 337-340:
	* Post-render scanline (240):
	* Vertical blanking lines (241-260):

NMI:
* 

Scrolling:
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
* Mirroring: ^e76062
	* NES only has enough VRAM for 2 nametables, therefore mirroring is used:
		* Vertical: $2000 equals $2800 and $2400 equals 2C00 (e.g. Super Mario Bros.)
		* Horizontal: $2000 equals $2400 and $2800 equals $2C00 (e.g. Kid Icarus)
		* One-screen: All nametables refer to the same memory at any given time, and mapper directly manipulates CIRAM address bit 10.
		* Four-screen: CIRAM is disabled and cartridge contains additional VRAM used for all 4 nametables.
		* Other: Some advanced mappers can present arbitrary combinations of CIRAM, VRAM or CHR ROM in nametable area. Rarely used.
* Background evaluation:
	* Conceptually, the PPU does this 33 times for each scanline:

		1. Fetch a nametable entry from $2000-$2FBF.
		2. Fetch the corresponding attribute table entry from $23C0-$2FFF and increment the current VRAM address within the same row.
		3. Fetch the low-order byte of an 8x1 pixel sliver of pattern table from $0000-$0FF7 or $1000-$1FF7.
		4. Fetch the high-order byte of this sliver from an address 8 bytes higher.
		5. Turn the attribute data and the pattern table data into palette indices, and combine them with data from sprite data using priority.
	* It also does a fetch of a 34th (nametable, attribute, pattern) tuple that is never used, but some mappers rely on this fetch for timing purposes.

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
	* Most programs write a copy of OAM womewhere in CPU addressable RAM (often $0200-$02FF) and copy it to OAM each frame using OAMDMA ($4014) register. Writing N to this register causes DMA circuitry inside 2A03/07 to fully initialize the OAM by writting OAMDATA 256 times using successive bytes from starting at address $100* N. CPU is suspended while transferring.
	* Address range to copy from could lie outside RAM, though only useful for static screens with no animation.
	* Not counting OAMDMA write tick, above procedure takes 513 CPU cycles (+1 on odd CPU cycles): first one (or two) idle cycles, then 256 pairs of alternating read/write cycles. (comparison: unrolled LDA/SDA loop would usually take 4x longer).
* Sprite 0 hits:
	* Sprites are conventionally numbered 0-63. Sprite 0 is sprite controlled by OAM addresses $00-$03, sprite 1 is controlled by $04-$07… sprite 63 is controlled by $FC- $FF.
	* While PPU is drawing picture, when opaque pixel of sprite 0 overlaps an opaque pixel of background, is sprite 0 hit. PPU detects this condition and sets bit  6of PPUSTATUS to 1 starting at this pixel, letting CPU know how far along PPU is drawing the picture.
	* Sprite 0 hit doesn’t happen:
		* If background or sprite rendering is disabled.
		* At x=0 to x=7 if left-side clipping window is enabled (bit 1-2 of PPUMASK is 0).
		* At x=255, for obscure reason related to pixel pipeline.
		* At any pixel where background or sprite pixel is transparent (2-bit color index from CHR pattern is %00).
		* If sprite 0 hit has already occurred this frame, Bit 6 of PPUSTATUS is cleared to 0 as dot 1 of pre-render line. Means only first sprite 0 hit in frame can be detected.
	* Sprite 0 hit happens regardless of:
		* Sprite priority. Sprite 0 can still hit the background behind.
		* Pixel colors. Only CHR pattern bits are relevant , not actual rendered colors, and any CHR color index except %00 is considered opaque.
		* The palette. Contents of the palette are irrelevant to sprite 0 hits. e.g. a black (0F) sprite pixel can hit a black ($0F) background as long as neither is the transparent index color index %00.
		* The PAL PPU blanking on left and right edges at x=0, x=1 and x=254, see [Overscan](https://www.nesdev.org/wiki/Overscan#PAL).
* Sprite overlapping:
	* Priority between sprites is determined by their address inside OAM. So a sprite displayed in front of another sprite in a scanline, sprite data that occurs first will overlap any other sprite after it. e.g. when sprites at OAM $0C and $28 overlap, sprite at $0C will appear in front.
* Internal operation:
		* In addition to primary OAM memory, PPU
* Dynamic RAM decay:
	* 

Palettes 2C02:


Memory Map:
* CHR-ROM/RAM:	
	* 0x0000 - 0x0FFF: Pattern Table 0.
	* 0x1000 - 0x1FFF: Pattern Table 1.
* PPU VRAM:
	* 0x2000 - 0x23FF: Nametable 0.
	* 0x2400 - 0x27FF: Nametable 1.
	* 0x2800 - 0x2BFF: Nametable 2.
	* 0x2C00 - 0x2FFF: Nametable 3.
* Mirror of VRAM (not sure where this is stored):
	* 0x3000 - 0x3EFF: Mirrors of 0x2000 - 0x2EFF.
* Always points to system palette:
	* 0x3F00 - 0x3F1F: Palette.
	* 0x3F20 - 0x3FFF: Mirrors of 0x3F00 - 0x3F1F.