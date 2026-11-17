# The front end (`RELOCS/GT.LVB`)

Read in Ghidra from the module as relocated at 0x8011F9BC (psxport manifest entry `c12_gt`, taken from a RAM dump
on the publisher card). Addresses are at that base; `Module_OpenGtLvb` keeps the buffer in gp+0x29C (0x800EFEAC),
and the offsets after each address are from it.

- `Gt_Enter` (0x8011FA64, +0xA8, the module entry): loads the front end's resources, sets the state word
  `gt_State` (0x80122FA0, +0x35E4) to 1 and the card count `gt_CardHold` (0x80122FE4, +0x3628) to 300.
- `Gt_Run` (0x8011FE34, the module's +0x14): until the leave request (0x800F1B44) is set, one `Gt_Frame` and the
  state's step per iteration.
- `Gt_Frame` (0x801211E0): `Gfx_DrawSync`, `VSync(2)` (returning to 0x80121204, +0x1848), the pad scan
  (0x80043DEC), the swap (0x80121524) and the state's draw.

## States

- 0 (drawn by 0x80120BE4): ends on Start/Cross (pressed edges 0x800F0664 & 0x1020) or when its count 0x80122FF0
  runs out, back to 1 with the card count at 300.
- 1, the publisher card ("Sony Computer Entertainment America presents" and the trademark lines): each frame
  decrements `gt_CardHold`; at 0 it goes to 2. Below 0x96 (the last 150 frames) any button (edges & 0x3FFF) goes
  to 2 at once, so on the console the card holds 5 s before a press is taken and 10 s without one.
- 2, the title (`PRESS START BUTTON`): starts the title music; Start/Cross goes to 3; idle past the count
  0x80122A34 (900 on later visits) calls 0x8012119C.
- 3, the menu (LOAD GAME / NEW GAME / OPTIONS).
