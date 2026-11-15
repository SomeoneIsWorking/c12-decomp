# C-12: Final Resistance decomp

A decompilation of **C-12: Final Resistance** (USA, `SCUS-94666`), set up with
[splat](https://github.com/ethteck/splat). It holds the build-matched split configuration for the main executable, a
symbol map from our own reverse engineering, and the RE notes. No C is matched yet.

The PC port [psx-c12](https://github.com/SomeoneIsWorking/psx-c12) consumes this repository as a submodule and reads
its guest addresses from the symbol map.

## Bring your own disc

No game bytes are ever committed here: not the executable, not the modules, not the split assembly or data. Copy the
executable from your own disc into `disc/` (gitignored):

```
disc/SCUS_946.66
```

The config carries the file's SHA-1 and splat refuses a mismatch. The images are listed in `docs/overlays.md`.

## Split

```
uv run --frozen splat split config/scus_946.66.yaml
```

Output goes to `asm/` and `build/`, both gitignored. splat is pinned in `pyproject.toml`/`uv.lock`.

## Layout

| path | what |
|---|---|
| `config/scus_946.66.yaml` | the splat config |
| `config/symbols/scus_946.66.txt` | the symbol map, splat `symbol_addrs` format |
| `docs/overlays.md` | the code images and their load addresses |
| `docs/re-frontier.md` | the ordered RE evidence chain |
| `docs/prior-art.md` | other C-12 work, licences and what we take |

## Licence

MIT (`LICENSE`), covering this repository's own files only. C-12: Final Resistance is © Sony Computer Entertainment;
nothing of it is distributed here.

Built with AI assistance (Claude and other LLMs).
