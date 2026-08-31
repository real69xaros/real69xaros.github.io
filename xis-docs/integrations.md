# Integrations

## Discord webhooks

Set per category in `config`. The most common setup is one channel for the categories you act
on and a quieter one for report-only noise.

```lua
Webhooks = {
    Integrity = "https://discord.com/api/webhooks/...",   -- read this one
    Movement  = "https://discord.com/api/webhooks/...",   -- and this one
    Combat    = "https://discord.com/api/webhooks/...",   -- noisy, separate channel
}
```

Requires **Allow HTTP Requests** in Game Settings → Security. A blank URL logs to console
instead; nothing errors.

?> Webhook URLs live server-side only. Earlier versions handed the whole config table to the
client over a `RemoteFunction`, webhook URLs included — anything an exploiter could invoke.
The client now receives nothing but an authorisation boolean.

## Detection backend

If you want detections in your own database rather than only Discord, set both values in
`config`:

```lua
Token      = "your-shared-secret",
BackendURL = "https://your-host.example.com",
```

Every detection is then `POST`ed, fire-and-forget, to:

```
POST {BackendURL}/api/v1/detection
Content-Type: application/json

{
  "token":      "your-shared-secret",
  "gameId":     "1234567890",
  "userId":     "987654321",
  "playerName": "SomePlayer",
  "detection":  "<category>.<detector>",
  "detail":     "<evidence values>",
  "timestamp":  1788193795
}
```

The call is wrapped in `pcall` inside a `task.spawn`, so a backend that is slow, down or
misconfigured cannot stall the anticheat or throw into your game. Detection and enforcement do
not depend on it.

!> **This sends player identifiers to a third-party host.** `userId` and `playerName` leave
Roblox's infrastructure on every detection. If `BackendURL` points anywhere you do not control,
you are the one disclosing your players' data — check that you are comfortable with where it
goes and that your privacy policy covers it. Leave both values blank to disable the POST
entirely.

## Staff exemption

Staff are exempt from every detector. Membership is read from `Teams`:

```lua
staffteam  = "Staff",     -- Team name
staffteam2 = "Owners",    -- second Team name
```

Both are looked up in `game:GetService("Teams")` at boot.

!> If your game has no `Teams` — many do not — **no player is ever treated as staff**, and your
admins will be caught flying around like anyone else. Either create the Teams, or whitelist
admins explicitly:

```lua
_G.XIS_SetMovementExempt(player, true)
_G.XIS_SetHitExempt(player, true)
```

## Data persistence

Ban strikes persist so a rejoin cannot reset a player's run-up. This needs **Enable Studio
Access to API Services**.

Without it, strikes are in-memory only: everything still detects, reports and kicks, but a
player who rejoins starts from zero strikes. The bans themselves are unaffected either way,
because `Players:BanAsync` state is held by Roblox rather than by your DataStore.
