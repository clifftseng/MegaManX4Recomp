# Local P3/P4 ("bench seats") — experimental

> 中文摘要：在原本的 X + Zero 雙人合作上，加入第 3、第 4 位本機玩家。
> P3/P4 是 P2 角色的分身（A 級：換色；Zero 可用原版黑色 Zero 配色）。
> 只支援本機 Unified 畫面；連線、分割畫面、即時存檔、倒帶在多人時停用。
> **尚未實機測試**（還沒有光碟可編譯），以下是設計與測試清單。

Status: written and syntax-checked against the framework headers. **Not yet
built or played** — a full build needs the user's SLUS-00561 disc to generate
the game's C.

## How to use

1. Launcher → **Mods → X + Zero Co-op** → **Extra local players**: `Player 3`
   or `Players 3 and 4`. Pick **Player 3 colors** / **Player 4 colors**.
2. Launcher → **Controls**: assign a device to Player 3 / Player 4, and turn
   **Multitap** on (the runtime only samples pads 3+ offline when it is on;
   the guest still sees two ordinary ports because `MAX_PLAYERS` is below 5).
3. Unified local co-op only. Fresh boot required.

## Design

The existing co-op already runs P2 by swapping P2's private copy of the
player records into the singleton native addresses ("projection") and calling
the original player routines a second time. P3/P4 reuse that unchanged:

- Each extra seat is a `BenchSeat` in `mmx4_coop_plugin.c` holding its own
  `PlayerContext`, inventory (HP/ammo/weapon/E-tank contents), previous input,
  colour choice and respawn timer.
- `bench_swap(e)` exchanges bench seat *e* with P2 in the private slot
  (`second`), outside any projection. While swapped in, every P2 path —
  the native player pass, `render_slot`, `hud_slot` — serves that seat.
- Each bench seat has a private copy of the counterpart palette
  (`bench_palettes`), installed as the scratch palette pointer during its
  projection so native weapon-colour writes stay per seat. An optional
  presentation-only tint is applied when its sprite banks are built.
- Camera: ordinary play centres on the mean of all living heroes. Bench
  seats are leashed to within 112 px of the P1/P2 centre (only the outward
  step of the current frame is undone, as for P1/P2).
- Death: a bench seat that dies respawns beside P1 after 180 ticks while P1
  is alive. Bench seats never count toward team survival.
- With `extra_players = 0` nothing rotates and P2's behaviour is unchanged
  (packet limit, HUD and render arenas, allocations are all identical).

Build changes: `MAX_PLAYERS 4` for the runtime targets and `players = 4` in
`game.toml` (both copies) so the host samples four pads.

## Known limitations (phase 1)

- P3/P4 are always copies of **P2's hero**. Choosing a different hero per
  seat (e.g. X, Zero, X) is not supported yet.
- Bench seats share P2's identity inside the combat/lifecycle modules: boss
  hit cooldowns, pickup ownership and door rules treat them as "seat 2".
- No pause menu of their own; weapon switching uses the in-game L1/R1.
- P2's private moving-floor copies are P2-only; bench seats use the shared
  floors.
- Not included in snapshots → save states and rewind are blocked while extra
  players are active. Playtest-diagnostics recordings only contain P1/P2.
- Netplay ignores extra players. Because `players = 4`, a netplay room can
  now admit 4 peers; peers 3/4 would have no hero.

## Test checklist (once a disc is available)

- [ ] `extra_players = 0`: two-player co-op unchanged (boss, door, death,
      pause menus, netplay smoke test).
- [ ] P3 appears beside P1 on stage start and after each room transition.
- [ ] P3 controls only its own hero; P1/P2 inputs unaffected.
- [ ] P3/P4 tints visible; Black on Zero copies.
- [ ] HUD: 2x2 grid (Stacked) / row (Side by side).
- [ ] Camera centres on everyone; leash stops P3 leaving the screen.
- [ ] P3 dies → respawns after ~3 s; P1 death with only P3 alive still
      triggers the team retry.
- [ ] Bosses take damage from P3; no crash when P3 rides Ride Armor / Chaser.
- [ ] `mmx4.coop.*` counters show no `invalid-sprite` or disabled codes.

## Next (phase 2, "B")

- Per-seat hero choice (X or Zero for each seat).
- Per-seat armour/variant (Ultimate Armor X for one seat only) — needs a
  per-seat sprite source instead of the global `x_variant`.
- Per-seat combat identity (cooldowns, pickups) and own pause menus.
