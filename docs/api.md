# Server-script API

Callable from any server script. Every grant is written to the console, because a suspended
check needs to be findable after the fact.

## Whitelist — per player, per check

Use this when one of your own systems legitimately breaks a movement rule.

```lua
-- Timed. PREFER THIS: it closes itself.
_G.XIS_Whitelist.add(player, "fly", 8)        -- 8 seconds
_G.XIS_Whitelist.add(player, "*", 5)          -- every movement check, 5 seconds

-- Indefinite, until you turn it off again.
_G.XIS_Whitelist.toggle(player, "fly", true)
_G.XIS_Whitelist.toggle(player, "fly", false)

_G.XIS_Whitelist.remove(player, "fly")
_G.XIS_Whitelist.clear(player)
_G.XIS_Whitelist.has(player, "fly")           --> boolean
_G.XIS_Whitelist.list(player)                 --> { fly = 6.2, noclip = "on" }
```

Duration clamps to 0.1–3600 seconds; the default is 30.

!> Prefer `add()` over `toggle()`. A toggle that a system forgets to clear is a permanent hole
for that player, and nothing will remind you it is open. Reach for `toggle()` only when you
genuinely cannot predict a duration, and clear it in the same script that set it.

### Check names

Names are forgiving — you do not have to remember the exact internal one.

| Write any of these | Suspends |
|---|---|
| `fly` `flight` `hover` `cframe_fly` | Fly, CFrame-fly and hover |
| `speed` | Speed cap |
| `walkspeed` `ws` | WalkSpeed cap |
| `noclip` `nocollide` | Noclip and local collision removal |
| `teleport` `tp` | Teleport-delta |
| `physics` `fling` `velocity` | Impossible velocity and the fling check |
| `spin` | Spinbot |
| `antiaim` | Anti-aim |
| `bhop` | Bunny hop |
| `gravity` | Gravity tamper |
| `jump` | Infinite jump |
| `combat` `wallhack` `los` `range` `position` | **All** hit validation for that player |
| `*` `all` `movement` | Everything whitelistable |

!> **Movement and combat differ in granularity.** Movement grants are *per check*. A combat
grant is *all-or-nothing*, because it works by skipping the whole hit validation for that
player rather than one check inside it. To disable a single hit check, do it globally with
`setCheck()` below.

### Worked examples

```lua
-- Launch pad: suspend fly + physics for the duration of the arc.
pad.Touched:Connect(function(hit)
    local plr = Players:GetPlayerFromCharacter(hit.Parent)
    if not plr then return end
    _G.XIS_Whitelist.add(plr, "fly", 3)
    _G.XIS_Whitelist.add(plr, "physics", 3)
    -- ...apply your impulse
end)

-- Cutscene: everything off, then straight back on.
local function playCutscene(plr, seconds)
    _G.XIS_Whitelist.add(plr, "*", seconds + 1)
    -- ...run the cutscene
end
```

## Hit validation — global, live

```lua
_G.XIS_HitValidation.setEnabled(false)         -- all hit validation off
_G.XIS_HitValidation.setCheck("los", false)    -- "los" | "pos" | "range"
_G.XIS_HitValidation.setKickOnStrikes(false)   -- validate and log, never kick
_G.XIS_HitValidation.setDebug(true)            -- log every declined hit
_G.XIS_HitValidation.status()                  -- reads the LIVE values back
```

`setCheck` takes:

| Check | Rejects |
|---|---|
| `los` | Shots that pass through server geometry |
| `pos` | Hits claimed where the server never saw the target |
| `range` | Hits beyond the weapon's range |

?> `setKickOnStrikes(false)` is the useful middle setting for a new install: bad hits are still
declined so cheaters get no damage, but nobody is removed while you are still learning your
own false-positive rate.

## Whole-player exemptions

```lua
_G.XIS_SetMovementExempt(player, true)   -- ALL movement checks, indefinite
_G.XIS_SetHitExempt(player, true)        -- ALL hit validation, indefinite
```

For systems that own a player's movement for an unbounded stretch — a carry mechanic, a
spectator camera, a vehicle seat you drive from a server script. Clear it in the same system
that set it.

## Sanctioned teleports

For a server-initiated move, prefer this over an exemption: it applies a **grace window**
rather than switching a check off, so the checks stay live either side of the jump.

```lua
ReplicatedStorage.BindableServerTeleport:Fire(player, targetCFrame, "ranked")
```

The third argument is a source name that must appear in `config.TeleportWhitelist`. An
unrecognised source is ignored and logged, so a cheat that finds the bindable cannot use it.

## Inspection

```lua
for _, line in ipairs(_G.XIS_Diag()) do print(line) end
print(_G.XIS_MovementStats())
```

`XIS_Diag()` returns a live health summary. `XIS_MovementStats()` returns counters — hitches,
snap-backs, spin and anti-aim hits, fling streaks — which is what you want when tuning
thresholds against a real session.

!> Both are server-VM only. `_G` is per-VM in Roblox, so a plugin, a client script or a
separate actor cannot see these. If a call returns `nil`, you are in the wrong VM.
