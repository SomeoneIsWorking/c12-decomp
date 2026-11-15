# Gameplay frame loop

Read in Ghidra from `SCUS_946.66` and checked in psx-c12's draw replay (the section run again at t = 1 links the same
ordering table, word for word, as the real frame).

## `Game_FrameLoop` (0x80038094), one iteration

1. `Gfx_DrawSync(0)` (0x8009E52C), `VSync(2)` while fewer than two fields have passed, then `Gfx_SwapAndSubmit`
   (0x800A0B80): flips the parity word gp+0xCB8, puts the draw and display environments, and for each camera on the
   list at gp+0xC90 links its DR_ENV packets (camera + 0x30 + parity * 0x1C) before the last slot of its ordering table,
   sends the table with `DrawOTag`, clears the next one and makes it the camera's build table (camera + 0x7C).
2. About fifty logic calls; `FUN_8006572C` collects the visible cells.
3. `Game_Draw(g_GameMode)` (0x80038440); the mode 0x800F1B4C is 0 in play and 1 in the pause view. Then the menu checks
   (`FUN_800386BC`), which draw nothing.

## `Game_Draw` (0x80038440)

`Task_RunAll(g_DrawTasks)`, the scene passes (0x8003A478, 0x80052064, 0x8003BDFC, 0x800A5A34), `Camera_DrawLists`
for the cameras at gp+0x258, gp+0xA08 and gp+0xA9C, `Camera_BuildMatrices` (0x8009CB34: each camera's view from the
MATRIX at camera + 0x80, rotation transposed, position into camera + 0xB8), `Obj_UpdateAll` in play only,
`Camera_DrawLists` for the main camera (gp+0x9E4, 0x800F05F4), `Obj_DrawAll`, the decal ring (0x8007E1CC,
0x8007ECCC), the terrain pass (0x800657B4) and 0x8005DD80.

It is more than a draw: `Obj_UpdateAll` (0x8002F324) moves objects. For each object on the list it runs the class's
method at class + 0x7C (the player's, 0x8001D5FC, moves it with collision through 0x8002D10C), the kind's physics
(`Obj_UpdateKind` 0x80029964, by class byte + 0x47) and `Obj_FollowMove` (0x800339A4), whose `Skeleton_Translate`
(0x8002950C) moves every joint's world translation by the object's step.

## Objects

`g_ObjectList` (0x800F1100) heads a list ended by 0x800F10FC; a record links on at +4. Class +0xC (update method
+0x7C, draw method +0x84, kind byte +0x47), world MATRIX +0x14 (rotation) with its translation at +0x28,
flags +0x60, +0x68 and +0x6C, physics +0x9C, skeleton slot +0x88.

## Skeletons and animation

The skeleton slot holds the skeleton's address S, a display-list model instance: type table S + 0x18 (0x800ECDCC:
compose +0xC `Skeleton_Compose`, draw +0x10 `Model_Draw`), parent MATRIX S + 0x14 (the object's), joint count u16
S + 0x1E, joints S + 0x24, 0x70 bytes each, and animation data S + 0x2C. A joint holds its flags +0, the address of
its animation clock +4, its track +8, its local rotation +0x10, its translation +0x24 and its world MATRIX +0x30.

`Anim_Sample` (0x800BBE78, for each instance through `Anim_SampleList` 0x800BBDFC) reads each joint's clock word: the
frame above bit 12, a 12-bit fraction below. With a fraction it interpolates the track's times and its two keys
(0x800BBB9C) and writes the local rotation (0x800BBA8C). `Skeleton_Compose` then makes each joint's world MATRIX from
its parent's (the object's, or joint `*(track + 6)`) and its local. Both run inside `Game_Draw`, so the clock is the
pose's input; the player's clock (0x801DD634 in the first mission) advances by about 0x1587 a tick.
