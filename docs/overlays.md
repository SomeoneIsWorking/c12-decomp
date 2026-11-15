# Code images — C-12: Final Resistance (USA `SCUS_946.66`)

| image | config | load | state |
|---|---|---|---|
| `SCUS_946.66` | `config/scus_946.66.yaml` | text at 0x80010000 from file offset 0x800, gp 0x800EFC10 | split; text to 0xAD694, data after it (boundaries splat's estimate, no rodata or bss split yet) |
| `RELOCS/GT.LVB` | none | relocated by `Module_Relocate` (0x800615F4) into the heap; measured at 0x8011F9BC..0x801239BC, entry blob + 0xA8 | no config: a relocatable module, split once its relocation table is read |

`Module_OpenGtLvb` (0x80059108) opens `RELOCS_GT_LVB`, allocates it, relocates it and calls its entry
(`docs/re-frontier.md`, runtime.module-load). Other `RELOCS/*.LVB` files on the disc are not yet listed.
