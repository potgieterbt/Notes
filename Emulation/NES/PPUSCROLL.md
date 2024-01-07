``` asm6502
; Set the high bit of X and Y scroll.
lda ppuctrl_value
ora current_nametable
sta PPUCTRL

; Set the low 8 bits of X and Y scroll.
bit PPUSTATUS
lda cam_position_x
sta PPUSCROLL
lda cam_position_y
sta PPUSCROLL
```
