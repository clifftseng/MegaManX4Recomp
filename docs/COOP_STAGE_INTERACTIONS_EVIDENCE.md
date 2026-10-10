# X4 shared stage interaction evidence

Static evidence uses the owner's original US executable `mmx4/SLUS_005.61`
and its trusted generated C. No original executable bytes or game assets are
included. Native runtime qualification is separate from this trace.

## Military Train background updates — feedback item 28

Slash Beast's native introduction `8006DC24` sets the foreground camera's
authored room bounds. Those limits do not stop the train's separate scenery
animation. The stage 8 entries in the first background mode table
`800F32D4` select mode 5, whose callback `800285E0` does nothing. This stage
therefore relies on shared actors to update the background origins.

The real online Split qualification QA79 disproved the initial actor mapping:
saved view 0 advanced by 8/4 pixels per tick, while view 1 remained at
background X3408/1448. Both players were healthy and idle after P2's Slash
entry; both foreground cameras stayed near X9392/Y208. The earlier wrappers
for small-pool types 24/25/26 modeled other native scenery actors, and did
not cover this train loop. Passing those models did not qualify item 28.

The 20-slot small actor pool starts at `801410C0`, stride `60` hex. Its native
update `8002196C` dispatches through `800F2A68`. The original executable's
relevant entries are:

| Type | Native update | Background behavior |
| --- | --- | --- |
| 19 | `800D8648`, active callback `800D872C` | Initializes actor `+18` to `00080000`; adds that speed to background 1 X at `800D878C` and its arithmetic half to background 2 X at `800D87A8`. |
| 21 | `800D9218`, active callback `800D9330` | Initializes the same speed; adds it to background 1 X at `800D9388` and its arithmetic half to background 2 X at `800D93A4`. The outer update also calls `800D9B48` to advance boss-phase layer metadata. |

These original top-level entries dispatch the active callbacks normally.
Additional active-function hooks are unnecessary for the observed coverage
gap. The main gameplay routine `80021158` updates the foreground camera,
then the two background helpers. The shared small actor pool also runs from
`80021104` after that gameplay step. Consequently these actors write scenery
after the co-op view snapshots. Calling the mode 5 helper for the other pane
cannot recreate that external movement.

Type 21's native region selector `800D993C` uses player X thresholds
5808/7648/9984/10000 and resets subphases when the region changes. Its earlier
phase 1 calls the full helper `800D964C` from `800D9480` only at subphase
`+6 == 2`. If `+7 != 0` and `PLAYER+BC == 0`, that helper additionally adds
`current_foreground_X - previous_foreground_X` to background 1 and an
arithmetic half to background 2. Its stopped-train `+7 == 0` branch either
decelerates or sets `+7 = 1` and shared limits; that transition tick adds no
foreground displacement. Testing only the resulting actor flags would
incorrectly apply parallax on the transition.

The revised Split adapter wraps types 19/21 once and records the actual
foreground branch by intercepting full native helper `800D964C` during that
outer call. It applies captured background deltas to both saved cameras,
without adding them to canonical RAM again. Only the other view's recorded
foreground contribution is replaced with that pane's own preceding motion.
Arithmetic half preserves native `sra` behavior for negative odd fixed units.
Changed authored limits, dead zones, shake and layer metadata are copied;
foreground current/previous origins remain private. The adapter does not
repeat shared actor effects, particles, story commands or animation updates.
Unified mode, private player projection and local drawing bypass it.
Section reset starts both snapshots from the new native origins.

The replacement model cases use the actual 8/4-pixel loop and native type 21
phase dispatch. They cover distinct pane origins, single original execution,
later camera snapshots, owner switches, opposite fractional movement,
native freeze gates, the stopped-train transition, region reset, initialization
metadata, modular uint32 origin arithmetic, shared scenes, cleanup, wrong
actor kind, projection, section reset, local drawing, another stage and
Unified. This revision has source inspection and a clean whitespace diff;
root owns compilation, model execution and repeat online native qualification.

Native qualification uses a real healthy connected two-peer Split match:
load stage 8, section 1, checkpoint 3; place both players at X9360/Y395 and
use native Right input to cross X9472 with either leading seat while the
other remains at X9360. After the warning/introduction, keep both living
players stationary and compare completed-frame captures over several seconds.
Both saved scenery panes must advance while the authored foreground stays
inside its boss limits. Record all active small-pool actors' bytes
`0/1/4/5/6/7/14` and word `18` to identify the actual train callback and phase.
Inspect canonical backgrounds at layer bases `80141A04` and `80141A58`,
current X/Y at `+8/+C`, previous X/Y at `+14/+18`, and saved camera state
reported by `mmx4.coop.split-state`. Repeat for both leading seats and
campaign rosters; verify matching deterministic state on both peers.

## Lightning Web wall-jump anchors — feedback item 22

The requested enhancement concerns X's Lightning Web weapon, earned from Web
Spider. The boss's damaging/capturing web actors are different: secondary
types 22/23 (`8009D74C`/`8009DD40`) have a null solid pointer and use native
player damage/capture. They are outside the requested enhancement.

Lightning Web is player shot type 1, native update `80092F08` (dispatch table
`800F25D8`). Its opening animation installs solid hitbox `80108800` at `+68`
and enables `+75` in `800932A0`. The update calls original solid contact
`8002E184`, then clears top/bottom contact bits from `PLAYER+71` according to
shot `+72` (`80092FA8`–`80092FFC`). Horizontal contact remains available to
the player's ordinary wall-jump action. Expiry clears the solid eligibility.

Native solid comparison `8002C36C` writes shot `+72` and player `+71`/`+4A`
from actual contact geometry; it has no X/Zero character check. Co-op keeps
each player's 16-shot pool private. The existing canonical solid retry reaches
P2 for P1's web, but a P2-owned web updates while projected, so the canonical
retry is skipped and P1 never contacts that web. The native post-contact
top/bottom cleanup also applies only to the singleton owner body. A narrow
enhancement must offer each live partner only the web's solid contact, retain
its per-seat contact history, and perform the same per-seat top/bottom cleanup.
It must not update the projectile animation, lifetime, movement, or damage
twice, and must not change Web Spider's enemy web behavior.

The separate `mmx4_coop_web.c` adapter handles only an active player shot 1
with enabled `+75` and the verified `80108800` solid pointer. It calls the
original `8002E184` once for X and offers the other living normal Zero body
the player portion of that collision: `8002E294` position response,
`8002E380` carry when that partner had prior contact, and `8002C36C` side
comparison. These are full native function entries in the trusted ranges;
each takes the victim as explicit `a1`, and carry takes prior side bits as
`a2`. The adapter preserves the caller's `a2` and original return value.

The noncurrent Zero body is copied to one deterministic guest mirror while
its owner and both private shot pools stay in place. Only the helpers' proven
mutable bytes return to the saved body: integer XY at `+A/+E`, separation
XY at `+6C/+6E`, wall-contact kind at `+4A`, and sides at `+71`. Fractional
coordinates, action, HP, inventory and campaign state are not copied back.
Top/bottom cleanup matches X's original web update. History is indexed by
allocating seat and the 16 native shot slots, so identical guest shot addresses
in different private pools remain distinct. Native allocator `8002AC0C`
returning a reused slot invalidates only that seat's old contact; stage reset
clears both sets, and the deterministic digest includes every history byte.

Queries exclude incoming actions 0/1, hidden/dead/vehicle bodies, scene and
door holds, unknown `+67` movement states, other relevant native holds,
and local drawing. `+67` is also used for normal airborne movement, so the
adapter accepts precisely the inspected action/control pairs: Jump 6/1
(`80034538`), Fall 7/FF (`80034604`), WallJump 10/1 (`800349F4`) and WallSlide
11/FF (`80034AFC`), plus ordinary control 0. Treating every FF value as a
script lock would disable web contact while Zero approaches or slides along
the anchor. Native `80033B34` requires Jump edge and positive wall-contact
kind at `+4A`; `800349F4` then starts actual action 10 with upward velocity
and horizontal velocity away from the surface. Both player arrangements,
active combat pose 48, and Jump/Fall/WallSlide/WallJump contact progression
have focused model coverage. Additional cases cover real contact geometry and
carry ordering, opposite wall sides, shot-pool aliasing, allocation reuse,
vertical cleanup, field/return/register preservation, story/arrival gates,
unopened/expired/other projectiles, reset and allocation failure. No projectile
update runs twice and no enemy web path changes.

Root owns integration, compilation, model execution and native wall-jump
qualification. Native qualification should select Lightning Web weapon 1 for
the X seat using the existing development weapon command, fire with original
controller input in an ordinary stage, then have Zero approach each opened
web side and press native Jump while moving into it. Verify actual normal
wall-jump action/trajectory in both rosters and camera modes, expiry/reuse,
and scene/warp exclusions. Counters `mmx4.coop.zero-web-queries` and
`mmx4.coop.zero-web-side-contacts` distinguish an offered native query from
accepted horizontal contact. The `mmx4.coop.zero-web-body-mirror` counter
publishes the existing guest query mirror address for read-only contact
inspection; it allocates no extra diagnostic sidecar. The owner has not yet visually qualified this
enhancement.

`tools/mmx4_web_probe.py` requires an explicit existing private TCP port and
never launches a process. With `--scenario web --campaign 0` or `1`, it uses
the observed Jungle platform at X56/Y139, native weapon-selection fixture 7
and Square input to create a real web. It reads both live solid boxes to align
Zero beside the opened anchor, requires direction input to produce actual
side contact, and then checks native Jump for action 10 plus an upward/away
trajectory. The safe setup is bounded to the observed initial platform's
X32–256/Y75–139 air column; a web outside that column is reported as an
unqualified setup rather than teleporting into unobserved terrain. For X/P2,
the existing diagnostic tail publishes one completed opened-web record with
WEB1 header at `+6F0` and the native `9C`-byte shot at `+700`. X/P1 uses the
original shot pool. The helper releases inputs on failure and retains JSON
observations and a native action 10 screenshot for root's visual review.

The same helper's `--scenario train --owner both` drives the original Slash
entry for each leading seat and compares background X movement in both saved
Split cameras during an idle fight. An existing private invulnerability
fixture prevents combat knockback during the measurement and is cleared
afterward. Both scenery panes must advance while the foreground remains
within one pixel of its initial camera origin. Source syntax was inspected
without connecting to or controlling a runtime; root owns these qualifications.

Actual Split views require an online match; requesting Split in an offline
configuration keeps Unified behavior. `tools/mmx4_netplay_runtime.py` supplies
`NetplayRuntime(existing_development_match)` for the unchanged native probe.
Read/capture requests use peer 0, setup fixtures use the existing admitted
`NetplayDevelopment` packets, and seat input goes to that seat's peer on the
host input layer. A both-player teleport uses one fixture 4/seat 2 command.
Direct RAM writes and other raw mutation requests are rejected. Its eight
offline fake-peer tests verify routing and completed-body reads, with sockets
prohibited; live Split/native CRC and visual qualification remain with root.
