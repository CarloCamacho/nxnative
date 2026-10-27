# Return of the Obra Dinn (`010032E00E6E2000`)

Base game 1.0.0 (v0), SDK 8.2.1. Unity 2017.4.28f1 IL2CPP with FMOD on NVN. Modules `main`,
`subsdk0` (the movie module) and `sdk`. No host change was needed and no rewriter cuts are
registered for this game.

On the Mac: the save-slot menu, the opening briefing letter and the first-person deck scene,
at 60 fps (frame time p50 16.66 ms, GPU p50 3.49 ms), with audio; no crash and no dropped frames
over a 40-second run. Two render pipelines fail to build at startup and their draws are skipped;
see [Open](#open).

## What it needs

| Setting | Where | Why |
|---|---|---|
| `modules main subsdk0 sdk` | [`game.tsv`](game.tsv) | `main` imports from all three; `subsdk0` is NintendoSDK_movie |
| `sys_resource 0x0` | [`game.tsv`](game.tsv) | the NPDM declares 0 MB of system resource: `IsVirtualAddressMemoryEnabled` is false, so no heap check runs |
| `mem_reserve 0x100000000`, `mem_alias 0x10000000`, `mem_stack 0x10000000`, `app_pool 0xCD500000` | [`game.tsv`](game.tsv) | `main` sets a heap of at most the pool − 64 MB: a 3328 MB heap region |
| `shader_groups /Data/=data` | [`game.tsv`](game.tsv) | the shaders are Shader objects in `Data/*.assets`, `data.unity3d` and the UnityFS bundles |
| `romfs_marker Data/sharedassets1.assets` | [`game.tsv`](game.tsv) | the RomFS is a normal folder tree |

## Files

- `Data/Resources/unity default resources` and `unity_builtin_extra` are serialized files with no
  type trees, and the game ships no bundle carrying a Shader tree of the same hash to borrow, so the
  profile relies on the Unity 2017.4 Shader layout in `dm/gfx/tools/unityfs.py`.
- 273 RomFS files, 1.95 GB, 13 UnityFS bundles.

## Shaders

- 1,260 shaders, split from the UnityFS bundles and `data.unity3d`. The game has no PC version to
  pair against, so `shader_pairs.tsv` holds a single vertex/pixel pair and most vertex shaders are
  translated without a known partner.

## Memory

- Resident footprint stayed between 950 MB and 1.2 GB over the runs; `mem_used` is 0x4600000 after
  the rewrite.

## Open

Two pipelines fail to build, once each at startup, and their draws are skipped:

```
failed to create render pipeline (vertex fc0fe6645a7a8a0a_b fragment 55b351df8df20311_b):
  Fragment input(s) `user(locn1)` mismatching vertex shader output type(s) or not written by vertex shader
failed to create render pipeline (vertex 4f00dcc565bde774_b fragment 7deb26f61143a68c_b): (same)
```

Everything visible renders correctly around them. Not investigated further yet.

Not tested: iPhone (no IPA built), and no `regress.py` route, so no `smoke.tsv`.
