# Configuration

Everything is set in two places: the `config` module for switches and thresholds, and
`XIS_Registry` for what each detection costs.

## Gates — switching whole categories off

Each detector category has a **gate**: a key in `config` that must be truthy for it to act at
all. Set a gate to `false` and that category stops entirely.

```lua
Fallback = {
    AntiTeleport            = true,
    AntiFly                 = true,
    AntiGravity             = true,
    AntiWallHack            = true,
    AntiKillAll             = true,
    AntiESP                 = false,   -- off by default, see below
    AntiHitboxExpander      = true,
    AntiCoreGuis            = true,
    AntiInject              = true,
    AntiAltAccount          = true,
    weaponsystemDetections  = true,    -- needs the Roblox Weapons Kit
    otherdetections         = true,
}
```

Whatever is off gets printed at every boot, so it can never be off by accident.

?> `AntiESP` ships **off**. ESP detection reads other players' GUI adornees, which produces
false positives in games that legitimately draw overlays on teammates — nameplates, objective
markers, revive indicators. Turn it on only if you have none of those.

`weaponsystemDetections` needs the Roblox Weapons Kit. If your game doesn't use it, set the
gate to `false`; the aim detectors are inert without it either way.

## Enforcement — the master switch

```lua
-- XIS_Report
CFG.EnforcementEnabled = true
```

`false` clamps **every** detector to `REPORT`, no matter what the registry says. Nothing kicks,
nothing bans, everything still logs. This is both your install mode and your bad-day switch —
see the [runbook](runbook.md).

## Ban behaviour

```lua
CFG.EnableBans      = true
CFG.BanThreshold    = 3      -- ban-eligible strikes before a ban lands
CFG.BanDuration     = -1     -- -1 = permanent, otherwise seconds
CFG.BanUniverse     = true   -- every place in the universe, not just this one
CFG.BanExcludeAlts  = true   -- do NOT propagate to suspected alt accounts
```

`BanExcludeAlts = true` is deliberate. Alt linkage is Roblox's inference, not yours, and a
wrong ban that spreads across someone's other accounts multiplies the damage. Turn it off only
once you trust your own false-positive rate.

## Rate limits

```lua
CFG.ActionCooldown  = 15   -- seconds between actions on the same player
CFG.WebhookCooldown = 10   -- seconds between embeds for the same detector
CFG.HistoryMax      = 15   -- detections kept per player for the run-up
```

`HistoryMax` is what populates the "what led up to this" block on a kick embed. Raising it
costs a little memory per player and makes reports considerably easier to read.

## Movement thresholds

The movement checks have their own table. The shipped values are tuned for a game with
vehicles, launch pads and 60 players; tighten them if your game is more constrained.

```lua
Movement = {
    maxSpeed          = 120,   -- studs/sec before the speed check fires
    maxWalkSpeed      = 40,    -- Humanoid.WalkSpeed ceiling
    maxAirTime        = 6,     -- seconds airborne
    teleportDelta     = 180,   -- studs in one sample
    minCheckDt        = 0.05,  -- sample interval
    minKickSev        = 6,     -- severity needed to accumulate toward a kick
    spawnGraceSeconds = 5,     -- checks suspended after spawn/respawn
}
```

!> `spawnGraceSeconds` protects against your own respawn logic. Lower it and you will start
seeing false teleport reports every time someone dies.

## Tuning what a detection costs

Open `XIS_Registry`, find the id, change one field.

```lua
["<category>.<detector>"] = {
    label    = "Human-readable description",
    severity = "HIGH",
    action   = "KICK",                     -- IGNORE | LOG | REPORT | KICK | BAN
    confirm  = { hits = 3, window = 30 },  -- N hits inside N seconds before acting
}
```

| Action | What happens |
|---|---|
| `IGNORE` | Nothing. The detector is off. |
| `LOG` | In-memory history only. Surfaces in the run-up attached to a later kick. **Where every unproven detector belongs.** |
| `REPORT` | History plus a Discord embed. No player-facing consequence. |
| `KICK` | As `REPORT`, plus the player is removed. |
| `BAN` | As `KICK`, plus a strike toward a `Players:BanAsync` ban. |

`confirm` requires N hits inside a window before acting. It exists on the two detectors that
have a real replication race behind them, and removing it reintroduces a false kick on both.

!> **Promotion discipline.** A detector you have retuned starts at `LOG` and is promoted only
after a clean populated session shows it names nobody innocent. This is not a formality — it is
where false mass-kicks come from.
