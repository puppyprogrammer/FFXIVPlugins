# FFXIVPlugins

**Plugins for FFXIV. One click. No catch.**

FFXIVPlugins is a community-run alternative distribution of plugins for
Final Fantasy XIV — a complete, ground-up fork of the entire goatcorp
Dalamud plugin ecosystem, built and operated by [CommsLink](https://commslink.net).

We mirror every layer of the upstream stack so AI-assisted tools,
experimental utilities, and quality-of-life mods that the official
repository can't or won't approve can be installed in-game, auto-updated,
and used normally — without depending on any goatcorp infrastructure.

> **Independent project.** Not affiliated with goatcorp, XIVLauncher, or
> Dalamud. We respect the upstream licenses (AGPLv3 / GPLv3 / MIT) and
> auto-merge upstream fixes; we just refuse the gatekeeping.

---

## Download

**Latest release:** [`v0.1.0`](https://github.com/puppyprogrammer/FFXIVPlugins/releases/tag/v0.1.0)

1. Grab **`FFXIVPlugins-Launcher-v0.1.0.zip`** from the release page.
2. Extract anywhere — the launcher is fully portable, no installer.
3. Run **`FFXIVPlugins.exe`**.

The zip contains exactly two files:

```
FFXIVPlugins.exe                         the launcher (self-contained .NET 9 single-file)
patcher/XIVLauncher.PatchInstaller.exe   the game patcher
```

No PDBs, no telemetry, no installer, no auto-update bait. The launcher
talks **only** to FFXIVPlugins infrastructure — there is zero traffic to
`kamori.goats.dev`, `goatcorp.github.io`, or any other upstream endpoint.

---

## Security improvements over upstream

We actively audit and patch security issues the upstream project hasn't
addressed. This is not theoretical — these are real vulnerabilities in
the code that ships to every XIVLauncher user today.

| # | Vulnerability | Upstream status | Our fix | PR |
|---|---|---|---|---|
| 1 | **No plugin download integrity verification.** Dalamud downloads plugin ZIPs and loads them with zero hash checking. A compromised CDN or MITM can serve arbitrary code that executes inside the game process. | Unfixed | SHA-256 hash field in plugin manifest; downloaded ZIPs verified before extraction. Mismatch = install aborted. Backwards-compatible (plugins without hashes still install with a warning). | [Dalamud #1](https://github.com/puppyprogrammer/Dalamud/pull/1) |

More fixes in progress. See the [security audit backlog](https://github.com/puppyprogrammer/FFXIVPlugins/issues)
for the full list.

---

## Why this exists

The official goatcorp ecosystem is *open source* but **not AI-friendly** —
they reject AI plugins on policy grounds. The technical infrastructure is
fine; the gatekeeping is purely social, applied at the plugin-approval and
distribution layers.

Dalamud already supports user-added "third-party plugin repositories", but
that path leaves the entire **runtime** (Dalamud itself, the launcher, the
auth/version API) under goatcorp's control. If goatcorp ever adds a signing
check, an allowlist, or a kill switch to the official Dalamud build, every
third-party repo dies overnight.

A full ecosystem fork is the only configuration that is structurally
immune to upstream lockout. So that's what we built.

---

## The sibling repos

Each upstream repo has a 1:1 fork under `puppyprogrammer/`. Names match
upstream so we can auto-merge upstream fixes without rename pain.

| Layer | Fork | Upstream | Role |
|---|---|---|---|
| **Plugin index** | [`DalamudPluginsD17`](https://github.com/puppyprogrammer/DalamudPluginsD17) | `goatcorp/DalamudPluginsD17` | TOML manifests pointing at plugin Git commits |
| **Built plugin CDN** | [`PluginDistD17`](https://github.com/puppyprogrammer/PluginDistD17) | `goatcorp/PluginDistD17` | Built plugin zips, served via Pages |
| **In-game framework** | [`Dalamud`](https://github.com/puppyprogrammer/Dalamud) | `goatcorp/Dalamud` | Injected into `ffxiv_dx11.exe`; loads plugins |
| **Game struct defs** | [`FFXIVClientStructs`](https://github.com/puppyprogrammer/FFXIVClientStructs) | `aers/FFXIVClientStructs` | Memory layouts every plugin uses |
| **Dalamud release CDN** | [`dalamud-distrib`](https://github.com/puppyprogrammer/dalamud-distrib) | `goatcorp/dalamud-distrib` | Built Dalamud zips, served via Pages |
| **Static assets** | [`DalamudAssets`](https://github.com/puppyprogrammer/DalamudAssets) | `goatcorp/DalamudAssets` | Fonts/icons Dalamud loads at startup |
| **Backend API** | [`XLWebServices`](https://github.com/puppyprogrammer/XLWebServices) | `goatcorp/XLWebServices` | The `kamori.goats.dev` server. ASP.NET 8. |
| **Windows launcher** | [`FFXIVQuickLauncher`](https://github.com/puppyprogrammer/FFXIVQuickLauncher) | `goatcorp/FFXIVQuickLauncher` | WPF launcher, patches game, injects Dalamud |
| **Linux/Steam Deck launcher** | [`XIVLauncher.Core`](https://github.com/puppyprogrammer/XIVLauncher.Core) | `goatcorp/XIVLauncher.Core` | Cross-platform launcher (not yet rebranded) |
| **CI build pipeline** | [`Plogon`](https://github.com/puppyprogrammer/Plogon) | `goatcorp/Plogon` | Docker plugin builder |
| **Plugin packager** | [`DalamudPackager`](https://github.com/puppyprogrammer/DalamudPackager) | `goatcorp/DalamudPackager` | MSBuild task plugins use to package |
| **Plugin SDK** | [`Dalamud.NET.Sdk`](https://github.com/puppyprogrammer/Dalamud.NET.Sdk) | `goatcorp/Dalamud.NET.Sdk` | MSBuild SDK most modern plugins target |

---

## Live infrastructure

Everything below is running today and serving real traffic to the
launcher and Dalamud running in-game.

| Component | URL | Status |
|---|---|---|
| **Backend API** | `https://ffxivplugins.commslink.net` | Live (Docker on EC2, nginx + Let's Encrypt) |
| **Plugin master** | `https://ffxivplugins.commslink.net/Plugin/PluginMaster` | Serving |
| **Dalamud release CDN** | `https://puppyprogrammer.github.io/dalamud-distrib/` | Live |
| **Static assets CDN** | `https://puppyprogrammer.github.io/DalamudAssets/` | Live |
| **Built plugin CDN** | `https://puppyprogrammer.github.io/PluginDistD17/` | Live |
| **Launcher releases** | [`releases/latest`](https://github.com/puppyprogrammer/FFXIVPlugins/releases/latest) | v0.1.0 |
| **Discord** | https://discord.gg/m3s5eDqsMw | Live |
| **FAQ / Docs** | https://commslink.net/FFXIVPlugins | Live |

The backend (XLWebServices) is patched to:

- target .NET 8 (upstream Dockerfile said .NET 7)
- skip the upstream availability gates that block startup when seeded
  data isn't byte-identical to goatcorp's
- run with Sentry disabled and a writable `FileCacheDirectory`
- proxy plugin metadata through our own forks instead of goatcorp's

---

## How it all routes

```
                            +--------------------------------------+
                            |  user runs FFXIVPlugins.exe          |
                            +-----------------+--------------------+
                                              |
                          +-------------------+----------------+
                          | launcher checks for Dalamud release|
                          |   ffxivplugins.commslink.net       |
                          |   /Dalamud/Release/Meta            |
                          +-------------------+----------------+
                                              |
                          +-------------------+----------------+
                          | downloads Dalamud zip from         |
                          |   puppyprogrammer.github.io/       |
                          |     dalamud-distrib/               |
                          +-------------------+----------------+
                                              |
                          +-------------------+----------------+
                          | injects Dalamud into ffxiv_dx11.exe|
                          | Dalamud reads MainRepoUrl =        |
                          |   ffxivplugins.commslink.net/      |
                          |     Plugin/PluginMaster            |
                          +-------------------+----------------+
                                              |
                          +-------------------+----------------+
                          | in-game plugin browser pulls zips  |
                          | from puppyprogrammer.github.io/    |
                          |   PluginDistD17/                   |
                          +------------------------------------+
```

Zero requests leave the FFXIVPlugins / CommsLink perimeter at any step.

---

## Notable launcher changes vs. upstream XIVLauncher

The launcher fork lives at
[`puppyprogrammer/FFXIVQuickLauncher`](https://github.com/puppyprogrammer/FFXIVQuickLauncher)
on the `master` branch. Concrete differences from goatcorp upstream:

- **Rebranded** to `FFXIVPlugins` — assembly name, window title, jump
  list, roaming data folder (`%LOCALAPPDATA%\FFXIVPlugins`), exe name.
- **CommsLink dark color palette** — `MaterialDesignOverrides.xaml`
  remaps every Material Design brush key (legacy + MD3) to the CommsLink
  surface tones (`#0d1117 / #161b22 / #1c2128 / #569cd6`). Tab strip,
  FAB Mini-Light buttons, raised buttons, validation, dividers — all
  redirected to the brand palette.
- **Dalamud is always-on.** The "In-Game" settings tab is gone. The
  first-time setup no longer asks whether to enable hooks. The "I'm
  running an unofficial version" warning is removed — being a fork is
  the entire point.
- **Backup tab removed** from settings (rarely used, legacy feature).
- **Settings simplified.** About / FAQ / GitHub buttons all point at
  `commslink.net/FFXIVPlugins`, the FFXIVPlugins Discord, and
  `puppyprogrammer/FFXIVPlugins` respectively.
- **Velopack auto-update bypassed.** The launcher is shipped as a
  portable single-file binary; we use GitHub Releases for distribution
  rather than goatcorp's Velopack feed.
- **Window doubled in height** to fit the new layout comfortably.
- **CI strips non-shipping files** so every build artifact contains
  exactly two `.exe`s — no PDBs, no leftover DLLs, no hash files.
- **`WINDOWS` constant hardcoded** in `XIVLauncher.Common.csproj` to fix
  an upstream bug where the symbol wasn't defined for `net9.0` (only
  `net9.0-windows`), causing a `clock_gettime` `DllNotFoundException`
  inside `ArgumentBuilder` at startup.

---

## Submitting a plugin

Plugin submission is a normal GitHub PR flow against the manifest repo,
[`puppyprogrammer/DalamudPluginsD17`](https://github.com/puppyprogrammer/DalamudPluginsD17).
There is no upstream-style category gate — AI plugins, experimental
tooling, and the rest are all welcome. There is, however, a security
review (AI-assisted, human-decided) on every PR.

### How to submit

1. **Fork** [`puppyprogrammer/DalamudPluginsD17`](https://github.com/puppyprogrammer/DalamudPluginsD17).
2. **Add a manifest** at `stable/<YourPluginName>/manifest.toml` (see
   the upstream [DIP17 docs](https://github.com/goatcorp/DalamudPluginsD17?tab=readme-ov-file#how-to-add-or-update-a-plugin)
   for the schema — we did not change it). The manifest pins your
   plugin's Git URL and a specific commit SHA, so your published build
   is reproducible and immutable.
3. **Optionally** add `stable/<YourPluginName>/images/` with screenshots
   for the in-game plugin browser.
4. **Open a PR** against `main`.

### What happens after you open the PR

- **AI preliminary review** — `ai-review.yml` runs automatically. It
  asks Claude (Haiku) to look at the diff, the manifest fields, and
  the linked plugin repo, then post a sticky comment under the PR
  with: a one-paragraph summary, a risk level (LOW/MEDIUM/HIGH), a
  bullet list of concrete findings, and a recommendation
  (APPROVE / APPROVE WITH NOTES / REQUEST CHANGES / BLOCK). This is
  **advisory only**.
- **Human final approval.** A FFXIVPlugins maintainer reads the AI
  review, sanity-checks anything it flagged, and either merges the PR
  or asks for changes. We do not auto-merge — final approval is
  always a human decision.
- **Build & ship.** When the PR is merged to `main`, `push.yml`
  triggers Plogon. Plogon clones your plugin at the pinned commit,
  builds it inside a sandboxed Docker image, and the resulting zip is
  committed to [`puppyprogrammer/PluginDistD17`](https://github.com/puppyprogrammer/PluginDistD17).
  A signal request to `https://ffxivplugins.commslink.net/Plogon/CommitStagedPlugins`
  refreshes the in-memory `PluginMaster`, and within about a minute
  every running launcher and in-game Dalamud sees your plugin in the
  browser.

### Why we review at all

AI gating is the upstream's policy problem; *security* is everyone's
problem. A Dalamud plugin runs as native code inside the user's game
process, with full memory access. We refuse goatcorp's
*category* gatekeeping but we still keep a *safety* review — an
unreviewed pipeline is irresponsible no matter what your policy on AI
is.

The review focuses on concrete signals: Is the manifest pinning a
specific commit? Does the linked repo exist and look like a Dalamud
plugin? Is the author known to us? Are there obvious red flags in the
diff (suspicious URLs, unowned manifests, encoded payloads)? It does
**not** waste your time on style nitpicks or general "best practices"
lectures.

### Authors who have submitted before

Once a maintainer has merged at least one PR from a given GitHub
author, future PRs from the same author will get the same AI review
but a faster human turnaround. We're not trying to make you jump
through hoops — we're trying to catch the one griefer in a thousand
without rejecting the other 999.

---

## Sustainability: how this stays current

Each fork runs three GitHub Actions workflows on a schedule:

- **`upstream-sync.yml`** — daily, opens a PR with the latest upstream
  commits. Never auto-merges — a human reviews to make sure goatcorp
  hasn't slipped in a kill switch or signing check.
- **`smoke-test.yml`** — runs on every push, asserts the FFXIVPlugins
  patches (`MainRepoUrl`, rebranding, brush overrides) are still intact.
  Catches bad merges before they ship.
- **`lkg-tag.yml`** — tags every successful build as `lkg-YYYYMMDD`
  (Last Known Good) so we can roll back instantly if a sync goes bad.

Game patch days (when SE updates FFXIV) require an updated
`FFXIVClientStructs` sigscan pass; we re-merge upstream ClientStructs
through the same `upstream-sync` flow.

---

## Roadmap

| Phase | Status | What |
|---|---|---|
| **0 — Inventory** | Done | Map every endpoint in the goatcorp stack. See [`STACK.md`](./STACK.md). |
| **1 — Fork & mirror** | Done | All sibling repos forked under `puppyprogrammer/`, static CDNs live on GitHub Pages. |
| **2 — Backend** | Done | Our XLWebServices instance running on `ffxivplugins.commslink.net` (Docker on EC2, nginx + Let's Encrypt). |
| **3 — Custom Dalamud** | Done | Dalamud built and published with `MainRepoUrl` repointed at our backend. Loads cleanly in-game. |
| **4 — Launcher** | Done | `FFXIVPlugins.exe` shipped as `v0.1.0`. Rebranded, themed, simplified, single-file. End-to-end verified: launcher → Dalamud → in-game plugin install. |
| **5 — AI-friendly pipeline** | Next | Replace DIP17 with a submission pipeline that doesn't gate AI plugins. Plogon fork already in place; need the submission UX and review policy. |
| **6 — Sustainability** | In progress | Daily upstream-sync workflows live across launcher / Dalamud / ClientStructs. LKG tagging in place. Sig-update runbook for game patch days still TODO. |

**Currently working on:** Phase 5 — the actual reason the project exists.
The full vendor-locked stack from launcher to plugin browser is now
operational; the next milestone is opening up an AI-plugin submission
flow that doesn't require begging an upstream reviewer.

---

## License

Each sibling repo retains its upstream license (AGPLv3 for Dalamud, GPLv3
for the launchers, MIT for ClientStructs, etc.). This meta repo is MIT.
