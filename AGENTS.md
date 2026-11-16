# AGENTS.md

ItemRack — a World of Warcraft **vanilla 1.12.1** (2004 client) gear/trinket-swap addon. Fork of `McPewPew/ItemRack`, targeting TurtleWoW-style servers. Use only the 1.12 client API — no modern WoW APIs.

## Layout

- `ItemRack.toc` — addon manifest (`## Interface: 11200`). Load order matters: `localization.lua` (defines `ItemRackText`) → `ItemRack.lua` → `ItemRack.xml` → `ItemRackSets.xml` → `mountGUID.lua` (defines `ItemRack.mountGUIDs`, used at runtime).
- `ItemRack.lua` — essentially all core logic (~5k lines): the `Rack` object and the swap queue.
- `README.md` — brief fork-facing intro/features; `readme.txt` is Gello's original full manual.
- `ItemRackFu/` — optional FuBar plugin; not loaded unless moved into the Addons root.
- `.github/workflows/sync-fork.yml` — auto-merges upstream `McPewPew/ItemRack` (`master`) into this fork daily; local edits to shared files can conflict on the next sync.

## Editing gotchas

- **Tabs for indentation; CRLF line endings.** Match both or every line diffs.
- **Lua 5.1, not 5.4+.** Nested loops are broken with `i,j = 99,99` (assigning to a `for` loop var), illegal since 5.4. A system `luac`/`lua` will false-positive with "attempt to assign to const variable"; there is no reliable offline syntax check.
- **Item IDs are strings** (`"12345:0:0"` = id:enchant:suffix), not numbers. `Rack.GetItemInfo` returns `(texture, itemID, itemName, itemSlot)`.
- **No build/test/lint/CI** — verify in-game only: `/reload`, then perform the swap.

## Swap engine

`Rack.EquipSet(setname)` fills a `Rack.SwapList` (slots 0–19), then enqueues `Rack.SwapQueue` phases in order: `INVTOBAG` (empty a slot) → `INVTOINV` (move between worn slots) → `BAGTOINV` (bag → slot) → `NEWSET`. Moves are asynchronous (1.12 item locks); the queue advances on `ITEM_LOCK_CHANGED` via `Rack.OnItemLockChanged` → `Rack.AnyLocked()`.

## Weapon-swap quirk (main-hand 16 / off-hand 17)

`PickupInventoryItem(16)` + `PickupInventoryItem(17)` is a **swap**, not a move. Moving a weapon out of main/off while a compatible weapon sits in a bag makes the client ghost-lock the just-bagged item (`AnyLocked()` stays true, item grayed-out) — only a re-log clears it.

`Rack.EquipSet` special-cases "main-hand weapon → off-hand, actual off-hand equipped, new main-hand from a bag": it routes the old weapon through `BAGTOINV` (chained via the client's "place previous item where the cursor originated" behavior) and flags it `deferredBAGTOINV` to run in a second BAGTOINV pass after the first settles. Preserve this when editing swap logic.
