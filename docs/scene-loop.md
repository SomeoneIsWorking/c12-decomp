# Scene loop, missions and the load page

Read in Ghidra from `SCUS_946.66`.

## `Game_Main` (0x8004B1F8)

`Game_Init` (0x8004B228: libraries, heap, `PROJFILE.MWD`, resource handlers), then `Scene_Loop` (0x8003736C), which
does not return.

## `Scene_Loop` (0x8003736C)

`g_SceneState` (0x800F1B40) selects the phase; `g_SceneStatePrev` (0x800F1B3C) keeps the last one.

- 6: the front end. `Module_OpenGtLvb` (0x80059108) loads `RELOCS/GT.LVB` and calls its entry, then the module's
  run (+0x14, `Module_RunGt` 0x800591D4) and close (+0x18, `Module_CloseGt` 0x80059200). The module sets the next
  state (2, or 7/8/9 for a loaded game, which become 2 with `gp+0x100` naming the restore).
- 2: `Mission_Load` (0x8003777C), then 3.
- 3: per section, `Section_Start` (0x8003798C), `Game_FrameLoop` (0x80038094) until it leaves, `Section_End`
  (0x80037BFC). The leave request `g_LeaveRequest` (0x800F1B44) the frame loop ends on picks the next state: 5 loads
  the mission at `g_MissionIndex` (state 2 again), 1, 2 and 3 return to the front end, 7 ends through state 5, 4
  starts the next section of the same mission.
- after 3: `Mission_Unload` (0x80037EF8).

`Mission_Advance` (0x800857A4) is the end of a mission: `g_MissionIndex` (0x800F1520) from the current record's next
mission (+0x36), wrapped to 0 past 0x17, `g_MissionSection` (0x800F1528) 0 and the leave request 5. `Mission_Load`
copies the index to `g_LoadedMission` (0x800F1524).

## The level table

`LevelTable` (0x800D1F78), one 0x5C-byte record per mission (24): +0 the mission module's file name (a string
pointer; records 0 and 1 are `RELOCS/TI.LVB` and name themselves next, NEW GAME starts at 2, `RONIN1.LVB`), +0x4 and +0x6 the two resource ids `Mission_Load` loads, +0x10 the section table (0x5C-byte
section records), +0x14 the module list, +0x34 the load page (0 none; bits 14-15 its style, the rest an index
into the text table at 0x800F1BC4), +0x36 the next mission, +0x3A the word 0x8005D438 reads (it runs the memory card
prompt 0x80059BD0 through gp+0x9D8), and hooks `Mission_Load` (+0x48), `Mission_Unload` (+0x50) and `Section_Load` (+0x54) call.

## The load page

`LoadPage_Begin` (0x8001068C, argument 1) shows the record's page while `Mission_Load` reads: `LoadPage_Init`
(0x80010CD4) sets up the text (`gp+0x14` = 1, flags 0x800F091A = 0x24, bit 2 waiting for a press). While it is set,
`LoadPage_Load` (0x800107FC) reads each resource asynchronously and draws the page (0x80010D6C, 0x80011850) under
`VSync(2)` until the read lands, two frames at least. `LoadPage_End` (0x80010968) with argument 1, from
`Section_Load` (0x8005D97C), keeps drawing until the text is out and, with bit 2, until the prompt's button (0x80102EF8 bit 0x20); then
clears `gp+0x14`. With argument 0 (from `Mission_Load`) it ends nothing, so the page carries over the section's
reads. On the console the prompt can only end the page once every read has landed; with the reads landing when
issued the page ends on the press alone. The page is `GRISHAM'S LOG` before the first mission.
