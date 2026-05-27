# AGENTS.md — herrnel/PX4-gazebo-models (branch: `ai-grand-prix`)

You (the AI agent) are operating inside a **fork** of PX4-gazebo-models.
This is a nested submodule of a fork of PX4-Autopilot, which is itself a
submodule of the parent project `AI-Grand-Prix` (`herrnel/Albatross`).

The directory you're in is checked out at:
`<repo-root>/external/PX4-Autopilot/Tools/simulation/gz/`

## Where you are

- Remotes:
  - `origin`  → `https://github.com/PX4/PX4-gazebo-models.git` (read-only — upstream)
  - `myfork`  → `https://github.com/herrnel/PX4-gazebo-models.git` (push target)
- Working branch: **`ai-grand-prix`** — every change goes here.
  Never commit to `main`.
- This repo is glob'd by PX4's CMake:
  `file(GLOB gz_worlds .../worlds/*.sdf)` — so the *filename* of any
  new world directly determines the `make px4_sitl gz_<model>_<world>`
  target name. Choose names carefully.

## Core rules

1. **Never `git reset --hard` or `git clean -fd`** without `git status`
   + `git stash list` first and showing the user what would be lost.
2. **Never edit on `main`.** Always work on `ai-grand-prix`. Verify with
   `git branch --show-current`.
3. **Never push to `origin`.** Push only to `myfork`.
4. **Don't break the CMake glob.** Adding a `.sdf` to `worlds/` will
   silently create a new make target. That's usually desired, but tell
   the user what target name will be generated.
5. **Asset URIs.** Worlds in this repo often reference models via
   `<include><uri>model://Foo</uri></include>`. Those models must be
   resolvable on `GZ_SIM_RESOURCE_PATH` at runtime. The parent project
   sets that path in `scripts/env.sh` (parent repo) to include the
   `mav_simulator` model trees. If you add a world that pulls in new
   models, update `scripts/env.sh` in the parent repo too.

## Normal change workflow

```bash
# 0. State check
git status
git branch --show-current   # must be 'ai-grand-prix'

# 1. Make the edits (edit/write tools, not sed/echo).

# 2. Commit
git add -A
git -c user.name="Nelson Herrera" -c user.email="ndanielherrera@icloud.com" \
    commit -m "<imperative subject>

<why; if adding a world, note the resulting make target name>"

# 3. Push
git push        # tracks myfork/ai-grand-prix
```

Then from the **parent repo**:

```bash
# Bump the inner submodule pointer inside PX4-Autopilot first:
scripts/bump-submodule.sh external/PX4-Autopilot/Tools/simulation/gz "<msg>"
# That commits the bump inside the PX4 fork on its ai-grand-prix branch
# and pushes it. Then bump PX4's pointer in the parent:
scripts/bump-submodule.sh external/PX4-Autopilot "Bump gazebo-models pointer: <msg>"
```

Two bumps because there are two parent-of-this layers.

## Syncing with upstream PX4-gazebo-models

```bash
# From the parent repo:
scripts/sync-upstream.sh external/PX4-Autopilot/Tools/simulation/gz main
```

Then bump as above.

## What's in the fork

- **Illini warehouse world** (`fe635e5`): `worlds/x3_illini_warehouse.sdf`
  — copied from `mav_simulator/mav_gazebo/worlds/x3_illini_warehouse.world`.
  References `model://X3` plus gates / warehouse models from the
  mav_simulator model trees. Generates make targets
  `gz_<model>_x3_illini_warehouse` for every airframe in PX4.

## When in doubt

Ask the user. Read git state before changing it.
