# Runbook

What to do when it is wrong. Read this before you go live.

## A detector is naming innocent players

```lua
-- XIS_Report
CFG.EnforcementEnabled = false
```

That clamps **every** detector to `REPORT`. Nothing kicks, nothing bans, everything still logs
with full context. One line, instead of hunting through 74 registry entries while your server
empties.

Then read the logs it keeps producing, find the offender, fix it, and flip it back.

For a single misbehaving detector, set that entry's `action` to `"LOG"` instead — same effect,
narrower blast radius, and the rest of the anticheat keeps working.

## Someone opens a ticket with a code

1. Search your webhook channel for the `Ref` value.
2. The embed has the detector, the evidence, their device, their ping, and the run-up of earlier
   unreported detections.
3. If it was wrong, set that detector to `LOG` and write down why in the registry entry's
   comment. Every threshold in this system has a story attached; add yours.

?> Resist the urge to reply with the detector name. The player asking is sometimes the one
testing bypasses, and "you were caught by the hover check" is a complete answer to their real
question.

## Checking it is alive

- Read the `_status` attribute on the `XIS_Server_v3` script in the Properties pane.
- Or call `_G.XIS_Diag()` from the server console.
- At boot the server prints its detector count, heartbeat slots, ban mode and everything the
  config has switched off.

If the detector count is not 74, the registry did not load cleanly and you are running with
gaps.

## A system of yours keeps tripping movement checks

Do not raise the threshold — that weakens the check for everyone. Whitelist the system instead:

```lua
_G.XIS_Whitelist.add(player, "fly", 3)
```

Raising a threshold is the right move only when you have measured that legitimate play genuinely
reaches it — a vehicle that genuinely outruns the cap, for instance.

## Players report being kicked during lag spikes

Movement checks read positions the server already has, so a client that stalls and then
catches up looks briefly like a teleport. Two things to check:

1. `spawnGrace` and `landingGrace` — if a respawn or a landing is involved, this is usually
   the fix.
2. `minKickSev` — raising it means low-severity movement hits stop accumulating toward a kick.
3. `maxTeleport` — the studs-in-one-sample threshold. A client that stalls and catches up
   moves a long way in one sample, so this is the field a lag spike actually trips.

If it is widespread rather than per-player, it is your server's frame time, not the anticheat.
`_G.XIS_MovementStats()` reports hitch counts, which will tell you which it is.

## Before you enable enforcement, checklist

- [ ] One full populated session in report-only mode, logs read
- [ ] Every system of yours that moves players unusually is whitelisted
- [ ] Webhooks pointed at a channel you will actually read
- [ ] `EnableBans = false` for the first stretch
- [ ] `setKickOnStrikes(false)` if you use the Weapons Kit
- [ ] You know where `EnforcementEnabled` is without looking it up
