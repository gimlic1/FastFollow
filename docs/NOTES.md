# FFXI Addon — FastFollow

> [!NOTE]
> **Quick Reference**
> - **Path:** `C:\Windower\addons\FastFollow\`
> - **Author / Version:** DiscipleOfEris / `1.2.1`
> - **Commands:** `//fastfollow`, `//ffo`
> - **Load Status:** `scripts/init.txt`
> - **Origin:** Third-Party (`https://github.com/DiscipleOfEris/FastFollow.git`, branch `master`)

High-precision multi-boxing follow addon using Windower IPC position streaming, automatic zone-line traversal, and cast/item movement pausing.

## What the Code Does
`FastFollow.lua` replaces FFXI's native `/follow` with a Windower IPC vector-steering loop:
- **Leader Side (`follow_me > 0`):** Every `prerender` frame, broadcasts `update <name> <zone> <x> <y>` via `windower.send_ipc_message()`. On outgoing `0x05E` (`REQUEST_ZONE`), broadcasts `zone <name> <zone_line> <type>`.
- **Follower Side (`following`):** On `ipc message` `update`, steers toward the leader via `windower.ffxi.run((target.x - self.x)/len, (target.y - self.y)/len)` whenever in the same zone and between `min_dist` (`settings.min^2`) and `max_dist` (`50.0^2`). On `zone`, spawns `zone()` which waits a random `0.2–2.7s`, waits until within `5.0` yalms of the leader's last coordinates, and injects an outgoing `0x05E` packet (throttling duplicate zone requests within `zone_suppress = 3s`).
- **Auto-Pause (`pauseon`):** When `spell`, `item`, or `dismount` is enabled, `outgoing chunk` intercepts `0x01A` (`MAGIC_CAST=0x03`, `DISMOUNT=0x12`) and `0x037` (`USE_ITEM`), stops movement (`windower.ffxi.run(false)`), delays packet re-injection by `pause_delay` (`0.1s`), and holds `casting` until the `action` event reports finish/interrupt (using `spell_cast_times.lua` for spell timings) or a fallback coroutine expires.
- **Distance Overlay (`info`):** Optionally displays distance to each IPC-tracked character in a `texts` box.

## Commands & Usage
- `//ffo followme` / `//ffo me` — Tell all other instances to follow this character.
- `//ffo follow <name>` (or `//ffo <name>`) — Follow a specific character.
- `//ffo stop` — Stop following on this character.
- `//ffo stopall` — Stop all characters from following/leading via IPC.
- `//ffo pauseon <spell|item|dismount|any>` — Toggle auto-pausing movement on casts, items, or dismounts.
- `//ffo pausedelay <sec>` — Set pause delay before re-injecting intercepted action packets.
- `//ffo min <dist>` — Clamp minimum follow distance (`0.2` to `10` yalms).
- `//ffo info [on|off]` — Toggle distance HUD overlay and IPC tracking.

## Events, Packets & Technical Hooks
- **Windower Events:** `unload`, `addon command`, `ipc message`, `prerender`, `outgoing chunk`, `action`
- **Packets:**
  - Outgoing `0x05E` (`REQUEST_ZONE` — monitored/suppressed on followers and injected to cross zone lines)
  - Outgoing `0x01A` (`ACTION` — intercepted/re-injected for `MAGIC_CAST` `0x03` and `DISMOUNT` `0x12`)
  - Outgoing `0x037` (`USE_ITEM` — intercepted/re-injected when pausing on item use)
- **Dependencies / Libraries:** `strings`, `tables`, `sets`, `coroutine`, `packets`, `resources`, `spell_cast_times`, `config`, `texts`, `logger`

## Local Settings & Customizations
`data/settings.xml`: `min=0.5`, `show=false`, `display` at `(0, 0)` with `Consolas` size `10`.

## Related Notes
- `FFXI Windower Overview`
- `FFXI Windower Addons Catalog`
