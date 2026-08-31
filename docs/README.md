# Overview

XIS AntiCheat is a server-authoritative anticheat for Roblox experiences. It ships as four
scripts, installs without touching your existing systems, and is tuned entirely from one
module.

## How it is put together

Four files, one job each. The split exists so that changing what a detection **costs** never
means editing the code that detects it.

| File | Location | Owns |
|---|---|---|
| `XIS_Registry` | `ServerScriptService/XIS AntiCheat` | Every detector's category, severity, action and ban eligibility. **You tune here, nowhere else.** |
| `XIS_Report` | `ServerScriptService/XIS AntiCheat` | The single path from "a detector fired" to log / webhook / kick / ban. |
| `XIS_Client_v3` | `ReplicatedFirst` | One LocalScript. Self-hides, then runs every client-side detector. |
| `XIS_Server_v3` | `ServerScriptService` | Transport, session liveness, the whitelist, and the server-authoritative movement and combat checks. |

## The rule everything rests on

The client reports **what it saw**, identified by a detector id. It never reports how serious
that is.

A client can lie, so severity and consequence are server-side data keyed by that id. Two things
follow. First, a cheat that forges a report cannot escalate it. Second, retuning the entire
system is one file — the registry — because that is the only place consequence is decided.

## Reports are never trusted from one side alone

Three detector families are re-checked on the server before anyone is touched, because the
client that reports them is usually a **witness**, not the offender.

- **Hitbox** — the reported player is re-measured against the server's own copy of their
  character. If the server sees a normal limb where the reporter saw an oversized one, the
  change never replicated, so the *reporter* made it locally.
- **ESP overlays** and **body movers** — the reported instance path is looked up on the server.
  If the server can see it too, it replicated *downward* and your own game created it. Dropped.

That asymmetry — client-created instances never replicate upward — is the load-bearing idea.
It is also what makes those detectors safe to leave on: a system of yours that later adds a
body mover and forgets to mark it trusted degrades them to a log line instead of kicking real
players.

## What you need

- A Roblox experience with **HTTP requests enabled** if you want Discord reporting.
- **API Services enabled** if you want ban strikes to persist across rejoins.
- No dependency on any particular framework, admin system or weapon kit. Detectors that need
  one are gated off automatically when it is absent.

## Where to go next

- [Install](install.md) — drop the four scripts in and verify first boot.
- [Configuration](configure.md) — gates, thresholds, webhooks.
- [Server-script API](api.md) — what your own scripts can call.
- [Runbook](runbook.md) — read this **before** you go live, not after.
