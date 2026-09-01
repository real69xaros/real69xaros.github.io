# Install

## 1. Place the scripts

```
ServerScriptService/
  XIS_Server_v3          (Script)
  XIS AntiCheat/
    XIS_Registry         (ModuleScript)
    XIS_Report           (ModuleScript)
    config               (ModuleScript)

ReplicatedFirst/
  XIS_Client_v3          (LocalScript)
```

`XIS_Client_v3` must be in `ReplicatedFirst` and nowhere else. `ReplicatedFirst` downloads
before the world renders, which is what lets the client detectors be running before an
autoexec script gets a chance to load.

## 2. Enable what you want to use

In Game Settings → Security:

| Setting | Needed for |
|---|---|
| Allow HTTP Requests | Discord reporting |
| Enable Studio Access to API Services | Ban strikes persisting across rejoins |

Neither is required for the anticheat to detect and kick. Without HTTP you get in-memory
history and console output; without API Services, strikes reset when a player rejoins.

## 3. Point it at a Discord channel

Open the `config` module and set the webhook URLs. Each category can have its own channel, or
they can all share one.

```lua
Webhooks = {
    Movement  = "https://discord.com/api/webhooks/...",
    Combat    = "https://discord.com/api/webhooks/...",
    Integrity = "https://discord.com/api/webhooks/...",
    -- ...
}
```

?> Leave a webhook blank and that category simply logs to console instead. Nothing errors.

## 4. Start in report-only mode

**Do this for your first session.** In `XIS_Report`:

```lua
CFG.EnforcementEnabled = false
```

Every detector is clamped to `REPORT`. Nothing kicks, nothing bans, everything still logs with
full context. Run a populated session, read what comes through, and only then turn it on.

!> Skipping this step is the single most common way to empty your own server. Every game has
systems that legitimately break movement rules — cutscenes, launch pads, vehicles, carry
mechanics, spectator cameras. Report-only mode is how you find yours before they cost you
players.

## 5. Check first boot

The server prints a summary. It looks like this:

```
[XIS AntiCheat] v3 online | 74 detectors | 12 heartbeat slots | bans on, threshold 3
[XIS AntiCheat] 2 detector(s) DISABLED by config:
   - <one detector> (action = IGNORE)
   - <another detector> (gate AntiESP is off)
[XIS AntiCheat] v3 server ready in 0.008s
```

- **Detector count** — if it is not 74, the registry did not load cleanly and you are
  running with gaps.
- **Heartbeat slots** — must match the number of slots the client build sends. A mismatch
  makes a slot look permanently frozen, which ejects players after the startup grace.
- **Ban mode** — confirms whether bans are armed and at what strike threshold.
- **Disabled list** — printed every boot on purpose, so a detector can never be off by
  accident. Anything unexpected in that list is a config mistake.

!> **Enforcement mode is NOT printed.** There is no boot line telling you whether you are
in report-only mode, so do not go looking for one — check `CFG.EnforcementEnabled` in
`XIS_Report` directly. This is the single most useful thing to be certain of before a
populated session, and it is the one thing the boot summary does not say.

You can also read the `_status` attribute on the `XIS_Server_v3` script in the Properties pane
for a live one-line health check without opening the console.

## 6. Exempt your own systems

Before enabling enforcement, wrap anything of yours that moves players in an unusual way. See
the [API reference](api.md) — in most cases this is one line:

```lua
_G.XIS_Whitelist.add(player, "fly", 4)
```

## 7. Turn it on

```lua
CFG.EnforcementEnabled = true
```
