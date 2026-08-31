# What it detects

74 detectors across ten categories. This page describes them at category level. Individual
detector ids, thresholds and confirm windows are deliberately **not** published — see the note
at the bottom.

| Category | Detectors | Gate | Highest action |
|---|---:|---|---|
| Integrity | 19 | always on | <span class="sev crit">BAN</span> |
| Movement | 20 | `AntiTeleport` `AntiFly` `AntiGravity` | <span class="sev high">KICK</span> |
| Visual | 9 | `AntiCoreGuis` `AntiESP` `AntiHitboxExpander` | <span class="sev high">KICK</span> |
| Injection | 7 | `AntiInject` | <span class="sev crit">BAN</span> |
| Combat | 6 | `AntiWallHack` `AntiKillAll` | <span class="sev med">REPORT</span> |
| Aim | 4 | `weaponsystemDetections` | <span class="sev med">REPORT</span> |
| State | 4 | `otherdetections` | <span class="sev high">KICK</span> |
| Account | 2 | `AntiAltAccount` | <span class="sev low">REPORT</span> |
| Memory | 2 | `AntiCoreGuis` | <span class="sev high">KICK</span> |
| Telemetry | 1 | always on | context only |

## Integrity

Tests that the Luau environment itself has not been tampered with: whether core globals are
still C functions rather than Lua closures, whether the DataModel metatable is still locked,
whether `error()` still propagates, whether remote-firing methods have been hooked, whether
core services have been renamed.

Also carries the **dead-man switch**. Every polling detector on the client bumps a counter each
pass, and those counters ride inside a signed handshake payload. A cheat that hooks one
detector to a no-op used to be invisible, because the script as a whole kept answering; now
that one counter freezes while the others climb, and the server can name the exact module that
stopped.

This is the only category that reaches `BAN`, because it tests Lua-level invariants rather than
the shape of any Roblox API — a C function is not a Lua closure, and that will not change out
from under you in a platform update.

## Movement

Server-authoritative. Speed, WalkSpeed, teleport delta, acceleration, air time, hover, fly,
CFrame-fly, noclip, local collision removal, gravity tampering, infinite jump, bunny hop,
spinbot, anti-aim, fake-seat, network ownership mismatch and impossible velocity.

Everything here is measured from positions the server already has, so nothing in this category
depends on the client being honest.

?> **Movement can never ban.** Enforced structurally, not by convention: the registry zeroes
ban strikes and downgrades a `BAN` action to `KICK` for this category, so the guarantee holds
even if someone edits an entry later. Movement checks have a documented false-positive history
in every anticheat ever written — snap-backs, seat-exit momentum, laggy clients — and a wrong
ban is not recoverable the way a wrong kick is.

## Visual

Cheat menus and overlays in the player's own GUI, known exploit GUI signatures, console output
signatures left behind by popular menus, ESP overlays adorned to other players, and hitbox
expansion on the head or body.

Hitbox reports are re-measured on the server before anyone is touched, which flips the
accusation onto whoever reported it — if the server sees a normal limb, the oversized one only
ever existed on the reporter's machine.

## Injection

Executor fingerprints: services and globals that only exist when an injector is present,
capability probes, `SaveInstance` attempts, and anti-AFK patterns that sever the `Idled` signal
to defeat idle-kick.

Most of this category is `KICK` rather than `BAN` on purpose. A detector that infers an
executor from an API's *shape* is one platform change away from mass-banning an innocent
playerbase; only the `SaveInstance` attempt is treated as ban-worthy.

## Combat

Hit validation, running on every damage claim: line of sight against server geometry, whether
the target was ever at the claimed position, whether the shot was inside weapon range, and
whether the client's claimed muzzle matches the server's.

Combat detections **decline the hit** — the cheater gets no damage — and report. They do not
kick on their own, and like movement they can never ban.

## Aim

Aimbot and silent-aim detection for games using the Roblox Weapons Kit: raycast callbacks
replaced or defined outside the weapon tree, and a decoy target that a legitimate raycast will
never pick.

Inert without the Weapons Kit. Set the `weaponsystemDetections` gate to `false` if your game
does not use it.

## State

Character and service state that a cheat leaves behind: unsanctioned body movers welded to the
character, an anchored root on a living unseated player, tampered Humanoid movement properties,
and GUI smuggled into a renamed service.

Body-mover reports are checked against the server first, so anything your own game creates is
dropped rather than acted on.

## Account

Alt-account signals and email verification status. Low severity and report-only — this is
context for a human reading a report, not grounds for action on its own.

## Memory

Instance-dumper and remote-spy detection. A weak-table canary that a garbage collector will
release, but an explorer holding a reference will not.

## Telemetry

Device, platform and ping, attached to reports as context. Never acted on.

---

!> **Why detector ids are not listed here.** A detector id and its threshold are precisely what
someone needs in order to work out what to change. The same reasoning is why a kick message
tells the player only their category and never the detector — a kick screen is otherwise a free,
instant feedback channel for testing bypasses.
>
> Kick messages follow the same rule: the player is told their category and never the
> detector. A kick screen is otherwise a free, instant feedback channel for testing
> bypasses.
