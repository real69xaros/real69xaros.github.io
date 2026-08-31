# Kicks and bans

What the player is told, what you are told, and why those are different.

## The player never learns which detector fired

A detector label is precisely what a cheater needs in order to work out what to change — and a
kick screen is a free, instant feedback channel for testing bypasses. So the player gets a
category-level message and an opaque reference code; the full detail goes to your webhook.

| Category | The player sees |
|---|---|
| Integrity, Injection, Memory | Tampering detected. |
| Visual, Aim, Combat, State | Exploit detected. |
| Movement | Movement exploit detected. |
| Account | Your account is not eligible to play. Try verifying your email. |

The first three share one string deliberately. A cheater must not be able to tell whether it
was their hooks, their executor or their explorer that gave them away — if the message
distinguishes them, they can bisect their own setup in three rejoins.

?> These strings are yours to change, and there is no technical reason not to. Just keep them
category-level: the moment a message names a mechanism, it becomes a bypass tool.

## Reference codes

Every kick and ban carries an 8-character code, shown to the player and attached to the report
as `Ref`.

```
Movement exploit detected.
Ref: C226340C
```

The code is opaque on its own — it maps to a detection only through your log. When someone
opens a ticket saying "I got kicked, code C226340C", that code is how you find out what
actually happened without ever having told them.

## What a report contains

Each embed carries the detector, the evidence values that triggered it, the player's device and
ping, and the **run-up**: the last N detections for that player that were not themselves
reported. The run-up is usually the most useful part — a single `KICK` with four `LOG` entries
behind it tells a very different story from one with none.

## Bans

Bans go through `Players:BanAsync`, so Roblox enforces them at **connection** time. A banned
account never reaches your server, which means a ban cannot be defeated by anything running on
the client.

| Setting | Ships as | Why |
|---|---|---|
| `BanThreshold` | `3` | Ban-eligible strikes before a ban lands. |
| `BanDuration` | `-1` | Permanent. Set to a number of seconds for temporary. |
| `BanUniverse` | `true` | Every place in the universe, not just the one that caught them. |
| `BanExcludeAlts` | `true` | Does **not** propagate to suspected alts. |

Strikes persist so a rejoin cannot reset the run-up. Roblox owns the ban state itself, so it
survives your DataStore having a bad day.

### Which categories can ban

Only **Integrity** and one Injection detector. Movement, Combat and Telemetry are structurally
ban-exempt: the registry zeroes their strike count and downgrades a `BAN` action to `KICK`
inside the resolver, so the guarantee holds even if a future edit sets `action = "BAN"` on one
of them.

!> This is not conservatism for its own sake. Movement and combat detections have a documented
false-positive history — snap-backs, seat-exit momentum, per-shot hit declines on a laggy
client. A wrong kick costs a player five minutes. A wrong permanent universe-wide ban costs you
the player and, if it happens at scale, your game's reputation.

## Turning bans off entirely

```lua
CFG.EnableBans = false
```

Every `BAN` becomes a `KICK`. Worth doing for your first month.
