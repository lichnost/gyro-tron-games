# gyro-tron-games

The community game registry for [Gyro-Tron](https://gitlab.com/secstant/gyro-tron-move) —
one JSON file per game under `registry/`, each approved by human review of a pull
request. This repository holds **pointers only**: game code lives in each author's own
repository, and no package is stored or served from here. The console's catalog
pipeline collects the registry at a pinned commit; accepting a game here makes it
*offered*, nothing more — the loader sandbox, not a signature, is the trust boundary
for community packages.

This repository's workflows are kept in the console repository
(`deploy/registry/`) and published here by its `tools/publish_registry.py`; the
exported SDK they run comes from [gyro-tron-dev](https://github.com/lichnost/gyro-tron-dev)
at the pinned commit named in each workflow.

## Submitting a game

1. Put the game in **your own repository**: the game folder (`game.json`, the `.c`,
   `icon.png`, assets), screenshots, and a `LICENSES.md` at the repository root
   declaring where every asset came from (template below). The game itself is plain C
   against one header — see the
   [SDK](https://github.com/lichnost/gyro-tron-dev) for the API, the packer and the
   PC emulator to try it on.
2. Open a pull request here adding one file to `registry/`:

   ```json
   {
     "id": "bunnyhop",
     "repo": "https://github.com/someone/bunnyhop",
     "commit": "0123abcd00000000000000000000000000000000",
     "summary": "Hop the bunny over fences",
     "dir": "game",
     "package": "dist/bunnyhop.tat"
   }
   ```

   | key | meaning |
   |---|---|
   | `id` | catalog-global, lowercase `[a-z0-9_]`, 1–15 chars, immutable once published |
   | `repo` | the author's repository (https) |
   | `commit` | the pinned full SHA — never a branch; an update is a commit bump |
   | `summary` | one line, what the store shows, ≤ 64 chars |
   | `dir` | optional: the game folder inside the repo (default: the root) |
   | `package` | optional: fetch this pre-built `.tat` from the same repo instead of packing the sources; must be repo-relative and the commit pins its bytes exactly |

   Unknown keys are refused; `dir` and `package` are mutually exclusive; paths may not
   climb out (`..`). The `id` must not collide with an existing official or community
   game — first come, first kept.
3. The `pr-check` workflow clones your repository at the pinned commit, packs (or
   fetches the pre-built package), validates the package (container, CRCs, api
   version, size ≤ 4 MB, icon ≤ 232 px, no Game Boy ROM descriptors, no smuggled
   signature — community stays unsigned) and posts the machine verdict as a PR
   comment. Then a maintainer reviews — every new game **and every update**; the
   review is the trust boundary.
4. Once merged, the game reaches players when the console repository advances its
   pinned copy of this registry (a merge request there, nightly or on demand).

Rules for the game folder itself: `LICENSES.md` at the repo root is required and must
declare the provenance of every asset you did not make; GB ROM-cartridge descriptors
(`game.json` with `"type": "gb"`) never enter the community channel.

## LICENSES.md (template for author repositories)

```markdown
# Asset licenses

Everything in this repository I made myself, except:

| file | author | license | where it came from |
|---|---|---|---|
| assets/tiles.png | J. Random | CC-BY 4.0 | https://example.org/tiles |

(Delete the table if there is nothing to declare; the declaration itself is required
so review can check it.)
```
