# ItemRack

Gear, set, and trinket management for World of Warcraft **1.12.1** (TurtleWoW-style servers).

A fork of [McPewPew/ItemRack](https://github.com/McPewPew/ItemRack) (itself based on Gello's original ItemRack), with TurtleWoW support and an off-hand weapon-swap fix.

## Features

- Per-slot quick-swap menus, item sets, keybinds, and event-driven auto-swapping (see `readme.txt` for full usage)
- TurtleWoW mount detection via spell IDs
- Goblin Brainwashing Device support (auto set-switch per specialization)
- Chinese (zhCN) localization
- Optional FuBar plugin (`ItemRackFu/`)

## Fork changes

- Fixed main-hand ↔ off-hand weapon swaps (e.g. dagger + offhand → axe + dagger) that previously left an item locked and required a re-log.

## Install

Copy the `ItemRack` folder (the one containing `ItemRack.toc`) into `Interface/AddOns`. For FuBar support, also copy `ItemRackFu` into `Interface/AddOns`.

## Mount not detected?

Run this while mounted and paste the output into an issue:

    /run for i=1,32 do local t,_,g=UnitBuff("player",i);if not t then break end print(t,g) end
