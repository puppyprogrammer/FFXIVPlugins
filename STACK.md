# Phase 0 — Goatcorp Stack Inventory

Inventory of every goatcorp-controlled component the FFXIV / Dalamud / XIVLauncher
ecosystem depends on, with concrete file:line citations of every URL the
launcher and Dalamud reach out to. This is the map we use to plan replacements.

All paths below are local clones under `H:/Development/`.

---

## 1. Repos cloned (Phase 0)

| Local path | Upstream | Role | Size note |
|---|---|---|---|
| `FFXIVQuickLauncher/` | `goatcorp/FFXIVQuickLauncher` | **Windows launcher** (WPF/.NET). Patches game, OTP, injects Dalamud. | shallow |
| `XIVLauncher.Core/` | `goatcorp/XIVLauncher.Core` | **Cross-platform launcher** (Steam Deck / Linux). Same purpose, different UI. | shallow |
| `Dalamud/` | `goatcorp/Dalamud` | **In-game framework** injected into ffxiv_dx11.exe. Loads plugins, ImGui, hooks. AGPLv3. | shallow |
| `FFXIVClientStructs/` | `aers/FFXIVClientStructs` | Memory layout / sig defs every plugin uses. **Breaks every game patch.** | shallow |
| `dalamud-distrib/` | `goatcorp/dalamud-distrib` | Static CDN of built Dalamud zips (`stg/latest.zip`, etc.). GitHub Pages. | shallow |
| `DalamudAssets/` | `goatcorp/DalamudAssets` | Static fonts/icons Dalamud loads at startup. (clone in progress, LFS) | shallow |
| `DalamudPackager/` | `goatcorp/DalamudPackager` | MSBuild task plugins use to package themselves. Trivial. | shallow |
| `Dalamud.NET.Sdk/` | `goatcorp/Dalamud.NET.Sdk` | MSBuild SDK most modern plugins target. | shallow |
| `Plogon/` | `goatcorp/Plogon` | Docker-based CI builder consumed by DalamudPluginsD17. | shallow |
| `XLWebServices/` | `goatcorp/XLWebServices` | **The backend API** (`kamori.goats.dev`). ASP.NET. **This is the lockout chokepoint.** | shallow |
| `DalamudPluginsD17-AIFriendly/` | `puppyprogrammer/DalamudPluginsD17` (fork of `goatcorp/DalamudPluginsD17`) | Plugin manifest index. Already cloned. | full |

### Repos identified but not yet cloned (low priority for Phase 0)

- `goatcorp/PluginDistD17` — built plugin zips. Huge. We only need its layout.
- `goatcorp/patchinfo` — game patch metadata used by the patcher.
- `goatcorp/bleatbot` — GitHub bot that handles `bleatbot, rebuild` PR commands.
- `goatcorp/operator` — internal ops tooling.
- `goatcorp/plogon-builder` — build helper image for Plogon.
- `goatcorp/XIVLauncher.PatchInstaller` (if separate) — patch unpacker.
- `goatcorp/dalamud-bugbait` — feedback collection backend (`kiko.goats.dev`).
- `goatcorp/goatcorp.github.io` — the static website (FAQ, faq/integrity/, etc.).

---

## 2. The lockout chokepoints — every URL the client reaches out to

This is the critical section. Each URL below is a place goatcorp could
unilaterally cut us off from. **All of them must be repointed at our infra
before we can claim independence.**

### 2A. `kamori.goats.dev` — XLWebServices (the API server)

This is the single most important host. It's a small ASP.NET app
(`XLWebServices/`) that wraps the static GitHub repos with a JSON API.

| Endpoint | Caller | File | What it returns |
|---|---|---|---|
| `/Plugin/PluginMaster` | **Dalamud** plugin installer | `Dalamud/Dalamud/Plugin/Internal/Types/PluginRepository.cs:28` | **The full plugin list (JSON).** Generated from `PluginDistD17` + manifests. **THIS is the central choke point.** |
| `/Plugin/History/{name}?track={track}` | Dalamud changelog UI | `Dalamud/Dalamud/Interface/Internal/Windows/PluginInstaller/DalamudChangelogManager.cs:20` | Per-plugin version history |
| `/Dalamud/Release/Meta` | Launcher + Dalamud | `FFXIVQuickLauncher/src/XIVLauncher.Common/Dalamud/DalamudBranchMeta.cs:49`, `Dalamud/Dalamud/Interface/Internal/Windows/BranchSwitcherWindow.cs:26` | List of Dalamud branches (release, stg, canary, etc.) |
| `/Dalamud/Release/VersionInfo?track={track}` | Launcher + Dalamud self-update | `FFXIVQuickLauncher/src/XIVLauncher.Common/Dalamud/DalamudLauncher.cs:48`, `Dalamud/Dalamud/Support/DalamudReleases.cs:18` | Currently advertised Dalamud version per branch |
| `/Dalamud/Release/Changelog` | Dalamud UI | `Dalamud/Dalamud/Interface/Internal/Windows/PluginInstaller/DalamudChangelogManager.cs:19` | Dalamud release notes |
| `/Dalamud/Asset/Meta` | Launcher | `FFXIVQuickLauncher/src/XIVLauncher.Common/Dalamud/AssetManager.cs:19` | Manifest of Dalamud asset files (fonts, icons) |
| `/Dalamud/Release/Runtime/Hashes/{version}` | Launcher | `FFXIVQuickLauncher/src/XIVLauncher.Common/Dalamud/DalamudUpdater.cs:501` | .NET runtime expected hashes |
| `/Dalamud/Release/Runtime/DotNet/{version}` | Launcher | `DalamudUpdater.cs:535` | .NET runtime download |
| `/Dalamud/Release/Runtime/WindowsDesktop/{version}` | Launcher | `DalamudUpdater.cs:536` | WindowsDesktop runtime download |
| `/Launcher/GetLease` | Launcher self-update | `FFXIVQuickLauncher/src/XIVLauncher/Updates.cs:35` | Gates whether the launcher is allowed to run / update |
| `/Launcher/GetFile` | Launcher self-update | `FFXIVQuickLauncher/src/XIVLauncher/Updates.cs:36` | Launcher binary download |
| `/Launcher/GetLauncherClientConfig` | Launcher (Win + Core) | `FFXIVQuickLauncher/src/XIVLauncher.Common/Util/DebugHelpers.cs:79`, `XIVLauncher.Core/src/XIVLauncher.Core/Net/LauncherClientConfig.cs:14` | Feature flags / kill switches |
| `/Proxy/Meta` | Launcher news | `FFXIVQuickLauncher/src/XIVLauncher/Windows/ChangelogWindow.xaml.cs:22` | News feed shown in launcher window |

**Source of truth:** `XLWebServices/XLWebServices/Controllers/PluginController.cs:94`
implements `PluginMaster` — it reads the manifests from the configured
`GitHub:PluginDistD17:Owner/Name` repo (`ConfigMasterService.cs:34-35`) and
builds the response. This is small (a few hundred lines) and very forkable.

The `Proxy=true` query flag rewrites plugin download URLs through Kamori
itself. With `proxy=false`, Dalamud will pull plugin zips directly from:

```
https://raw.githubusercontent.com/goatcorp/PluginDistD17/{branch}/{channel}/{plugin}/latest.zip
```
(`XLWebServices/XLWebServices/Controllers/PluginController.cs:72`)

### 2B. `goatcorp.github.io` — static GitHub Pages (low risk, easy to mirror)

| URL | Caller | File |
|---|---|---|
| `goatcorp.github.io/dalamud-distrib/stg/latest.zip` | Dalamud build CI; staging Dalamud download | `Dalamud/.github/workflows/main.yml:84` |
| `goatcorp.github.io/integrity/` | Launcher game-file integrity check | `FFXIVQuickLauncher/src/XIVLauncher.Common/Game/IntegrityCheck.cs:16` |
| `goatcorp.github.io/faq/...` | Browser-opened help links (many) | various |

These are pure static GitHub Pages. We can mirror by enabling Pages on our
forks of `dalamud-distrib`, `integrity`, and `goatcorp.github.io`.

### 2C. `kiko.goats.dev` — bug feedback (ignore-able)

| URL | Caller | File |
|---|---|---|
| `kiko.goats.dev/feedback` | Dalamud "send feedback" button | `Dalamud/Dalamud/Support/BugBait.cs:18` |

Telemetry/feedback only. Safe to point at `/dev/null` or commslink.net.

### 2D. SE-owned (we never touch)

The launcher also talks to `ffxiv-login.square-enix.com`, `patch-dl.ffxiv.com`,
`ffxiv.com`, etc. These are Square Enix's, not goatcorp's, and are not
lockout vectors for us — they're the same servers the official launcher uses.

---

## 3. The dependency graph

```
                        ┌─────────────────────────┐
                        │   FFXIV (the game)      │
                        └───────────▲─────────────┘
                                    │ inject DLL
                                    │
        ┌───────────────────────────┴──────────────────────┐
        │  Dalamud  (in-process, AGPLv3)                   │
        │  ─ loads plugins from PluginMaster JSON          │
        │  ─ self-updates from kamori.goats.dev            │
        └───────────────────────────▲──────────────────────┘
                                    │ launches & updates
                                    │
        ┌───────────────────────────┴──────────────────────┐
        │  XIVLauncher (Win)  /  XIVLauncher.Core (Linux)  │
        │  ─ patches game  ─ SE login  ─ OTP               │
        │  ─ pulls Dalamud zips from kamori                │
        │  ─ self-updates from kamori                      │
        └───────────────────────────▲──────────────────────┘
                                    │ HTTPS
                                    │
            ┌───────────────────────┴────────────────────────┐
            │   kamori.goats.dev   (= XLWebServices)         │
            │   ──────────────────────────────────────────   │
            │   /Plugin/PluginMaster   ← THE chokepoint      │
            │   /Dalamud/Release/...                         │
            │   /Launcher/...                                │
            │                                                │
            │   reads from GitHub:                           │
            │     ─ DalamudPluginsD17  (manifest index)      │
            │     ─ PluginDistD17      (built plugin zips)   │
            │     ─ dalamud-distrib    (Dalamud zips)        │
            │     ─ DalamudAssets      (fonts/icons)         │
            └────────────────────────────────────────────────┘
```

**Key insight:** the in-game plugin installer **never talks to GitHub
directly.** It talks to Kamori, which proxies/aggregates the GitHub repos.
That means:

1. Forking `DalamudPluginsD17` alone changes nothing for end users.
2. The minimum viable lockout-proof stack requires either
   (a) replacing Kamori with our own `pluginmaster.json` host AND patching
       Dalamud's `MainRepoUrl` constant to point at it, OR
   (b) running our own Kamori instance (XLWebServices is open source) on
       commslink.net and patching Dalamud's `MainRepoUrl` to point at it.

Option (b) is strictly better: it gives us version-info, changelogs, and
all the per-plugin endpoints "for free" because Dalamud's existing UI just
works against an unmodified XLWebServices clone.

---

## 4. Risk register

| Risk | Likelihood | Mitigation |
|---|---|---|
| Goatcorp ships a Dalamud build that hard-codes their `kamori.goats.dev` URL into a signed binary and refuses to load if the URL is changed | low–med | We ship our **own** Dalamud build (Phase 3). Our launcher only ever pulls Dalamud from our CDN. |
| Game patches break Dalamud sigs faster than we can keep up | **high** | Keep an auto-merge bot pulling upstream `Dalamud` + `FFXIVClientStructs` commits. Phase 6. |
| Goatcorp changes the Kamori response schema (`PluginMaster` JSON) | low | We control both server and client in our fork. Schema drift only matters if we try to stay wire-compatible with goatcorp, which we don't. |
| SE legal action / detection | low (no change from today) | Same risk surface as goatcorp; nothing we do increases it. |
| Loss of `FFXIVClientStructs` upstream cooperation | med | Vendor it; auto-merge upstream while it lasts; budget for taking it over if forced. |
| Plugin authors don't trust a fork and won't dual-list | med | Make the manifest format **byte-compatible** with goatcorp's so dual-listing is a one-line PR. |

---

## 5. Phase 0 deliverables — DONE ✅

- [x] Cloned 10 critical repos
- [x] Mapped every `kamori.goats.dev` endpoint to a file:line in the client
- [x] Identified `XLWebServices/Controllers/PluginController.cs:94` as the chokepoint
- [x] Confirmed `goatcorp.github.io/*` endpoints are pure static (mirrorable)
- [x] Built dependency graph + risk register

---

## 6. Recommended Phase 1 (next, awaiting your approval)

**Goal of Phase 1:** stand up our own static CDN clones, **without changing
any client code yet.** Verifiable end-to-end by you, with zero risk to your
existing FFXIV install.

Concrete steps:

1. Create empty repos under `puppyprogrammer/`:
   - `dalamud-distrib` — fork of goatcorp's
   - `DalamudAssets` — fork
   - `PluginDistD17` — fork (or new, since this is huge)
   - `integrity` — fork (game integrity hashes from `goatcorp.github.io/integrity/`)
2. Enable GitHub Pages on each. Verify the URLs respond:
   - `https://puppyprogrammer.github.io/dalamud-distrib/stg/latest.zip`
   - `https://puppyprogrammer.github.io/DalamudAssets/...`
   - etc.
3. Write a sync workflow that pulls upstream goatcorp updates daily so our
   mirrors stay fresh until we diverge.

**Test gate before Phase 2:** you visit each Pages URL in a browser and
confirm a 200 response with the expected content. Nothing on your machine
changes. If something is wrong we fix it before touching the launcher.

---

## 7. Recommended Phase 2 preview (after Phase 1 passes)

Stand up our own **XLWebServices** instance on commslink.net with config
pointing at our forks of `DalamudPluginsD17` and `PluginDistD17`. Verify
`https://api.commslink.net/Plugin/PluginMaster` returns a valid plugin list.

**Test gate before Phase 3:** `curl` returns valid JSON; we have not yet
touched the launcher or Dalamud. Still zero risk to your install.

---

## 8. What I need from you to proceed to Phase 1

1. Confirm the org name is `puppyprogrammer` for all forks (or specify
   another).
2. Confirm I should fork into sibling repos under `puppyprogrammer/*`
   (not into `puppyprogrammer/FFXIVPlugins`).
3. Create a GitHub personal access token with `repo` + `workflow` scope and
   tell me how you want to provide it (env var, file, or you run the `gh`
   commands yourself — I can prepare the exact commands).
4. Confirm `commslink.net` DNS is yours and you can add a subdomain
   (`api.commslink.net` or similar) when we get to Phase 2.

Once you give me those, I'll execute Phase 1 and stop at the test gate so
you can verify before we go further.
